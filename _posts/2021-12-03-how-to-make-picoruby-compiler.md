---
layout: default
title: "[Ruby Advent Calendar 2021] PicoRubyあるいはRuby言語のコンパイラについて"
date: 2021-12-01 12:01:42.932792
categories: 
---

これは[Ruby Advent Calendar 2021](https://qiita.com/advent-calendar/2021/ruby) 3日目の記事です。


PicoRubyというマイコン用のRubyをつくっています。
もうすこし正確に書くと、PicoRubyコンパイラというmruby互換の省メモリコンパイラをつくり、それをmruby/cバーチャルマシンと統合しています。


本稿では、Ruby言語のコンパイラについて書きます。


【注意】以下、とくに調べもせず記憶だけで書くので、不正確です。


コンパイラは一般に、3つの段階から構成されます：

- トークナイザ（スキャナあるいはレキサとも）
- パーサ（パーザと発音する人もいます。日本の大学でコンピュータサイエンスを学んだ人にこの傾向がありそう。まつもとさんとか）
- コードジェネレータ（たんにジェネレータとも）

トークナイザとパーサをコンパイラ内のフロントエンド、コードジェネレータをバックエンドに分類する考え方もあります。たしかドラゴンブックにそう書いてありました。

## トークナイザ

CRubyやmrubyでは、parse.yファイルにトークナイザの実装が含まれています。
「リエントラントではない」前提でこれらのパーサが書かれている（たぶん。後述します）ので、これらのトークナイザも再帰呼び出しを使用していません。たぶん。


対してPicoRubyでは、再帰呼び出しを使用します。例えばこういう場面です：

```
"abc_#{arg}_xyz"
```

文字列内のインターポレーション `#{arg}` の中身は `arg` というRuby文です。これを見つけると、トークナイザはトークナイザインスタンスを新たにつくり、いくつかのステータスをコピーして、 `#{}` の内側のトークナイズ処理（およびパース処理）を再帰呼び出しします。


この開発の初期には、PicoRubyコンパイラのプロトタイプをRubyで書いていました。
つまり、CRubyでmrubyコンパイラを書いていました（富山Ruby会議01でこれを見た角谷さんは、着地点はどこなんだ？と訝っていたようです）。


そのCRuby製のトークナイザを書いたときの私はC言語力が著しく低かったので、CRubyやmrubyのparse.yをきちんと読む根気がなく、なんとなく勘で自分のコンパイラをつくりはじめていました。


再帰呼び出しを使用したのは、この方法しか思いつかなかったからで、深い理由はありません。そのうち根本的な問題にぶち当たり、血を吐くかもしれません。


PicoRubyは省メモリであることが存在意義です。いまから考えると、再帰呼び出しはトークナイザの構造体を新たにつくってしまうし、リターンするまでスタックが深くなったままですので、省メモリ性能の面ではよろしくないのかもしれません。
しかし、マイコン用のRubyプログラムにそうそうたくさんの、あるいは多段にネストした文字列内挿式を書く人はいないはずだ、と自分を慰めています。


CRuby製のmrubyコンパイラをつくりながら、青木さんの「Rubyソースコード徹底開発（RHG）」をWebで読みました。
その後、古本も入手しました。未開封CD-ROMつきでたったの1500円（送料込み）だったので、メルカリもたまには使ってみるものです。


RHGでは、トークナイザ（青木さんは「レキサー」と呼んでいたかな？）とパーサについてそれぞれ章が分けられています。
コンパイラというと、一般にパーサに焦点があてられますが、レキサーの章が最難関だ、とまえがきあたりで宣言されていた記憶があります。


なぜかというと、たぶん、Rubyコンパイラのトークナイザは状態を複雑に管理するからです。違うかな。まあいいか、たしかそう。


行末にセミコロンが要らないとか、メソッド呼び出しの引数にカッコがあってもなくてもよいとかのRubyの書き味を実現するためには、トークナイザとパーサが手に手を取ってともに働かなければなりません。


Ruby言語コンパイラにおけるトークナイザは、実質的にはパーサの一部でもあります。
パーサの還元アクション内からトークナイザ側の状態を変更することもあり、密結合しています。


状態つきトークナイザ、たしかに難しいといえば難しいのですけど、個人的な思い出としては、プロトタイピングしたCRuby製のトークナイザをC言語へと移植する作業が最も辛いものでした。
ふだん如何に高級言語に守られて文字列処理しているかを嫌というほど思い知らされました。
セグメンテーションフォルトの嵐に巻き込まれ、そのお陰でGDBをつかえるようになりました。


それと、CRuby製のトークナイザではRubyの正規表現もつかっていました。
これは当初から嫌な予感しかしなかったのですが、その予感が的中しました。
標準Cライブラリにも正規表現はありますが、それがマイコンでも使えるとは思わない方がいいよね、というのは当初から薄々気づいていたことなのです。
でもとにかく動くものをサクッとつくりたい衝動に突き動かされ、glibcの正規表現を利用していました。


マイコン用のlibcはGNUのlibc（glibc）とは違うものをつかいます。
たとえばArmアーキテクチャ用ですと、newlibという標準Cライブラリがあり、省メモリ性能が優れています。


newlibをリンクしたPicoRubyコンパイラをPSoC5LP用にビルドしたところ、エラーもなくビルドが完了しました。
しかしマイコンにフラッシュして動かしてみると、ハードフォルトします。
予期していたことなのでそんなに驚かなかったですが、newlibにはregex.hはあるけどregex.cがなく、正規表現の関数実装が見つからないために実行時エラーを起こしていたのです。


そんなわけで、POSIX下方互換の軽量な正規表現実装も書きました。
これだけで1週間くらいかかりました。

## パーサ

みなさんご存じのとおり、CRubyは（mrubyも）YACCというパーサジェネレータを使用しています。
YACCは、BISONと呼ばれることもあります。


YACCをGNUプロジェクトが引き継いだときに、BISONという名前に変わったのだろうと思います。繰り返しますが、この記事は何も調べないで記憶と思い込みで書いています。


BISONのどこかのバージョンで、生成されるパーサがリエントラントになりました。
逆にいうと、YACC時代はリエントラントではなかった、ということです。


Rubyの開発が開始された前世紀の時点で、YACC/BISONがリエントラントになっていたかどうかわかりませんが、すくなくともFreeBSDでもRubyをビルドできるようにするためには、古い「バークレーYACC」（？）が読めるparse.yを書く必要があっただろうと想像しています。
CRubyのparse.yが再帰呼び出しを使用していないのには、このへんの経緯もあるのかもしれませんし、ないのかもしれません。もともと再帰する必要などないのでしょう（なにもわからない）。


PicoRubyのパーサジェネレータは、LEMONです。SQLiteのクエリ文をパースするパーサをジェネレートするためのパーサジェネレータがLEMONです。
LEMONのソースコードはたしかパブリックドメインです。裏付けを取る気がないならこんな微妙な情報は書かなければいいのにね、と思うかもしれません。わたくしもそう思います。


LEMONもBISONと同じLALR(1)のパーサジェネレータですので、BISONでやれることはLEMONでもだいたいできるのだろうと思います。
実際にPicoRubyコンパイラをつくっていても、結構複雑なRuby文をパースできているのでたぶん大丈夫です。


でもつくりはじめた当初は、大丈夫だという根拠はまったくありませんでした。
CRubyやmrubyのparse.yをただひたすら眺める日々を何日か過ごしました。
また、CRubyの豊富な実行オプションのどれか（なんだったか忘れました）をつかってパーサの状態遷移を手を替え品を替え出力させたり、Ripperをつかった構文解析の結果を眺めたりして、なんとなく勘をつかみました。

## コードジェネレータ

入力されたRuby文をフロントエンド（トークナイザとパーサ）が構文木とシンボルテーブルとなんやかんやにまとめあげます。
それをバーチャルマシンが実行可能なコードに変換するのがバックエンドたるコードジェネレータです。
バーチャルマシン向けのコードではなく、本当のマシン語に（一部分でもよいから）コンパイルすることができれば実行速度が速くなるのでしょうけど、そういうことはRubyPrizeを受賞するような人たちががんばっていますので期待しましょう。


PicoRubyをつくっているわたくしはRubyPrize最終ノミネートまで残らせていただきまして、これだけでも大変な名誉です。末代までの語り草です。ありがとうございます。


PicoRubyコンパイラをつくる途上、CRubyやmrubyのコードジェネレータは1ミリも読みませんでした。
いまではそこそこC言語が読み書きできます（ほんとか？）が、PicoRubyコンパイラはわたくしの最初の本格的なC言語プロダクトでして、既存のコードを読む力がありませんでしたので、まったくの勘でつくりはじめました。


ただ、最近ちょっとmrubyのコードジェネレータを読み始めています。
理由は、PicoRubyコンパイラを本家mrubyのコンパイラにする、という壮大なプロジェクトを進めているからです。
ここで、29時間くらい絶望していました。
ちゃんとmrubyのコードジェネレータを勉強してからPicoRubyコンパイラを書けばよかった、と反省しました。


理由は、mrubyの内部ではコードジェネレータはいわゆるmrbファイルに相当するような完成品のVMコードは生成せず、mrubyランタイムが実行するのに必要な3つのオブジェクト（パーサ、コンテキスト、スコープ）を生成するからです。
PicoRubyもそのようにつくっておけば、スムーズにmrubyと統合できたのでは？！と反省した、というわけです。


しかし、その反省はいったん撤回しました。
PicoRubyコンパイラはmruby/cとも統合されます。
mruby/cはmrubyと異なる設計のランタイムなので、PicoRubyコンパイラがランタイムに合わせようとすると二重の開発コストがかかってしまいます。


幸いというか当然というか、mrubyにもmruby/cにも完成品のVMコードをloadする関数がありますから、PicoRubyコンパイラはそいつに完成品のVMコードを渡すことにします。
そのせいで無駄にメモリを消費するのでは？という懸念がありますが、取り急ぎの実験として `puts "hello world"` を実行したところ、

- mruby（mruby-compiler + mrubyVM）画像の左側 : 140.0 KB
- MicroRuby（PicoRubyコンパイラ + mrubyVM）画像の右側 : 93.35 KB

というmassifの結果でした。

![](https://user-images.githubusercontent.com/8454208/144202486-4fdbba7e-a78b-46ae-95c1-1e26e403b6ee.png)

コンパイラの消費メモリが小さいことによる総合的な効果は十分に出るであろうと思われるため、当面はこの方法でつくります。
irbのためにはコンテキストオブジェクトの追加開発が必要で、これはもちろんやります。

...という方針でいいでしょうか？というメールをまつもとさんに送ったのですけど返事がなく、とくにアウトプットがなければ正常な証拠というコマンドラインツールの原則に従い、まつもとさんの承認をもらったことにします。


そろそろ書き疲れましたのでこのへんでお仕舞いにします。みなさまよいお年をお迎えください。

-----

あしたは @motoka1t さんと @okuramasafumi さんです！ 
