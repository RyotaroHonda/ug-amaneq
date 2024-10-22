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
- TDC precision:              ~20ps
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