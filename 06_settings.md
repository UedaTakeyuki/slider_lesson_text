# 設定メニュー

##<u>概要</u>
設定メニューの項目と操作方法を CUI、GUI の両者について理解する

##<u> CUI 実習手順</u>
自身の gc16 に terminal でログインする

### raspi-config
Linux なので個々の設定は原理的に設定ファイルの編集で行うのだが、`raspi-config` コマンドでメニューをつかった設定が可能

1. `sudo raspi-config` コマンドで、下記のような画面になる


```
pi@gc1624:~ $ pwd
/home/pi
```

現在、作業しているディレクトリは `pi` で、ディレクトリツリーの頂点(`/`ルート)から `pi` までの pass は `/home/pi` になる  

##<u> GUI 実習手順</u>
自身の gc15 に RemoteDesktop でログインする
