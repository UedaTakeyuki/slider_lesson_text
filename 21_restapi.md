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

### 実際のコードの確認
1. センサデータの送信
```
pi@gc1624:~ $ cat -n SCRIPT/slider/gen_sender.py
...
...
19
20	import requests
...
...
81	def send_data(payload):
82	    global ini
83	    if ini.get("send", "protocol") == "http":
84	#        r = requests.post(ini.get("server", "url_base") + 'postdata.php', data=payload, timeout=10, verify=False)
85	        r = requests.post(ini.get("server", "url_base") + 'postdata.php', data=payload, timeout=10, cert=os.path.dirname(os.path.abspath(__file__))+'/slider.pem', verify=False)
86	        slider.msg_log("by http.")
87	    elif ini.get("send", "protocol") == "mqtt":
88	        mqttclient.publish(ini.get("mqtt", "topic"), json.JSONEncoder().encode(payload))
89	        slider.msg_log("by mqtt.")
90
...
...
106
107	def send(serialid, name, value):
108	  global ini, public_serialid
109	  try:
110	    slider.msg_log ("start sending...")
111	    if ini.get("monitor", "mode") == "public":
112	      # public mode, so use public_serialid
113	      serialid = public_serialid
114	    now = datetime.datetime.now() # 時刻の取得
115	    now_string = now.strftime("%Y/%m/%d %H:%M:%S")
116	    payload = {'serial_id': serialid, 'name': name, 'datetime': now_string, 'data': value}
117	    send_data(payload)
118	    slider.msg_log ("end sending...")
119	  except IOError:
120	    slider.io_error_report()
121	  except:
122	    slider.unknown_error_report()
123
...
...
```  
ポイント
  - 20行: requests モジュールの import
  - 85行: requests.post() でデータの送信
  - 116行: payload の作成

2. 撮影画像の送信  
```
pi@gc1624:~ $ cat -n SCRIPT/slider/gen_pic_sender.py
...
...
    19
    20	import requests
...
...
    81	def send_data(payload, files):
    82	    global ini
    83	    if ini.get("send", "protocol") == "http":
    84	#        r = requests.post(ini.get("server", "url_base") + 'postpic.php', data=payload, files=files, timeout=10, verify=False)
    85	        r = requests.post(ini.get("server", "url_base") + 'postpic.php', data=payload, files=files, timeout=10, cert=os.path.dirname(os.path.abspath(__file__))+'/slider.pem', verify=False)
    86	        slider.msg_log("by http.")
    87	    elif ini.get("send", "protocol") == "mqtt":
    88	        mqttclient.publish(ini.get("mqtt", "topic"), json.JSONEncoder().encode(payload))
    89	        slider.msg_log("by mqtt.")
    90
...
...
   107	def send(serialid, filepath, device):
   108	  global ini, public_serialid
   109	  try:
   110	    slider.msg_log ( "start sending...")
   111	    if ini.get("monitor", "mode") == "public":
   112	      # public mode, so use public_serialid
   113	      serialid = public_serialid
   114	    now = datetime.datetime.now() # 時刻の取得
   115	    now_string = now.strftime("%Y/%m/%d %H:%M:%S")
   116	    files = {'upfile': open(filepath, 'rb')}
   117	    payload = {'serial_id': serialid, 'device': device, 'datetime': now_string}
   118	    send_data(payload, files)
   119	#    comand_str = 'curl --insecure -k -F "upfile=@' + filepath + '" -F "serial_id='+serialid+ '" -F "device='+device+'" '+requests.post(ini.get("server", "url_base")) + 'postpic.php'+ '--retry 30'
   120	#    print command_str
   121	#    p = subprocess.check_call(command_str, shell=True)
   122	    slider.msg_log ( "end sending...")
   123	  except IOError:
   124	    slider.io_error_report()
   125	  except:
   126	    slider.unknown_error_report()
```  
ポイント
  - 20行: requests モジュールの import
  - 85行: requests.post() で payload と ***file ハンドル*** を指定してデータの送信
  - 116行: file ハンドルの作成
  - 117行: payload の作成

### なりすまし
公開版の slider は（教育目的でコードをわざとシンプルにしているので）なりすましを防いでいない。
なりすましがどのように起きるのかを体験する

1. 隣の人の Serial_ID を教えてもらう
2. 上記３つ（ブラウザ、shell、pyhthon）のどれでも良いので隣の人の Serial_ID で適当な値を送信する
3. となりの人の monitor のデータが変わる

##<u>ポイント解説</u>
1. なりすましを防ぐため、実際のシステムでは API 認証がおこなわれる
