# LANケーブルでAPIPA接続

##<u>実習手順</u>

### ターミナルで接続
1. LAN ケーブルで PC と Raspberry Pi を直結する

<img src="pic/ss.2016-12-16 14.43.45.png" width="75%">

2. PC でターミナルを開き、ssh で接続

```bash:
  ssh pi@ホスト名.local
```

パスワードを入力してログインする

3. IP アドレスを直接指定してもログインできる
slider は APIPA で割り当てられた IP アドレスを表示する
<img src="pic/ss.2016-12-16 14.43.45.png" width="75%">

表示された IP アドレスを利用して

```bash:
  ssh pi@IPアドレス
```
でログインできる
### Remote Desktop でログイン

### FTP でログイン

##<u>ポイント解説</u>
1. Raspberry Pi の Eathernet Port は LAN ケーブルのストレート/クロスを認識して自動的に切り替える **Auto-MDIX** を備えているのでケーブルはストレートでもクロスでもどちらでもかまわない
2. LAN ケーブル で直結された Raspberry Pi と PC は互いに調整し、169.254.255.255 のサブネット内の衝突しない IP アドレスをお互いに自動的に割り当てる **APIPA（Automatic Private IP Addressing)** と呼ばれる **zeroconf** のプロトコルによって作成される一時的なネットワークを利用して通信をおこなう
3. slider は Linux 上で remote desktop Service をエミュレートするサービスを起動している
4. Linux や Apple 製品では ZeroConf（設定なしでのネットワーク上の名前の解決）を mDNS で行うのだが、Windows では mDNS の実装が独自（LLMNR）
5. mDNS では ホスト名.local で名前を検索（local ドメインという仮想的なドメイン）
