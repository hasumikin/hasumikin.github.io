---
layout: default
title: JS::Object#to_poroを使えと言ったな、あれは嘘だ
date:   2026-04-07 08:00:00
categories:
---


もう一昨年のことになるのですね。[PicoRuby.wasmをつくった](/2024/12/25/picoruby-wasm.html)という記事を書きました。

そのなかでPicoRuby.wasmでは`JS::Object#to_poro`というメソッドによってJSオブジェクトをてきとう[^1]にRubyオブジェクトへ変換できる、と書きました。しかし2026年現在、その仕様は捨てられました、というのが本稿の釈明です。

[^1]: いい意味でも悪い意味でもてきとう

## 経緯

最近、ピコルビーを玉突きリネームしました。従前PicoRubyと呼んでいたmruby/c VMベースのビルドをFemtoRubyへリネームし、MicroRubyと呼んでいたmruby VMベースのビルドをPicoRubyへリネームしました。[^2]

これはつまり、今後の主力製品はmruby VMベースだよ、という宣言でもあります。

そしてPicoRuby.wasmのバージョンを0.9.6から3.4.xへと一気に上げました。このタイミングで、VMがmruby/cからmrubyへと替わっています。バージョンナンバリングポリシーを変更し、PicoRubyもmrubyのバージョン番号に合わせることにしました。ただしメジャー番号（1番目の要素）とマイナー番号（2番目）までをmrubyに合わせ、パッチ番号（3番目）はPicoRuby独自に付けます。[^3]

[^2]: picoは`0.000 000 000 001`、femtoは`0.000 000 000 000 001`

[^3]: ちなみにR2P2もおなじ方針でバージョニングするようになっています。だから以下のリリース版PicoRubyもすでにmruby VMです [https://github.com/picoruby/picoruby/releases/](https://github.com/picoruby/picoruby/releases/)

## 一気に実用性を上げたPicoRuby.WASM

正直なところ、mruby/cベースではCRubyと同じ感覚で利用できないし、バージョン0.9.6までのPicoRuby.wasmはビミョーだなーと思っていました。mruby/cの言語機能の制限や組み込みメソッドの少なさが、CRubyに慣れている皆の衆には厳しいだろうな、と。

ことしに入ってからVMをmrubyにしたことで、CRubyとの互換性が高まりました。CRubyユーザでもほとんど違和感なくPicoRuby.wasmのアプリを書けるはずです。そして実用性を考えながらJSラッパーを書きすすめた結果、`JS::Object#to_poro`というワークアラウンドでは不便であることが見えてきたため、ちゃんと`to_s`とか`to_i`とかを書くように仕様変更しました。まあ最初からそうだろうなとは思っていたのですけど。AIコーディングのおかげで、このあたりの面倒でチマチマした実装がやりやすくなりました。

## RubyKaigi 2026にてPicoRuby.WASMについて話します

じつは昨年のRubyKaigi 2025でもmruby/cベースのPicoRuby.wasmについてトークすることは可能でしたが、1年寝かせました。熟成させた甲斐があったと思います。1年かけてVMを変更し、ちょっとずつ進化させてきた新生PicoRuby.wasmと、その上に構築したSPAフレームワークによる "たのしいフルスタックRubyライフ" をお届けします。[^4]

[^4]: なんと初日のキーノート直後: [https://rubykaigi.org/2026/presentations/hasumikin.html](https://rubykaigi.org/2026/presentations/hasumikin.html)



