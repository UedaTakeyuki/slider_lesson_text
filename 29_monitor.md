# 29.monitor サービス

##<u>概要</u>
システムを独立した小さなサービスの集合としてデザインすると変更しやすい柔軟なデザインになる  
これをマイクロサービスアーキテクチャと呼ぶ  
例として monitor サービスのアーキテクチャを紹介する  
すでに見たように monitor の postdata に値を POST するとブラウザの表示に反映された  
この一連の処理は独立した複数のサービスで実装している

##<u>実習手順</u>
自身の gc16 に terminal でログインする

### データの更新
postdata の処理をもう少し詳しくみてみる  
`自分の所属するmonitorのアドレス/SCRIPT/monitor/postdata.php`に対して下記の `curl` コマンドで適当なデータをポストする
```
pi@gc1624:~ $ curl -F "serial_id=00000000fbaa1f70" -F "name=temp" -F "data=80" http://gc1601.local/SCRIPT/monitor/postdata.php
```  
この時、サーバでなにがおきているのかを確認する  

自分が所属する monitor の RPi（それぞれ gc1601.local 及び gc1610.local）に terminal でログインし、`/var/www/html/SCRIPT/monitor/uploads/` に移動  
```
pi@gc1601:~ $ cd /var/www/html/SCRIPT/monitor/uploads/
```  
`ls` で一覧を確認すると、自分の RPi の Serial ID と同じ名前のフォルダがあるのでその下に移動する  
```
pi@gc1601:~ $ cd /var/www/html/SCRIPT/monitor/uploads/
pi@gc1601:/var/www/html/SCRIPT/monitor/uploads $ ls
0000000000000000  00000000391848bf  000000008d05061a  00000000e295c1c1
0000000008eb0991  000000004444903e  0000000099a20fe1  00000000fbaa1f70
000000000c3e5d05  00000000456c5ab4  000000009c482ea8  00000000fe486188
000000000e5ebbf5  0000000051a6c6b6  00000000b1f75ddc  ms.sh
000000000f6f8863  0000000053ac27d9  00000000b8600119  org
000000001b2015c6  0000000055210aa5  00000000cb650075
000000002159a35b  000000005d4d001f  00000000d6dc2e66
0000000032eae06b  0000000072760883  00000000d8daee79
pi@gc1601:/var/www/html/SCRIPT/monitor/uploads $ cd 00000000fbaa1f70/
```  
一覧を確認すると、以下のようなファイル構造になっている
```
pi@gc1601:/var/www/html/SCRIPT/monitor/uploads/00000000fbaa1f70 $ ls
1_temp.dini             500_video0.dini  humidity.csv         video0
2_humidity.dini         config.ini       humiditydeficit.csv
3_humiditydeficit.dini  cpu_temp.csv     SavePic.ini
4_CO2.dini.bak          fota.ini         temp.csv
```  
temp.csv ファイルを確認すると、先ほど curl で POST した値も反映されている

