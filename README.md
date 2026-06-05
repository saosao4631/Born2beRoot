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