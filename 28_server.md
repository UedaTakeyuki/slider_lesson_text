# 28.Raspberry Pi で Server を作成

##<u>概要</u>
Raspberry Pi の CPU は Server として十分な能力を持っており、実際、フランスの scaleway  のように ARM v7 CORE の CPU でべアメタルのサーバを提供するサービスもにぎわっている  
以下、Raspberry Pi をサーバとして利用するための設定を説明する

##<u>実習手順</u>
自身の gc16 に terminal でログインする

### 起動しているサーバの確認
lsof -i でネットワークコネクションの一覧を表示することができる  
PORT を LISTEN しているコマンドがサーバである
```
pi@gc1624:~ $ sudo lsof -i
COMMAND     PID        USER   FD   TYPE DEVICE SIZE/OFF NODE NAME
avahi-dae   425       avahi   12u  IPv4   6825      0t0  UDP *:mdns
avahi-dae   425       avahi   13u  IPv6   6826      0t0  UDP *:mdns
avahi-dae   425       avahi   14u  IPv4   6827      0t0  UDP *:59149
avahi-dae   425       avahi   15u  IPv6   6828      0t0  UDP *:36755
dnsmasq     513     dnsmasq    4u  IPv4   6840      0t0  UDP *:bootps
dnsmasq     513     dnsmasq    6u  IPv4   6843      0t0  UDP *:domain
dnsmasq     513     dnsmasq    7u  IPv4   6844      0t0  TCP *:domain (LISTEN)
dnsmasq     513     dnsmasq    8u  IPv6   6845      0t0  UDP *:domain
dnsmasq     513     dnsmasq    9u  IPv6   6846      0t0  TCP *:domain (LISTEN)
sshd        652        root    3u  IPv4   9981      0t0  TCP *:ssh (LISTEN)
sshd        652        root    4u  IPv6   9983      0t0  TCP *:ssh (LISTEN)
redis-ser   722       redis    4u  IPv4  11426      0t0  TCP localhost:6379 (LISTEN)
redis-ser   722       redis    5u  IPv4 158641      0t0  TCP localhost:6379->localhost:50266 (ESTABLISHED)
ntpd        723         ntp   16u  IPv4  10623      0t0  UDP *:ntp
ntpd        723         ntp   17u  IPv6  10624      0t0  UDP *:ntp
ntpd        723         ntp   18u  IPv4  10629      0t0  UDP localhost:ntp
ntpd        723         ntp   20u  IPv4  12449      0t0  UDP gc1624:ntp
shellinab   727 shellinabox    4u  IPv4  10636      0t0  TCP *:4200 (LISTEN)
xrdp        732        xrdp    6u  IPv4  10676      0t0  TCP *:3389 (LISTEN)
xrdp-sesm   739        root    6u  IPv4   7042      0t0  TCP localhost:3350 (LISTEN)
nginx       751        root    6u  IPv4  10677      0t0  TCP *:http (LISTEN)
nginx       751        root    7u  IPv6  10678      0t0  TCP *:http (LISTEN)
nginx       752    www-data    6u  IPv4  10677      0t0  TCP *:http (LISTEN)
nginx       752    www-data    7u  IPv6  10678      0t0  TCP *:http (LISTEN)
nginx       753    www-data    6u  IPv4  10677      0t0  TCP *:http (LISTEN)
nginx       753    www-data    7u  IPv6  10678      0t0  TCP *:http (LISTEN)
nginx       754    www-data    6u  IPv4  10677      0t0  TCP *:http (LISTEN)
nginx       754    www-data    7u  IPv6  10678      0t0  TCP *:http (LISTEN)
nginx       755    www-data    6u  IPv4  10677      0t0  TCP *:http (LISTEN)
nginx       755    www-data    7u  IPv6  10678      0t0  TCP *:http (LISTEN)
dhclient    941        root    6u  IPv4  10733      0t0  UDP *:bootpc
dhclient    941        root   20u  IPv4  10724      0t0  UDP *:53938
dhclient    941        root   21u  IPv6  10725      0t0  UDP *:11398
mosquitto 12749   mosquitto    4u  IPv4  47420      0t0  TCP *:9001 (LISTEN)
mosquitto 12749   mosquitto    5u  IPv4  47424      0t0  TCP *:1883 (LISTEN)
mosquitto 12749   mosquitto    6u  IPv6  47425      0t0  TCP *:1883 (LISTEN)
sshd      13559        root    3u  IPv4  46909      0t0  TCP gc1624:ssh->172.24.1.1:60022 (ESTABLISHED)
sshd      13565          pi    3u  IPv4  46909      0t0  TCP gc1624:ssh->172.24.1.1:60022 (ESTABLISHED)
python    20361        root    3u  IPv4 158640      0t0  TCP localhost:50266->localhost:6379 (ESTABLISHED)
```  
例えば、上のような場合、Server として `shellinabox`、 `xrdp`、 `nginx`、 `mosquitto` が Server として動いている。shellinabox と xrdp は既に説明したとおり Web ベースの shell と linux 用の Remote Desktop Service である
nginx はこれから説明する Web Server、mosquitto は後に説明する MQTT Broker である

