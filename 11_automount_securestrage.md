# 11.外部ストレージの auto mount とセキュアストレージ

##<u>概要</u>
`/` にマウントされるストレージは`/etc/fstab`によって起動時にマウントされるストレージの他に、USB に新たにストレージが装着されたことを認識して自動的にマウントする `auto mount`がある  
また、セキュアストレージの復号化とマウントを静的におこなってしまうと、その定義ファイルからストレージの暗号鍵がわかってしまうので、それをさけるために`手動でマウント`する方法もある

gc15 と gc16 では、auto mount のマウント先が異なる。gc15 が利用するデスクトップタイプの Raspbian Jessie と gc16 が利用するコマンドラインベースの Raspbian Jessie Lite では auto mount の方法が異なるからである

##<u>実習手順</u>

### gc16 の auto mount
SD カード R/W を Raspberry Pi の USB ポートに装着 ***せずに***、自身の gc16 に terminal でログインする

1.
```
pi@gc1624:~ $ lsusb
Bus 001 Device 005: ID 056e:7007 Elecom Co., Ltd
Bus 001 Device 003: ID 0424:ec00 Standard Microsystems Corp. SMSC9512/9514 Fast Ethernet Adapter
Bus 001 Device 002: ID 0424:9514 Standard Microsystems Corp.
Bus 001 Device 001: ID 1d6b:0002 Linux Foundation 2.0 root hub
```

2.
```
pi@gc1624:~ $ df
Filesystem     1K-blocks    Used Available Use% Mounted on
/dev/root        3404364 2662352    580380  83% /
devtmpfs          469536       0    469536   0% /dev
tmpfs             473868       0    473868   0% /dev/shm
tmpfs             473868    6400    467468   2% /run
tmpfs               5120       4      5116   1% /run/lock
tmpfs             473868       0    473868   0% /sys/fs/cgroup
/dev/mmcblk0p1   3507840  418864   3088976  12% /boot
/dev/mapper/i3    247791   78426    152265  34% /home/pi/SCRIPT
/dev/mapper/i4    247791  103988    126703  46% /var/www/html/SCRIPT
```

3.
```
pi@gc1624:~ $ ls /
bin   dev  home  lost+found  mnt  proc  run   srv  tmp  var
boot  etc  lib   media       opt  root  sbin  sys  usr
```

4.
```
pi@gc1624:~ $ ls /media
usb  usb0  usb1  usb2  usb3  usb4  usb5  usb6  usb7
```

5.
```
pi@gc1624:~ $ ls /media/usb0
pi@gc1624:~ $ ls /media/usb1
pi@gc1624:~ $
```

6.
```
pi@gc1624:~ $ lsusb
Bus 001 Device 006: ID 0bda:0109 Realtek Semiconductor Corp.
Bus 001 Device 005: ID 056e:7007 Elecom Co., Ltd
Bus 001 Device 003: ID 0424:ec00 Standard Microsystems Corp. SMSC9512/9514 Fast Ethernet Adapter
Bus 001 Device 002: ID 0424:9514 Standard Microsystems Corp.
Bus 001 Device 001: ID 1d6b:0002 Linux Foundation 2.0 root hub
```

7.
```
pi@gc1624:~ $ df
Filesystem     1K-blocks    Used Available Use% Mounted on
/dev/root        3404364 2662536    580196  83% /
devtmpfs          469536       0    469536   0% /dev
tmpfs             473868       0    473868   0% /dev/shm
tmpfs             473868    6444    467424   2% /run
tmpfs               5120       4      5116   1% /run/lock
tmpfs             473868       0    473868   0% /sys/fs/cgroup
/dev/mmcblk0p1   3507840  419256   3088584  12% /boot
/dev/mapper/i3    247791   78426    152265  34% /home/pi/SCRIPT
/dev/mapper/i4    247791  103988    126703  46% /var/www/html/SCRIPT
/dev/sda1          63503   20725     42778  33% /media/usb0
/dev/sda2        1814528  863108    850988  51% /media/usb1
```
8.
```
pi@gc1624:~ $ ls /media
usb  usb0  usb1  usb2  usb3  usb4  usb5  usb6  usb7
pi@gc1624:~ $ ls /media/usb0
bcm2708-rpi-0-w.dtb     bootcode.bin   fixup_x.dat       start_cd.elf
bcm2708-rpi-b.dtb       cmdline.txt    issue.txt         start_db.elf
bcm2708-rpi-b-plus.dtb  config.txt     kernel7.img       start.elf
bcm2708-rpi-cm.dtb      COPYING.linux  kernel.img        start_x.elf
bcm2709-rpi-2-b.dtb     fixup_cd.dat   LICENCE.broadcom
bcm2710-rpi-3-b.dtb     fixup.dat      LICENSE.oracle
bcm2710-rpi-cm3.dtb     fixup_db.dat   overlays
pi@gc1624:~ $ ls /media/usb1
bin   dev  home  lost+found  mnt  proc  run   srv  tmp  var
boot  etc  lib   media       opt  root  sbin  sys  usr
```

