# 22.制御アプリ read.py

## <u>目的</u>
IoT 端末の要件はデータを取得してデータを処理する事で、この処理の実装例として read.py を紹介する  

## <u>実習手順</u>
自身の gc16 に terminal でログインする

### 設定ファイル
設定ファイルには .ini ファイル、.yaml ファイル、.toml ファイルなどの他に、json, xml なども可能  
以下、.ini ファイルの例と .toml ファイルの例を説明する

#### .ini ファイル
配列やハッシュのようなデータ構造が不要で単に key=value の組でよければ .ini ファイルでもよい  
bash からも、python からも利用可能

##### bash からの利用
.ini ファイルは bash に source コマンドでとりこむ事ができる

1. /boot に移動  
```
pi@gc1624:~ $ cd /boot
```

2. $network になにも設定されていない事を確認

```
pi@gc1624:/boot $ echo $network

```
3. gc.ini を表示

```
pi@gc1624:/boot $ cat gc.ini
#network=pi
network=wpa
#network=adhoc
#adhoc_address=172.24.1.4
```

4. gc.ini を source で取り込む  
```
pi@gc1624:/boot $ source gc.ini
```  
network はない

5. gc.ini が取り込まれ、$network が設定されている
```
pi@gc1624:/boot $ echo $network
wpa
```

##### bash からの利用、別の方法
source は . とも書ける  

1. もう一つ、bash を立ち上げる。ここでは $network は設定されていない

```
pi@gc1624:/boot $ bash
pi@gc1624:/boot $ echo $network

```

2. . で gc.ini を取り込む  
```
pi@gc1624:/boot $ . gc.ini
pi@gc1624:/boot $ echo $network
wpa
```

##### python からの利用
ConfigParser モジュールを利用して python から ini ファイルを参照できる  

1. `SCRIPT/slider` に移動
```
pi@gc1624:~ $ cd SCRIPT/slider/
```
2. `gen_saver.ini` の内容を確認

```
pi@gc1624:~/SCRIPT/slider $ cat gen_saver.ini
[save]
#data_path=/media/usb0
data_path=/boot/DATA

[log]
log_file=/home/pi/LOG/gen_saver.log
```
3. インタラクティブモードで python を起動
```
pi@gc1624:~/SCRIPT/slider $ python
Python 2.7.9 (default, Sep 17 2016, 20:26:04)
[GCC 4.9.2] on linux2
Type "help", "copyright", "credits" or "license" for more information.
```
4. ConfitParser を import
```
>>> import ConfigParser
```
5. ConfigParser のインスタンスをつくり、.ini ファイルを読む
```
>>> ini = ConfigParser.SafeConfigParser()
>>> ini.read("gen_saver.ini")
['gen_saver.ini']
```
6. セクションとアイテムを指定して get
```
>>> ini.get("save","data_path")
'/boot/DATA'
```
7. exit で終了
```
>>> exit()
pi@gc1624:~/SCRIPT/slider $
```

#### .toml ファイル
.toml は構造を持つ設定を手軽に表現できる

#### config.toml
1. toml の例 として、slider の設定ファイルを確認する  
```
pi@gc1624:~/SCRIPT/slider $ cat -n config.toml
     1	[sensors]
     2	# data [[名前、単位、送信、保存],...]
     3
     4	#  [sensors.mh_z19]
     5	#  data = [["CO2","ppm","gen_sender","gen_saver"]]
     6
     7	  [sensors.dht22]
     8	  data = [["temp","℃","gen_sender","gen_saver"],
     9	  				["humidity","%","gen_sender","gen_saver"],
    10	  				["humiditydeficit","g/㎥","gen_sender","gen_saver"]]
    11
    12	[imaging]
    13	# data [[イメージ種別、デバイス数、送信、保存],...]
    14	# イメージ種別 = [ "pic" | "movie" ]
    15	# デバイス数 = [ "one" | "all" | "dummy"]
    16
    17		[imaging.uvc]
    18	#	data =["pic","one","gen_pic_sender", "gen_pic_saver"]
    19		data =["pic","all","gen_pic_sender", "gen_pic_saver"]
    20	#	data =["pic","dummy","gen_pic_sender", "gen_pic_saver"]
```  
toml の構文は以下  

