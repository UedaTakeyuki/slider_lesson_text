# 1. 完成品の動作確認

## <u>目的</u>
先ず完成品の動作を確認し、システムの全体像を把握する

システムが以下の要素で構成されていること、およびそれらの関係を理解する

- データを取得する端末
- データの通知を受けるサーバ
- サーバに蓄積されたデータを閲覧するブラウザ

## <u>実習手順</u>

### 端末の起動
Raspberry Pi には起動スイッチ等はない  
Micro USB や GPIO を通じて 5V の電力を供給すると起動する

1. USB ケーブルで RPi に給電する

  1. A端子を PC に接続する  
  1. マイクロ B 端子を RPi に接続する  
  1. 正常に給電が開始されると RPi の赤い LED が点灯し、SDカードへのアクセスに応じて緑の LED が点滅する

2. Raspberry Pi が正常に起動すると boot シーケンスからslider アプリケーションが起動されて LCD の表示がはじまる

  1. 時計として時刻を表示する  
  ただし、RPi は RTC を持たず、表示される日時は正しくない
  2. 毎時30秒付近で以下を数秒ずつ表示する

    - 自身に割り当てられた IP アドレス
    - 現在の気温
    - 現在の湿度

  3. それらをスピーカーから合成音声で読み上げる  
<img src="pic/ss.2017-04-17 20.11.44.png" width="75%">


<a name="setdatetime"/>

### 時刻の設定


slider の Web アプリケーションを使って PC の時刻で Raspberr Pi の時刻を合わせる

1. PC で chrome を開き、IP アドレスを指定して自分の Raspberry Pi に接続する  
IPアドレスは[こちら](classenvironment.md)で確認できる。以下の画面になる  

<img src="pic/ss.2017-03-08 21.01.34.png" width="75%">

2. Set DateTime を開く。真っ白な画面が開くのと同時に Raspberry Pi の LCD の時刻が PC の時刻と同期する

3. (任意：時刻が同期される理由の確認) chrome の「検証」を開き、Source タブで "dt.php" のコードを開く。このコードは Ajax でクライアントの時刻をサーバに POST している。

<a name="monitor"/>
### サーバの確認
1. PC で chrome を開き、IP アドレスを指定して自分の Raspberry Pi が所属するネットワークのサーバーに接続する。サーバーの IP アドレスは[こちら](classenvironment.md)で確認できる。以下の画面になる  

<img src="pic/ss.2017-03-08 21.06.35.png" width="75%">

2. monitor 配下の自分の Raspberry Pi のホスト名をクリック   
下記のログイン画面が開く  

<img src="pic/ss.2017-03-08 21.06.47.png" width="75%">

初期 ID と PW　は以下

- ID: g4
- PW: g4

3. ログインすると自分の Raspberry Pi のデータを表示するページが開く  

<img src="pic/ss.2017-03-08 21.07.06.png" width="75%">

1分間隔でデータが更新される  

4. （任意）上記画面の「設定変更」ボタンでログインパスワードと、グラフの表示件数が変更できる  
　
<img src="pic/ss.2017-03-08 21.07.20.png" width="75%">

### (任意) BYOD (Bring Your Own Device)の利用
私物スマフォ を教室内のネットワークに参加させ、上記ブラウザ操作を PC からではなく自身のスマフォで行ってもよい  
接続のための SSID と Pass Key は [こちら](classenvironment.md)で確認できる。


## <u>ポイント解説</u>
1. 正常に起動しない理由として最も多いのが SD カードが緩み。この場合は緑のLEDが点滅しない、もしくは点灯したまま点滅しない
2. RPi は PC のようなバッテリー付きの内部時計(RTC: Real Time Clock) を持たない。通常はネットワークの先の NTP(Network Time Protocol) Server から時刻情報を受け取って補正するのだが、NTP Server に接続することのできない環境では、前回の終了時の時刻になってしまう。
3. 原始的な時刻補正だが、ネットワークが近いので小さい遅延で時刻の同期が出来る。RTC も NTP も使えない環境では便利