### データの取得
単に URL をパラメタとして `curl` コマンドを実行すると GET が送信される  
`自分の所属するmonitorのアドレス/SCRIPT/monitor/data.php`に対して以下のようにパラメタ`serial_id=自分のRPiの Serial_id`を指定して curl で GETを発行する  
```
pi@gc1624:~ $ curl http://gc1601.local/SCRIPT/monitor/data.php?serial_id=00000000fbaa1f70
```  
レスポンスとして下記のような json データが帰って来る  
```
{"serial_id":"00000000fbaa1f70","temp":[{"datetime":"2017\/04\/04 20:15:04","data":21.8},{"datetime":"2017\/04\/04 20:14:03","data":21.8},{"datetime":"2017\/04\/04 20:13:03","data":21.7},{"datetime":"2017\/04\/04 20:12:05","data":21.7},{"datetime":"2017\/04\/04 20:11:03","data":21.6},{"datetime":"2017\/04\/04 20:10:06","data":21.6},{"datetime":"2017\/04\/04 20:09:04","data":21.6},{"datetime":"2017\/04\/04 20:08:03","data":21.5},{"datetime":"2017\/04\/04 20:07:05","data":21.5},{"datetime":"2017\/04\/04 20:06:05","data":21.4},{"datetime":"2017\/04\/04 20:05:04","data":21.3}],"humidity":[{"datetime":"2017\/04\/04 20:15:05","data":38.6},{"datetime":"2017\/04\/04 20:14:03","data":38.9},{"datetime":"2017\/04\/04 20:13:04","data":38.8},{"datetime":"2017\/04\/04 20:12:05","data":38.9},{"datetime":"2017\/04\/04 20:11:03","data":38.9},{"datetime":"2017\/04\/04 20:10:07","data":38.9},{"datetime":"2017\/04\/04 20:09:04","data":39.2},{"datetime":"2017\/04\/04 20:08:04","data":39.1},{"datetime":"2017\/04\/04 20:07:05","data":39.2},{"datetime":"2017\/04\/04 20:06:05","data":39.4},{"datetime":"2017\/04\/04 20:05:04","data":39.6}],"humiditydeficit":[{"datetime":"2017\/04\/04 20:15:05","data":11.8},{"datetime":"2017\/04\/04 20:14:04","data":11.7},{"datetime":"2017\/04\/04 20:13:04","data":11.7},{"datetime":"2017\/04\/04 20:12:05","data":11.7},{"datetime":"2017\/04\/04 20:11:04","data":11.6},{"datetime":"2017\/04\/04 20:10:07","data":11.6},{"datetime":"2017\/04\/04 20:09:05","data":11.5},{"datetime":"2017\/04\/04 20:08:04","data":11.5},{"datetime":"2017\/04\/04 20:07:05","data":11.5},{"datetime":"2017\/04\/04 20:06:05","data":11.4},{"datetime":"2017\/04\/04 20:05:04","data":11.3}]}
```  
とても見にくい表示なので、下記のようにレスポンスを `jq .` にリダイレクトする  
```
pi@gc1624:~ $ curl http://gc1601.local/SCRIPT/monitor/data.php?serial_id=00000000fbaa1f70 | jq .
  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed
100  1691    0  1691    0     0   5722      0 --:--:-- --:--:-- --:--:--  5732
{
  "serial_id": "00000000fbaa1f70",
  "temp": [
    {
      "datetime": "2017/04/04 20:16:03",
      "data": 21.8
    },
    {
      "datetime": "2017/04/04 20:15:04",
      "data": 21.8
    },
    {
      "datetime": "2017/04/04 20:14:03",
      "data": 21.8
    },
    {
      "datetime": "2017/04/04 20:13:03",
      "data": 21.7
    },
    {
      "datetime": "2017/04/04 20:12:05",
      "data": 21.7
    },
    {
      "datetime": "2017/04/04 20:11:03",
      "data": 21.6
    },
    {
      "datetime": "2017/04/04 20:10:06",
      "data": 21.6
    },
    {
      "datetime": "2017/04/04 20:09:04",
      "data": 21.6
    },
    {
      "datetime": "2017/04/04 20:08:03",
      "data": 21.5
    },
    {
      "datetime": "2017/04/04 20:07:05",
      "data": 21.5
    },
    {
      "datetime": "2017/04/04 20:06:05",
      "data": 21.4
    }
  ],
  "humidity": [
    {
      "datetime": "2017/04/04 20:16:04",
      "data": 38.6
    },
    {
      "datetime": "2017/04/04 20:15:05",
      "data": 38.6
    },
    {
      "datetime": "2017/04/04 20:14:03",
      "data": 38.9
    },
    {
      "datetime": "2017/04/04 20:13:04",
      "data": 38.8
    },
    {
      "datetime": "2017/04/04 20:12:05",
      "data": 38.9
    },
    {
      "datetime": "2017/04/04 20:11:03",
      "data": 38.9
    },
    {
      "datetime": "2017/04/04 20:10:07",
      "data": 38.9
    },
    {
      "datetime": "2017/04/04 20:09:04",
      "data": 39.2
    },
    {
      "datetime": "2017/04/04 20:08:04",
      "data": 39.1
    },
    {
      "datetime": "2017/04/04 20:07:05",
      "data": 39.2
    },
    {
      "datetime": "2017/04/04 20:06:05",
      "data": 39.4
    }
  ],
  "humiditydeficit": [
    {
      "datetime": "2017/04/04 20:16:04",
      "data": 11.8
    },
    {
      "datetime": "2017/04/04 20:15:05",
      "data": 11.8
    },
    {
      "datetime": "2017/04/04 20:14:04",
      "data": 11.7
    },
    {
      "datetime": "2017/04/04 20:13:04",
      "data": 11.7
    },
    {
      "datetime": "2017/04/04 20:12:05",
      "data": 11.7
    },
    {
      "datetime": "2017/04/04 20:11:04",
      "data": 11.6
    },
    {
      "datetime": "2017/04/04 20:10:07",
      "data": 11.6
    },
    {
      "datetime": "2017/04/04 20:09:05",
      "data": 11.5
    },
    {
      "datetime": "2017/04/04 20:08:04",
      "data": 11.5
    },
    {
      "datetime": "2017/04/04 20:07:05",
      "data": 11.5
    },
    {
      "datetime": "2017/04/04 20:06:05",
      "data": 11.4
    }
  ]
}
```  
同様に`pic.php` にパラメタ`serial_id=自分の RPi の serial_id`と`device=video0` を指定して GET を発行する  
```
pi@gc1624:~ $ curl "http://gc1601.local/SCRIPT/monitor/pic.php?serial_id=00000000fbaa1f70&device=video0"
{"serial_id":"00000000fbaa1f70","device":"video0","latest_pic_name":"20170331171304.jpeg","ymd":"20170331"}
```  
デバイス`Video0`の直近の撮影画像の情報が返る

