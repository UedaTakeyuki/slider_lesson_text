# バックアップ&リストア

##<u>目的</u>
Raspberry Pi の SD カードは複数のパーティションを持つこと、ext4 のファイルシステムが Windows や Mac でサポートされていない事などから、ファイル単位でのバックアップではなくストレージ全体のバイトイメージを取得する方法が一般的

しかし、この処理はかなり神経を使う。バイトイメージの戻し先を間違えると自分の PC のストレージが破壊されてしまう

Raspbian Jessie で、SD card copier が追加され、状況は少しはよくなったとはいえ、まだ問題が残る

そもそも、土砂崩れの危険があるような場所での監視は高価な PC ではなく安価な Raspberry Pi で行いたいと同様、Raspberry Pi のバックアップ&リストアのような危険な作業は PC ではなく RPi で行いたい

そこで、RPi を使って　RPi の SD カードのバックアップを取る方法を紹介する

といっても、現在 Linux を実行している SD カードをバックアップすることはできないので、バックアップ対象の Raspbian の SD カードを USB で外付けして、別の Raspbian を使ってバックアップをとる

##<u>実習手順</u>
自身の gc16 に terminal でログインする  

この gc16 で、Jessie Liete の SD カードのバックアップを取得する  

外付け SDカード reader/writer をつかって jl の SDカードを RPi の USB に装着する  
尚、jl (Raspbian Jessie Lite) は 2GB の SD カードであることを確認しておく


### Raspberry Pi による Raspberry Pi のバックアップ

1. jl の SD カードの使用状況の確認
```
pi@gc1624:~ $ df
Filesystem     1K-blocks    Used Available Use% Mounted on
/dev/root        3404364 2653828    588904  82% /
devtmpfs          469536       0    469536   0% /dev
tmpfs             473868       0    473868   0% /dev/shm
tmpfs             473868    6412    467456   2% /run
tmpfs               5120       4      5116   1% /run/lock
tmpfs             473868       0    473868   0% /sys/fs/cgroup
/dev/mmcblk0p1   3507840   25216   3482624   1% /boot
/dev/sda2        1814528  863108    850988  51% /media/usb0
/dev/sda1          63503   20725     42778  33% /media/usb1
/dev/mapper/i3    247791   78381    152310  34% /home/pi/SCRIPT
/dev/mapper/i4    247791  103988    126703  46% /var/www/html/SCRIPT
```

jl は FAT のブート用パーティションが /dev/sda1 として /media/usb1 にマウントされており、ext4  の linux ファイルシステムが /dev/sda2 として　/media/usb0 にマウントされている
尚、FAT と ext4 のどちらが usb0 でどちらが /usb1 になるかはタイミング次第のようである  
マウントされているので通常のファイルシステムと同様に読み書きすることができる

```
pi@gc1624:~ $ ls /media/usb0
bin   dev  home  lost+found  mnt  proc  run   srv  tmp  var
boot  etc  lib   media       opt  root  sbin  sys  usr
pi@gc1624:~ $ ls /media/usb1
bcm2708-rpi-0-w.dtb     bootcode.bin   fixup_x.dat       start_cd.elf
bcm2708-rpi-b.dtb       cmdline.txt    issue.txt         start_db.elf
bcm2708-rpi-b-plus.dtb  config.txt     kernel7.img       start.elf
bcm2708-rpi-cm.dtb      COPYING.linux  kernel.img        start_x.elf
bcm2709-rpi-2-b.dtb     fixup_cd.dat   LICENCE.broadcom
bcm2710-rpi-3-b.dtb     fixup.dat      LICENSE.oracle
bcm2710-rpi-cm3.dtb     fixup_db.dat   overlays
```

2. この jl のバックアップを gc16 の /boot/DATA の下に取得する  
現在の /boot の使用状況は 3GByte 以上空きがある

```
pi@gc1624:~ $ df /boot
Filesystem     1K-blocks  Used Available Use% Mounted on
/dev/mmcblk0p1   3507840 25216   3482624   1% /boot
```

