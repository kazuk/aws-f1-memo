# AWS F1 インスタンスを使う上でのメモ書き

## 日本語リソース

ほぼ無い感じ。（だから書いてる）

あまりにも寂しいので slack 作るだけは作ってみた。

[jajp-aws-fpga-f1](https://join.slack.com/jajp-aws-fpga-f1/shared_invite/MjA3MzgzNjIxMDQzLTE0OTkxNDAwNTItY2U3NDlhMjRhMA)

質問とかに答えられる自信は全くない。

とりあえず Developer AMI で place & root 中にマッタリする程度に使ってもらえれば。



以下、まだ未整理な備忘録


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
<dt>AFI</dt> <dd>AWS FPGA Image</dd>
<dt>CL</dt> <dd>customer logic</dd>
<dt>DCP</dt> <dd>design check point </dd>
<dt>HDK</dt> <dd>Hardware Development Kit</dd>
<dt>Shell</dt><dd>AWS F1 インスタンス上で動作する FPGA とのインターフェースを行うCLIツール群</dd>
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

