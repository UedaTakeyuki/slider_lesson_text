# 35.クライアントとサーバを兼ねる構成

## <u>目的</u>
gc16 をルーターとして実行しつつ、slider と monitor を起動すると、その場の監視データを自身が提供するインターネット接続を通して Web で配信する監視システムになり、インターネット接続が存在しないような場所で無線で測定値を提供したい場合などに利用できる他、複数台の gc16 で構築した MANET ネットワークで遠隔地まで無線で中継をおこなうこともできる  

本章では、gc16 にクライアントとサーバを兼ねさせる構成を設定する方法を説明する

## <u>実習手順</u>

### 送信先の変更
送信先を `http://monitor.uedasoft.com` に変更する

1. 自身の gc16 に terminal でログインする
2. /home/pi/SCRIPT/slider に移動  
```
pi@gc1624:~ $ cd SCRIPT/slider/
```

3. gen_sender.ini を編集  
10行目のように `url_base` を `http://localhost/SCRIPT/monitor/` にする  
末尾の `/` は必要   
また、`[monitor]`セクションの `mode`を`public`にする  
```
pi@gc1624:~/SCRIPT/slider $ cat -n gen_sender.ini
     1	[send]
     2	protocol=http
     3	protocol_old=mqtt
     4
     5	[server]
     6	url_base_gal3=https://klingsor.uedasoft.com/tools/151001/
     7	url_base_gal4=https://klingsor.uedasoft.com/tools/151108/
     8	url_base_klingsor=https://klingsor.uedasoft.com/tools/160613/gal4_server/
     9	url_base_c9=https://gal5-ueda.c9users.io/
    10	url_base=http://localhost/SCRIPT/monitor/
    11	#url_base=http://gc1601.local/SCRIPT/monitor/
    12
    13	[mqtt]
    14	host=klingsor.uedasoft.com
    15	topic=gal/gal4/
    16	id_base=ueda-
    17
    18	[log]
    19	log_file=/home/pi/LOG/gen_sender.log
    20
    21	[monitor]
    22	mode=public
    23	#mode=private
pi@gc1624:~/SCRIPT/slider $
```
4. 同様に gen_pic_sender.ini を編集  
`url_base` を `http://localhost/SCRIPT/monitor/` にして
`[monitor]`セクションの `mode`を`public`にする  

```
pi@gc1624:~/SCRIPT/slider $ cat -n gen_pic_sender.ini
     1	[send]
     2	protocol=http
     3	protocol_old=mqtt
     4
     5	[server]
     6	url_base_gal3=https://klingsor.uedasoft.com/tools/151001/
     7	url_base_gal4=https://klingsor.uedasoft.com/tools/151108/
     8	url_base_klingsor=https://klingsor.uedasoft.com/tools/160613/gal4_server/
     9	url_base=http://localhost/SCRIPT/monitor/
    10	#url_base=http://gc1601.local/SCRIPT/monitor/
    11
    12	[mqtt]
    13	host=klingsor.uedasoft.com
    14	topic=gal/gal4/
    15	id_base=ueda-
    16
    17	[log]
    18	log_file=/home/pi/LOG/gen_sender.log
    19
    20	[monitor]
    21	mode=public
    22	#mode=private
pi@gc1624:~/SCRIPT/slider $
```

### router として設定
gc16を[routerとして設定](27_router.md#pi_network)して起動する  

### 端末を pi ネットワークに接続
PCやモバイル機器を以下のネットワークに接続する  
- SSID: pi
- psk:  Raspberry

### ブラウザで monitor に接続
下記のように `172.24.1.1`に接続
<img src="pic/ss.2017-04-06 11.46.18.png" width="75%">  
`Set DateTime`で RPi の時計を端末の時計であわせることができる  
`motoin` を選択すると、下記のようなログイン画面が開くので、id, pw ともに `00`でログイン  
<img src="pic/ss.2017-04-06 11.46.32.png" width="75%">  
以下のように `monitor` が利用できる
<img src="pic/ss.2017-04-06 11.50.52.png" width="75%">  

### 表示要素の変更
表示要素の変更は gc16 の `var/www/html/SCRIPT/monitor/uploads/0000000000000000` フォルダで .dini ファイルを編集する  
```
pi@gc1624:/var/www/html/SCRIPT/monitor/uploads/0000000000000000 $ ls
1_temp.dini             config.ini    humiditydeficit.csv  video1
2_humidity.dini         cpu_temp.csv  SavePic.ini
3_humiditydeficit.dini  fota.ini      temp.csv
500_video0.dini         humidity.csv  video0
```