```
[table名1]
  key1 = value1
  key2 = value2
  [table名1.table名2]
  key3 = value3
  key3 = [item1, item2, item3 ...] # array
  key4 = [[item1, item2, item3 ...],
          [item1, item2, item3 ...],
          [item1, item2, item3 ...]] # array of array

```
#### python からの利用
いろいろなパッケージが提供されているのだがここでは pytoml を用いる

1.  インタラクティブモードで python を起動
```
pi@gc1624:~/SCRIPT/slider $ python
Python 2.7.9 (default, Sep 17 2016, 20:26:04)
[GCC 4.9.2] on linux2
Type "help", "copyright", "credits" or "license" for more information.
>>>
```

2. pytoml を import
```
import pytoml as toml
```

3. .toml ファイルを開き、ファイルハンドルを引数として pytoml.load()
```
>>> with open("config.toml") as fin:
...   config = pytoml.load(fin)
...
```

4. python の dict データとして config を参照

```
>>> config
{u'sensors': {u'dht22': {u'data': [[u'temp', u'\u2103', u'gen_sender', u'gen_saver'], [u'humidity', u'%', u'gen_sender', u'gen_saver'], [u'humiditydeficit', u'g/\u33a5', u'gen_sender', u'gen_saver']]}}, u'imaging': {u'uvc': {u'data': [u'pic', u'all', u'gen_pic_sender', u'gen_pic_saver']}}}

>>> config["sensors"]
{u'dht22': {u'data': [[u'temp', u'\u2103', u'gen_sender', u'gen_saver'], [u'humidity', u'%', u'gen_sender', u'gen_saver'], [u'humiditydeficit', u'g/\u33a5', u'gen_sender', u'gen_saver']]}}

>>> config["sensors"]["dht22"]
{u'data': [[u'temp', u'\u2103', u'gen_sender', u'gen_saver'], [u'humidity', u'%', u'gen_sender', u'gen_saver'], [u'humiditydeficit', u'g/\u33a5', u'gen_sender', u'gen_saver']]}

>>> config["sensors"]["dht22"]["data"]
[[u'temp', u'\u2103', u'gen_sender', u'gen_saver'], [u'humidity', u'%', u'gen_sender', u'gen_saver'], [u'humiditydeficit', u'g/\u33a5', u'gen_sender', u'gen_saver']]

>>> config["sensors"]["dht22"]["data"][0]
[u'temp', u'\u2103', u'gen_sender', u'gen_saver']
>>> config["sensors"]["dht22"]["data"][0][1]
u'\u2103'
>>> config["sensors"]["dht22"]["data"][0][2]
u'gen_sender'
>>> config["sensors"]["dht22"]["data"][0][3]
u'gen_saver'
```  
各々のセンサについて、一回の呼び出しで取得できるデータの組の各々について、`名前`、`単位`、`送信処理`、`保存処理`が array で定義される

### read.py の構造
IoT クライアントの主な機能は`センサデータを取得して処理する`事だが、slider では read.py がそれにあたる。config.toml ファイルを読み、個々のセンサの個々の取得データについて定義されている送信処理、保存処理を実行する

