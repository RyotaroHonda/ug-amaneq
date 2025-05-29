# Streaming low-resolution TDC with 10GbE

## Overview

The streaming low-resolution TDC with 10GbE (Str-LRTDC 10G) is a continuous readout TDC with 1ns timing precision.
This firmware is basically the same as Str-LRTDC, but the data link protocol is replaced with SiTCP-XG.

[Github repository](https://github.com/AMANEQ-official/StrLrTdc-10G)

```
- Unique ID:                  0x61c4

- Number of inputs:           128
- Timing measurements:        Both edges
- TDC precision:              1ns
- Double hit resolution:      ~8ns
- Max TOT length:             4000ns

- Link protocol:              SiTCP-XG
- Default IP:                 192.168.10.10
- Data link speed:            1Gbps

- Data word width:            64bit
- Acceptable max input rate:  ~125MHz/board
- System clock freq.:         125MHz
```

### History

|Version|Date|Changes|
|:----:|:----|:----|
|v2.0|2025.5.29| 1st version. Functionalities are the same as those of StrLrTdc v2.9.|

## Functions

データリンクスピードを除き、全ての機能がStr-LrTdcと同一である。
