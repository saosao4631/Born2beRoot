*This project has been created as part of the 42 curriculum by ksaotome.*

# Born2beRoot
 
仮想マシン上に、厳格なセキュリティポリシーを適用した最小構成の Debian サーバーを構築する課題。
 
---
 
## Description
 
VirtualBox 上に GUI を持たない Debian サーバーを構築し、以下を実装した。
 
- **暗号化 LVM** によるパーティション設計（LUKS の内側に LVM を構成）
- **ユーザー / グループ管理**（`ksaotome` を `sudo` および `user42` グループに所属させる）
- **パスワードポリシー**（有効期限・複雑性を PAM と `login.defs` で強制）
- **sudo の厳格な運用**（試行回数制限・入出力ログ・TTY 必須・PATH 制限）
- **SSH**（ポート 4242、root ログイン禁止）
- **UFW**（既定で受信拒否、4242 番のみ開放）
- **monitoring.sh**（起動時および 10 分ごとに、システム情報を `wall` で全端末へ放送）
**ボーナスパートは実装していない。** 本 README が扱うのは必須パートのみである。
 
Git リポジトリに置くのは本ドキュメントと仮想ディスクの署名のみで、仮想マシン本体は含めない。
 
```
.
├── README.md       # 本ドキュメント
└── signature.txt   # 仮想ディスク(.vdi)の SHA1 署名
```
 
---
 
## Instructions
 
### 環境
 
| 項目 | 値 |
|---|---|
| ホスト OS | Linux |
| 仮想化ソフト | VirtualBox |
| ゲスト OS | Debian（最新安定版） |
| RAM | （要記入）MB |
| CPU | （要記入） |
| 仮想ディスク | （要記入）GB / VDI |
| ホスト名 | `ksaotome42` |
| 一般ユーザー | `ksaotome` |
 
### 署名の取得
 
提出する `signature.txt` は、VM を停止した状態で仮想ディスクのハッシュを取って作成する。
 
```bash
cd ~/VirtualBox\ VMs/<VM名>/
sha1sum <VM名>.vdi
```
 
VM を起動し直すと署名は変わるため、提出後は起動しないか、評価用にディスクを複製しておく。
 
### 起動と接続
 
```bash
# 起動すると LUKS のパスフレーズを求められる（正常な挙動）
#   Please unlock disk sda5_crypt:
 
# ホスト側から SSH 接続（ポート転送を設定している場合）
ssh -p 4242 ksaotome@localhost
```
 
root での SSH ログインは拒否される。これは意図した設定である。
 
### 各要件の確認
 
```bash
# OS とバージョン
head -n 2 /etc/os-release
 
# パーティション構成
lsblk
 
# AppArmor が起動時から有効か
sudo aa-status
 
# SSH が 4242 で待ち受けているか
sudo ss -tunlp | grep sshd
 
# ファイアウォールの状態と既定ポリシー
sudo ufw status verbose
 
# パスワードの有効期限設定
sudo chage -l ksaotome
 
# sudo の設定内容
sudo cat /etc/sudoers.d/born2beroot
 
# 所属グループ
groups ksaotome
```
 
### monitoring.sh の停止
 
スクリプト自体を書き換えずに止めるには、実行しているのが cron であることを利用する。
 
```bash
# 一時的に止める
sudo systemctl stop cron
 
# cron の設定から該当行を外して止める
sudo crontab -e
 
# 再開
sudo systemctl start cron
```
 
---
 
## Project description
 
### OS の選択：Debian
 
課題では Debian と Rocky Linux から選択できるが、以下の理由で Debian を選んだ。
 
- 課題文が、システム管理の初学者には Debian を強く推奨している
- SELinux より AppArmor のほうが設定が単純で、限られた時間で「なぜそう設定したか」まで理解できる見込みが立った
- ホスト環境が Linux であり、`apt` 系の操作に馴染みがあった
- 日本語・英語ともに参照できる情報が多く、詰まったときに自力で調べ切れる
### パーティション設計
 
必須要件である「LVM を用いた暗号化パーティションを 2 つ以上」を満たす構成にした。
 
```
sda
├── sda1   /boot            暗号化なし
├── sda2                    拡張パーティション
└── sda5   LUKS 暗号化
      └── sda5_crypt
            └── LVM (VG)
                  ├── root   /
                  ├── swap   [SWAP]
                  └── home   /home
```
 
