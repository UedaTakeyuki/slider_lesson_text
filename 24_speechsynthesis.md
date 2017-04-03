# 24.Speech Synthesis

##<u>目的</u>
前の章で、音声合成の出力を `alsa` に入力することで Raspberry Pi を簡単に喋らせることができた  
この章では音声合成エンジンについて詳細を確認する

##<u>実習手順</u>

### say
Web アプリ `say` を使って自分の Raspberry Pi に好きな言葉をしゃべらせてみる

1. PC で chrome を開き、上で調べた IP アドレスで自分の Raspberry Pi に Web で接続する。下記の画面になる  
<img src="pic/ss.2017-03-08 21.01.34.png" width="75%">

2. `say`をクリックすると下記の画面になる  
<img src="pic/ss.2017-04-03 12.21.48.png" width="75%">

3. text imput `pharse` に好きな言葉を入れて `say` ボタンをクリックすると RPi が喋る  
<img src="pic/ss.2017-04-03 12.22.09.png" width="75%">

4. （任意）隣の人等、他人の RPi を遠隔で喋らせることもできる

### say の実装
say がどのように実装されているかを確認する

1. 自身の gc16 に terminal でログインする
2. `cd /var/www/html/SCRIPT/` に移動してファイル一覧を見る  
```
pi@gc1624:~ $ cd /var/www/html/SCRIPT/say
pi@gc1624:/var/www/html/SCRIPT/say $ ls
config.ini       espeak.zh.sh   jsay.mei.sh   openjtalk.setup.sh
espeak.fr.sh     jsay_fifo.sh   jsay.sh       package.json
espeak.setup.sh  jsay.mei.2.sh  node_modules  say.php
```
3. say.php を開く  
```
pi@gc1624:/var/www/html/SCRIPT/say $ cat -n say.php
     1	<?php
     2	if($_SERVER["REQUEST_METHOD"] == "POST"){
     3	  # 設定の読み込み
     4	  $configfile = "config.ini";
     5	  $ini = parse_ini_file($configfile);
     6
     7	  $command = 'sudo sh -c "'.$ini["command_path"].' '. $_POST["phrase"] . ' | aplay > /dev/null &"';
     8	  exec($command);
     9	}
    10	?>
    11	<!DOCTYPE html>
    12	<html lang="ja">
    13	<head>
    14	  <meta charset="UTF-8">
    15	  <meta name="viewport" content="width=device-width, initial-scale=1">
    16	  <link href="node_modules/bootstrap3/dist/css/bootstrap.min.css" rel="stylesheet">
    17	  <title>say</title>
    18	</head>
    19	<body>
    20	  <script src="node_modules/jquery/dist/jquery.min.js"></script>
    21	  <script src="node_modules/bootstrap3/dist/js/bootstrap.min.js"></script>
    22	  <div class="container-fluid">
    23	    <div class="input-group">
    24	      <form action="" method="POST" >
    25	        phrase
    26	        <input type="text" name="phrase" id="phrase">
    27	        <span class="input-group-tn">
    28	          <input type="submit" value="say" class="btn btn-default">
    29	        </span>
    30	      </form>
    31	    </div>
    32	  </div>
    33	</body>
    34	</html>
```  
ポイントは以下
  - php スクリプトは、`<?php` と `?>` で囲まれた部分がサーバで実行され、残りはそのまま表示される
  - HTML 部分は、入力されたテキストを POST で送信するシンプルな FORM
  - 2行目: サーバへのリクエストが `GET`（最初に開いた状態）か `POST`（FORM が実行された状態）かを確認し、POST でなければ何もしない
  - 5行目: `parse_ini_file()` が php での .ini ファイルパーサ
  - 6行目: shell スクリプトの作成、.ini ファイルの "command_path" に設定されているコマンドに、FORM で入力された `phrase` を渡すスクリプト文字列 `$command` を作成
  - 7行目: `exec($command)` で、$command を外部 shell に渡す（shell スクリプトとして同期実行）

