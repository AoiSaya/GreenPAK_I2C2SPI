# I2C-to-SPI bridge and GPIO extender using GreenPAK. (緑豆壱拾陸號)

GreenPAK用のデザインデータです。  
SLG46826V(STQFN) または SLG46826G(TSSOP) に対応しています。  
I2CインタフェースからSPIに対応したデバイスへのアクセスを可能にします。
同時にGPIOを追加することもできます。  

## 機能
- スレーブアドレス指定で最大4つのSPIデバイスへのアクセスが可能
- リードアフターライト形式のSPIに対応
- 双方向データ端子に対応
- 4chのGPIOが利用可能
- I2C入力に約10kΩのプルアップ抵抗を内蔵
- ライト中のデータリードには非対応


### 本デザインの使用イメージ
パスコンは省略してあります。  
<img src="img/SampleSch.jpg" width="800">  

## ピンアサイン

STQFN Pin # | TSSOP Pin # | 機能名 | IO | 内蔵抵抗 | SLG46826端子名 | 機能
--- | --- | --- | --- | --- | --- | ---
STQFN Pin # | TSSOP Pin # | 機能名 | IO | 内蔵抵抗 | SLG46826端子名 | 機能						
1 | 20 | VDD |  |  | VDD |  2.3V～5.5V
2 | 19 | SS0 | O | - | IO0 | SPI ch0 Slave Select, active low
3 | 18 | SS1 | O | PU10k | IO1 | SPI ch1 Slave Select, active low
4 | 17 | SS2 | O | - | IO2 | SPI ch2 Slave Select, active low
5 | 16 | SS3 | O | - | IO3 | SPI ch3 Slave Select, active low
6 | 15 | SCL2 | I | PU10k | IO4 | I2C SCL for SPI
7 | 14 | SDA2 | I/O | PU10k | IO5 | I2C SDA for SPI
8 | 13 | SCL | I | - | SCL | SCL for SLG46826
9 | 12 | SDA | I/O | - | SDA | SDA for SLG46826
10 | 11 | DC | O | - | IO6 | data / command select for LCD
11 | 10 | GND |  |  | GND |  GND
12 | 9 | SCK | O | - | IO7 | Serial Clock
13 | 8 | SO | I/O | PU1M | IO8 | Serial Data Output
14 | 7 | VDD2 |  |  | VDD2 |  2.3V～5.5V
15 | 6 | SI | I | PU1M | IO9 | Serial Data Input
16 | 5 | NC | - | PU10k | IO10 | Do Not Connect (internal use)
17 | 4 | GPIO0 | I/O | PU1M | IO11 | general-purpose input/output 0
18 | 3 | GPIO1 | I/O | PU1M | IO12 | general-purpose input/output 1
19 | 2 | GPIO2 | I/O | PU1M | IO13 | general-purpose input/output 2
20 | 1 | GPIO3 | I/O | PU1M | IO14 | general-purpose input/output 3
  
### 内蔵抵抗
各端子の内蔵抵抗を有効にしてありますので、I2Cの外付けプルアップ抵抗を省略できます。  
記号はおおよその抵抗値を表しており、値は以下の通りです。  
- PU10k: Pull-up 10k ohm  
- PU1M: Pull-up 1M ohm  


### スレーブアドレス

SPIデバイスの選択はI2Cのスレーブアドレスの下位2ビットで行います。  
スレーブアドレスの上位5ビットは、設計ツール上でPGENのデータを書き換えるか、動作時にSLG46826へのI2Cアクセスで変更することができます。デフォルト値におけるSS0～SS3とスレーブアドレスとの対応は以下の通りです。
pin | |slave address|
SS| | W | R | select bit | default |  | hex
SS0 | SO | SI | xxxxx00 | 1010100 | 0x | 54
SS1 | SO | SI | xxxxx01 | 1010101 | 0x | 55
SS2 | SO | SI | xxxxx10 | 1010110 | 0x | 56
SS3 | SO | SO | xxxxx11 | 1010111 | 0x | 57


## コンフィギュレーションレジスタ
動作中にSLG46826のI2Cレジスタを書き換えることで、GPIOを用いた入出力やSPIの動作モードなどを切り替えることができます。
GPIOのデフォルトは全端子入力、SPIはモード0、SO端子は出力オンリーです。
SLG46826のI2Cアドレスは、0x08です。0x09～0x0Fはコンフィグ用に予約されていますのでアクセスしないでください。
変更するには設計ツールでI2Cのプロパティを書き換えてください。


address | bit | Definition | default
0x7A | [7:6] | "GPIO3 control
00:input, 01:reserve, 10:output 0, 11:output 1" | 0x00
 | [5:4] | "GPIO2 control
00:input, 01:reserve, 10:output 0, 11:output 1" | 
 | [3:2] | "GPIO1 control
00:input, 01:reserve, 10:output 0, 11:output 1" | 
 | [1:0] | "GPIO0 control
00:input, 01:reserve, 10:output 0, 11:output 1" | 
0x92 | [7] | Reserve | 0x54
 | [6:2] | Slave address[6:2] for SPI function | 
 | [1:0] | Reserve | 
0x03 | [7:0] | "SO timing
0x46: SPI mode 0 or 2, 0x50: SPI mode 1 or 3" | 0x46
0x90 | [7:0] | "SCK polarity
0x88: SPI mode 0 or 1, 0x87: SPI mode 2 or 3" | 0x88
0x0C | [7:0] | "Bidirectional SO support for reading
0x3F:normal, 0x5B:SS0, 0x6B:SS2, 0x6A:SS3, 0x55:SS0 and SS1, 0x56:SS0 and SS2, 0x42:SS1 and SS3, 0x40:always" | 0x3F


## 設計データ

「GreenPAK6 Designer」で  
I2C2SPI.gp6  
を開き、SLG46826Vに焼いてください。
SLG46826G に焼く場合は、File-Project info で Packageを「TSSOP-20」に変更してください。

## 免責事項

当方は、利用者に対して、このデザインおよびこの資料（以下、本デザイン等）に関する当方または第三者が有する著作権、特許権、商標権、意匠権及びその他の知的財産権をライセンスするものではありませんし、本デザイン等の内容についていかなる保証をするものでもありません。また当方は、本デザイン等を用いて行う一切の行為について何ら責任を負うものではありません。本デザイン等の情報の利用、内容によって、利用者にいかなる損害、被害が生じても、当方は一切の責任を負いません。ご自身の責任においてご利用いただきますようお願いいたします。  


## Author  

[GitHub/AoiSaya](https://github.com/AoiSaya)  
[Twitter ID @La_zlo](https://twitter.com/La_zlo)  