### Web Server
gc16 は Web Server として nginx（エンジンエックス、と発音する）を使って、すでにみてきたように以下の Web インターフェースを提供している  
<img src="pic/ss.2017-03-08 21.01.34.png" width="75%">

1. nginx のインストールスクリプトは以下
```
pi@gc1624:~ $ cat -n install/nginx.setup.sh
     1	# NGINX, php
     2	sudo apt-get install nginx
     3	sudo apt-get install php5-fpm
     4	sudo sed -i 's|index index.html index.htm|index index.php index.html index.htm|g' /etc/nginx/sites-enabled/default
     5	sudo sed -i 's|#location ~ \\\.php$ {|location ~ \\\.php$ {|' /etc/nginx/sites-enabled/default
     6	sudo sed -i 's|#\tinclude snippets/fastcgi-php.conf;|\tinclude snippets/fastcgi-php.conf;|g' /etc/nginx/sites-enabled/default
     7	sudo sed -i 's|#\tfastcgi_pass unix:/var/run/php5-fpm.sock;|\tfastcgi_pass unix:/var/run/php5-fpm.sock; }|g' /etc/nginx/sites-enabled/default
```  
ポイントは  
  - 2行: nginx のインストール
  - 3行: php5-fpm のインストール
  - 4 - 7行: nginx 設定ファイルへの php の設定

