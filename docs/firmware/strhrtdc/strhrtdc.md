# Streaming high-resolution TDC

## Overview

Streaming high-resolution TDC (Str-HRTDC)は20ps精度の連続読み出しTDCです。
専用のメザニンカードであるmezzanine HR-TDCカードが必要で、メザニン上のFPGAとAMANEQ本体のFPGAが連携して動作する事で1つのモジュールとして動作します。
それぞれのファームウェアを区別するときはAMANEQ側をStr-HRTDC Base、メザニン側Mezzanine (Mzn) Str-HRTDCと表記します。

[Github repository (AMANEQ)](https://github.com/AMANEQ-official/StrHrTdcBase)

[Github repository (Mezzanine)](https://github.com/AMANEQ-official/MznStrHrTdc)

```
- Unique ID (AMANEQ):         0xC480
- Unique ID (Mezzanine):      0x804c

- Number of inputs:           64
- Timing measurements:        Both edges
- TDC precision:              ~23ps
- Double hit resolution:      ~2ns
- Max TOT length:             4000ns

- Link protocol:              SiTCP-XG
- Default IP:                 192.168.10.10
- Data link speed:            10Gbps

- Data word width:            64bit
- Acceptable max input rate:  ~125MHz/board
- System clock freq.:         125MHz
```

### History

|Version|Date|Changes|
|:----:|:----|:----|
| AMANEQ |||
|v2.4|2024.6.4|事実上の初期版|
| Mezzanine |||
|v2.4|2024.6.4|事実上の初期版|

## Functions

![BL-DIAGRAM](block-diagram.png "Simplified block diagram of Str-HRTDC."){: #BL-DIAGRAM width="100%"}

[図](#BL-DIAGRAM)はStr-HRTDCの簡易ブロックダイアグラムです。
AMANEQ側にデータリンクとMIKUMARIポートが存在するため、外部との通信や同期はAMANEQが行います。
メザニンとはメザニンベースコネクタを通じてつながっており、複数の通信プロトコルでAMANEQ上のFPGAと通信を行います。
Str-HRTDC Base上にはMikumariClockHubと同様にクロックを受信するためのLACCPと再送信するためのLACCPが存在し、まずAMANEQ上の時刻を同期して、更にメザニン上の時刻同期を行います。
そのため、Str-HRTDC上には3つの時刻ドメインが存在し、各々が内部のハートビートカウンターを頼りに動いています。

Str-HRTDC Base上にはstreaming TDCの機能の内back merger以降の機能が実装されており、メザニンカードからデータを集めてSiTCP-XGを通じてPCへ送信する事が役割です。
AMANEQ本体には時間を測定する機能は実装されておらず、メザニンカードを搭載していない場合何もできません。
Scalerは限定的な情報を記録しているのみです。詳しくは後述します。
Base ファームウェアはスタンドアロンモードを実装しており、AMANEQ単体で動作する事が可能です。
システムクロック信号の生成にはCDCE62002を使用しているので、**CDCE62002を未設定のAMANEQではこのファームウェアは動作しません。**
125 MHzの入力から500 MHzと125 MHzのクロック信号を生成するようにCDCE62002を設定してください。

Str-HRTDCとしての主機能は殆どがメザニン上のFPGAに実装されています。
メザニンカードにつき32ch入力が可能です。
**入力信号規格はLVDSのみです。AMANEQ本体とは違い、ECLやPECLはサポートしていません。**
メザニン上のstreaming TDCブロックにはfront-mergerまでが実装されており、時間分解能が20psである事以外は、構造的に殆どStr-LRTDCと同様です。
Streaming TDCとしての基本的な動作についてはStr-LRTDCを参照してください。
Scaler機能についてもStr-LRTDCと同様ですが、システムインフォメーションを2つのメザニンカードは独立に記録しているので、その取扱いに注意が必要です。これについても後述します。

本ファームウェアに搭載されている通信プロトコルはSiTCP-XGです。
**Gigabit Ethernetでは動作しません。相手側も10Gigabit Ethernetで通信可能な機器である必要があります。**
また、auto-negotiationは働かないです。

![PORT-MAP](port-map.png "Port map of Str-HRTDC"){: #PORT-MAP width="80%"}

[図](#PORT-MAP)はTDC入力チャンネル番号とMIKUMARIのポート番号を示しています。
MIKUMARIのポートは、0番がupper mezzanine、1番がlower mezzanine、2番が受信用のポートにアサインされています。

### LED and DIP switch

MIKUMARIシステムを利用している場合、1-3番がすべて点灯していれば正常です。
スタンドアロンの場合、1番と3番が点灯していれば正常です。

|LED #||Comment|
|:----:|:----|:----|
|1| PLL locked| 全ての内部クロック信号が正常に出力されている状態です。 |
|2| MIKUMARI (2) link up| MIKUMARIポートの2番がリンクアップしている状態です。 |
|3| Ready for DAQ| 時刻同期が完了し、DAQを走らせられる状態である事を示します。 |
|4| DAQ is running| データ読み出し中である事を示します。 |

Str-HRTDC Baseは2枚ともメザニンカードを搭載しているケースと、upper slotにだけメザニンカードを搭載しているケースをサポートしています。
Lower slotにだけメザニンカードを搭載すると動作しません。
**搭載メザニン数に応じてDIP 3番を適切に設定してください。**

|DIP #||Comment|
|:----:|:----|:----|
| AMANEQ |||
|1| SiTCP IP setting | 0: デフォルトIPを使用します <br> 1: ユーザー設定のIPを使用します (要ライセンス)。|
|2| Standalone mode | 0: MIKUMARIシステムを使用します<br>1: ローカル発振器を使用しスタンドアロンモードになります|
|3| Lower Mzn absent | 0: 2枚ともメザニンカードを搭載している場合 <br> 1: Upper mezzanine slowのみ使用している場合|
|4| NIMOUT setting | 0: NIMOUT-1からハートビート信号が出力されます<br>1: NIMOUT-1からLACCPがトリガー信号が出力されます|
| Mezzanine |||
|1| Not in use | |
|2| Not in use | |
|3| Not in use | |
|4| Not in use | |

## Local bus modules

Str-HRTDCでは2つのFPGAが接続されているので、メザニン側のFPGAを制御するためにlocal busの拡張が必要になります。
2つのFPGAは[図](#BUS-BRIDGE)に示すようにbus bridge primary/secondaryを通じて接続されています。
AMANEQ側のbus bridge primary (BBP) はlocal bus moduleです。
AMANEQ側から見るとメザニンFPGAへの通信はBBPへの読み書きに見えます。
一方、メザニン側のbus bridge secondaryはlocal bus controllerから見ると外部リンクに見えます。

BBPを介してのアクセスには通常の通信よりも余計に手順がかかります。
そのため、hul-common-libで用意している読み書き用の実行体では煩雑になってしまうため、amaneq-softで一連の通信をまとめたwrite/read_mzn_registerという実行体を用意しています。
メザニンFPGAへアクセスするときはこの実行体を使用してください。

メザニンベースコネクタを介して2つのFPGAつながっているため、しっかりつながっていないと通信エラーを起こします。
一度エラーを起こすとlocal busが応答しなくなるため、リセットボタンによるハードウェアリセットか、MIKUMARIによる遠隔リセットが必要になります。

![BUS-BRIDGE](bus-bridge.png "Local bus bridge"){: #BUS-BRIDGE width="80%"}

Str-LRTDC Baseには8個のローカルバスモジュールが存在します。
以下がローカルバスアドレスのマップです。

|Local Module|Address range|
|:----|:----|
|Mikumari Utility        |0x0000'0000 - 0x0FFF'0000|
|DAQ Controller          |0x2000'0000 - 0x2FFF'0000|
|BusBridgePrimary upper  |0x3000'0000 - 0x3FFF'0000|
|BusBridgePrimary lower  |0x4000'0000 - 0x4FFF'0000|
|Scaler                  |0x8000'0000 - 0x8FFF'0000|
|CDCE62002 Controller    |0xB000'0000 - 0xBFFF'0000|
|Self Diagnosis System   |0xC000'0000 - 0xCFFF'0000|
|Flash Memory Programmer |0xD000'0000 - 0xDFFF'0000|
|Bus Controller          |0xE000'0000 - 0xEFFF'0000|

Mezzanine Str-LRTDCには5個のローカルバスモジュールが存在します。
メザニン側は使用するアドレスのレンジが異なっています。

|Local Module|Address range|
|:----|:----|
|Mikumari Utility        |0x0000 - 0x0FFF|
|DAQ Controller          |0x1000 - 0x1FFF|
|Streaming TDC           |0x2000 - 0x2FFF|
|Scaler                  |0x8000 - 0x8FFF|
|Self Diagnosis System   |0xC000 - 0xCFFF|
|Bus Controller          |0xE000 - 0xEFFF|

## Module reset



## Streaming-TDC block

Streaming TDCの基本構造はStr-LRTDCと同様です。まずは、Str-LRTDCのページを参照してください。
ここではHR-TDCに特有の事を記述します。

### Data merging block

Str-HRTDCではfront-mergerとback-mergerが物理的に2つのFPGAに分かれているため、限られた信号線でstreaming TDC全体を制御しています。
そのため、高負荷になった時の制御がうまくいかずデータ破損がStr-LRTDCに比べると起こりやすいです。
また、破損が発生した時に復帰がうまくいかない事もあります。
多くの場合で、データ破損はback-merger上のlocal heartbeat frame number mismatchとして検出されます。
Self recovery modeをONにしている場合、この時自動復帰を試みますが、前述のように自動復帰できない事があります。
Delimiter flagsを監視し、このエラーが発生し続けるようであればDAQを止めてモジュールリセットを試みてください。