1. read.py を確認する
```
pi@gc1624:~/SCRIPT/slider $ cat -n read.py
...
...
    14	import pytoml as toml
    15	with open(os.path.dirname(os.path.abspath(__file__))+'/config.toml', 'rb') as fin:
    16	  config = toml.load(fin)
...
...
    21	def read():
    22	  ############################################################
    23	  # sensors
    24	  #
    25	  slider.msg_log("read.py started.")
    26
    27	  for sensor_name, sensor_settings in config["sensors"].items():
    28	    slider.msg_log(sensor_name)
    29
    30	    # road reader.
    31	    reader = importlib.import_module(sensor_name)
    32	    try:
    33	      value = reader.read()
    34	    except IOError:
    35	      slider.io_error_report()
    36	      continue
    37	    except:
    38	      slider.unknown_error_report()
    39	      continue
    40
    41	    if (value is not None):
    42	      datas = sensor_settings['data']
    43	      for data in datas:
    44	        slider.msg_log(data[2])
    45	        if data[2]: # Send
    46	          # read specified sender.
    47	          try:
    48	            sender = importlib.import_module(data[2])
    49	            sender.send(serialid, data[0], value[data[0]]) # serialid, name, value
    50	          except IOError:
    51	            slider.io_error_report()
    52	          except:
    53	            slider.unknown_error_report()
    54	        slider.msg_log(data[3])
    55	        if data[3]: # Save
    56	          # read specified saver.
    57	          try:
    58	            saver = importlib.import_module(data[3])
    59	            saver.save(serialid, data[0], value[data[0]]) # serialid, name, value
    60	          except IOError:
    61	            slider.io_error_report()
    62	          except:
    63	            slider.unknown_error_report()
    64
    65	  ############################################################
    66	  # image
    67	  #
    68	  for imagedevice_name, data in config["imaging"].items():
    69	    imagedevice_settings = data["data"]
    70	    slider.msg_log( imagedevice_name)
    71	    if imagedevice_name == 'uvc': # USB カメラなら
    72	      devices = []
    73	#      d = datetime.datetime.today()
    74	#      now = d.strftime("%Y%m%d%H%M%S")
    75	      if imagedevice_settings[1] == "one":
    76	        devices = ["video0"]
    77	      elif imagedevice_settings[1] == "all":
    78	        # UVC カメラデバイスの数だけ
    79	        devices = videodevices.videodevices_basename()
    80	      elif imagedevice_settings[1] == "dummy":
    81	        # dummy 画像の取得
    82	        devices = ["video0"]
    83
    84	      for videodevice in devices:
    85	        videodevice_now = datetime.datetime.today().strftime("%Y%m%d%H%M%S")
    86	        filepath = '/tmp/'+videodevice_now+'.jpg'
    87	        slider.msg_log( filepath)
    88	        if imagedevice_settings[1] == "dummy":
    89	#          command_str = 'fswebcam '+filepath+' -d TEST -r 320x240'
    90	          command_str = os.path.dirname(os.path.abspath(__file__))+'/photographier.sh '+filepath+' dummy 320x240'
    91	        else:
    92	#          command_str = 'fswebcam '+filepath+' -d /dev/'+videodevice+' -D 1 -S 20 -r 320x240'
    93	          command_str = os.path.dirname(os.path.abspath(__file__))+'/photographier.sh '+filepath+' '+videodevice+' 320x240'
    94	        slider.msg_log( command_str)
    95	        try:
    96	          p = subprocess.check_call(command_str, shell=True)
    97	          slider.msg_log ('p = ' + str(p))
    98	        except IOError:
    99	          slider.io_error_report()
   100	          continue
   101	        except:
   102	          slider.unknown_error_report()
   103	          continue
   104
   105	        if imagedevice_settings[2]: # Send
   106	          # read specified sender.
   107	          slider.msg_log(imagedevice_settings[2])
   108	          try:
   109	            sender = importlib.import_module(imagedevice_settings[2])
   110	            sender.send(serialid, filepath, videodevice) # serialid, name, value
   111	          except IOError:
   112	            slider.io_error_report()
   113	          except:
   114	            slider.unknown_error_report()
   115
   116	        if imagedevice_settings[3]: # Save
   117	          # read specified saver.
   118	          slider.msg_log(imagedevice_settings[3])
   119	          try:
   120	            saver = importlib.import_module(imagedevice_settings[3])
   121	            saver.save(serialid, videodevice, filepath) # serialid, device, picfilepath
   122	          except IOError:
   123	            slider.io_error_report()
   124	          except:
   125	            slider.unknown_error_report()
   126
   127	  slider.msg_log("read.py ended.")
   128
   129	if __name__ == '__main__':
   130	  read()
```  
ポイント  
  - 14 - 16行: config.toml の読み込み
  - 21行: 以下、read() の定義
  - 27行: sensors テーブルから、sensor_name と sensor_settings の組を読む  sensor_settings は array
  - 31行: センサー名と同名の python モジュールをダイナミックロード
  - 33行: ダイナミックロードしたセンサモジュールの read() メソッドを実行  
  read の戻り値はデータの名前と値の組
  - 48行: このデータについて定義されている送信処理と同名の python モジュールをダイナミックロード
  - 49行: ダイナミックロードした sender の send() メソッドで、データを送信
  - 58行: このデータについて定義されている保存処理と同名の python モジュールをダイナミックロード
  - 59行: ダイナミックロードした sender の save() メソッドで、データを保存

### センサの追加
アダプタモジュールの read() 関数のおかげで、センサの追加時に read.py に一切の変更は不要で、単に config.toml への設定の追加だけでよい  
例として、先ほど作成した cpu.py アダプタをを利用して CPU温度の取得を追加してみる

1. `slider` 配下に移動
```
pi@gc1624:~ $ cd SCRIPT/slider/
```