| 論理ボリューム | サイズ | マウントポイント |
|---|---|---|
| root | （要記入） | `/` |
| swap | （要記入） | `[SWAP]` |
| home | （要記入） | `/home` |
 
設計上の判断は次のとおり。
 
- **`/boot` は暗号化しない。** カーネルと初期 RAM ディスクは復号前に読まれる必要があるため、ここを暗号化するとブートできない。ディスク先頭に独立した平文の領域として確保する。
- **残りを LUKS で暗号化する。** ディスクを物理的に持ち出されても、パスフレーズなしには中身を読めない。起動のたびにパスフレーズを求められるのはこのためである。
- **暗号化層の内側に LVM を置く。** 逆順（LVM の中を個別に暗号化）にすると、ボリュームごとに復号が必要になる。暗号化を一段にまとめることで、パスフレーズの入力は起動時の一度で済む。
- **サイズは用途に対して最小限にした。** 課題文が「適切に動作しつつ不要なディスク消費を避ける」ことを求めているため、GUI を持たないサーバーとして必要な分だけを割り当てた。
### ユーザー管理
 
root のほかに、ログイン名と同じ `ksaotome` を作成し、`sudo` と `user42` の両グループに所属させた。
 
```bash
sudo groupadd user42
sudo usermod -aG sudo,user42 ksaotome
```
 
`sudo` グループは管理コマンドの実行権限、`user42` は課題が要求する確認用のグループである。日常の操作は一般ユーザーで行い、必要なときだけ `sudo` で昇格する。root で直接ログインして作業しない運用にすることで、操作の記録が sudo のログに残る。
 
### パスワードポリシー
 
**有効期限** — `/etc/login.defs`
 
```
PASS_MAX_DAYS   30
PASS_MIN_DAYS   2
PASS_WARN_AGE   7
```
 
**複雑性** — `libpam-pwquality` を導入し、`/etc/pam.d/common-password` の `pam_pwquality.so` の行に指定
 
```
minlen=10 ucredit=-1 lcredit=-1 dcredit=-1 maxrepeat=3 reject_username difok=7 enforce_for_root
```
 
| 設定 | 意味 |
|---|---|
| `minlen=10` | 最低 10 文字 |
| `ucredit=-1` `lcredit=-1` `dcredit=-1` | 大文字・小文字・数字をそれぞれ最低 1 文字 |
| `maxrepeat=3` | 同一文字の 3 連続まで（4 つ続くと拒否） |
| `reject_username` | ユーザー名を含むパスワードを拒否 |
| `difok=7` | 前回のパスワードと 7 文字以上異なること |
| `enforce_for_root` | root にも適用 |
 
`difok` は root のパスワードには適用されない（課題文の記載どおり）。設定後、root を含む全アカウントのパスワードをポリシーに適合するものへ変更した。
 
### sudo の設定
 
`/etc/sudoers` を直接編集せず、`visudo -f /etc/sudoers.d/born2beroot` で専用ファイルを作成した。構文エラーのまま保存すると sudo が使えなくなるため、必ず `visudo` の文法チェックを通す。
 
```
Defaults        passwd_tries=3
Defaults        badpass_message="（任意のメッセージ）"
Defaults        logfile="/var/log/sudo/sudo.log"
Defaults        log_input, log_output
Defaults        iolog_dir="/var/log/sudo"
Defaults        requiretty
Defaults        secure_path="/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin"
```
 
| 設定 | 目的 |
|---|---|
| `passwd_tries=3` | 総当たりを抑えるため、入力試行を 3 回に制限 |
| `badpass_message` | 失敗時に任意のメッセージを表示 |
| `logfile` / `log_input, log_output` | 誰が何を実行し、何が出力されたかを記録 |
| `requiretty` | TTY からの実行を必須にし、スクリプト経由の実行を防ぐ |
| `secure_path` | PATH を固定し、細工されたバイナリが優先されるのを防ぐ |
 
ログの保存先は事前に作成しておく。
 
```bash
sudo mkdir -p /var/log/sudo
```
 
### SSH
 
`/etc/ssh/sshd_config` を編集した。
 
```
Port 4242
PermitRootLogin no
```
 
既定の 22 番から変更するのは、無差別なスキャンや総当たりの大半が 22 番を狙うためで、それ自体が防御になるわけではない。root ログインの禁止のほうが本質的で、攻撃者に「必ず存在する特権アカウント名」を与えないという意味を持つ。
 
