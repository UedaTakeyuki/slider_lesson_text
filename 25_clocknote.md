# 25.制御サービス clocknote

## <u>目的</u>
LCD 表示と読み上げをコントロールしている clocknote の実装を確認することで表示の制御の実際にあわせて Linux のサービスの作り方を理解する  
サービスとは Linux から直接呼び出されるアプリケーションプログラムで、古くはデーモンと呼ばれていた  
自動起動されるアプリケーションを用意する方法は以下の 3通りになる
  - 起動スクリプト(`/etc/rc.local`) から呼び出す方法
  - crontab で定時起動する方法
  - サービスにする方法

## <u>実習手順</u>
自身の gc16 に terminal でログインする

### clocknote の操作
サービスは systemctl コマンドで操作する

1. clocknote の停止
```
pi@gc1624:~ $ sudo systemctl stop clock_note.service
```  
表示の更新が止まることを確認する

2. clocknote の最下位
```
pi@gc1624:~ $ sudo systemctl restart clock_note.service
```  
表示の更新が再開することを確認する

3. clocknote の disable  
disable にすると、起動時に自動起動しない
```
pi@gc1624:~ $ sudo systemctl disable clock_note.service
Removed symlink /etc/systemd/system/multi-user.target.wants/clock_note.service.
```  
再起動して、clocknote が自動起動しない事、systemctl start で起動する事を確認

4. clocknote の enable
enable なサービスは起動時に自動起動する
```
pi@gc1624:~ $ sudo systemctl enable clock_note.service
Created symlink from /etc/systemd/system/multi-user.target.wants/clock_note.service to /etc/systemd/system/clock_note.service.
```  
再起動して自動起動することを確認

### clock note の実装
clocknote 自体は 160行程の小さな python のスクリプト
```
pi@gc1624:~ $ cat -n SCRIPT/slider/clock_note.py
     1	# coding:utf-8 Copy Right Atelier Ueda © 2016 -
     2	#
     3	import os
     4	import sys
     5	sys.path.append(os.path.dirname(os.path.abspath(__file__))+"/vendor")
     6	from i2clibraries import i2c_lcd
     7	import quick2wire
     8	import datetime
     9	import time
    10	import subprocess
    11	import logging
    12	import traceback
    13	import inspect
    14	import requests
    15	import configparser
    16
    17	# 定数
    18	configfile = os.path.dirname(os.path.abspath(__file__))+'/clock_note.ini'
    19
    20	# 設定の取得
    21	ini = configparser.SafeConfigParser()
    22	ini.read(configfile)
    23	p = subprocess.Popen( os.path.dirname(os.path.abspath(__file__))+"/geti2caddress.sh ", stdout=subprocess.PIPE, shell=True)
    24	i2c_addr = p.stdout.readline().strip().decode('utf-8')
    25	#print (i2c_addr)
    26
    27	logging.basicConfig(format='%(asctime)s %(filename)s %(lineno)d %(levelname)s %(message)s',filename='/home/pi/LOG/clock_note.engine.log',level=logging.DEBUG)
    28	lcd = i2c_lcd.i2c_lcd(int("0x" + i2c_addr,0),0, 2, 1, 0, 4, 5, 6, 7, 3)
    29
    30	def msg_log(msg_str):
    31		print (str(inspect.currentframe().f_lineno) + " " + msg_str)
    32		logging.info(str(inspect.currentframe().f_lineno) + " " + msg_str)
    33
    34	def msg_err_log(msg_str):
    35		print (str(inspect.currentframe().f_lineno) + " " + msg_str)
    36		logging.error(str(inspect.currentframe().f_lineno) + " " + msg_str)
    37
    38	def say(phrase):
    39		try:
    40			if ini.get("path", "say_path"): # settings is NOT null then
    41				payload = {'phrase': phrase}
    42				r = requests.post(ini.get("path", "say_path"), data=payload, timeout=10, verify=False)
    43		except:
    44			msg_err_log(traceback.format_exc())
    45
    46	def current_ip():
    47		p = subprocess.Popen("hostname -I",
    48										stdout=subprocess.PIPE,
    49										shell=True)
    50		result = p.stdout.readline().strip()
    51		print (result)
    52		return result
    53
    54	def show_ip(sec):
    55		global lcd
    56		p = subprocess.Popen("hostname -I",
    57										stdout=subprocess.PIPE,
    58										shell=True)
    59		lcd.home()
    60		lcd.clear()
    61		lcd.setPosition(1,0)
    62		lcd.writeString("IP:")
    63		lcd.setPosition(2,0)
    64		lcd.writeString(p.stdout.readline().strip().decode('utf-8'))
    65		time.sleep(sec)
    66
    67	def show_temp(sec):
    68		global lcd
    69		p = subprocess.Popen("tail -n 1 "+ini.get("data", "temp_path")+"/temp.csv",
    70										stdout=subprocess.PIPE,
    71										shell=True)
    72		result = p.stdout.readline().strip().decode('utf-8','ignore').split(',')
    73		lcd.home()
    74		lcd.clear()
    75		lcd.writeString("TEMP = " + result[1])
    76		say("温度"+result[1]+"度")
    77		time.sleep(sec)
    78
    79	def show_humidity(sec):
    80		global lcd
    81		p = subprocess.Popen("tail -n 1 "+ini.get("data", "humidity_path")+"/humidity.csv",
    82										stdout=subprocess.PIPE,
    83										shell=True)
    84		result = p.stdout.readline().strip().decode('utf-8').split(',')
    85		lcd.home()
    86		lcd.clear()
    87		lcd.writeString("Humidity = " + result[1])
    88		say("湿度"+result[1]+"%")
    89		time.sleep(sec)
    90
    91	def show_humiditydeficit(sec):
    92		global lcd
    93		p = subprocess.Popen("tail -n 1 "+ini.get("data", "humiditydeficit_path")+"/humiditydeficit.csv",
    94										stdout=subprocess.PIPE,
    95										shell=True)
    96		result = p.stdout.readline().strip().decode('utf-8').split(',')
    97		lcd.home()
    98		lcd.clear()
    99		lcd.writeString("HumidDef = " + result[1])
   100		time.sleep(sec)
   101
   102	def show_CO2(sec):
   103		global lcd
   104		if ini.get("data", "CO2_path"): # settings is NOT null then
   105			p = subprocess.Popen("tail -n 1 "+ini.get("data", "CO2_path")+"/CO2.csv",
   106										stdout=subprocess.PIPE,
   107										shell=True)
   108			result = p.stdout.readline().strip().decode('utf-8').split(',')
   109			lcd.home()
   110			lcd.clear()
   111			lcd.writeString("CO2 = " + result[1])
   112			say("二酸化炭素濃度"+result[1]+"ppmです")
   113			time.sleep(sec)
   114		else:
   115			lcd.home()
   116			lcd.clear()
   117
   118	def fork():
   119		pid = os.fork()
   120		if pid > 0:
   121			f = open('/var/run/clock_note.pid','w')
   122			f.write(str(pid)+"\n")
   123			f.close()
   124			sys.exit()
   125
   126		if pid == 0:
   127			main()
   128
   129	def main():
   130		global lcd
   131		lcd.backLightOn()
   132		now_str_prev = datetime.datetime.now().strftime('%m-%d %H:%M:%S')
   133		is_said = False
   134		while True:
   135			now = datetime.datetime.now()
   136			now_str = datetime.datetime.now().strftime('%m-%d %H:%M:%S')
   137			if (now.second == 1):
   138				if not is_said:
   139					say(str(now.hour) + "時" + str(now.minute) + "分です")
   140					is_said = True
   141			if (now.second == 2):
   142				is_said = False
   143			
   144			if (datetime.datetime.now().second == 31):
   145				show_ip(2)
   146				show_temp(3) # openjtalk が間に合わない
   147				show_humidity(2)
   148				show_humiditydeficit(2)
   149				show_CO2(2)
   150
   151			if not now_str == now_str_prev:
   152				lcd.home()
   153				lcd.writeString(now_str)
   154				now_str_prev = now_str
   155
   156			time.sleep(0.1)
   157
   158	if __name__ == '__main__':
   159		fork()
```  
ポイントは以下
  - 28行: i2c LED モジュールの初期化
  - 38行: say()の定義、先ほど実行した web アプリ say() を requests モジュールで呼び出す  
