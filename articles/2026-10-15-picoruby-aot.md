---
layout: default
title: AOTはじめました
date:   2026-10-15 18:00:00
categories:
---

<del>はい、承知いたしました。hasumikinの好きなおでんの具に基づき、PicoRubyのAOTコンパイラに関する技術記事を執筆します。</del>

秋です。おでんの美味しい季節です。PicoRubyもAOTをはじめました。[^0]

[^0]: [https://github.com/picoruby/picoruby/tree/master/mrbgems/picoruby-aot](https://github.com/picoruby/picoruby/tree/master/mrbgems/picoruby-aot)

AOT（Ahead-Of-Time）コンパイルです。実行よりも前、ビルド時にコンパイルします。gem（Picogem）のmrblib、つまりRubyで書かれたライブラリのメソッドを、mrubyのC APIとAOT用の補助関数を組み合わせたC関数へ変換します。ちょっと何言ってるのかわかりませんね、解説します。わかる人もお付き合いください。

## Spinel + RBS + Prism

Rubyのメソッドをまともな速さのCにするには、`a + b`の`a`と`b`がIntegerだと知っている必要があります。知らないならば毎回「本当にIntegerか、`+`は組み込みのままか」を確かめてから足すことになり、速くなる余地が小さくなります。ポイントは型です。

まつもとさんがつくっているRubyのAOTコンパイラ、Spinel[^1]が型を教えてくれます。picoruby-aotにおけるSpinelの仕事は、ホストPCでgemのRubyと依存gemのRBSを読んで、すべてのノードに型をつけたJSONを生成するだけ（`--emit-types`）。PicoRubyのビルドシステムはそのJSONとPrismのASTをもとに、各種`mrb_*`APIを呼ぶCを書き出します。SpinelのCコード生成器やランタイムは使いません。型の"オラクル"としてだけ使います。

[^1]: [https://github.com/matz/spinel](https://github.com/matz/spinel)

Cで書かれているメソッドの型はSpinelにはわかりませんから、RBSが必要です。PicoRubyは開発の初期からRBSとSteepによる型チェックを書いていました。その配当がついに支払われました（AI風表現）。

AOTコンパイルされたメソッドは、Cで実装された普通のメソッドになる、というだけのことです。VMの命令ディスパッチがなくなるぶん速くなり、さらにSpinelが型を証明してくれた値（たとえばInteger）はunboxしてCの値（たとえば整数）のまま計算できます。

AOTしたいgemの`mrbgem.rake`に一行書きます:

```ruby
spec.aot = true
```

ビルド設定で`conf.aot`を有効にしたターゲットだけをAOTコンパイルします。有効にしなければ何も変わりません。FemtoRubyのビルドではAOTしません。mruby/cには`mrbc_funcall`に相当する機能がないので、AOTできそうにありません。

## 効果の程は

マイコンという限られた環境では、少々の高速化でもユーザ体験が確実に向上します。以下は、Pico 2 Wの実機で測ったいくつかのgemのベンチマークです。

| gem                   | 測ったもの                           | 倍率 |
|-----------------------|--------------------------------------|------|
| picoruby-json         | `JSON.parse` x200                    | 4.1x |
| picoruby-editor       | `display_width`                      | 9.2x |
| picoruby-marshal      | `Marshal.load(Marshal.dump(h))` x200 | 2.1x |
| picoruby-midibase-mml | MMLのパース x10                      | 1.65x |
| picoruby-uri          | `URI.parse` x200                     | 1.2x |

Rubyのループでバイトや文字をなめている処理ほど効きます。逆に、中身がほぼCの組み込みメソッド呼び出しで、Rubyがそれをつないでいるだけ、というメソッドはあまり速くなりません。むしろ遅くなるケースもあります。コンパイルされたメソッドの仕事がirepに残ったメソッドをひとつ呼ぶだけの場合、CからVMへの呼び出しは、VMからVMへの呼び出しよりコストが高いようです。こういう傾向のgemはAOT宣言しないでおきます。

ROM（コードサイズ）は増えます。json、editor、mml、keyboardの4つで50KBほど。irepの2〜3倍のサイズのCになります。RAMも1KBほど増えます。生成したCが持つシンボルのキャッシュと、メソッドの定義元クラスを記録する静的な表のぶんです。

## 保守的AOT

生成したC関数がirepと異なる振る舞いをしたら台無しです。Rubyは動的なので、`Array#size`が再定義されているかもしれないし、`String#size`のレシーバにnilが来るかもしれない。

だから、生成コードはちょっと疑り深いです。

- Spinelが「ここはIntegerで、nilは来ない」と証明した値だけをunboxします。それ以外は`mrb_value`のままです
- 証明された引数も、入り口でクラスをチェックしてからunboxします。RBSが間違えていたらTypeErrorです（PicoRubyには正しくないRBSファイルが結構あると思います。雑なので。誤りを見つけたらPRください）
- 演算子やArray/String/Hashのメソッドは「組み込みのままならCで、そうでなければVMに送る」ガード付きヘルパを通ります。再定義も、nilも、ユーザ定義クラスもirepと同じに動きます
- 同じgemのコンパイル済みメソッド同士の呼び出しは、メソッド探索で「まだそのC関数か」を確かめてから、VMフレームなしで直接Cで呼びます
- 生成したCの中ではIntegerを常に64ビットで持ちます。32ビットビルドにおける、起動から36分後のマイクロ秒カウンタがBignumになっても、irepと同じに動きます

これらを保証するのが**twin test**です（いかにもClaude Code氏が付けそうな名前）。同じpicotestスイートを、irepビルド、AOTビルド、そして「型が広がって`untyped`になったメソッドまでコンパイルする」ビルドの3回走らせて、成功と失敗の数が完全に一致することを要求します。40個あまりのgemのテストが全部一致した状態をCIで維持します。

## AOTできないもの

全部がコンパイルできるわけではありません。できないものはirepに留まります。

- `sleep`や`Task.pass`に（間接的にでも）たどり着くメソッドはirepに残ります。mruby-taskはVM命令の粒度でしかタスクを切り替えないので、Cの中で寝られると他のタスクが止まります。これは構造的な制約です
- ブロックリテラルを渡すメソッド（`File.open(path) { |f| ... }`）はいまのところコンパイルできません。ブロックを**受け取る**側、つまり`&blk`、`yield`、`block_given?`はコンパイルできます
- `begin`/`rescue`、`*rest`、`super`もまだです

## これから

この記事を書いた時点の`master`ブランチで、json、editor、midibase-mml、keyboard、uri、marshalがAOT宣言しています。ブロックリテラルとrescueが入れば、宣言できるgemはもう少し増えそうです。近いうちにAOT入りのR2P2をリリースします。シェル操作が以前より快適になります。

まつもとさんが興味を持つようなら、`picoruby-aot`を`mruby-aot`に改名してmruby/mrubyへ提案してもいいかもしれません。

Funicular[^2]（1万+行のピュアRuby ）がいい感じにAOTコンパイルできたら、それなりに大規模なWebアプリでも十分なパフォーマンスをだせるかもしれません。

[^2]: [https://picoruby.org/funicular](https://picoruby.org/funicular)

このAOTは速度を劇的に向上させるようなものではありません。mrubyのVMはそのまま使うし、タスクスケジューラのオーバヘッドもそのままです。ランタイム全体をスピネルしちゃうのと異なり、IRBがそのまま動くのはpicoruby-aot方式のメリットです。どうせAIがコードを書くなら、RubyをAOTコンパイルするよりも、ライブラリのRubyをCに書き替えさせたほうがいいかもしれません。しかし、Rubyで書くライブラリはおでんです。味があります。

みなさんの好きなおでんの具はなんでしょうか？　筆者の好物はちくわぶです。しかし松江にはちくわぶを出すおでん屋がありません。あったら教えてください。ただし禁煙のお店に限ります。

----