/boot への書き込みは root 権限が必要なので sudo su で root になる

```
pi@gc1624:~ $ sudo su
root@gc1624:/home/pi#
```


3. dd コマンドで /dev/sda (外付けSDカード全体)のバイトイメージを取得し、gzip で圧縮して、/boot/DATA/js.gz ファイルに保存する  
--fast は圧縮率は少し甘いが圧縮が高速（といっても PC 程早くはない）

PC と RPi とでの速度比較の記事が[こちら](http://qiita.com/UedaTakeyuki/items/4a78c38b3070e8ef57e5)にある


```
root@gc1624:/home/pi# dd if=/dev/sda bs=1M | /bin/gzip --fast > /boot/DATA/jl.gz
```

4. バックアップが終了するまで約 9分ほどかかる。出来たバックアップファイルのサイズを見る

```
root@gc1624:/home/pi# ls -la /boot/DATA/jl.gz
-rwxr-xr-x 1 root root 1151548322 Mar 18 12:38 /boot/DATA/jl.gz
```

約1GByte ある。使用状況が半分程度の 2G Byte の SD のイメージファイルを圧縮して 1GByte とは大きすぎるのではないか？  

実は、SD カードの未使用領域は 0クリアされていないのでランダムなゴミが詰まっている。そこで、バックアップを取る前に以下のように jl の ext4 の未使用領域をゼロクリアする  
この処理は ４分30秒程かかる

```
root@gc1624:/media/usb1# cd /media/usb0
root@gc1624:/media/usb0# ls
bin   dev  home  lost+found  mnt  proc	run   srv  tmp	var
boot  etc  lib	 media	     opt  root	sbin  sys  usr
root@gc1624:/media/usb0# cat /dev/zero > boo
cat: write error: No space left on device
```

これで、中身が全て0の巨大なファイル boo で占められたので、boo を削除する

```
root@gc1624:/media/usb0# rm boo
```


5. 再度バックアップをとる。cd コマンドで /media/usb0 （バックアップ対象）から抜けて、先程と同じ dd コマンドを実行する。  
```
root@gc1624:/media/usb0# cd
root@gc1624:~# dd if=/dev/sda bs=1M | /bin/gzip --fast > /boot/DATA/jl.gz
```

6. 今度は 5分ぐらいで終わる。出来たバックアップファイルのサイズを見る
```
root@gc1624:~# ls -l /boot/DATA/jl.gz
-rwxr-xr-x 1 root root 403016699 Mar 18 13:17 /boot/DATA/jl.gz
```
今度は、400MByte ぐらいに減っている

### 取得したバックアップファイルの PC での操作
gc16 の /boot/DATA は PC からも読み書きできる領域なので、取得した jl.gz を PC にコピーする

1. RPi を停止
2. 外付け SD カード reader/writer から jl の SD カードを抜いてケースにしまう
3. gc16 の SD カードを RPi から抜いて、外付け SD カード reader/writer に装着
4. 外付け SD カード reader/writer を PC に挿す
5. boot という名前のデバイスが見えるのでファイルマネージャーで開く。DATA の下に jl.gz がある

### Raspberry Pi による Raspberry Pi のリストア
1. 外付け SD カード reader/writer に jl のカードを挿し、RPi の USB port に装着
2. RPi を gc16 で再起動
3. 先程と同様に df コマンドで /dev/sda が出来ている事を確認する

```
pi@gc1624:~ $ df
Filesystem     1K-blocks    Used Available Use% Mounted on
/dev/root        3404364 2656820    585912  82% /
devtmpfs          469536       0    469536   0% /dev
tmpfs             473868       0    473868   0% /dev/shm
tmpfs             473868   12336    461532   3% /run
tmpfs               5120       4      5116   1% /run/lock
tmpfs             473868       0    473868   0% /sys/fs/cgroup
/dev/mmcblk0p1   3507840 1543376   1964464  44% /boot
/dev/sda2        1814528  863108    850988  51% /media/usb0
/dev/sda1          63503   20725     42778  33% /media/usb1
/dev/mapper/i3    247791   78381    152310  34% /home/pi/SCRIPT
/dev/mapper/i4    247791  103988    126703  46% /var/www/html/SCRIPT
pi@gc1624:~ $
```

4. リストアをする前に、外付け SD の FAT を加えておく（実習で、バックアップ＆リストアの効果を見る為）

```
root@gc1624:/media/usb1# ls
a.txt			bcm2710-rpi-cm3.dtb  fixup_db.dat      overlays
bcm2708-rpi-0-w.dtb	bootcode.bin	     fixup_x.dat       start_cd.elf
bcm2708-rpi-b.dtb	cmdline.txt	     issue.txt	       start_db.elf
bcm2708-rpi-b-plus.dtb	config.txt	     kernel7.img       start.elf
bcm2708-rpi-cm.dtb	COPYING.linux	     kernel.img        start_x.elf
bcm2709-rpi-2-b.dtb	fixup_cd.dat	     LICENCE.broadcom
bcm2710-rpi-3-b.dtb	fixup.dat	     LICENSE.oracle
root@gc1624:/media/usb1# cat a.txt
hello
```

4. リストア。js.gz を解凍しながら dd で SDカード(/dev/sda)に書く
```
root@gc1624:/media/usb1#
root@gc1624:~# gzip  -dc /boot/DATA/jl.gz | sudo /bin/dd of=/dev/sda bs=1M
```

これも 5分ほどかかる  
正常にリストアできて、a.txt がなくなっていることを確認する

```
root@gc1624:~# ls /media/usb1
bcm2708-rpi-0-w.dtb	bootcode.bin   fixup_x.dat	 start_cd.elf
bcm2708-rpi-b.dtb	cmdline.txt    issue.txt	 start_db.elf
bcm2708-rpi-b-plus.dtb	config.txt     kernel7.img	 start.elf
bcm2708-rpi-cm.dtb	COPYING.linux  kernel.img	 start_x.elf
bcm2709-rpi-2-b.dtb	fixup_cd.dat   LICENCE.broadcom
bcm2710-rpi-3-b.dtb	fixup.dat      LICENSE.oracle
bcm2710-rpi-cm3.dtb	fixup_db.dat   overlays
```

### (任意) gc15, gc16 の相互バックアップ
gc15, gc16 共に、/boot/DATA には相手のバックアップを取ることができる程度の空きがあるので、交互にバックアップをとってみる。おそらく30分以上かかるので、昼休み等に

### （任意）高速バックアップ
圧縮に時間がかかっているので、圧縮せずに、2G Byte の SD カード全体を 2G Byte のファイルにバックアップをとることができる

  - バックアップ：dd if=/dev/sda of=/boot/DATA/js.img
  - リストア：dd if=/boot/DATA/js.img of=/dev/sda bs=1M

dd コマンドそのものである

###（任意）xz によるバックアップ
gzip は圧縮率よりも実行時間を優先したアルゴリズムと言われている。圧縮率を優先した xz も用意してあるので、xz を使ってバックアップ&リストアをおこなってみる。恐ろしく時間がかかるものの圧縮率は少しあがる

  - バックアップ：dd if=/dev/sda bs=1M | /usr/bin/xz > /boot/DATA/jl.xz
  - リストア：/usr/bin/xz -dc /boot/DATA/jl.xz | sudo /bin/dd of=/dev/sda bs=1M

##<u>ポイント解説</u>
1. Windows や Mac でバックアップをとる場合は、バックアップ対象が動いている状態で空き領域を一旦 /dev/zero で埋めれば、おなじようにバックアップサイズが劇的に小さくなる
2. 一旦クリーンアップした未使用領域もつかっているうちにだんだんゴミがたまってきて、やはりクリーンアップが有効になる
3. gc15, gc16 は NTFS, HFS をサポートしている（ドライバをインストールしている）ので、USB 外付けの HDD を廃棄前に 0 クリアするのにも利用出来る
4. dd, 1M などの魔法の言葉の解説は[こちら](http://qiita.com/UedaTakeyuki/items/4f97240b5fd7fc8a4d0b)