2. 上のスクリプトの 4 - 7行で修正した nginx 設定ファイル
```
pi@gc1624:~ $ cat -n /etc/nginx/sites-enabled/default
     1	##
     2	# You should look at the following URL's in order to grasp a solid understanding
     3	# of Nginx configuration files in order to fully unleash the power of Nginx.
     4	# http://wiki.nginx.org/Pitfalls
     5	# http://wiki.nginx.org/QuickStart
     6	# http://wiki.nginx.org/Configuration
     7	#
     8	# Generally, you will want to move this file somewhere, and start with a clean
     9	# file but keep this around for reference. Or just disable in sites-enabled.
    10	#
    11	# Please see /usr/share/doc/nginx-doc/examples/ for more detailed examples.
    12	##
    13
    14	# Default server configuration
    15	#
    16	server {
    17		listen 80 default_server;
    18		listen [::]:80 default_server;
    19
    20		# SSL configuration
    21		#
    22		# listen 443 ssl default_server;
    23		# listen [::]:443 ssl default_server;
    24		#
    25		# Self signed certs generated by the ssl-cert package
    26		# Don't use them in a production server!
    27		#
    28		# include snippets/snakeoil.conf;
    29
    30		root /var/www/html;
    31
    32		# Add index.php to the list if you are using PHP
    33		index index.php index.html index.htm index.nginx-debian.html;
    34
    35		server_name _;
    36
    37		location / {
    38			# First attempt to serve request as file, then
    39			# as directory, then fall back to displaying a 404.
    40			try_files $uri $uri/ =404;
    41		}
    42
    43		# pass the PHP scripts to FastCGI server listening on 127.0.0.1:9000
    44		#
    45		location ~ \.php$ {
    46			include snippets/fastcgi-php.conf;
    47		#
    48		#	# With php5-cgi alone:
    49		#	fastcgi_pass 127.0.0.1:9000;
    50		#	# With php5-fpm:
    51			fastcgi_pass unix:/var/run/php5-fpm.sock; }
    52		#}
    53
    54		# deny access to .htaccess files, if Apache's document root
    55		# concurs with nginx's one
    56		#
    57		#location ~ /\.ht {
    58		#	deny all;
    59		#}
    60	}
    61
    62
    63	# Virtual Host configuration for example.com
    64	#
    65	# You can move that to a different file under sites-available/ and symlink that
    66	# to sites-enabled/ to enable it.
    67	#
    68	#server {
    69	#	listen 80;
    70	#	listen [::]:80;
    71	#
    72	#	server_name example.com;
    73	#
    74	#	root /var/www/example.com;
    75	#	index index.html;
    76	#
    77	#	location / {
    78	#		try_files $uri $uri/ =404;
    79	#	}
    80	#}
```  
ポイントは以下
  - 30行: document root は /var/www/html
  - 33行: index ファイルと認識するファイル一覧、左優先
  - 45,46,51行: php の設定のコメントアウトを外す

3. document route の確認  
```
pi@gc1624:~ $ ls /var/www/html
index.nginx-debian.html  index.php  SCRIPT
```  
index.nginx-debian.html と index.php がいるが、index.php が優先  
SCRIPT は gc16 のセキュアストレージ、すでに説明したようにこの中のスクリプトは悪意の第三者から保護される  
```
pi@gc1624:~ $ ls /var/www/html/SCRIPT/
BackupPi_2  gcidx  lost+found  monitor  say  sdt
```

4. html の作成  
実際に Web ページを作ってみる  
まず、`/var/www/html` に移動し、`gpio` というフォルダを作成する  
```
pi@gc1624:~ $ cd /var/www/html
pi@gc1624:/var/www/html $ sudo mkdir gpio
```  
pgio の所有者を`nginx`の起動ユーザである`www-date`に、所属グループを`pi`にして、group に書き込み権限をあたえる  
尚、一般ニューザに書き込み権限を与えると乗っ取りのリスクが発生する  

