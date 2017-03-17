# Raspbeery Piの操作（任意）

##<u>概要</u>
コマンドラインで login した後の作業は基本的に下記の３点になる

1. ファイルの操作（参照、編集、移動、削除、作成、実行）
2. ディレクトリの移動
3. コマンドの実行

以降の章での実習でも必要になるので、コマンドラインでの操作に自身がない場合は本章の実習の実施を進める。  
日常的にコマンドラインでの作業に精通している場合、本章の実習は不要

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
SCRIPT の先頭の `drwxrwxr-x`で、d は SCRIPT がフォルダ（ディレクトリファイル）であることを示す。以下は３文字づつ所有者の権限(rwx)、グループの権限、(rwx)、その他ユーザの権限(r-x)を表す

3. ディレクトリを SCRIPT の中に移動し、ファイルの一覧を見る
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

### ファイルの編集
先ほど作成した a.txt ファイルを再度開く
```
nano a.txt
```

3行目を削除する。カーソルキーで３行目に移動し、`contl + K` で行を削除する  
カット&ペーストのペーストは `contl + U`
指定行への移動は `contl + -`。文字列の検索は `contl + W`
