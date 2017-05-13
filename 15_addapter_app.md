# 15.センサを読むソフトウェア

##<u>概要</u>
センサのインターフェースは一般に`i2c`や`spi`等のプロトコルの上でのコマンドとレスポンスの組となる。
特殊なプロトコルを使うもの以外、一般に`センサーのドライバ`という物は存在しない。ドライバとしてカーネルで動作するのは`i2c`や`spi`等のプロトコル部分であって、コマンド-レスポンスはユーザーランドのアプリケーションとして実装されるのが一般的。また、特殊なプロトコルを利用するセンサーであってもそのためのドライバが用意されることは一般にまれで、たんに`GPIO`の接点インターフェースを利用した、やはりユーザランドのアプリケーションとしてライブラリが用意されることが一般的

このように、センサのデータを読むソフトウェアは一般にユーザランドで動作する CUI アプリケーションのライブラリとして提供される。そこで、そのデータを取得し、自分が利用したい形式に変換するためのグルーコードを用意する必要がある。逆に言えば、たんにライブラリを呼び出し、データの形式を変更するグルーコードを用意するだけでセンサのデータを簡単に取得することができる

##<u>実習手順</u>
自身の gc16 に terminal でログインする

### 温湿度センサの利用

1. 温湿度センサー dht22 のポーリングアプリケーションは色々とあるが、ここでは technion さんの [lol_dht22](https://github.com/technion) を使わせて頂く  
lol_dht22 を git clone して make するスクリプトは以下  
```
cat -n /home/pi/SCRIPT/slider/vendor/dht22.setup.sh
```  
このスクリプトを実行した結果、ポーリングアプリは下記に設定される  
`/home/pi/SCRIPT/slider/vendor/lol_dht22`  

2. 上記のポーリングアプリを実行するまえに、センサ利用が競合しないように現在のセンシング処理を止める  
現在、一分おきに繰り返しているセンシング処理は crontab で行なっているので、コメントアウトする
```
crontab -e
```  
末尾で read.py を 1分毎に起動するようにしているのでここを変更する

```
この行を　↓
*/1 * * * * sudo python /home/pi/SCRIPT/slider/read.py

先頭に # を追加してコメントアウト
＃*/1 * * * * sudo python /home/pi/SCRIPT/slider/read.py
```  
`ctl-x`で保存すると read.py （センサを読んで送信する処理）が停止する  


3. `/home/pi/SCRIPT/slider/vendor/lol_dht22` を実行する  
```
pi@gc1624:~/SCRIPT/slider/vendor/lol_dht22 $ sudo ./loldht 29
Raspberry Pi wiringPi DHT22 reader
www.lolware.net
Humidity = 44.20 % Temperature = 19.00 *C
```  
パラメタの 29 は、dht22 の３線のうち、信号線が繋がっている GPIO.29 の指定。この番号を変えれば別の GPIO で dht22 を利用できる  

4. この出力文字列をJson に変換する  
```
pi@gc1624:~/SCRIPT/slider/vendor/lol_dht22 $ cd ../..
pi@gc1624:~/SCRIPT/slider $ sudo python dht22.py
{"temp": 19.0, "humidity": 44.2}
```  
このアプリケーションの実体は以下  
```
pi@gc1624:~/SCRIPT/slider $ cat -n /home/pi/SCRIPT/slider/dht22.py
     1	# coding:utf-8
     2	# Copy Right Atelier Grenouille  © 2015 -
     3	#
     4	# require: 'lol_dht' https://github.com/technion/lol_dht22
     5	# return:  {"temp": , "humidity":}
     6
     7	#import subprocess
     8	import os
     9	import sys
    10	#import commands
    11	#import subprocess
    12	import ConfigParser
    13	import subprocess32 as subprocess
    14	import re
    15	import slider_utils as slider
    16	import json
    17
    18	# 設定値の取得
    19	configfile = os.path.dirname(os.path.abspath(__file__))+'/'+os.path.splitext(os.path.basename(__file__))[0]+'.ini'
    20	#print configfile
    21	ini = ConfigParser.SafeConfigParser()
    22	ini.read(configfile)
    23
    24	def dht22(gpio):
    25	  global ini
    26	  try:
    27	    if ini.get("mode", "run_mode") == "dummy":
    28	      result = {"temp":30.0, "humidity":30.0}
    29	    else:
    30	      p = subprocess.Popen(os.path.abspath(os.path.dirname(__file__))+"/vendor/lol_dht22/loldht " + str(gpio) + " |grep Hum", stdout=subprocess.PIPE, stderr=subprocess.PIPE, shell=True)
    31	      std_out, std_err = p.communicate(None, timeout=10)
    32	      result = std_out.strip()
    33
    34	      # read result
    35	      match = re.match(r'Humidity = (.*) % Temperature = (.*) \*C',result)
    36	      temp = float(match.group(2))
    37	      humidity =float(match.group(1))
    38
    39	      # drop bad value.
    40	      if temp < -1000  or temp > 1000:
    41	        temp = None
    42	      if humidity < -1000 or humidity > 1000:
    43	        humidity = None
    44	      result = {"temp":temp, "humidity":humidity}
    45	    return result
    46	  except IOError:
    47	    slider.io_error_report()
    48	  except:
    49	    slider.unknown_error_report()
    50
    51	# http://d.hatena.ne.jp/Rion778/20121203/1354546179
    52	def HumidityDeficit(t,rh): # t: 温度, rh: 相対湿度
    53	    ret = AbsoluteHumidity(t, 100) - AbsoluteHumidity(t, rh)
    54	#    print "HD = " + str(ret)
    55	    return ret;
    56
    57	# http://d.hatena.ne.jp/Rion778/20121203/1354461231
    58	def AbsoluteHumidity(t, rh):
    59	    ret = 2.166740 * 100 * rh * tetens(t)/(100 * (t + 273.15))
    60	#    print "AH = " + str(ret)
    61	    return ret
    62
    63
    64	#  飽和水蒸気圧
    65	#  function GofGra(t){};
    66	# http://d.hatena.ne.jp/Rion778/20121126/1353861179
    67	def tetens(t):
    68	    ret = 6.11 * 10 ** (7.5*t/(t + 237.3))
    69	#    print "tetens = " + str(ret)
    70	    return ret
    71
    72	def read():
    73	  global ini
    74	  result = dht22(ini.get("gpio", "gpio"))
    75	  if result is not None:
    76	    result["humiditydeficit"] = ('%.1f' % HumidityDeficit(result["temp"],result["humidity"]))
    77	    return result
    78
    79	if __name__ == '__main__':
    80	  print json.dumps(dht22(ini.get("gpio", "gpio")))
    81
```  
ポイントは以下  
  - 30行: loldht を subprocess で実行
  - 35行: 実行結果の文字列からパターンマッチでデータを取り出す
  - 39 - 43行: 異常値を捨てる
  - 51 - 70行: センサから取得した温度と湿度を使って飽差を計算
  - 72: read() インターフェースを用意しておく。異なるセンサに対して共通インターフェースを定義すると制御が簡単になる
  - 76: センサからの取得値だけではなく、計算して作成した飽差も戻り値に混ぜる

### センサの追加
CPU の温度を取得するアプリケーションを作成する。dht22.py と同様、以下のインターフェースで作成する

  - read() を持つ。戻り値は `{データ名: 値文字列, ...}` の形式

1. CPU 温度は `vcgencmd measure_temp` で取得できる
```
pi@gc1624:~ $ vcgencmd measure_temp
temp=37.0'C
```

2. python をインタラクティブに起動して subprocess で取得してみる
```
Python 2.7.9 (default, Sep 17 2016, 20:26:04)
[GCC 4.9.2] on linux2
Type "help", "copyright", "credits" or "license" for more information.
>>> import subprocess
>>> p = subprocess.Popen("vcgencmd measure_temp", stdout=subprocess.PIPE, shell=True)
>>> value = p.stdout.readline()
>>> value
"temp=41.9'C\n"
```  

3. 値の末尾の改行が邪魔なので strip() する。末尾の空白類を strip してくれる  
```
>>> value.strip()
"temp=41.9'C"
```

4. `=` を区切り文字として前後に分ける  
```
>>> pair=value.strip().split('=')
>>> pari[0]
Traceback (most recent call last):
  File "<stdin>", line 1, in <module>
NameError: name 'pari' is not defined
>>> pair[0]
'temp'
>>> pair[1]
"41.9'C"
>>>
```

5. 単位がいらないので、正規表現をつかって数値だけとりだす
```
>>> import re
>>> match_result = re.match('([0-9,.]*).*', pair[1])
>>> match_result.group(1)
'41.9'
```

6. `{データ名: 値文字列, ...}` の形式にする
```
>>> result = {'cpu_temp': match_result.group(0)}
>>> result
{'cpu_temp': '41.9'}
```

7. 以上を纏めて、SCRIPT/slider 配下に cpu.py というファイルを作る
```
pi@gc1624:~/SCRIPT/slider $ cd
pi@gc1624:~ $ cd SCRIPT/slider/
pi@gc1624:~/SCRIPT/slider $ nano cpu.py
```  
内容は以下

```
import subprocess
import re

def remove_unit(val_unit_str):
  match = re.match('([0-9,.]*).*', val_unit_str)
  return match.group(1)

def read():
  p = subprocess.Popen("vcgencmd measure_temp", stdout=subprocess.PIPE, shell=True)
  value = p.stdout.readline().strip().split('=')[1]
  return {'cpu_temp': remove_unit(value)}

if __name__ == '__main__':
  value = read()
```  
同じ物が cpu_shell.py に用意してあるのでコピーしても良い

```
pi@gc1624:~/SCRIPT/slider $ cp cpu_shell.py cpu.py
```

7. python で実行してみる  
```
pi@gc1624:~/SCRIPT/slider $ python cpu.py
{'cpu_temp': '46.7'}
pi@gc1624:~/SCRIPT/slider $ python -m cpu
{'cpu_temp': '46.7'}
pi@gc1624:~/SCRIPT/slider $ python
Python 2.7.9 (default, Sep 17 2016, 20:26:04)
[GCC 4.9.2] on linux2
Type "help", "copyright", "credits" or "license" for more information.
>>> import cpu
>>> cpu.read()
{'cpu_temp': '46.2'}
>>>
```
