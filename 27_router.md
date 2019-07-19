# 27.Raspberry Pi で Mobile Router を作成

## <u>概要</u>
モバイルルーターは以下の機能をもつコンピュータである
  - Access Point 機能の提供
  - DHCP server 機能の提供
  - ルーティング  

商用のモバイルルータのプロダクトも内部では小型の Linux ボードで上記を実装していることが多く、逆もまた同様で Raspberry Pi のような小型の汎用 Linux ボードも簡単にモバイルルータにすることができる

## <u>実習手順</u>
自身の gc16 に terminal でログインする

### Access Point 機能
`hostapd` を使って Access Point 機能を提供することができる

1. hostapd は使用する NIC のドライバ毎にコンパイル時の設定が異なるため、複数種類の NIC に対応するためには複数の hostapd コマンドを用意しておき、NIC のドライバに合わせて使う必要がある  
```
pi@gc1624:~ $ ls install/hostapd*
install/hostapd.conf  install/hostapd_mac80211  install/hostapd_r8188eu
```  
ドライバの種類は　ethtool で確認することができる
```
pi@gc1624:~ $ sudo ethtool -i wlan0
driver: brcmfmac
version: 7.45.41.26
firmware-version: 01-df77e4a7
bus-info: mmc1:0001:1
supports-statistics: no
supports-test: no
supports-eeprom-access: no
supports-register-dump: no
supports-priv-flags: no
```  
以下のように値だけ取り出すことができる  
```
pi@gc1624:~ $ sudo ethtool -i wlan0 | grep driver | cut -c9-
brcmfmac
```

2. hostapd の設定
```
pi@gc1624:~ $ cat -n install/hostapd.conf
     1	interface=wlan0
     2	#driver=nl80211
     3	hw_mode=g
     4	ssid=
     5	wpa_passphrase=
     6	channel=4
     7	macaddr_acl=0
     8	wmm_enabled=10
     9	auth_algs=3
    10	beacon_int=100
    11	ignore_broadcast_ssid=0
    12	wpa=2
    13	wpa_key_mgmt=WPA-PSK
    14	wpa_pairwise=TKIP
    15	rsn_pairwise=CCMP
```  
ポイント
  - 1行: AP にする interface 名
  - 3行: SSID（ここでは消してある）
  - 4行: wpa_passphrase（ここでは消してある）

### DHCP server 機能
`dnsmasq`, `isc-dhcp-server` が利用できる。ここでは `dnsmasq` を解説する

1. 設定は `/etc/dnsmasq.conf`
```
pi@gc1624:~ $ cat -n /etc/dnsmasq.conf
...
152	# Uncomment this to enable the integrated DHCP server, you need
153	# to supply the range of addresses available for lease and optionally
154	# a lease time. If you have more than one network, you will need to
155	# repeat this for each network on which you want to supply DHCP
156	# service.
157	#dhcp-range=192.168.0.50,192.168.0.150,12h
158	interface=wlan0
159	no-dhcp-interface=eth0
160	dhcp-range=172.24.1.50,172.24.1.150,12h
...
```  
ポイント
  - 158行: dhcp で IP address を配布する先は wlan0
  - 159行: eth0 への dhcp request には答えない
  - 160行: 配布するアドレスは 172.24.1.50 - 172.24.1.150 で、リース時間は 12時間

### IPフォーワーディングとルーティング
AP としての接続機能を提供し、dhcp としてIP address を配布することで IP network へ参加する機能を提供し、残るのは Router の主たる機能である `IP forwarding` と `routing`である
IP forwarding とは、外から来たパケットを別のネットワークに転送する機能で、例えば `wlan0` で AP 機能を提供していて、3G ドングル等を通じて `ppp0` でインターネットに接続しているような時に、wlan0 で接続してきた端末にインターネットへの接続を提供するような機能である  
routing とは本来、ルーティングとはフォーワーディングを行う際、どのネットワークから来たパケットをどのネットワークに流すかを判断する処理である  
いくつかやり方があるのだが、ここでは iptables を使う方法を説明する

1. フォーワーディングの許可  
通常、Linux カーネルはフォーワーディングを禁止するようになっているので、`proc` ファイルを通してフォーワーディングを許可する
```
echo 1 > /proc/sys/net/ipv4/ip_forward
```

2. iptable の FORWARD の設定  
以下は eth0 が外部につながっているとして、eth0 へのフォーワーディングを許可させるスクリプト  
```
pi@gc1624:~/SCRIPT/wvdial $ cat 2eth0.sh
# routing from wlan0 to eth0, refer  http://akkagi.info/20160628_web/
echo 1 > /proc/sys/net/ipv4/ip_forward
iptables -t nat -A POSTROUTING -o eth0 -j MASQUERADE
iptables -A FORWARD -i eth0 -o wlan0 -m state --state RELATED,ESTABLISHED -j ACCEPT
iptables -A FORWARD -i wlan0 -o eth0 -j ACCEPT
```  
以下は ppp0 が外部につながっているとして、ppp0 へのフォーワーディングを許可させるスクリプト  
```
pi@gc1624:~/SCRIPT/wvdial $ cat 2ppp0.sh
# routing from wlan0 to eth0, refer  http://akkagi.info/20160628_web/
echo 1 > /proc/sys/net/ipv4/ip_forward
iptables -t nat -A POSTROUTING -o ppp0 -j MASQUERADE
iptables -A FORWARD -i ppp0 -o wlan0 -m state --state RELATED,ESTABLISHED -j ACCEPT
iptables -A FORWARD -i wlan0 -o ppp0 -j ACCEPT
```

<a name="pi_network"/>
### gc16 の起動時設定  

gc16 は設定で router として起動させることができる  

1. gc16 を shutdown  
```
pi@gc1624:~ $ sudo shutdown -h 0
```

2. gc16 の SD カードを PC に挿し、PC のテキストエディタで /boot/gc.ini の設定を network=wpa から network=pi に変更する
以下は修正例
```
network=pi
#network=wpa
#network=adhoc
#adhoc_address=172.24.1.4
```

3. SD カードを gc16 に戻し RPi を起動
4. PC の WiFi の設定で利用できるルーターを確認。同名のネットワークが沢山できているのが見える
5. RPi を停止し、SD カードを抜いて PC で /boot/gc.ini の設定を元のとおり network=wpa にもどす  
以下は修正例  
```
#network=pi
network=wpa
#network=adhoc
#adhoc_address=172.24.1.4
```  
この設定で、以下のような AP, DNS の設定で起動する
- AP
  - ssid:
  - psk:
- DNS  
  - 自身のIP アドレス: 172.24.1.1
  - 配布するIP アドレス: 172.24.1.50 - 172.24.1.150



6. （任意）hostapd で SSID を一意なものに変更して起動する。PC のネットワーク接続を pi もしくは pi2 から自分設定した SSID のものにアクセスし、172.24.1.1 にログインする  
hostapd の設定を元に戻して終了
