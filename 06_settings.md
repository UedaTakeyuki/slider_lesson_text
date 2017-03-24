# 設定メニュー

##<u>概要</u>
設定メニューの項目と操作方法を CUI、GUI の両者について理解する

##<u> CUI 実習手順</u>
自身の gc16 に terminal でログインする

### raspi-config
Linux なので個々の設定は原理的に設定ファイルの編集で行うのだが、`raspi-config` コマンドでメニューをつかった設定が可能

1. `sudo raspi-config` コマンドで、下記のような画面になる

<img src="pic/ss.2017-03-23 18.48.45.png" width="75%">  

カーソルキーでメニューアイテムを移動する  
`<Select>` と `<Finish>`は　← → キーで選択する

`1 Expand Filesystem` は、サイズの大きい SD カードにコピーした際に余った領域の端まで ext4 のパーティションを拡張してくれる。先に説明したとおり、SD カードの互換性をなくすので数MByte あまらせたほうが良いので、これを使うことはほぼない

`2 Boot Options` は、boot 終了後に自動的にログインするか、ログインを求める状態で終わるか等の設定  

`4 Wait for Network at Boot` は Network の接続がおわるまで rc.local の実行を待ってくれる設定

`8 Overclock` は、メニューによる Overclock の設定。あまり過激でない Overclock が可能だが、過激な Overclock 及び Underclock は別途設定ファイルの操作が必要

2. （任意）`2 Change User Password` でパスワードを変更してみる
3. （任意）`5 Internationalisation Options` は TimeZone、Locale、キーボードなどの設定。現在の設定を確認する

4. （任意）`Enable Camera` は MIPI CSI-2 インターフェースの Raspberry Pi 公式カメラを有効にする設定。多くのメモリを消費するので、不要であれば disable にしておく。現在の設定を確認

5. `9 Advanced Options` を選択すると、下記の設定メニューになる
<img src="pic/ss.2017-03-23 21.16.00.png" width="75%">

メモリを CPU/GPU でどのようにわけるか  
後で利用する I2Cの有効/無効（起動時にI2Cのドライバをロードするかどうか）などの設定メニューがある  

##<u> GUI 実習手順</u>
自身の gc15 に RemoteDesktop でログインする

1. [Raspberry メニュー] - [Preferences] - [Raspberry Pi Configration] を開く  
<img src="pic/ss.2017-03-23 21.50.13.png" width="75%">
2. 設定アプリには `System`、`Interfaces`、`Performance`、`Localisation` のタブ
がある。
<img src="pic/ss.2017-03-23 21.50.30.png" width="75%">
<img src="pic/ss.2017-03-23 21.50.35.png" width="75%">
<img src="pic/ss.2017-03-23 21.50.41.png" width="75%">
<img src="pic/ss.2017-03-23 21.50.48.png" width="75%">
