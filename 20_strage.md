# 20.データの保存先

##<u>概要</u>
データの保存先には以下の選択枝がある
  - 外部ストレージ
  - /boot
  - tmpfs
  - セキュアストレージ
  - その他
外部ストレージには USB メモリ、USB HDD、USB SD r/w、その他なんでも接続することができるが、RPi が USB に供給できる電流に限りがある（RPi 3 で 1.2A、B+ or 2B で 0.6A or 1.2A、A+ で0.5A）ので、HDD 等ドライブ電流の大きなデバイスを使う場合はセルフパワーの USB ハブを使ったほうが良い  
外部ストレージのファイルシステムのフォーマットは、RPi に追加できるものであればなんでも良い。gc15, gc16 は以下に対応している
 - Windows: FAT, NTFS
 - Mac: HFS
 - Linux: ext4


/boot は FAT32 でフォーマットしてあるので、ここに保存したデータは SD カードを PC に挿せば PC から読み書きが可能  

tmpfs は RPi の起動時に初期化される
セキュリティ要件などで起動時ではなく電源断時の消去が好ましい場合は Ramdisk にする方法がある


セキュアストレージは起動時以外は他の PC から読み書きすることができない


その他の場所に保存したデータは、SDカードの中身を他の Linux PC （等、ext4 のドライバを持つ PC）から読むことができるが、Windows や Mac からは（別途、ext4 のドライバをインストールしない限り）読み書きすることができない


##<u>実習手順</u>
自身の gc16 に terminal でログインする


### /tmp の挙動の確認
1. 現在の /tmp の状態の確認
```
pi@gc1624:~ $ ls /tmp
```
2. /tmp の下に a.txt ファイルを作る
```
pi@gc1624:~ $ echo hello > /tmp/a.txt
pi@gc1624:~ $ ls /tmp
```
3. shutdown ***しないで*** RPi の電源を切る
4. RPi から gc16 の SD カードを抜いて、USB SD r/w に挿して RPi の USB port に装着
5. RPi を ***gc15*** で起動し、***terminal*** でログインする
6. USB外付けドライブを確認する  
```
pi@gc1524:~ $ ls /media/pi
158717B0012C7F83  3598ef8e-09be-47ef-9d01-f24cf61dff1d  boot
```  
boot は外付けの gc16 SD の /boot フォルダ  
UUID は外付けの gc16 SD の ext4 のフォルダ

7. `/media/pi` にマウントされた gc16 SD の ext4 のフォルダを参照する  
```
pi@gc1524:/media/pi/3598ef8e-09be-47ef-9d01-f24cf61dff1d $ ls
bin   dev  home  lost+found  mnt  proc  run   srv  tmp  var
boot  etc  lib   media       opt  root  sbin  sys  usr
```
8. `tmp` の中身を確認する。先程作成した `a.txt` が存在していて、中身を参照できる
```
ls tmp
cat tmp/a.txt
```

9. `tmp` に `b.txt` を作成する
```
echo hello > tmp/b.txt
ls tmp
cat tmp/b.txt
```

10. 再度、shutdown ***せずに*** RPi の電源切断
11. 再度、gc16 で RPi を起動し、/tmp フォルダを確認する
```
ls /tmp
```  
a.txt も、先程作成した b.txt も消えている

12. 再度、/tmp の下に a.txt ファイルを作る
```
pi@gc1624:~ $ echo hello > /tmp/a.txt
pi@gc1624:~ $ ls /tmp
```
13. shutdown ***して*** RPi の電源を切る
```
sudo shutdown -h 0
```
14. 再度、gc16 の SD カードをRPi の USB port に装着し、RPi は ***gc15*** で起動し、***terminal*** でログインして、外付けの gc16 の /tmp フォルダを確認する
```
pi@gc1524:~ $ cd /media/pi/3598ef8e-09be-47ef-9d01-f24cf61dff1d
pi@gc1524:/media/pi/3598ef8e-09be-47ef-9d01-f24cf61dff1d $ ls tmp
```  
shutdown しても、a.txt は残っている

### /tmp の ramdisk化
実行中のアプリケーションの鍵を /tmp 配下に展開してコンテンツの復号を行うような運用を考える。pi アカウントのパスワードの強度が健全で、悪意の第三者に不正にログインされる心配がないのであれば、運用中は鍵は安全であるが、悪意の第三者に RPi の SD カードを ***物理的に盗まれてしまうと鍵も盗まれる***  

これを避けるために、/tmp をメモリ上にマウントすることができる。SD カードを盗まれてもそこに鍵はない  
余談だが、よく見かける誤解なのだが tmpfs の ramdisk 化と SD カードの寿命とは全く関係が ***ない***

