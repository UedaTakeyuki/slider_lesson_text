# 5.基本的な Folder Structure

##<u>概要</u>
Raspberry Pi の OS (Raspbian) の基本的なフォルダ構成を確認しておく

##<u>実習手順</u>
自身の gc16 に terminal でログインする

### ディレクトリの移動 と一覧の表示
1. `ls -l /` コマンドで、ルート配下のファイル一覧を確認

```
pi@gc1624:~ $ ls -l /
total 84
drwxr-xr-x   2 root root  4096 Jan  9 10:28 bin
drwxr-xr-x   7 root root 16384 Jan  1  1970 boot
drwxr-xr-x  15 root root  3580 Mar 17 16:17 dev
drwxr-xr-x 101 root root  4096 Feb 27 21:53 etc
drwxr-xr-x   3 root root  4096 Sep 23 11:26 home
drwxr-xr-x  19 root root  4096 Jan  9 10:27 lib
drwx------   2 root root 16384 Sep 23 12:52 lost+found
drwxr-xr-x  10 root root  4096 Jan  9 10:29 media
drwxr-xr-x   2 root root  4096 Sep 23 11:20 mnt
drwxr-xr-x   3 root root  4096 Sep 23 11:27 opt
dr-xr-xr-x 180 root root     0 Jan  1  1970 proc
drwx------   4 root root  4096 Jan 17 08:55 root
drwxr-xr-x  22 root root   880 Mar 17 16:18 run
drwxr-xr-x   2 root root  4096 Jan  9 10:28 sbin
drwxr-xr-x   2 root root  4096 Sep 23 11:20 srv
dr-xr-xr-x  12 root root     0 Jan  1  1970 sys
drwxrwxrwt   8 root root  4096 Mar 17 19:23 tmp
drwxr-xr-x  10 root root  4096 Sep 23 11:20 usr
drwxr-xr-x  12 root root  4096 Oct 15 19:22 var
```

### boot フォルダ
boot フォルダの中身を確認する

```
pi@gc1624:~ $ ls -l /boot
```

Linux のブートに必要となるファイルがここに格納されている  
`kernel.img` 及び `kernel7.img` は Linux Kernel そのもの。こんなに小さい


```
pi@gc1624:~ $ ls -l /boot/overlays
```
`overlays` の中身はデバイスドライバ。４章で説明する gpio や i2c に関するドライバもここにある。

理由は３章で説明するが、`/boot` は単なるマウントポイントで実体は以下で確認できる
```
pi@gc1624:~ $ df
```

`/boot` の実体は `/dev/mmcblk0p1`。この意味も３章で説明する  
/boot フォルダは 3.5G Byte と、8G Byte SD カードの半分をしめている。４章で説明するように gc15, gc16 では利便性のために /boot のサイズを拡張しているからであり、通常の素の raspbian では 60M Byte 程度である

`/` と `/boot` のマウントの定義は `/etc/fstab` にある
```
pi@gc1624:~ $ cat /etc/fstab
proc            /proc           proc    defaults          0       0
/dev/mmcblk0p1  /boot           vfat    defaults          0       2
/dev/mmcblk0p2  /               ext4    defaults,noatime  0       1
# a swapfile is not a swap partition, no line here
#   use  dphys-swapfile swap[on|off]  for that
```

### dev フォルダ
デバイス（仮想）ファイル
```
pi@gc1624:~ $ ls -l /dev
```

主なファイルは以下
- `video0` USB 外付けでカメラ
- `mmcblk0` 本体の SD カード
- `sda` 外部ストレージ
- `tty` tty

### etc フォルダ
設定ファイル
```
pi@gc1624:~ $ ls -l /etc
```

### home フォルダ
ユーザーのホームディレクトリ
```
pi@gc1624:~ $ ls -l /home
```

### sys フォルダ
sysfs  
kernel のデータを user land に export する仮想ファイルシステム
```
pi@gc1624:~ $ ls -l /sys
```

たとえば、SD カードの cid は
`cat /sys/block/mmcblk0/device/cid`

CPU温度は
`cat /sys/class/thermal/thermal_zone0/temp`

### tmp フォルダ
仮ファイルの置き場
```
pi@gc1624:~ $ ls -l /tmp
```
ここのファイルは再起動時に自動的に削除される。ramdisk をマウントすることもある

### var フォルダ
成長するファイルの保存場所
```
pi@gc1624:~ $ ls -l /var
```

主な `/var` 配下の主なフォルダは以下
- `/var/log` 各種ログファイル
- `/var/www/html` Web サーバーのドキュメントルート