### gc15 の auto mount
R/W を Raspberry Pi の USB ポートに装着 ***せずに***、自身の gc16 に terminal でログインする

1.
```
pi@gc1524:/var/www/html $ ls /media
pi
```

```
pi@gc1524:/var/www/html $ ls /media/pi
158717B0012C7F83
```

```
pi@gc1524:~ $ ls /media/pi/158717B0012C7F83/
ls: cannot open directory /media/pi/158717B0012C7F83/: Permission denied
pi@gc1524:~ $ sudo ls /media/pi/158717B0012C7F83/
pi@gc1524:~ $
```

2.
```
pi@gc1524:~ $ sudo ls /media/pi/
158717B0012C7F83  adc806ed-d763-4eab-8319-b7ecfb276845	boot
```

```
pi@gc1524:~ $ sudo ls /media/pi/158717B0012C7F83/
pi@gc1524:~ $
```

```
pi@gc1524:~ $ ls /media/pi/adc806ed-d763-4eab-8319-b7ecfb276845/
bin   dev  home  lost+found  mnt  proc  run   srv  tmp  var
boot  etc  lib   media       opt  root  sbin  sys  usr
```

```
pi@gc1524:~ $ ls /media/pi/boot/
bcm2708-rpi-0-w.dtb     bootcode.bin   fixup_x.dat       start_cd.elf
bcm2708-rpi-b.dtb       cmdline.txt    issue.txt         start_db.elf
bcm2708-rpi-b-plus.dtb  config.txt     kernel7.img       start.elf
bcm2708-rpi-cm.dtb      COPYING.linux  kernel.img        start_x.elf
bcm2709-rpi-2-b.dtb     fixup_cd.dat   LICENCE.broadcom
bcm2710-rpi-3-b.dtb     fixup.dat      LICENSE.oracle
bcm2710-rpi-cm3.dtb     fixup_db.dat   overlays
```

### セキュアストレージ
gc16 の SD カードを挿した SD カード R/W を Raspberry Pi の USB ポートに装着 ***して***、自身の gc15 に RemoteDesktop でログインする

1. メニュー [Raspberry]-[Preferences]-[GParted]を開く  
<img src="pic/ss.2017-03-10 14.52.00.png" width="75%">

2. GPartedを使うには sudo 可能なユーザーでの sudo が必要なので、gc15 の pi アカウントのパスワード（変更していなければ `gc15pw`）を入力する  
<img src="pic/ss.2017-03-10 14.52.25.png" width="75%">

3. GParted アプリの右上のデバイス選択メニューで `/dev/sda` を選択して、USB ポートに装着した gc16 のストレージのパーティション構造を観察する  
<img src="pic/ss.2017-03-10 14.52.53.png" width="75%">  
以下のようになっている
  - unallocated
  - /dev/mmcblk0p1 fat32
  - /dev/mmcblk0p2 ext4
  - /dev/mmcblk0p3 crypt-luks
  - /dev/mmcblk0p4 crypt-luks
  - unallocated

crypt-luks が暗号化ファイルシステム

4. terminal ソフトを開いて `df` を確認する  

```
pi@gc1524:~ $ df
Filesystem     1K-blocks    Used Available Use% Mounted on
/dev/root        4766652 4149744    377192  92% /
devtmpfs          469536       0    469536   0% /dev
tmpfs             473868       0    473868   0% /dev/shm
tmpfs             473868    6528    467340   2% /run
tmpfs               5120       4      5116   1% /run/lock
tmpfs             473868       0    473868   0% /sys/fs/cgroup
/dev/mmcblk0p1   2094056   25736   2068320   2% /boot
/dev/mapper/i3    247791   16666    214025   8% /home/pi/SCRIPT
/dev/mapper/i4    247791   95374    135317  42% /var/www/html/SCRIPT
tmpfs              94776       0     94776   0% /run/user/1000
/dev/sda1        3507840  419320   3088520  12% /media/pi/boot
/dev/sda2        3404364 2662576    580156  83% /media/pi/3598ef8e-09be-47ef-9d01-f24cf61dff1d
```

5.
```
pi@gc1524:~ $ ls /media/pi/3598ef8e-09be-47ef-9d01-f24cf61dff1d
bin   dev  home  lost+found  mnt  proc  run   srv  tmp  var
boot  etc  lib   media       opt  root  sbin  sys  usr
```

6.
```
pi@gc1524:~ $ ls /media/pi/3598ef8e-09be-47ef-9d01-f24cf61dff1d/home/pi
2013-10-27 13.36.31.jpg  haarcascade_frontalface_default.xml  SCRIPT
a.php                    I2C_LCD_driver.py                    ssd1306.py
a.py                     install                              ssd1306.pyc
a.txt                    led.sh                               ssd1306_test.py
bk.txt                   LOG                                  tf.py
facemosaic.py            mosaic.jpg
gc_ssd1306.pyc           old
```

7.
```
pi@gc1524:~ $ ls /media/pi/3598ef8e-09be-47ef-9d01-f24cf61dff1d/home/pi/SCRIPT/
```
