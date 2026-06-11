*This project has been created as part of the 42 curriculum by ksaotome.*


☓
ヴァーチャルボックスインストール？
https://www.virtualbox.org/wiki/Downloads

コマンド？サイトから？


➜  ~ cat /etc/os-release 

NAME="Ubuntu"

◦ Debian vs Rocky Linux
→オープンソースのLinuxディストリビューションの種類(Linuxの種類)
Debianの方が設定が簡単。

| | 系列 | 命令 |
| --- | --- | --- |
| Debian | Debian(Ubuntu) |
| Rocky Linux | Red Hat（RPM） |  |


◦ AppArmor vs SELinux
◦ UFW vs firewalld

---

◦ VirtualBox vs UTM
→仮想化ソフト
仮想環境を作成できる

|| OS | CPU | 命令 |
| --- | --- | --- | --- |
| VirtualBox | Windows/Linux | Intel/AMD | x86_64 |
| UTM | Mac M1/M2/M3 | Apple Silicon | ARM |

https://qiita.com/RanmaRanma/items/d708d42674928ccc3e42

VirtualBoxはx86_64専用
Mac M1はCPUの種類が根本的に違うのでVirtualBoxがそもそも動かない
UTMはARM上でも動くように作られているので、Mac M1でも使える

今回はlinuxで環境構築のためVirtualBox

---
|| Debian | Rocky Linux |
| --- | --- | --- |
|| AppArmor | SELinux |
| セキュリティモジュール | UFW | firewalld |
| 仮想化ソフト | VirtualBox | UTM |




https://zenn.dev/kodyi/articles/1440a8e2ee0f8b
https://noreply.gitbook.io/born2beroot/installing-the-virtual-machine/important-advice
https://note.com/costly_to_exist/n/n9e42f74eb013




debian公式サイト
https://www.debian.org/download

isoファイルとは
https://qiita.com/kuri_kuri/items/60d85ec04111dc0e7115


￼ You have selected to skip unattended guest OS install, the guest OS will need to be installed manually.

z自動インストールを防ぐ（パーテーションが勝手に決まり暗号化LVMが作れない

暗号化LVMとは bitロッカー的な
https://qiita.com/nfwork01/items/14a9d7e5090f25f34aaa


EFIとは　BIOSの進化版てきな
https://wa3.i-3-i.info/word12797.html


Guided - use entire disk and set up encrypted LVM


SCSI3 (0,0,0) (sda) - XX GB ATA VBOX HARDDISK

SCSI3 (0,0,0) — ディスクの接続場所を示す番号です。(チャンネル, ID, LUN) という形式で、要するに「どのポートにつながっているディスクか」を表しています。d
(sda) — Linuxがこのディスクに付けた名前です。1台目のディスクは「sda」、2台目があれば「sdb」となります。
XX GB — そのディスクの容量です。
ATA VBOX HARDDISK — ディスクの製品名にあたる部分です。「VBOX」とあるので、これはVirtualBox（仮想マシンソフト）が作った仮想ハードディスクであることがわかります。物理的な本物のディスクではありません。


https://qiita.com/sofareels/items/84d45d398c849a6153c2

https://www.debian.org/doc/manuals/debian-reference/