# 3.Linux の基本操作（任意）

##<u>概要</u>
コマンドラインで login した後の作業は基本的に下記の３点になる

1. ファイルの操作（参照、編集、移動、削除、作成、実行）
2. ディレクトリの移動
3. コマンドの実行

以降の章での実習でも必要になるので、コマンドラインでの操作に自身がない場合は本章の実習の実施を進める。  
コマンドラインでの作業に精通している場合、本章の実習は不要

##<u>実習手順</u>
自身の gc16 に terminal でログインする

### ディレクトリの移動 と一覧の表示
1. `pwd` コマンドで、現在作業しているディレクトリ（Current Working Directroy）が以下のように表示される

```
pi@gc1624:~ $ pwd
/home/pi
```

現在、作業しているディレクトリは `pi` で、ディレクトリツリーの頂点(`/`ルート)から `pi` までの pass は `/home/pi` になる  


2. `ls` コマンドで、現在のディレクトリ配下のファイルの一覧の表示。`ls -l`で、詳細情報付きで表示
```
drwxrwxr-x  5 root pi      1024 Feb 27 21:55 SCRIPT
-rw-r--r--  1 pi   pi       951 Nov 22 21:59 ssd1306.py
```

詳細情報の意味は先頭ブロックから順番に以下

|位置|意味|
|:--:|:--:|
|ファイルの権限|詳細は以下|
|ファイルのハードリンク数|ファイルシステム内で、リンクされている数。Linux ではファイルは複数箇所からリンクできる|
|所有者のid|ファイルを操作するユーザがこのidをもつ時、下記の権限で所有者の権限が付与される|
|グループのid|ファイルを操作するユーザがこのidを持つ時、下記の権限でグループの権限が付与される|
|サイズ|ファイルのサイズ|
|更新日|ファイルの更新日|
|ファイル名|ファイルの名前|



先頭の `drwxrwxr-x` や `-rw-r--r--` 等の権限の意味は以下  

|位置|権限|意味|
|:--:|:--:|:--:|
|1文字目|ファイルの種類|d:ディレクトリ、-:通常のファイル|
|2文字目|オーナーの読み取り権限|r:読み取り可、-:読み取り不可|
|3文字目|オーナーの書き込み権限|w:書込み可、-:書込み不可|
|4文字目|オーナーの実行権限|x:実行可、-:実行不可|
|5文字目|グループの読み取り権限|r:読み取り可、-:読み取り不可|
|6文字目|グループの書き込み権限|w:書込み可、-:書込み不可|
|7文字目|グループの実行権限|x:実行可、-:実行不可|
|8文字目|一般ユーザーの読み取り権限|r:読み取り可、-:読み取り不可|
|9文字目|一般ユーザーの書き込み権限|w:書込み可、-:書込み不可|
|10文字目|一般ユーザーの実行権限|x:実行可、-:実行不可|

権限は、オーナー、グループ、一般ユーザーの or で付与される。例えば `pi` アカウントでログインしている場合、上記、`ssd1306.py` によって `pi` は所有者でありかつ所属グループである。ファイルの書き込み権限はオーナとしては書き込み可(1)、グループとしては不可(0)なので、or を取って書き込み可(1)の権限が付与される

SCRIPT の先頭の `drwxrwxr-x`で、d は SCRIPT がフォルダ（ディレクトリファイル）であることを示す。以下は３文字づつ所有者の権限(rwx)、グループの権限、(rwx)、その他ユーザの権限(r-x)を表す

3. `cd` コマンドでディレクトリを SCRIPT の中に移動し、ファイルの一覧を見る
```
pi@gc1624:~ $ cd SCRIPT/
pi@gc1624:~/SCRIPT $ pwd
/home/pi/SCRIPT
pi@gc1624:~/SCRIPT $ ls -l
total 16
drwx------ 2 root root 12288 Nov 12 20:59 lost+found
drwxr-xr-x 6 pi   pi    3072 Mar  8 17:11 slider
drwxr-xr-x 3 pi   pi    1024 Feb 27 22:22 wvdial
```
slider というフォルダがあるので、この下に入り、ファイルの一覧を見る
```
pi@gc1624:~/SCRIPT $ cd slider/
pi@gc1624:~/SCRIPT/slider $ ls
```

`cd` は change directory という意味