```bash
sudo sshd -t                  # 構文チェック
sudo systemctl restart ssh
```
 
### UFW
 
```bash
sudo apt install ufw
sudo ufw allow 4242
sudo ufw enable
```
 
**順序に注意する。** 4242 を開ける前に有効化すると、SSH 接続中の自分が切断される。UFW の既定ポリシーは受信を拒否するため、明示的に許可した 4242 以外はすべて遮断される。`ufw enable` により、次回以降の起動時にも自動で有効になる。
 
### monitoring.sh
 
システム情報を収集し、`wall` で全端末へ放送する bash スクリプト。cron に登録し、起動時および 10 分ごとに実行する。
 
**表示項目** — アーキテクチャとカーネル、物理 CPU 数、仮想 CPU 数、メモリ使用量と使用率、ディスク使用量と使用率、CPU 負荷率、最終起動日時、LVM の有効可否、アクティブな TCP 接続数、ログイン中のユーザー数、IPv4 アドレスと MAC アドレス、sudo で実行されたコマンド数。
 
**cron への登録**
 
```bash
sudo crontab -e
```
 
```
@reboot      bash /path/to/monitoring.sh
*/10 * * * * bash /path/to/monitoring.sh
```
 
root の crontab に登録しているのは、sudo のログなど root 権限がないと読めないリソースを参照するためである。実行は root の 1 プロセスに集約し、表示は `wall` が全端末へ配る。
 
---

### Debian vs Rocky Linux

→オープンソースのLinuxディストリビューションの種類(Linuxの種類)
Debianの方が設定が簡単。

| | 系列 | 命令 |
| --- | --- | --- |
| Debian | Debian(Ubuntu) |
| Rocky Linux | Red Hat（RPM） |  |

### aptitude vs apt

→Debian 系のパッケージ管理ツール。
**apt** は標準的で高速
**aptitude** は依存関係の解決がより高度で、対話的なインターフェースを持つ。

### AppArmor vs SELinux

| | AppArmor | SELinux |
| --- | --- | --- |
| 識別方法 | パス | ラベル |
| 標準採用 | Debian / Ubuntu | RHEL / Rocky |
| モード | enforce / complain | enforcing / permissive / disabled |
| 状態確認 | aa-status | sestatus, getenforce |

### UFW vs firewalld

→ファイアウォール管理ツール。
**UFW**（"Uncomplicated FireWall"）はコマンドが単純、単一サーバーの基本的なポート制御に向く。
**firewalld**（RHEL 系で一般的）はゾーンの概念を持ち、動的な再読み込みや複雑な構成に強い。

### VirtualBox vs UTM

→仮想環境を作成できるもの

|| OS | CPU | 命令 |
| --- | --- | --- | --- |
| VirtualBox | Windows/Linux | Intel/AMD | x86_64 |
| UTM | Mac M1/M2/M3 | Apple Silicon | ARM |

## Resources

### 参考資料

- https://qiita.com/sofareels/items/84d45d398c849a6153c2
- https://www.debian.org/doc/manuals/debian-reference/
- https://qiita.com/nfwork01/items/14a9d7e5090f25f34aaa
- https://wa3.i-3-i.info/word12797.html
- https://qiita.com/RanmaRanma/items/d708d42674928ccc3e42
- https://zenn.dev/kodyi/articles/1440a8e2ee0f8b
- https://noreply.gitbook.io/born2beroot/installing-the-virtual-machine/important-advice
- https://note.com/costly_to_exist/n/n9e42f74eb013

### 参考メモ

SCSI3 (0,0,0) (sda) - XX GB ATA VBOX HARDDISK

SCSI3 (0,0,0) — ディスクの接続場所を示す番号です。(チャンネル, ID, LUN) という形式で、要するに「どのポートにつながっているディスクか」を表しています。d
(sda) — Linuxがこのディスクに付けた名前です。1台目のディスクは「sda」、2台目があれば「sdb」となります。
XX GB — そのディスクの容量です。
ATA VBOX HARDDISK — ディスクの製品名にあたる部分です。「VBOX」とあるので、これはVirtualBox（仮想マシンソフト）が作った仮想ハードディスクであることがわかります。物理的な本物のディスクではありません。

### AIの利用

AIは学習と確認の補助として、次の用途に使用した。

- 課題PDFの要件整理
- READMEの構成と説明の見直し
- 作成時のエラー処理
