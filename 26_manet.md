# 26.MANET(Mobile Ad-hoc Network)

## <u>目的</u>
MANET はピア間でメッシュトポロジを構成しつつ定期的にルート情報を再構築することで移動にも対応できるアドホックなメッシュネットワークである  
通常のスター型トポロジと違ってメッシュ構成のため障害に強く、ルーター等の用意も不要という特徴があり、災害時の仮設ネットワークや構造物のメンテナンスなど多くの応用が期待されているネットワークプロトコルである  
Raspberry Pi では簡単に MANET を構築できる

## <u>実習手順</u>

### 講師の作業: routerでの OLSRD の準備
1. router に WiFi ドングルをもう一つ追加
2. wlan1 を `adhoc_address=174.24.1.1` で adhoc モードで起動  
```
sudo ifdown wlan1
sudo ifup wlan1=adhoc -o address=174.24.1.1
```
3. wlan1 で olsrd を起動し、デバッグ情報を　a.txt にリダイレクト
```
sudo olsrd -i wlan1 -d 1
```

### 実習: MANET の起動
174.24.1.x ネットワークに皆で参加し MANET を構築する  

1. `/boot/gc.ini` を編集する(PC に SDカードを挿して PC のエディタでも良いし、gc16 に接続してでもよい)
2. 現状で、network=wpaになっている
```
#network=pi
network=wpa
#network=adhoc
#adhoc_address=172.24.1.4
```  
これを、`network=adhoc` に変更し、さらに `adhoc_address` のコメントアウトを外し、[教室の環境](classenvironment.md) の `MANETの実習` のテーブルを参照して自分の `adhoc_address` を設定する  
下記は設定例  
```
#network=pi
#network=wpa
network=adhoc
adhoc_address=174.24.1.51
```  
保存して、gc16 を再起動
3. PC から自分の属するルーター（それぞれ、`172.24.1.1`、`173.24.1.1`）に terminal でログインする。id=pi, pw=gc16  
4. 起動時に自動的に olsrd を起動するのだが、デバッグ情報が見えないので一度、olsrd を stop して再起動する
```
sudo pkill -9 -f olsrd
sudo olsrd -i wlan0 -d 1
```　
表示の意味は以下、詳細は[こちら](http://www.olsr.org/docs/README-Link-Quality.html)  
  - LQ: Link Quality
  - NLQ: neighbor's view of the link quality. "Hello" メッセージの届いた率
  - EXT: ext metric = 1/(LQ*NLQ)
5. 教室内ではどうしても 1-hop の NEIGHBORS に固まってしまうのであまり面白くないのだが、実際はいろいろと動き回るとそれに適応したメッシュ（マルチホップでパケットを届けられるメッシュ）が構成される

### WiFi の ADHOC モードでの起動の確認
1. `CNTL + z`　で olsrd を止める
2. ネットワーク設定を確認する
```
pi@gc1601:~ $ cat -n /etc/network/interfaces
     1	# interfaces(5) file used by ifup(8) and ifdown(8)
     2
     3	# Please note that this file is written to be used with dhcpcd
     4	# For static IP, consult /etc/dhcpcd.conf and 'man dhcpcd.conf'
     5
     6	# Include files from /etc/network/interfaces.d:
     7	source-directory /etc/network/interfaces.d
     8
     9	auto lo
    10	iface lo inet loopback
    11
    12	iface eth0 inet manual
    13
    14	#allow-hotplug wlan0  
    15	#iface wlan0 inet static  
    16	iface pi inet static  
    17	    address 172.24.1.1
    18	    netmask 255.255.255.0
    19	    network 172.24.1.0
    20	    broadcast 172.24.1.255
    21
    22	iface wpa inet dhcp
    23	    wpa-conf /etc/wpa_supplicant/wpa_supplicant.conf
    24
    25	iface adhoc inet static
    26	#    address 172.24.1.1
    27	    netmask 255.255.255.0
    28	#    network 172.24.1.0
    29	#    broadcast 172.24.1.255
    30	wireless-channel 1
    31	wireless-mode ad-hoc
    32	wireless-essid pi
    33	wireless-key 01234567890123456789012345
    34
    35	#allow-hotplug wlan0
    36	#iface wlan0 inet manual
    37	#    wpa-conf /etc/wpa_supplicant/wpa_supplicant.conf
    38
    39	allow-hotplug wlan1
    40	iface wlan1 inet manual
    41	    wpa-conf /etc/wpa_supplicant/wpa_supplicant.conf
```  
定義は `iface` インタフェース名 ... という形をしていて、  
`ifup` コマンドで WiFi を起動する時にインターフェース名を指定して起動するとそのインターフェースが選ばれる  
例えば `ifup wlan1=adhoc -o address=174.24.1.1`と実行すると、wlan1 にインターフェース名 adhoc のインターフェースが設定される。その際、-o でパラメタを渡すことができ、この例では address に 174.24.1.1 を設定している  
25行 - 33行の adhoc だが、ポイントは以下
- 31行: wifi dongle のモードを ad-hoc に
- ネットワークの`channel(30行)`,`essid(32行)`,`key(33行)` を合わせる

3. /boot/gc.ini の設定を wpa に戻して再起動
