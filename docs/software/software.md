# Software

ここではRBCP経由でAMANEQをスロー制御するためのソフトウェアについて述べます。
SPADI-Aで開発中のNestDAQソフトウェアについては、NestDAQのユーザーガイドを参照してください。

## hul-common-lib

[Github repository](https://github.com/spadi-alliance/hul-common-lib)

hul-common-libはHULのファームウェアをベースにして開発されたも回路用の共通ライブラリです。
AMANEQ、CIRASAME、RAYRAWの制御はこの共通ライブラリで行う事が可能です。
**非常に残念ながらHULは古いソフトから移行が済んでおらず、現在hul-common-libではHULファームウェアの制御は出来ません。**

**Directory structure**

```
.
└── hul-common-lib/
    ├── HulCore
    └── Common
```

HulCoreにはRBCP通信を行うクラス等、共通のライブラリメソッドが格納されています。
コンパイルするとこの部分をまとめたshared libraryが生成されます。
後述するamaneq-softは、このshared libraryを必要とします。

Common内にはいくつか実行体を作るためのソースファイルが格納されています。
生成される実行体で、AMANEQの制御は全て行う事が出来るようになっています。
他のソフトへ制御シーケンスを移植する際には、これらソースファイルの記述を参考にしてください。

### Executable files

hul-common-libをコンパイルすると生成される実行体について説明します。
**全ての実行体は引数無しで実行するとusageが表示されます。**
取るべき引数を確認するために活用してください。

#### write_register

SiTCPのRBCP経由でlocal bus moduleへレジスタを書き込むための実行体です。RBCP通信クラスを包含しています。
`write_register`と`read_register`の2種類でAMANEQのファームウェアは全て制御できるようになっていますが、制御手順が非常に煩雑な機能については別途実行体を用意しています。

IPアドレス192.168.10.16のローカルアドレス0x40100000へ0x1Aという値を2-byte幅で書き込む例です。
```shell
               [IP]          [Addr]      [val] [bytes]
write_register 192.168.10.16 0x40100000  0x1A  2
```
この関数は4-byte書き込みまで対応しています。
RBCPは本来1-byte幅でしか同じアドレスに書けないのですが、プログラムの中で同一ローカルアドレスに対して他バイトの値を書き込んでいるように見せかけています。

レジスタ長が8-bitの倍数でない場合、例えば13-bit幅のレジスタに2-byte書き込みを行った場合、存在しない上位3-bit部分の値は何を入れても無視されます。

#### read_register

SiTCPのRBCP経由でlocal bus moduleからレジスタを読み出すための実行体です。
読み出し結果は標準出力へ表示されます。

IPアドレス192.168.10.16のローカルアドレス0x40100000から2-byte幅で値を読み出す例です。
```shell
              [IP]          [Addr]     [bytes]
read_register 192.168.10.16 0x40100000 2
```

#### check_spi_device

ボード上のSPIフラッシュメモリの型番を調べます。マニュファクチャIDをメモリから読み出して判別しているので、正常に読み出せていてもソフト側がそれを知らないとunknownと表示されてしまいます。
そのような現象に遭遇したらgithub repositoryでissueを挙げていください。

#### erase_eeprom

SiTCPのライセンスやIPアドレスなどの情報が格納されている不揮発性メモリ（EEPROM）のデータを全消去する実行体です。
[SiTCPのTCP接続が再接続の際に非常に待たされる](https://hul-official.gitlab.io/hul-ug/practical/main/)という現象を解決するために必要な実行体です。
通常は使用しません。

#### flash_memory_programmer

MCSファイルをSiTCP経由でフラッシュメモリに書くための実行体です。
通常MCSファイルはJTAG経由でフラッシュメモリへ書き込みますが、ローカルバスモジュールのFMPを経由して書き込むためのプログラムです。
当然ですが、FMPが実装されたファームウェアがFPGA上で動いている事が前提条件です。

フラッシュメモリの消去、書き込み、ベリファイとJTAGで書き込むときと同じ手順を踏みます。
ベリファイの結果最後に`MCS is verified`と表示されることを確認してください。
Mismatchと表示された場合、通信エラー等で書き込んだデータと読み出したデータが一致していません。
Verifiedと表示されるまで繰り返し書き込んでください。
Mismatchと表示されているのに電源を落としたりFPGAをリコンフィグしてしまったりすると、ボードが起動しなくなります。
その場合はJTAGケーブルを使って対処してください。

RBCPを用いて数MBあるファイルを送信するので、何度もRBCP通信が行われます。
RBCPはUDP通信のためパケットロスの可能性があり、ネットワークが不安定だったり、他の通信パケットが沢山流れている環境だと失敗しやすいです。
また、無線経由でも失敗しやすいです。
`flash_memory_programmer`を使う際には、他のプログラムから該当AMANEQへアクセスするのをやめ、クリーンな状況で書き込んでください。

MCSをフラッシュメモリへ書き込んでもFPGAの中身はそのままです。
書いた内容をFPGAへ反映させるためにはボードの電源を一度落とすか、`reconfig_fpga`を実行してFPGAを再コンフィグしてください。

#### gen_user_reset

Local Bus ControllerからBCTリセットを発行します。
FPGA内部の多くのローカルバスモジュールがリセットされるのでレジスタの再設定を行ってください。

#### get_version

ファームウェアのIDとバージョンを取得します。
何かおかしいと思った時にとりあえず実行するプログラムです。

#### inject_sem_error

開発者向けのデバッグツールです。SEMを通じて任意の領域にエラーを発生させるために使います。ユーザー向けの機能ではないのでここでは説明を省略します。

#### mcs_converter

MCSからSPIフラッシュに書き込むためのバイナリファイルを作成します（MCSはテキストファイル）。`flash_memory_programmer`内ではバイナリ変換を同時に行っているので、このプログラムはデバッグ時以外に使うことは無いと思います。
Vivadoが生成するとBINファイルと一致するかどうかは調べていません。誰か調べてください。

#### read_scr

スケーラーを読み出し、その結果をバイナリデータファイルへ保存する実行体です。
読み出しバイト数の取得、FIFOが空である事の確認（およびFIFOリセット）、ラッチリクエストの送信、FIFO読み出し、ファイル書き出し、の一連のプロセスを実行します。

以下の例では指定したIPアドレスのボードから読み出した結果をscaler.datへ保存します。引数の数が2つだと上書きモード、3つだと追記モードで動作します。追記モードでは既にファイルが存在する時、前のデータを消さずに追記します。
```shell
read_scr 192.168.10.16 scaler.dat      (Overwrite mode)
read_scr 192.168.10.16 scaler.dat 1    (Append mode)
```

繰り返しスケーラー読み出しをする場合、この実行体をシェルスクリプトで何度も呼び出してもあまり高い周波数で読めません。
プログラムを実行するオーバーヘッドがかさんだり、プログラム内で毎回読み出しバイト数を確認していたりするためです。
高速に何度も読み出したい場合、この実行体のソースコードを参考にして自分でプログラムを書いてください。

#### read_sem

SEMにアクセスしてSEUを修正した回数、修正不可能な状態になっていないかの確認を行います。
放射線が気になる環境では定期的に読み出して、修正不能なエラーが発生していないか確認してください。
もし、`uncorrectable_error`フラグが立っていたらFPGAを再コンフィグしてください。

#### read_xadc

XADCにアクセスしてFPGAの温度と電源電圧値を取得します。

#### reconfig_fpga

SPIフラッシュメモリからFPGAを再コンフィグする命令を送信します。FPGA即座にダウンするのでRBCP通信のタイムアウトが発生しますが、正常な動作です。
10秒程度してから再アクセスしてください。

#### reset_sem

SEMの状態をリセットします。SEUを修正た回数も0に戻ります。

#### reset_sitcp

SiTCPコアのリセットを行います。

#### send_cbtinit

MIKUMARIリンクの物理層をリセットします。デバッグ以外の用途では使用しません。

#### set_cdce62002

CDCE62002 ICへレジスタを書き込むための実行体です。
AMANEQが納品されたら必ず1度実行する必要があるプログラムです。
CDCE62002には多数のパラメータがあり、1つ1つ手動で設定するのはユーザーには難しいので、いくつかのプリセットを用意しています。
どのようなプリセットがあるかはプログラムのusageに書いてあります。

以下は125 MHz入力から500 MHzと125 MHzの出力を作るための設定を書き込む時の例です。
```shell
set_cdce62002 192.168.10.16 in_125_out_500_125
```

書き込みを行った後、レジスタリードバックを行っています。
この時0でない値が返ってきている事を確認してください。もしなんどやっても0が返ってくる場合、何かトラブルがあるかもしれません。

#### set_ctrl_eeprom

EEPROMへ制御レジスタを書き込みます。デバッグ以外では使用しません。

#### set_hbfstate

MikuClockRootおよびMikuClockHubに対して使用する実行体です。
ハートビートフレーム状態のON/OFFを切り替えます。LACCPを通じてフレーム状態は下流の全てのモジュールに伝搬します。

第2引数に`on/off`を入力して状態を変更してください。
```shell
set_hbfstate 192.168.10.16 on
set_hbfstate 192.168.10.16 off
```

MIKUMARI Utilityが`0x0000'0000-0x0FFF'0000`のアドレス範囲に存在しないファームウェアでは、第3引数にアドレスオフセットを与えてください。

#### set_max1932

AMANEQでは使用しない機能です。CIRASAMEとRAYRAW上のAPDバイアスチップへレジスタを送信するために使います。

#### set_sitcpreg

SiTCPの予約レジスタ領域にアクセスしてSiTCPのパラメータを書き換えます。
SiTCPの説明書をよく読んで使用してください。

#### show_laccp

LACCPのリンク状況と、自身の時刻オフセットを調べる機能です。

`-- Link Up status --`はLACCPのリンクアップ状態を示すビット列です。
ファームウェアによっては複数のLACCPが動いているので、全て1列に表示しています。
ビット位置がポート番号に対応しており、ポート番号とボード上の位置は各ファームウェアの記述を参照してください。
1でLACCPがリンクアップしている事を示します。

`-- My Offset --`はそのファームウェアのセカンダリLACCPの情報を示します。つまり、受信ポートです。
Heartbeat count offsetはMIKUMARIパルスの往復時間をシステムクロックのunit interval 精度で示しています。
ファイバーが長くなると大きくなる数字です。
Local fine offsetとLACCP fine offsetは1つ上流のモジュールと最上流のモジュールに対するクロック信号の位相差をそれそれ示しています。

`Partner IP address`の項目では相手側のボードのIPアドレスををポート番号ごとに表示しています。

#### show_mikumari

MIKUMARIリンクの状況を詳細に調べるための実行体です。何か問題がある時に使う事が多いと思います。

`-- Link Up Status --`では物理層（CBT）、リンク層（MIKUMARI）、時刻同期層（LACCP）の接続状態を一覧できます。
途中まではアップしているのに上位プロトコルがアップしていないときは、ファイバーの抜き差しなどをトライしてみるといいでしょう。

#### strdaq

TCP接続を行いデータを収集するためのプログラムです。**動作確認用の非常に簡易的なプログラムであり、このソースコードを移植して独自のDAQソフトウェアを作る事は推奨しません。**
このプログラムを実行するには`data`ディレクトリを作成しておく必要があります。

```shell
       [IP]          [RUN no.]
strdaq 192.168.10.16 1
```
上記の例ではrun1.datというファイルが`data`ディレクトリ以下に生成されます。
プログラムを止めるためには`Ctrl-C`を押下してください。

#### verify_mcs

SPIフラッシュメモリからデータを読み出して、比較対象のMCSファイルと一致するかどうかを確認します。
`flash_memory_programmer`に組み込まれているプロセスですが、確認だけしたい時に使います。


### DAQ data taking

連続読み出しDAQではロスレスでデータをPCが受け取る事が前提なので、トリガー型DAQのときよりもソフトウェアのパフォーマンスが重要になります。
ソケットからのデータ受信、処理、ファイル書き込みはそれぞれ独立したプロセスである事が望ましく、また各プロセス間はqueueing bufferで接続されている事が必要です。
つまり、それなりにちゃんと設計されたソフトウェアが必要です。
`strdaq`実行体はこれらを全て1つのプロセス内で行っています。**strdaqプログラムを使って物理実験を行う事は全く推奨しません。**

イベント毎にデータを読み出していたHULと異なり、連続的にデータが返ってくるAMANEQではDAQ読み出しプログラムはかなり簡素になりました。
実際にソースを見てもらうと分かりますが、TCP接続を確立して、ソケットを読んで、データをファイル書き込みしているだけです。
他には何もしていません。
そのため、わざわざ`strdaq`プログラムを使う必要性も薄く、正直なところ簡易的な試験であれば`nc`コマンドをリダイレクトした方がいいかもしれません。

## amaneq-soft

[Github repository](https://github.com/spadi-alliance/amaneq-soft)

hul-common-libに共通化できなかった各ファームウェア固有の機能をまとめたものです。
コンパイルするためには先にhul-common-libをインストールしている必要があります。

### Str-LRTDC

#### set_tdcmask

Str-LRTDCはTDCマスクをセットするためのレジスタが5つもあり、個別にセットすると大変なのでまとめて設定するための実行体です。
例えば、16-31chと64-79chにマスクを設定したい場合以下のようになります。
```shell
set_tdcmask 192.168.10.16 0xffff0000 0x0 0xffff 0x0 0x0
```