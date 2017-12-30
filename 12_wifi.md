# 12.WiFiの設定

## <u>概要</u>
GUI　及び CUI での WiFi の設定を理解する  

RPi で作成したプロダクトを現場のネットワークに接続させようとしてデスクトップどころかコンソールさえも使う事が出来ない現場の作業環境で困ってしまうことがよくある。そこで、`RPi 自身に起動時に RPi の WiFi を設定させる`方法を紹介する

## <u>実習手順</u>

### GUI 環境での WiFi の設定
1. 自身の gc15 に RemoteDisktop でログインする  
2. 画面右上の WiFi アイコンをクリックすると WiFi の設定選択アプリが開く  
<img src="pic/ss.2017-03-23 21.44.27.png" width="75%">


### CUI 環境での WiFi の設定
CUI 環境での WiFi の設定は `wpa_cli` コマンドを使う

1. 自身の gc16 に terminal でログインする  

2. `sudo wpa_cli` で wpa_cli のインタラクティブモードに入る

```
pi@gc1624:~ $ sudo wpa_cli
wpa_cli v2.3
Copyright (c) 2004-2014, Jouni Malinen <j@w1.fi> and contributors

This software may be distributed under the terms of the BSD license.
See README for more details.


Selected interface 'wlan0'

Interactive mode
```
インタラクティブモードに入らない方法もある。後で addwpa の説明にあわせて解説する

3. `list_network` コマンドで、psk を知っている SSID の一覧を表示する  

```
> list_network
network id / ssid / bssid / flags
0	Buffalo-G-E854	any	[DISABLED]
1	pi	any	[CURRENT]
2	wx01-ceca2c	any
>
```

4. `scan` コマンドで、近辺の AP の SSID をスキャンする  

```
> scan
OK
<3>CTRL-EVENT-SCAN-STARTED
<3>CTRL-EVENT-SCAN-RESULTS
> scan_result
...
...
```

5. この後、add_network, set_network, enable_network, save_config などのコマンドで AP の追加と設定の保存を手動で行うのだが、詳細は `man wpa_cli` を参照
`quit` コマンドで wpa_cli を抜ける  

```
> quit
pi@gc1624:~ $
```

5. 起動時には、自分が SSID と psk を知っている AP を探して接続を行う。SSID と psk は `/etc/wpa_supplicant/wpa_supplicant.conf ` に保存されている  
 
```
cat /etc/wpa_supplicant/wpa_supplicant.conf
```  
`wpa_cli` コマンドによる設定も、`save_config` するとこちらに反映される

6. 参考まで、`wpa_cli` で WPS(Wi-Fi Protected Setup) をつかった設定も可能で、起動時のジャンパ線の設定によって自動的に WPS で設定させる実装例も[試した事がある](http://qiita.com/UedaTakeyuki/items/e63f4c06ab2814f4fb27)のだが、  
そもそも WPS 自体があまり存在を知られておらず、次に説明する `addwpa` のほうが一般の方に使ってもらいやすかった

### addwpa
Raspberry Pi に WiFi で接続して設定を行なっている環境では、WiFi の設定の変更は大変にむづかしい。設定変更が反映された瞬間に現在の接続が切れてしまい、正しく設定変更ができたのか確認するタイミングがない  
そこで、RPi の WiFi 設定は通常は RPi にディスプレイ、マウス、キーボードを繋いだ stand alone 環境でおこなうのだが、出先や現場などでそのような環境を用意することは難しい  
また、RPi ベースのプロダクトを提供し「そちらのネットワーク環境に合わせて WiFi を登録して使ってください」と言われた相手が困ってしまうことになる  
そこで、RPi 自体に WiFi　の設定をさせる addwpa を紹介する  
gc15, gc16 では、addwpa の仕組みを使って SD カードに PC 等でアクセスポイントの SSID と psk を書いておけば、初回の起動時に自動的にアクセスポイント情報の設定を行う

1. RPi を停止し、gc16 の SD カードを抜く  

```
sudo shutdown -h 0
```

2. gc16 の SD カードを抜いて、USB SD カード reader/writer に装着し、PC に挿す

3. PC のファイルマネージャーには `boot` というパーティションが見える  
ここに登録したいアクセスポイントの SSID と psk のみの２行だけのテキストファイルを addwpa.txt という名前で作る  
```bash:/boot/addwpa.txt
SSID
psk
```  
例えば、SSID が "Buffalo-G-E854"、psk が　"ppppjjjj333tj"の場合、addwpa.txt は以下のようになる  
```bash:/boot/addwpa.txt
Buffalo-G-E854
ppppjjjj333tj
```

4. 起動時に呼び出される addwpa は addwpa.txt ファイルがあると、前述の wpa_cli コマンドをインタラクティブモードではなくパラメタ指定で実行して、指定されたAPの登録をおこなう  
addwpa の実体は以下  
```
pi@gc1624:~ $ cat -n install/addwpa/addwpa.sh
     1	#!/bin/sh -eu
     2	# http://qiita.com/youcune/items/fcfb4ad3d7c1edf9dc96
     3	#trap 'echo NG' ERR
     4
     5	addwpafile=/boot/addwpa.txt # path of addwpa.txt.
     6	wpaconf=/etc/wpa_supplicant/wpa_supplicant.conf
     7	if [ -e $addwpafile ]; then # exist addwpa.txt file then:
     8	  nkf -Lu --overwrite $addwpafile # convert CRLF or CR to LF.
     9	  echo hello
    10	  ssid=`head -n 1 $addwpafile`
    11	  psk=`head -n 2 $addwpafile | tail -n 1`
    12	  echo $ssid
    13	  echo $psk
    14	  if [ -n "$ssid" -a -n "$psk" ]; then # confirm ssid and psk
    15	    # add network statement to the end of wpa_supplicant.conf
    16	    nn=`wpa_cli add_network | tail -n 1` # added network number
    17	    wpa_cli set_network $nn ssid \"$ssid\"
    18	    wpa_cli set_network $nn psk \"$psk\"
    19	    wpa_cli enable_network $nn
    20	    wpa_cli save_config
    21	    # remove addwpa.txt
    22	    rm $addwpafile
    23	    reboot
    24	  fi
    25	fi
```

5. addwpa は[github](https://github.com/UedaTakeyuki/addwpa)で公開してメンテしている  
また、qiita に[解説記事](http://qiita.com/UedaTakeyuki/items/b64c63ade185303628eb)がある
