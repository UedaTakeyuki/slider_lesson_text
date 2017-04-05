# 29.マイクロサービス

##<u>概要</u>
サービスを独立した小さなサービスの集合としてデザインすると変更しやすい柔軟なデザインになる

##<u>実習手順</u>
自身の gc16 に terminal でログインする

### データの更新
```
pi@gc1624:~ $ curl -F "serial_id=00000000fbaa1f70" -F "name=temp" -F "data=80" http://gc1601.local/SCRIPT/monitor/postdata.php
```

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
pi@gc1601:/var/www/html/SCRIPT/monitor/uploads/00000000fbaa1f70 $ ls
1_temp.dini             500_video0.dini  humidity.csv         video0
2_humidity.dini         config.ini       humiditydeficit.csv
3_humiditydeficit.dini  cpu_temp.csv     SavePic.ini
4_CO2.dini.bak          fota.ini         temp.csv
```

### データの取得
```
pi@gc1624:~ $ curl http://gc1601.local/SCRIPT/monitor/data.php?serial_id=00000000fbaa1f70
{"serial_id":"00000000fbaa1f70","temp":[{"datetime":"2017\/04\/04 20:15:04","data":21.8},{"datetime":"2017\/04\/04 20:14:03","data":21.8},{"datetime":"2017\/04\/04 20:13:03","data":21.7},{"datetime":"2017\/04\/04 20:12:05","data":21.7},{"datetime":"2017\/04\/04 20:11:03","data":21.6},{"datetime":"2017\/04\/04 20:10:06","data":21.6},{"datetime":"2017\/04\/04 20:09:04","data":21.6},{"datetime":"2017\/04\/04 20:08:03","data":21.5},{"datetime":"2017\/04\/04 20:07:05","data":21.5},{"datetime":"2017\/04\/04 20:06:05","data":21.4},{"datetime":"2017\/04\/04 20:05:04","data":21.3}],"humidity":[{"datetime":"2017\/04\/04 20:15:05","data":38.6},{"datetime":"2017\/04\/04 20:14:03","data":38.9},{"datetime":"2017\/04\/04 20:13:04","data":38.8},{"datetime":"2017\/04\/04 20:12:05","data":38.9},{"datetime":"2017\/04\/04 20:11:03","data":38.9},{"datetime":"2017\/04\/04 20:10:07","data":38.9},{"datetime":"2017\/04\/04 20:09:04","data":39.2},{"datetime":"2017\/04\/04 20:08:04","data":39.1},{"datetime":"2017\/04\/04 20:07:05","data":39.2},{"datetime":"2017\/04\/04 20:06:05","data":39.4},{"datetime":"2017\/04\/04 20:05:04","data":39.6}],"humiditydeficit":[{"datetime":"2017\/04\/04 20:15:05","data":11.8},{"datetime":"2017\/04\/04 20:14:04","data":11.7},{"datetime":"2017\/04\/04 20:13:04","data":11.7},{"datetime":"2017\/04\/04 20:12:05","data":11.7},{"datetime":"2017\/04\/04 20:11:04","data":11.6},{"datetime":"2017\/04\/04 20:10:07","data":11.6},{"datetime":"2017\/04\/04 20:09:05","data":11.5},{"datetime":"2017\/04\/04 20:08:04","data":11.5},{"datetime":"2017\/04\/04 20:07:05","data":11.5},{"datetime":"2017\/04\/04 20:06:05","data":11.4},{"datetime":"2017\/04\/04 20:05:04","data":11.3}]}
```

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

```
pi@gc1624:~ $ curl "http://gc1601.local/SCRIPT/monitor/pic.php?serial_id=00000000fbaa1f70&device=video0"
{"serial_id":"00000000fbaa1f70","device":"video0","latest_pic_name":"20170331171304.jpeg","ymd":"20170331"}
```
