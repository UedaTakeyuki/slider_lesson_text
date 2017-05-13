# 30.Ajax API の作り方

##<u>概要</u>
ajax API はレスポンス文字列が HTML ではなく json 文字列であることを除けば通常の Web Server と同じであり、ajax API を作るための特別のミドルウェアは不要で php 等で普通に作成できる  
例として、先ほど作成した gpio の一覧を返す Web Application の ajax 版を作成する

##<u>実習手順</u>
自身の gc16 に terminal でログインする

1. `/var/www/html/gpio` に移動
```
pi@gc1624:~ $ cd /var/www/html/gpio
```

2. テキストエディタで ajax.php を以下の内容で作成する

```
<?php
if(isset($_GET['gpio'])) {
    $pin = $_GET['gpio'];
    $gpio['gpio_'.$pin] = substr(`sudo gpio read $pin`, 0, -1);
} else {
    for ($pin = 1; $pin < 30; $pin++){
    $gpio['gpio_'.$pin] = substr(`sudo gpio read $pin`, 0, -1);
    }
}

$json_str = json_encode($gpio);

header('Content-Type: application/json');

echo $json_str;
exit;
```  
ポイント
  - javascript から GET で呼び出す設計の ajax.API
  - ajax からのパラメタは GET のパラメタとして受け取ることができる
  - レスポンスヘッダ + json 文字列をレスポンスとして返す

3. ブラウザで確認  
URL は`自分の RPi のホスト名/gpio/ajax.php`  
<img src="pic/ss.2017-04-05 16.05.59.png" width="90%">  
また、`?gpio=`のパラメタを指定して、特定の gpio の値のみ受け取ることもできる
<img src="pic/ss.2017-04-05 16.09.53.png" width="90%">  

4. gc16 の terminal から curl と jq を使って接続する

```
pi@gc1624:/var/www/html/gpio $ curl localhost/gpio/ajax.php?gpio=2
{"gpio_2":"0"}

pi@gc1624:/var/www/html/gpio $ curl localhost/gpio/ajax.php | jq .
  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed
100   398    0   398    0     0    215      0 --:--:--  0:00:01 --:--:--   215
{
  "gpio_1": "0",
  "gpio_2": "0",
  "gpio_3": "0",
  "gpio_4": "0",
  "gpio_5": "0",
  "gpio_6": "0",
  "gpio_7": "1",
  "gpio_8": "1",
  "gpio_9": "1",
  "gpio_10": "1",
  "gpio_11": "1",
  "gpio_12": "0",
  "gpio_13": "0",
  "gpio_14": "0",
  "gpio_15": "1",
  "gpio_16": "0",
  "gpio_17": "0",
  "gpio_18": "1",
  "gpio_19": "0",
  "gpio_20": "0",
  "gpio_21": "1",
  "gpio_22": "1",
  "gpio_23": "0",
  "gpio_24": "0",
  "gpio_25": "0",
  "gpio_26": "0",
  "gpio_27": "0",
  "gpio_28": "0",
  "gpio_29": "1"
```