2. `config.toml` のバックアップを取得
```
pi@gc1624:~/SCRIPT/slider $ cp config.toml config.toml.bak
```

3. 現在の`config.toml`を確認
```
pi@gc1624:~/SCRIPT/slider $ cat -n config.toml
     1	[sensors]
     2	# data [[名前、単位、送信、保存],...]
     3
     4	#  [sensors.mh_z19]
     5	#  data = [["CO2","ppm","gen_sender","gen_saver"]]
     6
     7	  [sensors.dht22]
     8	  data = [["temp","℃","gen_sender","gen_saver"],
     9	  				["humidity","%","gen_sender","gen_saver"],
    10	  				["humiditydeficit","g/㎥","gen_sender","gen_saver"]]
    11
    12	[imaging]
    13	# data [[イメージ種別、デバイス数、送信、保存],...]
    14	# イメージ種別 = [ "pic" | "movie" ]
    15	# デバイス数 = [ "one" | "all" | "dummy"]
    16
    17		[imaging.uvc]
    18	#	data =["pic","one","gen_pic_sender", "gen_pic_saver"]
    19		data =["pic","all","gen_pic_sender", "gen_pic_saver"]
    20	#	data =["pic","dummy","gen_pic_sender", "gen_pic_saver"]
```

4. 11行目以降に`cpu`の記載を追加  
  - センサモジュール名は`cpu`
  - データは以下
    - 名前: `cpu_temp`
    - 単位: `℃`
    - 送信: `gen_sender`
    - 保存: `gen_saver`
完成例は以下  
```
pi@gc1624:~/SCRIPT/slider $ cat -n config.toml
     1	[sensors]
     2	# data [[名前、単位、送信、保存],...]
     3
     4	#  [sensors.mh_z19]
     5	#  data = [["CO2","ppm","gen_sender","gen_saver"]]
     6
     7	  [sensors.dht22]
     8	  data = [["temp","℃","gen_sender","gen_saver"],
     9	  				["humidity","%","gen_sender","gen_saver"],
    10	  				["humiditydeficit","g/㎥","gen_sender","gen_saver"]]
    11	### ↓ 追加 ###
    12	  [sensors.cpu]
    13	  data = [["cpu_temp","℃","gen_sender","gen_saver"]]
    14	### ↑ 追加 ###
    15
    16	[imaging]
    17	# data [[イメージ種別、デバイス数、送信、保存],...]
    18	# イメージ種別 = [ "pic" | "movie" ]
    19	# デバイス数 = [ "one" | "all" | "dummy"]
    20
    21		[imaging.uvc]
    22	#	data =["pic","one","gen_pic_sender", "gen_pic_saver"]
    23		data =["pic","all","gen_pic_sender", "gen_pic_saver"]
    24	#	data =["pic","dummy","gen_pic_sender", "gen_pic_saver"]
```

5. `/boot/DATA` に `cpu_temp.csv` が出来ていることを確認  
  
```
pi@gc1624:~/SCRIPT/slider $ ls /boot/DATA
cpu_temp.csv  humidity.csv  humiditydeficit.csv  lena_std.jpg  temp.csv  video0

pi@gc1624:~/SCRIPT/slider $ cat /boot/DATA/cpu_temp.csv
2017/03/31 19:18:24,46.2,00000000fbaa1f70
```

### read.py の呼び出し
センサデータの取得は`一定間隔での繰り返し`や`特定のイベントをトリガーとして` read.py が呼び出される事で取得して処理される。後者は GPIO の章ですでに、水センサをトリガとして read を呼び出す処理を実装した。ここでは前者を確認する  
linux の場合、定時処理は `cron` を使うとシンプルになる  

