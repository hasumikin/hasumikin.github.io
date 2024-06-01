---
layout: default
title: "[PRK Firmware Advent Calendar 2021] PRK Firmware is 何？"
date: 2021-11-29 11:54:08.046575
categories: 
---

この記事は[PRK Firmware Advent Calendar 2021](https://adventar.org/calendars/7086)の12月1日のエントリです。


- PRK Firmwareとは、自作キーボードのファームウェアの一種です
  - 自作キーボードとは、キースイッチやキーキャップなどの部品をユーザ自らが選択し、好みに合ったキーレイアウトや外観のキーボードキットを購入し（あるいは自ら設計・製造し）、組み立て（あるいは完成品または半完成品を購入し）、キーマップを自ら定義し（あるいはデフォルトのキーマップをそのまま利用し）、快適なタイピング人生を送るための道具です
    - 一部には、キーボードをコレクションすることによって資産形成できると主張する人もいます
    - キースイッチは機械部品であり、潤滑油を塗布することで滑らかな動作になるため、潤滑そのものを趣味にする人もいます
    - キーキャップは直接に指と触れるパーツであり、キーボードの外観にも大きな影響を与えます
      - 形状が使用感を大きく左右するため様々な種類（プロファイル）が考案され、Profileの原義である横顔が示すとおり、主に横から見たときの形状がそれぞれの特徴を表しています
    - キーマップとは、どのキーがキーボード上のどの位置にあるかを定義するものです
      - 一般的なキーボードと異なり、レイヤという概念を導入することで、少ないキーの数でキーボード機能を十全に果たすこともできます
        - レイヤによって入力されるキーが変更されることは、シフトキーによって大文字と小文字が入れ替えられたり数字と記号が入れ替えられたりすることに似ています
  - ファームウェアとは、ハードウェア（本稿の文脈ではキーボードに搭載されたマイコン）上で動作するソフトウェアです
    - マイコンとは、小さなコンピュータのことです
    - マイコンには様々な種類がありますが、2021年現在のPRK Firmwareが動作するのはRP2040というMCUをベースにしたマイコンボードです
      - MCUとは、狭義のマイコンです（他方、マイコンボードのことを広義のマイコンと捉えることが可能です。後述）。通常、ひとつのチップのなかにCPUやROMやRAMやペリフェラルが集積されています（ただしRP2040にはROMが搭載されていません）
        - ペリフェラルとは、MCUに付加されている機能の総称です。例として、MCUが外部とデジタルデータをやりとりするためのデジタルピン機能、アナログ電圧値をデジタル値に変換する機能、各種の通信規格を制御する機能などがあり、MCUによって搭載される種類や数量が異なります
    - マイコンボードとは、プロトタイピングや趣味の電子工作のために製造され市販されるもので、MCUに加えてUSBポートや電源回路やMCUを補うROMなどのパーツをひとつの基板にアセンブルし、一般には2.54mmピッチのピンを引き出すことでMCUの機能を容易に利用できるようにしたものです
  - PRKとはPico Ruby Keyboardの頭文字を表します
    - PicoRubyとはmrubyのワンチップマイコン（MCU）向け実装です
      - mrubyとは組み込み用途のためのRuby実装です
        - Rubyとは、まつもとゆきひろさんによって策定されるプログラミング言語の指針です
        - Ruby実装とは、指針としてのRubyを実際に動作するソフトウェアにしたものです
        - 組み込み用途とは、筆者もよくわからないくらい広い概念である気がするので説明を省略します
          - 少なくとも、マイコンのファームウェアを「組み込みソフトウェア」と呼ぶのは正しい用法です
      - mubyの実装は、mrubyコンパイラとmrubyバーチャルマシンによって成り立っています
        - mrubyコンパイラがVMコード（中間コード、バイトコードなどとも呼ばれます）を生成し、それをmrubyバーチャルマシンが実行するという言語実装コンセプトにより、コンパイラを動作環境に組み込まない、というアプリケーション設計が可能です
      - PicoRubyの実装は、PicoRubyコンパイラとmruby/cバーチャルマシンによって成り立っています
        - いずれも、一般的なコンピュータではなく、メモリの小さなMCUで動作することを目的に開発されています
          - 32ビットアーキテクチャの場合、128KB程度のROMと32KB程度のRAMがあれば、murby/cバーチャルマシンとコンパイル済みのVMコードによる小～中規模のアプリケーションを実行できます
          - PicoRubyコンパイラをともに組み込む場合、ROMは上記の3倍、RAMはコンパイルするコード量により2倍から4倍程度あることが目安になります
      - PicoRubyのRubyコンパイラが生成するVMコードは、mrubyのVMコードの仕様に準拠しています（2021年現在では下方互換）
  - PRK Firmwareは、実装の多くの部分がRubyで書かれ、PicoRubyコンパイラでVMコードへコンパイルされています
    - マイコンペリフェラルを操作する関数、動作にミリ秒以下の遅延を許したくない場面にC言語が使用されています
  - ユーザは、キーマップ（keymap.rb）をRubyで書くことができます
  - ユーザがキーマップをコンパイルする必要はなく、マスストレージデバイスとしてPCにマウントされたRP2040マイコンボードにkeymap.rbをドラッグ＆ドロップするだけで、マイコンボード上のPicoRubyコンパイラが自動的にコンパイルし、新たなキーマップが適用されます
  - キーボードユーザは、keymap.rbに独自の実装を追加することで、キーボードの機能を拡張することができます
  - これらについて、RubyKaigi Takeout 2021での[発表動画とスライド](https://rubykaigi.org/2021-takeout/presentations/hasumikin.html)がありますのでご参照ください