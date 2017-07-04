# AWS F1 インスタンスを使う上でのメモ書き

## 日本語リソース

ほぼ無い感じ。（だから書いてる）

あまりにも寂しいので slack 作るだけは作ってみた。

[jajp-aws-fpga-f1](https://join.slack.com/jajp-aws-fpga-f1/shared_invite/MjA3MzgzNjIxMDQzLTE0OTkxNDAwNTItY2U3NDlhMjRhMA)

質問とかに答えられる自信は全くない。

とりあえず Developer AMI で place & root 中にマッタリする程度に使ってもらえれば。



以下、まだ未整理な備忘録


## ハード構成

[AWS Shell Interface Specification](https://github.com/aws/aws-fpga/blob/master/hdk/docs/AWS_Shell_Interface_Specification.md) に記述されている。 Shell というのが F1 インスタンス上で FPGA とのやり取りをする為のコマンド群

FPGA チップは Xilinx の VU9P が使われている。

 [UltraScale+ FPGAs Product Tables and Product Secection Guide(pdf)](https://www.xilinx.com/support/documentation/selection-guides/ultrascale-plus-fpga-product-selection-guide.pdf) 

ざっくりと言えば、ハイエンドの中ぐらい。なぜか PCIe Gen3x16/Gen4x8 のチャネル数は VU シリーズで最強の6チャネル、150G Interlakenも最強の9チャネルという事でAWSの中の人の選定理由が透けて見える感はある。（好きなの選べるならコレだな感はある）

F1.16xlarge では FPGA を4個、 F1.4xlarge では FPGAは1個使える。

DDR4 RDIMM 72bit ECC メモリを4チャネル(A,B,C,D)、各4GB接続。 Shell からアクセス可能なのは1チャネルの(C)のみ。

F1.16xlarge では FPGA 間は AXI-Stream で接続されているのでホストを介さずに FPGA 間でストリームを流せる。

## CPU 側アプリケーションからの FPGA アクセス

カーネルモードドライバが入ってれば普通に libc から open,pwrite,pread,fsync,poll で済むって感じ？

OpenCL ランタイム経由でのあれこれはきれいに隠蔽されてるというか、クラウドでそこまで下は触らせられないよね。

### EDMA 経由での DMA IO

[Using AWS EDMA in C/C++ application](https://github.com/aws/aws-fpga/tree/master/sdk/linux_kernel_drivers/edma)

単純に言えば `open("/dev/edma0_queue_0")` して `pwrite` `fsync` で書き込み。 `pread` で読み込み。

### FPGA からユーザープログラムへの割り込み

[Using AWS FPGA user-defined interrupts in C/C++ application](https://github.com/aws/aws-fpga/blob/master/sdk/linux_kernel_drivers/edma/user_defined_interrupts_README.md)

単純に言えば `open("/dev/fpga0_event4")` とかして `poll`



## リンク

[Official repository of the AWS EC2 FPGA Hardware and Software Development Kit](https://github.com/aws/aws-fpga)

[Forum: FPGA Development](https://forums.aws.amazon.com/forum.jspa?forumID=243&start=0)

[Announcement: Upcoming F1 FPGA v1.3.0 Shell and Developer Kit Release](https://forums.aws.amazon.com/ann.jspa?annID=4774)

## AFI のネットリストとかへのアクセス

できないようになっている

AES 暗号化されて RSA で認証してる。

これによって AWS 向けに作ったものは AWS でしか使えない。(Xilinxが開発ツールを無償提供する条件なんだと思う)

[netlist encryption & authentication](https://forums.aws.amazon.com/thread.jspa?threadID=255980&tstart=0)

## FPGA Developper AMI の起動

[FPGA Developer AMI](https://aws.amazon.com/marketplace/pp/B06VVYBLZZ) を起動して接続する。

```
Last login: Fri Jun 16 19:05:52 2017 from 72-21-196-69.amazon.com
 ___ ___  ___   _     ___  _____   __    _   __  __ ___
| __| _ \/ __| /_\   |   \| __\ \ / /   /_\ |  \/  |_ _|
| _||  _/ (_ |/ _ \  | |) | _| \ V /   / _ \| |\/| || |
|_| |_|  \___/_/ \_\ |___/|___| \_/   /_/ \_\_|  |_|___|
AMI Version:        1.2.4
Readme:             /home/centos/src/README.md
GUI Setup Steps:    /home/centos/src/GUI_README.md
AMI Release Notes:  /home/centos/src/RELEASE_NOTES.md
Xilinx Tools:       /opt/Xilinx/
Developer Support:  https://github.com/aws/aws-fpga/blob/master/README.md#developer-support
[centos@ip-172-31-4-157 ~]$ ls /opt/Xilinx/
DocNav  license  SDK  SDx  Vivado  Vivado_HLS

```

 とりあえず、SDx と Vivado_HLS とかは入ってる。



## 略語

<dl>
<dt>AFI</dt> <dd>AWS FPGA Image / Amazon FPGA Image </dd>
<dt>AXI-4</dt> <dd>ARM Advanced eXtensible Interface</dd>
<dt>AXI-4 Stream</dt> <dd>ARM Advanced eXtensible Stream Interface</dd>
<dt>CL</dt> <dd>customer logic / Custom Logic</dd>
<dt>DCP</dt> <dd>design check point </dd>
<dt>DW</dt> <dd>Doubleword: referring to 4-byte (32-bit) data size</dd>
<dt>HDK</dt> <dd>Hardware Development Kit</dd>
<dt>M</dt> <dd>Typically refers to the Master side of an AXI bus.</dd>
<dt>S</dt> <dd>Typical refers to the Slave side of AXI bus.</dd>
<dt>Shell / SH</dt><dd>AWS platform logic responsible for taking care of the FPGA external peripherals, PCIe, DRAM, and Interrupts. AWS F1 インスタンス上で動作する FPGA とのインターフェースを行うCLIツール群</dd>
<dt>ShVersion</dt> <dd>Shell Version</dd>
</dl>



## Forum から

### AGFI の個数制限

 デフォルトで 100 、AWS Support リクエストをすれば 500 まで増やせる。

  [about to hit the 100 AGFI limit, please lift it](https://forums.aws.amazon.com/thread.jspa?threadID=258914&tstart=0)

### `pci_peek` / `pci_poke` とバックプレッシャー

 ソフトウェアとは AXI インターフェースを介して `pci_peek` / `pci_poke`でやり取りできるけど、2.5 μ秒で応答しないと AXI プロトコル違反になる(AXI protocol violation)

  [AXI Timeouts](https://github.com/aws/aws-fpga/blob/master/hdk/docs/HOWTO_detect_shell_timeout.md)

```
$ sudo fpga-describe-local-image -S 0 --metrics
```

 [Do the pci_peek/poke interfaces expose backpressure to software?](https://forums.aws.amazon.com/thread.jspa?threadID=258846&tstart=0)

