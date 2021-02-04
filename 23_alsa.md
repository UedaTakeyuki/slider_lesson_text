# 23.音の再生

## <u>目的</u>
ALSA (Advanced Linux Sound Architecture)は カーネルモジュール、ドライバ及びユーザランドで動作するユーティリティから構成される サウンドの ***入出力*** を制御するLinux の標準のモジュール     
本章で ALSA のユーティリティーを使って音を出す方法、音を制御する方法を体験する

##<u>実習手順</u>
自身の gc16 に terminal でログインする

### 準備：音声ファイルの作成
以下の実習で使うためにまず、音声合成ライブラリをつかってサンプルの音声ファイルをする

1. `/var/www/html/SCRIPT/say` に移動し、ファイル jsay.mei.sh のコピーを作る
```
pi@gc1624:~ $ cd /var/www/html/SCRIPT/say
pi@gc1624:/var/www/html/SCRIPT/say $ cp jsay.mei.sh jsay.mei.2.sh
```

2. エディタで jsay.mei.2.sh を編集し、
  - `TMP=/tmp/jsay.wav` を `TMP=/home/pi/jsay.wav` に変更
  - 末尾行をコメントアウト  
以下のようになっていれば正解
```
pi@gc1624:/var/www/html/SCRIPT/say $ cat -n jsay.mei.2.sh
     1	#!/bin/sh
     2	#http://shokai.org/blog/archives/6893
     3	#http://physicom.digick.jp/?p=5283
     4	#http://moblog.absgexp.net/openjtalk/
     5
     6	TMP=/home/pi/jsay.wav
     7
     8	#cd /usr/share/hts-voice/nitech-jp-atr503-m001
     9	#cd /usr/share/hts-voice/mei_happy
    10	echo start
    11	echo "$1" | open_jtalk \
    12	-x /var/lib/mecab/dic/open-jtalk/naist-jdic \
    13	-m /usr/share/hts-voice/MMDAgent_Example-1.6/Voice/mei/mei_normal.htsvoice \
    14	-a 0.5 \
    15	-ow $TMP && \
    16	aplay --quiet $TMP
    17	# rm -f $TMP
```

3. jsay.mei.2.sh に適当な文字列を与えて wav ファイルを作る
```
pi@gc1624:/var/www/html/SCRIPT/say $ ./jsay.mei.2.sh 春は長雨
start
```  
`/home/pi` に `jsay.wav` というフォルダが出来ているので、以降このファイルを使う

### wav ファイルの再生
aplay コマンドで再生する

1. スピーカーの接続を再確認
2. /home/pi に移動して、aplay で再生
```
pi@gc1624:/var/www/html/SCRIPT/say $ cd
pi@gc1624:~ $ aplay jsay.wav
Playing WAVE 'jsay.wav' : Signed 16 bit Little Endian, Rate 48000 Hz, Mono
```

### ボリュームの設定
ボリューム等の設定は alsamixer で行う

1. alsamixer
```
pi@gc1624:~ $ alsamixer
```

2. 下記の画面になるので ↑ ↓ キーでボリュームを変更し、ESC で保存する  
<img src="pic/ss.2017-03-31 21.00.19.png" width="75%">

3. aplay で jsay.wav を再生し、ボリュームが変わっている事を確認
```
pi@gc1624:~ $ aplay jsay.wav
```