```
pi@gc1624:/var/www/html $ ls -la
total 25
drwxr-xr-x 4 root root 4096 Apr  4 22:16 .
drwxr-xr-x 3 root root 4096 Oct 15 19:22 ..
drwxr-xr-x 2 root root 4096 Apr  4 22:16 gpio
-rw-r--r-- 1 root root  764 Apr  4 22:14 gpio.html
-rw-r--r-- 1 root root  764 Apr  4 22:15 gpio.php
-rw-r--r-- 1 root root 1163 Jan 10 22:13 index.nginx-debian.html
lrwxrwxrwx 1 root root   36 Mar 24 22:40 index.php -> /var/www/html/SCRIPT/gcidx/index.php
drwxrwxr-x 8 root pi   1024 Mar 24 22:36 SCRIPT

pi@gc1624:/var/www/html $ sudo chown www-data gpio
pi@gc1624:/var/www/html $ sudo chgrp pi gpio
pi@gc1624:/var/www/html $ sudo chmod g+w gpio
pi@gc1624:/var/www/html $ ls -la
total 25
drwxr-xr-x 4 root     root 4096 Apr  4 22:16 .
drwxr-xr-x 3 root     root 4096 Oct 15 19:22 ..
drwxrwxr-x 2 www-data pi   4096 Apr  4 22:16 gpio
-rw-r--r-- 1 root     root  764 Apr  4 22:14 gpio.html
-rw-r--r-- 1 root     root  764 Apr  4 22:15 gpio.php
-rw-r--r-- 1 root     root 1163 Jan 10 22:13 index.nginx-debian.html
lrwxrwxrwx 1 root     root   36 Mar 24 22:40 index.php -> /var/www/html/SCRIPT/gcidx/index.php
drwxrwxr-x 8 root     pi   1024 Mar 24 22:36 SCRIPT
```  
作成した `gpio` 配下に移動  
```
pi@gc1624:/var/www/html $ cd gpio
pi@gc1624:/var/www/html/gpio $
```  
以下の内容で `index.html` ファイルを作成する
```
<!DOCTYPE html>
<html lang="ja">
<head>
    <meta charset="UTF-8">
    <title>Raspberry Pi GPIO</title>
</head>
<body>
    <p>GPIO 1 = </p>
    <p>GPIO 2 = </p>
    <p>GPIO 3 = </p>
    <p>GPIO 4 = </p>
    <p>GPIO 5 = </p>
    <p>GPIO 6 = </p>
    <p>GPIO 7 = </p>
    <p>GPIO 8 = </p>
    <p>GPIO 9 = </p>
    <p>GPIO 10 = </p>
    <p>GPIO 11 = </p>
    <p>GPIO 12 = </p>
    <p>GPIO 13 = </p>
    <p>GPIO 14 = </p>
    <p>GPIO 15 = </p>
    <p>GPIO 16 = </p>
    <p>GPIO 17 = </p>
    <p>GPIO 18 = </p>
    <p>GPIO 19 = </p>
    <p>GPIO 20 = </p>
    <p>GPIO 21 = </p>
    <p>GPIO 22 = </p>
    <p>GPIO 23 = </p>
    <p>GPIO 24 = </p>
    <p>GPIO 25 = </p>
    <p>GPIO 26 = </p>
    <p>GPIO 27 = </p>
    <p>GPIO 28 = </p>
    <p>GPIO 29 = </p>
</body>
</html>
```  
作成した `index.html` を、アドレス `自分のgc16のホスト名.local/gpio`でブラウザに表示する    
以下のように表示される  
<img src="pic/ss.2017-04-04 22.21.26.png" width="75%">

5. php スクリプトの作成  
html ファイルは静的なファイルをそのまま表示するだけなのでユーザやシステムとのインタラクションを反映させることができない  
そこで、先に作成した html ファイルをひな形として php スクリプトを作成する  

まず、`index.html` を `index.php` にコピーする  
```
pi@gc1624:/var/www/html/gpio $ cp index.html index.php
```  
エディタで index.php を開き、`<p>GPIO 1 = </p>` の行に以下のように php のコードを追加する  
```
<p>GPIO 1 = <?php echo rtrim(`sudo gpio read 1`) ?>></p>
```  
先ほどと同じアドレスで再度ブラウザで表示（再読み込み）すると、下記のように表示される
<img src="pic/ss.2017-04-05 7.33.12.png" width="75%">  
ポイント
  - index.php が index.html に優先されている
  - gpio コマンドを使って gpio1 の値を読んで表示している  

最期に、下記のように変更
```
<!DOCTYPE html>
<html lang="ja">
<head>
    <meta charset="UTF-8">
    <title>Raspberry Pi GPIO</title>
</head>
<body>
<?php
  for ($i = 1; $i < 30; $i++){
    echo '<p>GPIO '.$i.' = '.rtrim(`sudo gpio read $i`).'</p>';
  }
?>
</body>
</html>
```  
再表示すると、以下のように表示される  
<img src="pic/ss.2017-04-05 15.55.46.png" width="75%">  