1. cron の設定を確認
```
pi@gc1624:~/SCRIPT/slider $ crontab -l
# DO NOT EDIT THIS FILE - edit the master and reinstall.
# (/tmp/crontab.kef0xk/crontab installed on Wed Jul  6 21:10:33 2016)
# (Cron version -- $Id: crontab.c,v 2.13 1994/01/17 03:20:37 vixie Exp $)
# Edit this file to introduce tasks to be run by cron.
#
# Each task to run has to be defined through a single line
# indicating with different fields when the task will be run
# and what command to run for the task
#
# To define the time you can provide concrete values for
# minute (m), hour (h), day of month (dom), month (mon),
# and day of week (dow) or use '*' in these fields (for 'any').#
# Notice that tasks will be started based on the cron's system
# daemon's notion of time and timezones.
#
# Output of the crontab jobs (including errors) is sent through
# email to the user the crontab file belongs to (unless redirected).
#
# For example, you can run a backup of all your user accounts
# at 5 a.m every week with:
# 0 5 * * 1 tar -zcf /var/backups/home.tgz /home/
#
# For more information see the manual pages of crontab(5) and cron(8)
#
# m h  dom mon dow   command
# 時 分 日 月 曜 実行コマンド
#*/5 * * * * python /home/pi/SCRIPT/fota.py
*/5 * * * * python /home/pi/SCRIPT/slider/statelog.py
*/1 * * * * sudo python /home/pi/SCRIPT/slider/read.py
```  
read.py は1分毎によびだされる

2. 設定を変更して 2分間隔でよびだされるようにする
```
pi@gc1624:~/SCRIPT/slider $ crontab -e
```

末尾の read.py の設定の先頭の `*/1` を `*/2` に変更して保存

```
pi@gc1624:~/SCRIPT/slider $ crontab -e
crontab: installing new crontab
```
```
pi@gc1624:~/SCRIPT/slider $ crontab -e
No modification made
```  
ブラウザでデータの更新が2分おきになった事を確認

3. 設定を変更して read.py の呼び出しを止める
```
pi@gc1624:~/SCRIPT/slider $ crontab -e
```

末尾の read.py の設定の先頭の `*/` を `#*/2` に変更して保存

```
pi@gc1624:~/SCRIPT/slider $ crontab -e
crontab: installing new crontab
```
```
pi@gc1624:~/SCRIPT/slider $ crontab -e
No modification made
```  
ブラウザでデータの更新が止まった事を確認

### PC からの cron の設定
実際の現場での slider の使用においてデータ取得間隔の変更のために Raspberry Pi にログインして cron の設定を変更する必要があるのではかなり不便である。  
そこで、slider では PC から変更できる /boot 領域に cron の設定を置いている

1. gc16 を再起動
```
pi@gc1624:~ $ sudo reboot
```

2. gc16 に再度 login
3. crontab で先ほどコメントアウトした末尾の read.py の起動が、1分間隔に戻っている事を確認
```
pi@gc1624:~ $ crontab -l
# DO NOT EDIT THIS FILE - edit the master and reinstall.
# (/tmp/crontab.kef0xk/crontab installed on Wed Jul  6 21:10:33 2016)
# (Cron version -- $Id: crontab.c,v 2.13 1994/01/17 03:20:37 vixie Exp $)
# Edit this file to introduce tasks to be run by cron.
#
# Each task to run has to be defined through a single line
# indicating with different fields when the task will be run
# and what command to run for the task
#
# To define the time you can provide concrete values for
# minute (m), hour (h), day of month (dom), month (mon),
# and day of week (dow) or use '*' in these fields (for 'any').#
# Notice that tasks will be started based on the cron's system
# daemon's notion of time and timezones.
#
# Output of the crontab jobs (including errors) is sent through
# email to the user the crontab file belongs to (unless redirected).
#
# For example, you can run a backup of all your user accounts
# at 5 a.m every week with:
# 0 5 * * 1 tar -zcf /var/backups/home.tgz /home/
#
# For more information see the manual pages of crontab(5) and cron(8)
#
# m h  dom mon dow   command
# 時 分 日 月 曜 実行コマンド
#*/5 * * * * python /home/pi/SCRIPT/fota.py
*/5 * * * * python /home/pi/SCRIPT/slider/statelog.py
*/1 * * * * sudo python /home/pi/SCRIPT/slider/read.py
pi@gc1624:~ $
```

4. shutdown
```
pi@gc1624:~ $ sudo shutdown -h 0
```

5. gc16 の SDカードを Raspberry Pi から抜き、USB SD r/w で PC に挿し、テキストエディタで `/boot/crontab.slider.txt` を確認  
read.py の繰り返し間隔が 1分になっているので 2分間隔に変更して保存

6. gc16 の SD カードを Raspberry Pi に装着して起動し、データの更新が 2分間隔になっていることをブラウザで確認

7. 自身の gc16 に terminal でログイン

8. `/boot/crontab.slider.txt` を修正して read.py を 1分間隔に戻す

9. `sudo reboot` で再起動  
再起動後、データの更新が 1分間隔に戻っていることをブラウザで確認