4. `config.ini` を確認  
```
pi@gc1624:/var/www/html/SCRIPT/say $ cat -n config.ini
     1	command_path=/var/www/html/SCRIPT/say/jsay.mei.sh
     2	#command_path=/home/pi/install/aquestalkpi/AquesTalkPi
     3	#command_path=/var/www/html/SCRIPT/say/espeak.fr.sh
     4	#command_path=/var/www/html/SCRIPT/say/espeak.zh.sh
```  
結局、say の実体は、入力された文字列を `jsay.mei.sh` に渡しているだけだった

5. jsay.mei.sh の実体の確認  
```
pi@gc1624:/var/www/html/SCRIPT/say $ cat -n jsay.mei.sh
     1	#!/bin/sh
     2	#http://shokai.org/blog/archives/6893
     3	#http://physicom.digick.jp/?p=5283
     4	#http://moblog.absgexp.net/openjtalk/
     5
     6	TMP=/tmp/jsay.wav
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
    17	rm -f $TMP
```  
ポイント  
  - 11行目: 入力文字列を open_jtalk に渡す
  - 12 - 15行目: open_jtalk のオプション
  - 13行目: 音響ファイルに mei を利用
  - 15行目: 出力を /tmp/jsay.wav
  - 16行目: aplay で /tmp/jsay.wav を再生
open_jtalk は名古屋工業大学で開発されている`日本語`音声合成エンジン  
ライセンスは Modified BSD license.

6.（任意）open_jtalk のインストール  
```
pi@gc1624:/var/www/html/SCRIPT/say $ cat openjtalk.setup.sh
```

### espeak（任意）
open-jtalk は`日本語`以外の言語では espeak が多言語に対応しており、英語だけで6方言、アフリカーンス、ボスニア語、フランス語、デンマーク語、ギリシャ語、クルド語、ラトビア語、タミル語など多くの言語に対応した言語定義ファイルと音響ファイルが用意されている  
ライセンスは GPL.
以下、espeak で中国語を喋らせる

1. config.ini ファイルを編集して command_path を espeak.zh.sh に変更する  
以下は修正例
```
#command_path=/var/www/html/SCRIPT/say/jsay.mei.sh
#command_path=/home/pi/install/aquestalkpi/AquesTalkPi
#command_path=/var/www/html/SCRIPT/say/espeak.fr.sh
command_path=/var/www/html/SCRIPT/say/espeak.zh.sh
```

2. ブラウザで say を開き、適当な中国語を喋らせてみて、日本語読みではなくちゃんと中国語で読み上げていることを確認  
<img src="pic/ss.2017-04-03 14.35.03.png" width="75%">

3. config.ini を元に戻す
```
pi@gc1624:/var/www/html/SCRIPT/say $ git checkout config.ini
```

4. espeak のインストール
```
pi@gc1624:/var/www/html/SCRIPT/say $ cat espeak.setup.sh
```

### AquesTalkPi の利用(任意)
AquesTalk は高速な音声合成、極めて自然な発話ができる等の多くの利点を持つ商用の日本語音声合成エンジンで、TV放送などいろいろな場所でつかわれている  
Raspberry Pi 用の`おためし版`AquesTalkPi が公開されていて非商用目的に利用できる他、ライセンスを購入して商用利用することも可能

1. config.ini ファイルを編集して command_path を AquesTalkPi に変更する  
以下は修正例
```
#command_path=/var/www/html/SCRIPT/say/jsay.mei.sh
command_path=/home/pi/install/aquestalkpi/AquesTalkPi
#command_path=/var/www/html/SCRIPT/say/espeak.fr.sh
#command_path=/var/www/html/SCRIPT/say/espeak.zh.sh
```

2. ブラウザで say を開き、適当な事をしゃべらせてもよいし、単に時刻、温湿度の読み上げを聴いても良い

3. config.ini を元に戻す
```
pi@gc1624:/var/www/html/SCRIPT/say $ git checkout config.ini
```
