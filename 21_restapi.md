# HTTP REST API

##<u>目的</u>

##<u>実習手順</u>

### ブラウザからの送信
自身の gc16 に terminal でログインする

1. PC で chrome を開き、IP アドレスを指定して自分の Raspberry Pi が所属するネットワークのサーバーに接続する。サーバーの IP アドレスは[こちら](classenvironment.md)で確認できる。以下の画面になる  

<img src="pic/ss.2017-03-08 21.06.35.png" width="75%">

2. monitor 配下の自分の Raspberry Pi のホスト名をクリック   
下記のログイン画面が開く  

<img src="pic/ss.2017-03-08 21.06.47.png" width="75%">

初期 ID と PW　は以下

- ID: g4
- PW: g4

3. PC で chrome を開き、IP アドレスを指定して自分の Raspberry Pi に接続する。IPアドレスは[こちら](classenvironment.md)で確認できる。以下の画面になる  

<img src="pic/ss.2017-03-08 21.01.34.png" width="75%">

4. Set DateTime を開く。真っ白な画面が開くのと同時に Raspberry Pi の LCD の時刻が PC の時刻と同期する

5. 自分の RPi の Serial ID を再度、確認しておく
```
pi@gc1624:~ $ python -m piserialnumber
00000000fbaa1f70
```

6. ブラウザをもう一枚ひらき、`自分の所属するmonitorのアドレス/SCRIPT/monitor/postdata.php`にアクセスする  
以下のような`データアップロード`画面が開く
<img src="pic/ss.2017-03-28 19.18.46.png" width="75%">

7. `serial_id` に自分の Serial_id を、`name` に `temp`と、datetime は空白のまま、`data` に適当な数値を入れて登録ボタンを押す
<img src="pic/ss.2017-03-28 19.19.08.png" width="75%">

8. `monitor` をみると、入力した値が温度として反映されている
<img src="pic/ss.2017-03-28 19.18.17.png" width="75%">

### shell からの送信
curl コマンドをつかって shell から HTTP POST を送信することができる

1. 自身の gc16 に terminal でログインする
2. 先程、ブラウザで開いた`自分の所属するmonitorのアドレス/SCRIPT/monitor/postdata.php`に対して下記の `curl` コマンドでデータをポストする  
```
pi@gc1624:~ $ curl -F "serial_id=00000000fbaa1f70" -F "name=temp" -F "data=80" http://gc1601.local/SCRIPT/monitor/postdata.php
```  
ポイントは以下
  - curl コマンドのパラメタの URL では、プロトコル文字列（先頭の `http:`）を省略しない
  - POST のパラメタは `-F "パラメタ名=値文字列"` で指定する
    - `serial_id=` には自分の RPi の Serial ID を指定
    - `name=` には `temp` を指定
    - `data=` には適当な値を指定

3. `monitor` を見ると、入力した値が温度として反映されている
<img src="pic/ss.2017-03-28 19.46.39.png" width="75%">

### python からの送信
python の場合、HTTP POST を送信する手段はいくつもある  

- 上記の `curl` を `subprocess()` で実行する方法
- pycurl
- httpie
- requests

ここでは `request` をつかった方法を説明する  
使い方を一言で説明すると、以下のステップになる

- requests モジュールを import
- payload を作る
- HTTP のメソッド（post や get）で送信

POST で作っていたプログラムをやはり GET に変えたい、という時にメソッドを変えるだけで済むので便利

1. 自身の gc16 に terminal でログインする
2. python をインタラクティブモードで実行  
```
pi@gc1624:~ $ python
Python 2.7.9 (default, Sep 17 2016, 20:26:04)
[GCC 4.9.2] on linux2
Type "help", "copyright", "credits" or "license" for more information.
```
3. requests モジュールを import  
```
>>> import requests
```
4. payload を作る  
```
>>> payload = {'serial_id': "00000000fbaa1f70",
... 'name': "temp",
... 'data': "120.5"}
```  
  - `serial_id:` には自分の RPi の Serial ID を指定
  - `name:` には `temp` を指定
  - `data:` には適当な値を指定

5. url は先程 curl で指定したものと同じ  
```
>>> url="http://gc1601.local/SCRIPT/monitor/postdata.php"
```

6. post メソッドで送信
```
>>> requests.post(url, data=payload, timeout=10, verify=False)
<Response [200]>
```  
ポイント  
  - `timeout=10` で 10秒のタイムアウトを設定できる
  - `veryfy` は、url が `https:` であった時にサーバの証明書を veryfi するかどうか
  - サーバからの OK レスポンス `<Response [200]>` が帰ってくる

7. `monitor` を見ると、入力した値が温度として反映されている
<img src="pic/ss.2017-03-28 20.14.21.png" width="75%">


### なりすまし
公開版の slider は（教育目的でコードをわざとシンプルにしているので）なりすましを防いでいない。
なりすましがどのように起きるのかを体験する

1. 隣の人の Serial_ID を教えてもらう
2. 上記３つ（ブラウザ、shell、pyhthon）のどれでも良いので隣の人の Serial_ID で適当な値を送信する
3. となりの人の monitor のデータが変わる

##<u>ポイント解説</u>
1. なりすましを防ぐため、実際のシステムでは API 認証がおこなわれる
