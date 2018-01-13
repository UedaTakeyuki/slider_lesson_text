# ネットワークを介したlogin

##<u>概要</u>
PC と Raspberry Pi を同じネットワーク内に参加させてネットワーク経由でアクセスする方法。Raspberry Pi にディスプレイとキーボード、マウスを接続して操作する代わりに PC のキーボード、マウス、ディスプレイを利用して Raspberry Pi を操作する

##<u>実習手順</u>
最初に自分の Raspberry Pi の IP アドレスを[こちら](classenvironment.md)で確認する。

### コマンド文字列で操作 （CUI）
1. コマンドライン操作用の ***gc16*** の SD カードで Raspberry Pi を起動する
2. PC で ***teraterm*** 等のターミナルソフトを起動し、上で調べた IP アドレスで自分の Raspberry Pi に login する。login id と password は以下

  - ID: pi
  - PW: gc16pw


### デスクトップで操作(GUI)
1. GUI操作用の ***gc15*** の SD カードで Raspberry Pi を起動する
2. PC で RemoteDesktop を開きく、上で調べた IP アドレスで自分の Raspberry Pi に接続
3. 下記のように login 画面が開く
<img src="pic/ss.2016-12-16 14.44.02.png" width="75%">  
login id と password は以下

  - ID: pi
  - PW: gc15pw

login すると、以下のようなデスクトップが開く
<img src="pic/ss.2016-12-16 14.44.26.png" width="75%">

###（任意）RemoteDesktop でのコマンドライン接続
デスクトップを持たない ***gc16*** にも RemoteDesktop で接続することができ、terminal アプリケーションの代用にすることができる

1. GUI操作用の ***gc16*** の SD カードで Raspberry Pi を起動する
2. PC で RemoteDesktop を開きく、上で調べた IP アドレスで自分の Raspberry Pi に接続
3. 下記のように login 画面が開く
<img src="pic/ss.2016-12-16 14.44.02.png" width="75%">  
login id と password は以下

  - ID: pi
  - PW: gc16pw

login すると、以下のようなコンソール画面になる
<img src="pic/ss.2016-12-16 14.44.14.png" width="75%">


### shell in a box の利用(CUI)
gc15, gc16 では shell in a box がインストールしてあり、Web ブラウザ経由で tty にログインすることができる。多くのクラウドサービスの Web ログインと同じようにつかうことができる

1. PC で chrome を開き、上で調べた IP アドレスで自分の Raspberry Pi に Web で接続する。下記の画面になる

<img src="pic/ss.2017-03-08 21.01.34.png" width="75%">

2. shell in a box をクリックすると下記の画面になるのでログインする

<img src="pic/ss.2017-03-09 21.34.35.png" width="75%">

3. （任意）raspberry pi に shell in a box をインストールするスクリプトは以下  
/home/pi/install/gc_setups/shellinabox.setup.sh  
興味があれば、cat コマンドなどで参照されたい

外部ネットワークに接続できない本教室の環境では参照することはできないのだが、参考までに gc_setups のセットアップスクリプト群は以下で公開している

https://github.com/UedaTakeyuki/gc_setups

### （任意）別の Raspberry Pi を踏み台にしてログイン
Linux 同士であれば、IPアドレスではなく以下のようにホスト名で login できる
```
 ホスト名.local
```


##<u>ポイント解説</u>
1. この方法は、ブートシーケンスが正常に終了してネットワークに接続できることが前提になる。実際、開発途中ではブートシーケンスが正常に終了しなくなる事もしばしばあり、そのようなときは問題が解決するまではこの方法では login できない。実習ではおこなわないが、問題の解決のためにブートシーケンスのログを見るには以下の方法がある
  - Raspberry Pi にディスプレイ、キーボード、マウスを直結して起動する。ブートシーケンスもディスプレイに表示される
  - Raspberry Pi と PC を Serial ケーブルでつなぎ、デバッグ用の tty と接続して Raspberry Pi を起動する。ブートシーケンスも表示される
  - 起動に失敗した Raspberry Pi の SD カードの /var/log/syslog ファイル（失敗するまでのブートシーケンスが記録されている）を読む。PC で読む場合は、Linux のファイルシステムである ext4 のドライバを PC にインストールする必要がある。別の正常に起動する Raspberry Pi （例えば、正常起動できなくなった Raspberry Pi の SD カードのバックアップなど）に、正常起動しなかった SD カードを SD カードリーダーなどを介して USB でマウントして読むこともよく行う
2. 操作の利便性のために Raspberry Pi 上で RDP のサービスを起動している。Windows から Raspberry Pi のデスクトップ利用する場合は RDP が便利。また、Android や iOS の各種 RDP アプリケーションからも利用できる。Raspberry Pi のデスクトップを共有する手段としては他に以下の方法がある。
  - XWindow：　Mac など、XWindow を標準でもっているクライアントからの接続の場合、一番自然
  - VNC：　これもよく利用される
