# Mikumari Clock Root

## Overview

Mikumari Clock Primary (MikuClockPrim)は17台の下流モジュールを同期する事が出来る、クロック分配ネットワーク上のROOTモジュールです。
ファームウェア内部に64chの1ns精度連続読み出しTDCも有しています。

[Github repository](https://github.com/AMANEQ-official/MikuClockPrim)

```
- Unique ID:                  0xF000

- Number of clock port:       17

- Number of inputs:           64
- Timing measurements:        Both edges
- TDC precision:              1ns
- Double hit resolution:      ~8ns
- Max TOT length:             4000ns

- Link protocol:              SiTCP
- Default IP:                 192.168.10.16
- Data link speed:            1Gbps

- Data word width:            64bit
- Acceptable max input rate:  ~28MHz/board
```

### History

|Version|Date|Changes|
|:----:|:----|:----|
|v2.5|2024.6.9|事実上の初期版|

## Functions

![BL-DIAGRAM](block-diagram.png "Simplified block diagram of MikuClockPrim."){: #BL-DIAGRAM width="90%"}