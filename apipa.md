# LANケーブルで直結

## <u>概要</u>
出先や現場など、PC と Raspberry Pi が共通に接続できるネットワークがない場合でも、LANケーブルを直結することで PC と Raspberry Pi の２者だけで通信をすることができる  
本教室の環境のように、PC によってはあまり通信が安定しない場合もあるようだが、非常時の手段として利用できる

## <u>実習手順</u>

### ターミナルで接続
1. LAN ケーブルで PC と Raspberry Pi を直結する

<img src="pic/ss.2016-12-16 14.43.45.png" width="75%">  
RPi に 169.254.x.y の IP アドレスが割り当てられる

slider のプログラムによって毎分 30秒頃、自動で調停さられた IP アドレスが表示されるので、メモを取る  
（スマフォで写真を取るなどの方法も有効）

2. PC でターミナルを開き、IP アドレスを指定しt ssh で接続

```bash:
  ssh pi@IPアドレス
```

パスワードを入力してログインする

3. Mac, Linux, Windows10, もしくは 古い Windows であっても bonjour protocol がインストールしてあれば、下記のように ホスト名.local で接続することができる

```bash:
  ssh pi@ホスト名.local
```
### (任意)Remote Desktop でログイン
同様に、Remoto Desktop でも利用することができる

## <u>ポイント解説</u>
1. Raspberry Pi の Eathernet Port は LAN ケーブルのストレート/クロスを認識して自動的に切り替える **Auto-MDIX** を備えているのでケーブルはストレートでもクロスでもどちらでもかまわない
2. LAN ケーブル で直結された Raspberry Pi と PC は互いに調整し、169.254.255.255 のサブネット内の衝突しない IP アドレスをお互いに自動的に割り当てる **APIPA（Automatic Private IP Addressing)** と呼ばれる **zeroconf** のプロトコルによって作成される一時的なネットワークを利用して通信をおこなう
3. slider は Linux 上で remote desktop Service をエミュレートするサービスを起動している
4. Linux や Apple 製品では ZeroConf（設定なしでのネットワーク上の名前の解決）を mDNS で行うのだが、Windows では mDNS の実装が独自（LLMNR)
5. mDNS では ホスト名.local で名前を検索（local ドメインという仮想的なドメイン）
6. 古い Windows でも iTune や bonjour print server をインストールすることで mDNS を利用できるようになり、ホスト名.local の名前を検索できるようにる。Windows で Raspberry Pi を利用する際によく利用されている