1. 自身の gc16 に terminal でログインする
2. /etc/fstab を編集する
```
pi@gc1624:~ $ sudo nano /etc/fstab
proc            /proc           proc    defaults          0       0
/dev/mmcblk0p1  /boot           vfat    defaults          0       2
/dev/mmcblk0p2  /               ext4    defaults,noatime  0       1
# a swapfile is not a swap partition, no line here
#   use  dphys-swapfile swap[on|off]  for that

# ramdisk for /tmp
#tmpfs           /tmp  tmpfs   defaults,size=16m,noatime,mode=1777      0      $
```  
末尾行のコメントアウト(先頭の`#`)を外して保存する

3. 編集結果を確認し、/tmp 以下を削除してから再起動する
```
pi@gc1624:~ $ cat /etc/fstab
proc            /proc           proc    defaults          0       0
/dev/mmcblk0p1  /boot           vfat    defaults          0       2
/dev/mmcblk0p2  /               ext4    defaults,noatime  0       1
# a swapfile is not a swap partition, no line here
#   use  dphys-swapfile swap[on|off]  for that

# ramdisk for /tmp
tmpfs           /tmp  tmpfs   defaults,size=16m,noatime,mode=1777      0       0
pi@gc1624:~ $ sudo rm /tmp/*
pi@gc1624:~ $ sudo reboot
```

4. 再度ログインして `df` を確認
```
pi@gc1624:~ $ df
Filesystem     1K-blocks    Used Available Use% Mounted on
/dev/root        3404364 2675912    566820  83% /
devtmpfs          469536       0    469536   0% /dev
tmpfs             473868       0    473868   0% /dev/shm
tmpfs             473868    6396    467472   2% /run
tmpfs               5120       4      5116   1% /run/lock
tmpfs             473868       0    473868   0% /sys/fs/cgroup
tmpfs              16384      24     16360   1% /tmp
/dev/mmcblk0p1   3507840   36272   3471568   2% /boot
/dev/mapper/i3    247791   78564    152127  35% /home/pi/SCRIPT
/dev/mapper/i4    247791  104044    126647  46% /var/www/html/SCRIPT
```  
設定どおり、/tmp は 16M Byte

5. /tmp の下に a.txt ファイルを作る
```
pi@gc1624:~ $ echo hello > /tmp/a.txt
pi@gc1624:~ $ ls /tmp
```
6. shutdown ***しないで*** RPi の電源を切る
7. gc16 の SD カードをRPi の USB port に装着し、RPi は ***gc15*** で起動し、***terminal*** でログインして、外付けの gc16 の /tmp フォルダを確認する
```
pi@gc1524:~ $ cd /media/pi/3598ef8e-09be-47ef-9d01-f24cf61dff1d
pi@gc1524:/media/pi/3598ef8e-09be-47ef-9d01-f24cf61dff1d $ ls tmp
```  
shutdown しても、a.txt は残って***いない***

8. gc16 に戻り、/etc/tmpfs を元に戻す（末尾行を`#`でコメントアウトする)

### セキュアストレージ
`/home/pi/SCRIPT` と `/val/www/html/SCRIPT` はセキュアストレージ。起動時に復号化してマウントしてあり、その手順は `obfscation` でガードしてある。ガードが必要な ***永続的なファイル*** はここに保存しておけば、（運用中は復号化されているので）pi アカウントのパスワードが強固で不正にログインされない限り、盗難等にあっても悪意の第三者から内部を読み書きする手段はない

1. 7. gc16 の SD カードをRPi の USB port に装着し、RPi は ***gc15*** で起動し、***terminal*** でログインする
2. 外付けの gc16 の ext4 のストレージに移動する
3. `home/pi/` の中身を確認
```
pi@gc1524:/media/pi/3598ef8e-09be-47ef-9d01-f24cf61dff1d $ ls home/pi/
```  
普通に参照できる
4. `home/pi/SCRIPT/`の中身を確認
```
pi@gc1524:/media/pi/3598ef8e-09be-47ef-9d01-f24cf61dff1d $ ls home/pi/SCRIPT/
pi@gc1524:/media/pi/3598ef8e-09be-47ef-9d01-f24cf61dff1d $
```  
なにもない
5. `var/www/html`の中身を確認
```
pi@gc1524:/media/pi/3598ef8e-09be-47ef-9d01-f24cf61dff1d $ ls var/www/html
index.nginx-debian.html  index.php  SCRIPT
```  
普通に参照できる
g. `var/www/html/SCRIPT`の中身を確認
```
pi@gc1524:/media/pi/3598ef8e-09be-47ef-9d01-f24cf61dff1d $ ls var/www/html/SCRIPT
pi@gc1524:/media/pi/3598ef8e-09be-47ef-9d01-f24cf61dff1d $
```  
なにもない

##<u>ポイント解説</u>
1. /tmp を ramdisk で運用していると /tmp が溢れて RPi が止まってしまうことがある。/tmp 自体を ramdisk にするのではなく、適当なフォルダ（/home/pi/tmp 等）を ramdisk 化し、セキュアな情報を含む一時ファイルをそこで運用するようにしたほうがよい
