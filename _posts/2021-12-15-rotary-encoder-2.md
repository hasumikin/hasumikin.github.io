---
layout: default
title: "[PRK Firmware Advent Calendar 2021] ロータリエンコーダ完全に理解した（後編）"
date: 2021-12-13 10:35:47.342899
categories: 
---

おはようございます。[2021年PRK Firmwareアドベントカレンダー](https://adventar.org/calendars/7086) 15日目の朝です。


さて後編です（[前編はこちら](/hasumin/rotary-encoder-1)）。

## 何に気づいたのか

QMKのロータリエンコーダ動作を観察していたら、つまみに「バネの力を感じている間」（前編を参照のこと）はいっさい出力（アサインしているキーコードの出力）がなく、クリックストップ位置までつまみを動かしてはじめてキーコードが出力されます。
どんなにゆっくりとつまみを回しても、たとえば10秒かけてひと目盛ぶん回しても、バネの力を感じる区間で小さく何往復しても、そのあいだキーボードはうんともすんとも言いません。


バネの力を感じている間というのは、AB接点の状態で言うと `0b01` `0b11` `0b10` のいずれかですよね（しつこいですが前編を読んでください）。
おそらくほとんど `0b11` の状態なんだろうと思いますが、実際には接点がこすれ合っているあいだにバウンスバウンスしているわけです。
で、ここで気づいたのです。

## 途中の状態って（あんま）関係なくね？

そうなのです。
どうせチャタリングが盛大に発生してしまい、不規則な状態遷移しか検出できないのです。
だったら「最初と最後だけ見ればいいじゃん！」ってことです。


たとえば、時計回りの回転が始まったら、接点はこう遷移しますね：

```
A: 0->1
B: 0->0
```

`0b00` -> `0b10 ` の変化をみつけたら、「よっしゃ時計回りがはじまった」と記憶しておき、あとは `0b00` つまり停止状態になるまで待てばよいのです。
途中の `0b10->0b11` とか `0b11->0b01` の遷移を正確にくそ真面目に見張る必要はないのです。


前編では状態を4ビットで持っていましたが、「回転方向」を含めて6ビットに変更しました：

```
0b111111
  ^^     期待される回転方向（10なら時計回り、01なら反時計回り、00なら停止）
    ^^   前回のAB状態（ほんとうはこれは不要だが、デバッグのために残している）
      ^^ 現在のAB状態
```

あとは、

- 前回状態と現在状態が同じならなにもしない
- 停止状態から回転がはじまったらそれを記憶する
- 停止状態への状態遷移だけをひたすら待ち、ひと目盛うごいた！という記録を残す

これだけでよいのでした。
QMKのソースコードを読むことなく、QMKとほとんど同等の使用感を得ることができました。
できてみれば簡単なのですけどね。


ただし、結果がほぼ同じというだけです。QMKの当該ソースをまったく読んでないので、内部でやっていることは違うかもしれません。

[詳しくはソースコード](https://github.com/picoruby/prk_firmware/blob/0.9.8/src/rotary_encoder.c#L25-L74)を読んでください。丁寧にコメントも書いておいたので、この記事とあわせて読めば理解しやすいでしょう。


そんなわけです。チャオ
