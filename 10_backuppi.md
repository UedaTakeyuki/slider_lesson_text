# 10.BackupPi と拡張 boot 領域
##<u>目的</u>
先の実習でおこなったバックアップ＆リストアだが、Web UI をかぶせて Web から操作できるようにしておくと本当に便利になる。この Web UI は BackupPi として[こちら](https://github.com/UedaTakeyuki/BackupPi_2)にコードを[こちら](http://qiita.com/UedaTakeyuki/items/4a78c38b3070e8ef57e5)に記事を公開している  
このぐらい気楽にバックアップがとれるようになると、開発作業中に頻繁にバックアップをとるようになる  

##<u>実習手順</u>
gc16 で RPi を起動  
外付け SDカード reader/writer をつかって jl の SDカードを RPi の USB に装着する  

### BackupPi でバックアップ

1. PC で chrome を開き、自分の Raspberry Pi に Web で接続する。下記の画面になる

<img src="pic/ss.2017-03-08 21.01.34.png" width="75%">

2. `BackupPi_2` をクリックすると下記の画面になる

<img src="pic/ss.2017-03-24 18.06.34.png" width="75%">

2. 保存先ファイル名を  jl.gz に変更して `バックアップ開始`をクリック
<img src="pic/ss.2017-03-24 18.06.34.png" width="75%">

4. バックアップの進捗を定期的に報告
<img src="pic/ss.2017-03-24 18.07.12.png" width="75%">

5. バックアップ完了
<img src="pic/ss.2017-03-24 18.27.00.png" width="75%">

### 取得したバックアップファイルの PC での操作
バックアップは gc16 sdカードの ***拡張boot領域*** /boot 配下の /boot/DATA に保存されているので、PC から読み書きできる

### BackupPi でバックアップ

1. PC で chrome を開き、自分の Raspberry Pi に Web で接続する。下記の画面になる

<img src="pic/ss.2017-03-08 21.01.34.png" width="75%">

2. `BackupPi_2` をクリックすると下記の画面になる

<img src="pic/ss.2017-03-24 18.06.34.png" width="75%">

3. `リストア`タブをクリック

<img src="pic/ss.2017-03-24 18.06.34.png" width="75%">

4. `復元ファイル選択`リストから復元するバックアップファイルを選択し、`リストア開始`をクリック

<img src="pic/ss.2017-03-24 18.28.18.png" width="75%">

4. リストアの進捗を定期的に報告
<img src="pic/ss.2017-03-24 18.28.30.png" width="75%">

5. リストア完了
<img src="pic/ss.2017-03-24 18.34.41.png" width="75%">
