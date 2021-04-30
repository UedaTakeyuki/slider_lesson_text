# 34.monitor サービスの利用（任意）

## <u>目的</u>
実習で利用した RPi は商用の `monitor` サービスの無償アカウントが用意してある
インターネット接続して送信先を商用 `monitor` サービスに変更すると、gc15, gc16 はインターネット上のリアルな IoT システムとして機能する

## <u>実習手順</u>

### 送信先の変更
送信先を `http://monitor.uedasoft.com` に変更する

1. 自身の gc16 に terminal でログインする
2. /home/pi/SCRIPT/slider に移動  
```
pi@gc1624:~ $ cd SCRIPT/slider/
```

3. gen_sender.ini を編集  
12行目のように `url_base` を `http://monitor.uedasoft.com/` にする  
末尾の `/` は必要   
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
    10	url_base_local=http://localhost/SCRIPT/monitor/
    11	#url_base=http://gc1601.local/SCRIPT/monitor/
    12	url_base=http://monitor.uedasoft.com/
    13
    14	[mqtt]
    15	host=klingsor.uedasoft.com
    16	topic=gal/gal4/
    17	id_base=ueda-
    18
    19	[log]
    20	log_file=/home/pi/LOG/gen_sender.log
    21
    22	[monitor]
    23	#mode=public
    24	mode=private
```
4. 同様に gen_pic_sender.ini を編集  
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
     9	url_base_localhost=http://localhost/SCRIPT/monitor/
    10	#url_base=http://gc1601.local/SCRIPT/monitor/
    11	url_base=http://monitor.uedasoft.com/
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
    22	#mode=public
    23	mode=private
```

### インターネット接続の付与
gc15, gc16 に、BYOD (Bring Your Own Device) のモバイルルータやスマフォの ssid と psk を [addwpa](12_wifi.md) で設定する

1. gc16 の SD カードを USB SD カード r/w に装着し、PC に挿す

2. PC のファイルマネージャーには `boot` というパーティションが見える  
ここに登録したいアクセスポイントの SSID と psk のみの２業だけのテキストファイルを addwpa.txt という名前で作る  
```bash:/boot/addwpa.txt
SSID
psk
```  
例えば、SSID が "Buffalo-G-E854"、psk が　"ppppjjjj333tj"の場合、addwpa.txt は以下のようになる  
```bash:/boot/addwpa.txt
Buffalo-G-E854
ppppjjjj333tj
```

3. RPi を再起動