4. 上のディレクトリに戻る
```
pi@gc1624:~/SCRIPT/slider $ cd ..
pi@gc1624:~/SCRIPT $ cd ..
pi@gc1624:~ $ pwd
/home/pi
```

`..` が、一つ上のディレクトリを表す

5. ファイル名はタブキーで補完ができる。
例えば、`cd S` まで打った所でタブキーを押すと `cd cd SCRIPT/` とファイル名を補完してくれる

### ファイルの作成
1. nano エディタを使い、ファイルを作成する

```
cd /home/pi
nano a.txt
```

以下のような画面になる

<img src="pic/ss.2017-03-17 21.09.51.png" width="75%">

なんでもいいので文字列を入力してみる

<img src="pic/ss.2017-03-17 21.34.09.png" width="75%">

下の欄にはコマンドが表示されている。`^`は `contl +` の意味。`contl + G` で HELP ファイルが表示される。HELP の表示を終了して元に戻るには `contl + X`  
編集の終了は `contl + X` 保存するかどうか聞かれるので　`y` と答える  
<img src="pic/ss.2017-03-17 21.41.39.png" width="75%">

、保存するファイル名を再確認されるので、そのままでよければそのまま改行する  
<img src="pic/ss.2017-03-17 21.41.56.png" width="75%">
ここで別のファイル名を入力すると別名で保存される

### ファイルの簡単な作成
echo コマンドで文字列を表示できる。表示先をファイル名にリダイレクトするとファイルに保存される

1. `echo abc` と入力。abc と表示される
2. `echo abc > c.txt` と入力。なにも表示されない
3. `ls` c.txt が作成されている

### ファイルの編集
先ほど作成した a.txt ファイルを再度開く
```
nano a.txt
```

3行目を削除する。カーソルキーで３行目に移動し、`contl + K` で行を削除する  
カット&ペーストのペーストは `contl + U`
指定行への移動は `contl + -`。文字列の検索は `contl + W`

### ファイルのコピー
`cp a.txt b.txt` で、ファイル a.txt を b.txt にコピー

### ファイルの表示
`cat a.txt` で　a.txt を表示  
尚、コマンド名の cat は `concatenate` の短縮で、本来このコマンドは複数のファイルを結合したものを表示する  
`cat a.txt b.txt` で、両者を結合したファイルの表示

### ファイルの削除
`rm a.txt b.txt` で、ファイルa.txt と b.txt 削除

### sudo
`sudo` の名の由来は `substitute user do` だが、よく`superuser do`と呼ばれるとおり、コマンドをスーパーユーザーの権限で実行する。sudo は許可されたユーザーのみが使うことができ、`pi`ユーザは`sudo`を許可されている

1. `cd /boot` で boot ディレクトリに移動
```
pi@gc1624:~ $ cd /boot
pi@gc1624:/boot $
```
2. `echo abc > c.txt` を実行しても、権限がないためコマンドの実行を拒否される
```
pi@gc1624:/boot $ echo abc > c.txt
-bash: c.txt: Permission denied
```
3. `sudo sh -c 'echo abc > c.txt'` を実行。c.txt ファイルが作成される
```
pi@gc1624:/boot $ sudo sh -c 'echo abc > c.txt'
pi@gc1624:/boot $ ls
bcm2708-rpi-b.dtb       COPYING.linux       FSCK0001.REC      LICENSE.oracle
bcm2708-rpi-b-plus.dtb  crontab.slider.txt  gc_cid            overlays
bcm2708-rpi-cm.dtb      c.txt               gc.ini            sc.txt
bcm2709-rpi-2-b.dtb     DATA                gc_issue.txt      sliders.tiff
bcm2710-rpi-3-b.dtb     fixup_cd.dat        gc_log.txt        start_cd.elf
bcm2710-rpi-cm3.dtb     fixup.dat           issue.txt         start_db.elf
bootcode.bin            fixup_db.dat        kernel7.img       start.elf
cmdline.txt             fixup_x.dat         kernel.img        start_x.elf
config.txt              FSCK0000.REC        LICENCE.broadcom
pi@gc1624:/boot $ sudo rm c.txt
```
`sudo echo abc > c.txt` としなかったのは、sudo がリダイレクトの先までに及ばないため、sh コマンドで別の shell を起こし、その全体に sudo を適用した。