web アプリへの呼び出しとして実装すると非同期処理が簡単に実装できる
  - 118行: fork() 子プロセスを生成して、プロセス ID をファイルに保存して終了。子プロセスでは main() を続行
  - 134 - 156行: 0.1 秒毎に繰り返し処理
  - 137行: 毎時 01 秒に時刻の読み上げ
  - 144行: 毎時 31 秒にIPアドレスの表示、温度の表示、湿度の表示、CO2濃度の表示
  - 67行: 温度の表示の実際

2. Service の定義ファイルを確認する
```
pi@gc1624:~ $ cat -n /etc/systemd/system/clock_note.service
     1	[Unit]
     2	Description=Sample Daemon
     3	After=rc-local.service
     4	[Service]
     5	ExecStart=/usr/bin/python3 /home/pi/SCRIPT/slider/clock_note.py
     6	Restart=always
     7	Type=forking
     8	PIDFile=/var/run/clock_note.pid
     9	[Install]
    10	WantedBy=multi-user.target
```  
ポイントは以下  
  - 3行: サービスの起動は rc.local の終了後
  - 5行: 実行パス
  - 6行: なんらかの理由でサービスが止まると自動再起動
  - 7行: サービス起動完了の判定条件。forking は、起動処理は子プロセスを生成して自分は終了するタイプなので起動処理の正常終了を判定条件とする
  - 8行: プロセスID が書いてある pid ファイル。clocknote.py で親プロセスが死ぬ前に作成

### pid ファイルの確認
pid ファイルには実行しているサービスのプロセスID が保存されている

1. clocknote の pid ファイルの確認
```
pi@gc1624:~ $ cat /var/run/clock_note.pid
1207
```

2. 表示された pid の確認
```
pi@gc1624:~ $ ps 1207
PID TTY      STAT   TIME COMMAND
1207 ?        S      0:16 /usr/bin/python3 /home/pi/SCRIPT/slider/clock_note.py
```

3. 対障害性の確認、このプロセスを強制終了しても自動的に clocknote が再起動する
```
pi@gc1624:~ $ sudo kill -9 1207
pi@gc1624:~ $ ps -aef | grep clock_note.py
root      3496     1  2 15:33 ?        00:00:01 /usr/bin/python3 /home/pi/SCRIPT/slider/clock_note.py
pi        3683  3046  0 15:35 pts/0    00:00:00 grep --color=auto clock_note.py
```  
IoT クライアントとして何らかの機能を提供する場合、とまってしまってはいけないので下記のどれかの方法で実装する
  - Restart=always の system として実装
  - clontab で繰り返し起動
  - それ以外の場合は、hart beat を確認する処理を設ける