### センサの追加
もう一度、自分が所属する monitor の RPi（それぞれ gc1601.local 及び gc1610.local）に terminal でログインし、`/var/www/html/SCRIPT/monitor/uploads/` の下の自分の RPi のフォルダ配下の一覧を表示する  
```
pi@gc1601:/var/www/html/SCRIPT/monitor/uploads/00000000fbaa1f70 $ ls
1_temp.dini             500_video0.dini  humidity.csv         video0
2_humidity.dini         config.ini       humiditydeficit.csv
3_humiditydeficit.dini  cpu_temp.csv     SavePic.ini
4_CO2.dini.bak          fota.ini         temp.csv
```  
前に slider に追加した cpu_temp.csv が無事に送信されているが、ブラウザに表示されていない  
<img src="pic/ss.2017-04-05 15.12.46.png" width="75%">  

このデータをブラウザの表示に追加する
表示に追加するためには設定ファイル `dini(display initial)` ファイルの追加でよい  

まず、1_temp.dini をコピーして 4_cpu_temp.dini を作成する  
```
pi@gc1601:/var/www/html/SCRIPT/monitor/uploads/00000000fbaa1f70 $ sudo cp 1_temp.dini 5_cpu_temp.dini
```  
`4_cpu_temp.dini` を (sudoを付けて)テキストファイルで開き、以下のように編集して保存する  
```
pname=CPU温度
fname=cpu_temp
dname=cpu_temp
unit=℃
```  
ここで、fname はデータの保存されている csv ファイルの base 名、その他はグラフ表示に利用される文字列  

ブラウザを再読み込みすると、以下のようにCPU温度の表示が追加されている  
<img src="pic/ss.2017-04-05 15.21.27.png" width="75%">  

### 表示データの順番の変更
monitor は、dini ファイルのソート順昇順にデータを表示する  
表示順を変えるには、dini ファイルのファイル名の先頭の数字を変える  
例えば、CPU 温度を気温の次に表示するには CPU 温度の dini ファイルのファイル名を気温と湿度間にいれればよい  
```
pi@gc1601:/var/www/html/SCRIPT/monitor/uploads/00000000fbaa1f70 $ sudo mv 4_cpu_temp.dini 25_cpu_temp.dini
```  
<img src="pic/ss.2017-04-05 15.28.35.png" width="75%">  
