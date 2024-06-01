---
layout: default
title: "RubyKaigi2019で発表した内容について〈mrubyc-test編〉"
date: 2019-04-23 13:08:24.426298
categories: 
---

去る2019年4月19日、RubyKaigi2019（福岡）に登壇させていただきました。

RubyKaigiでの登壇は2018年につづき2回めでした。

----

[RubyKaigi2019にて生まれて初めて英語プレゼンをしてきた(3/3)](http://shimane.monstar-lab.com/hasumin/taked-in-english-on-RubyKaigi2019-1)という記事もあります

----

発表スライドはこちらにあります。


[https://slide.rabbit-shocker.org/authors/hasumikin/RubyKaigi-2019/](https://slide.rabbit-shocker.org/authors/hasumikin/RubyKaigi-2019/
)


発表では2つのgemを紹介しました。技術の詳細については上の資料、そのうち公開されるであろう録画ビデオ、そしてなによりもGitHub上のソースコードを見ていただくとして、ここでは簡単な概要と当日話さなかったことを書きます。全2回。

## mrubyc-test

まずはmrubyc-testというRubyGemです。


リポジトリはこちら : [https://github.com/mrubyc/mrubyc-test](https://github.com/mrubyc/mrubyc-test)

### 動機

これはmruby/cという「マイコン向けmruby実装」のためのテストツールです。APIの設計は[Test::Unit](https://github.com/test-unit/test-unit)を参考にしています。


2017年にv1.0がリリースされたmruby/cですが、長らくテストツールがなかった（実はあったことを後で知ったのですがこれは後述）ため、アプリ開発が大変っていうか、mruby/c自体の開発も大変そうでした。


たまにmruby/cそのものがリグレッションしてしまったりして、自分のアプリコードに原因をもとめてもさっぱりわからず途方に暮れた日もありました（遠い目）。


そんなわけで以前からテストツールをつくりたいなーとぼんやり考えていました。RubyKaigi2019の登壇プロポーザルの締め切りが2019年1月14日（成人の日）で、その日は3連休の最終日でした。


ちょうどその3連休にうちの神さんが風邪をひきまして、ヒマになったのでプロポーザル書きつついっちょつくってみようか！ってことで2日間くらいでプロポーザルに必要なレベルのPoCをつくったわけです。


その後ちょこちょこ手直ししまして、mruby/c本体の開発チームに「こんなのつくったから本体のテストもこれでやりませんか？」と提案したところ、採用されました。だからmrubyc/mrubyc-testなのです。

### 動作概要

mrubyc-testはGemですので、CRubyで実装されています。CRubyの魔法パワーを利用してmruby/cのアプリコードやテストコードを解析し、テストに必要なstubやmockのコードを動的に生成し、単一バイナリへとコンパイルしてmruby/cのVM上でテストを実行するという仕組みです。


採用される直前に知ったのですが、実はmrubycチームは独自開発のテストツールを持っていたのです。しかもCRubyで情報を集めてmruby/cのバイナリにするという点もまったく同じ発想のものでした。


以下の点でわたくしがつくったもののほうが実用的だったので、こちらが正式ツールに採用されました（、と理解しています）。


- mrubyc-testはgemになっていたため、配布が容易だった
- 同様にgemなのでTravis CIのセットアップもやりやすかった（わたくしがやりました）
- stubとmockの機能があった


stubやmockがないと、mruby/c本体はともかくとして、アプリ開発には使い勝手が悪いです。なぜなら、ハードウェア上で起きることと切り離してアプリのロジックを単体テストしたいからです。これはスライド中でも説明していますので興味のある方は見てみてください。


このテストツールは、/cのつかない方のmrubyにも適用可能かもしれません。コンパイル時のツールチェイン呼び出しをそれ用に変更すれば基本的には動きそうです。が、mrubyにはmrbgemsというライブラリ管理の仕組みがあり（mruby/cにはそういう仕組みがない。ないし、つくられる予定もないし、なくてよいと思っていますが、スニペットをさくっと共有するくらいの仕組みはほしいかも。来年のRubyKaigiでこのことを話せるかな？）、もちろんすでにmruby-testというのがあるので、mrubyc-testはmruby/c専用にしておきます。そのほうがメンテナンスが楽だし。

### 手伝ってくれるひとウェルカム

で、メンテナンスの件。


わたくしは基本的にはTest::Unitユーザなのですが、mrubyc-test自体のテストはRSpecで書いています。
Test::Unit風のAPIデザインをもったテスティングフレームワークのテストをTest::Unitで書くと頭が爆発しそうになるためです。いま自分がどっちを書いているのかわからなくなる。。。

選択肢があるって素晴らしいことですね。


ですがそのspecたちもまだまだ最低限しなかなく、荒野の城バカま（...ちょ、紺屋の白袴を変換できないのかこのIME...）状態です。
みなさまからのspec追加、リファクタリング、そしてもちろん機能改善などのPRをお待ちしております！

----

[後編はこちら](http://shimane.monstar-lab.com/hasumin/RubyKaigi2019-mrubyc-debugger)
