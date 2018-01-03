# 16.USB WebCamの利用

## <u>概要</u>
Raspberry Pi では財団公式の高速高画質なカメラを利用することも、特殊な Serial Camera を利用することも、安価な USB WebCam を利用することもできる。USB WebCam と Linux 間の通信は USB のデバイスクラスの一つ UVC（USB Video Class) を介して Linux 上の標準のドライバでおこなわれる。そのため、USB WebCam の利用には特別なドライバや設定は不要で、単に USB にカメラを挿せば自動的に認識され、`/dev/video0` 等のデバイスとしてすぐに利用できる  

## <u>実習手順</u>
自身の gc16 に terminal でログインする

### usb 機器の一覧
usb に接続されている機器の一覧を表示する

```
pi@gc1624:~ $ lsusb
Bus 001 Device 004: ID 0bda:0109 Realtek Semiconductor Corp.
Bus 001 Device 009: ID 0c45:62e0 Microdia MSI Starcam Racer
Bus 001 Device 003: ID 0424:ec00 Standard Microsystems Corp. SMSC9512/9514 Fast Ethernet Adapter
Bus 001 Device 002: ID 0424:9514 Standard Microsystems Corp.
Bus 001 Device 001: ID 1d6b:0002 Linux Foundation 2.0 root hub
```

`lsusb -v` オプションをつけると詳細情報が表示される。UVC 機器の場合、サポートする画角等のインターフェース情報も表示される

### video デバイス
カメラが認識されると video のデバイスが作成される
```
pi@gc1624:~ $ ls /dev/video*
/dev/video0
```

### fswebcam
インターネットに接続されている Raspberry Pi で、`apt-cache search`コマンドで apt のパッケージマネージャを検索できる。webcam というキーワードで検索すると 43個見つかる  
```
pi@gc1624:~ $ apt-cache search webcam | wc -l
43
pi@gc1624:~ $ apt-cache search webcam
cameramonitor - Webcam monitoring in system tray
camorama - gnome utility to view and save images from a webcam
cheese - tool to take pictures and videos from your webcam
cheese-common - Common files for the Cheese tool to take pictures and videos
feh - imlib2 based image viewer
freetuxtv - Internet television and radio player
fswebcam - Tiny and flexible webcam program
gir1.2-cheese-3.0 - tool to take pictures and videos from your webcam - gir bindings
...
```  
色々とためしてみて気に入ったのを使えばいいのだが  
ここでは、その中の一つ fswebcam を利用する  
```
pi@gc1624:~ $ apt-cache search fswebcam
fswebcam - Tiny and flexible webcam program
```

1. /boot/DATA 配下に移動  
```
pi@gc1624:~ $ cd /boot/DATA
pi@gc1624:/boot/DATA $
```

2. `sudo fswebcam a.jpg` コマンドで撮影

3. /boot/DATA に a.jpg というファイルができている。RPi をshutdown して SD カードを RPi から抜き USB SD reader/writer で PC に挿し、PC で開く

### 複数台のカメラ

1. 隣の人から USB WebCam を借りて、２台の WebCam を接続する

2. ２台の video のデバイスが作成される
```
pi@gc1624:~ $ ls /dev/video*
/dev/video0
/dev/video1
```

3. 2台の WebCam を別々の方向にむけて撮影  
`sudo fswebcam -d /dev/video0 a.jpg`
`sudo fswebcam -d /dev/video1 b.jpg`

4. SD カードを抜いて、PC で確認

### fswebcam のオプション

manpage でオプションを確認  
`man fswebcam`  
よく使うオプション  
  - -r: 画角、-r 320x240
  - -D: 遅延、-D 1
  - -S: スキップ、-S 20

### gc15 で撮影写真の確認
gc15 では、撮影した写真をそのままデスクトップのイメージビューアで確認できる
1. 自身の gc15 に RemoteDesktop でログインする
2. terminal を開いて `fswebcam` で撮影する  
<img src="pic/ss.2017-03-28 14.00.05.png" width="75%">
3. 画面左上、３つめのアイコンのファイルマネージャを開く  
<img src="pic/ss.2017-03-28 14.00.15.png" width="75%">
4. a.jpg があるのでクリックすると
<img src="pic/ss.2017-03-28 14.02.25.png" width="75%">
5. 撮影画像がイメージビューアで開く
<img src="pic/ss.2017-03-28 14.02.58.png" width="75%">
