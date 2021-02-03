# 13.GPIO

## <u>目的</u>
GPIOの操作について習得する。Linux では gpio は /sys から shell で操作することができるのだが、Python 等の言語環境から利用する便利なライブラリも多数用意されているので、好きな物を使えばよいのだが、Raspberry Pi に特化したライブラリとして `Wiring Pi` も便利

## <u>実習手順</u>
自身の gc16 に terminal でログインする

### gpio readall
順番が sysfs と前後してしまう（sysfs のほうが原理的）のだが、便利なツールなので最初に紹介する。Wiring Pi をインストールすると一緒に幾つかの utility がインストーされ、`gpio` コマンドが利用できるようになる。  
`gpio` コマンドの `readall` オプションで、現在の gpio の全ての状態が表示される

```
pi@gc1624:~ $ gpio readall
 +-----+-----+---------+------+---+---Pi 3---+---+------+---------+-----+-----+
 | BCM | wPi |   Name  | Mode | V | Physical | V | Mode | Name    | wPi | BCM |
 +-----+-----+---------+------+---+----++----+---+------+---------+-----+-----+
 |     |     |    3.3v |      |   |  1 || 2  |   |      | 5v      |     |     |
 |   2 |   8 |   SDA.1 | ALT0 | 1 |  3 || 4  |   |      | 5V      |     |     |
 |   3 |   9 |   SCL.1 | ALT0 | 1 |  5 || 6  |   |      | 0v      |     |     |
 |   4 |   7 | GPIO. 7 |   IN | 1 |  7 || 8  | 1 | OUT  | TxD     | 15  | 14  |
 |     |     |      0v |      |   |  9 || 10 | 0 | OUT  | RxD     | 16  | 15  |
 |  17 |   0 | GPIO. 0 |   IN | 0 | 11 || 12 | 0 | IN   | GPIO. 1 | 1   | 18  |
 |  27 |   2 | GPIO. 2 |   IN | 0 | 13 || 14 |   |      | 0v      |     |     |
 |  22 |   3 | GPIO. 3 |   IN | 0 | 15 || 16 | 0 | IN   | GPIO. 4 | 4   | 23  |
 |     |     |    3.3v |      |   | 17 || 18 | 0 | IN   | GPIO. 5 | 5   | 24  |
 |  10 |  12 |    MOSI | ALT0 | 0 | 19 || 20 |   |      | 0v      |     |     |
 |   9 |  13 |    MISO | ALT0 | 0 | 21 || 22 | 0 | IN   | GPIO. 6 | 6   | 25  |
 |  11 |  14 |    SCLK | ALT0 | 0 | 23 || 24 | 1 | OUT  | CE0     | 10  | 8   |
 |     |     |      0v |      |   | 25 || 26 | 1 | OUT  | CE1     | 11  | 7   |
 |   0 |  30 |   SDA.0 |   IN | 1 | 27 || 28 | 1 | IN   | SCL.0   | 31  | 1   |
 |   5 |  21 | GPIO.21 |   IN | 1 | 29 || 30 |   |      | 0v      |     |     |
 |   6 |  22 | GPIO.22 |   IN | 1 | 31 || 32 | 0 | IN   | GPIO.26 | 26  | 12  |
 |  13 |  23 | GPIO.23 |   IN | 0 | 33 || 34 |   |      | 0v      |     |     |
 |  19 |  24 | GPIO.24 |   IN | 0 | 35 || 36 | 0 | IN   | GPIO.27 | 27  | 16  |
 |  26 |  25 | GPIO.25 |   IN | 0 | 37 || 38 | 0 | IN   | GPIO.28 | 28  | 20  |
 |     |     |      0v |      |   | 39 || 40 | 0 | IN   | GPIO.29 | 29  | 21  |
 +-----+-----+---------+------+---+----++----+---+------+---------+-----+-----+
 | BCM | wPi |   Name  | Mode | V | Physical | V | Mode | Name    | wPi | BCM |
 +-----+-----+---------+------+---+---Pi 3---+---+------+---------+-----+-----+
```

最下行に RPi のボードのバージョンが表示され、その上は各ピンの ***名前*** と ***モード*** と ***値*** が表示されている

値は `V` 欄に 0 か 1 かで表示されている  

次に `Mode` だが、GPIO は、***インプットモード***、***アウトプットモード***、***altinative function mode*** のモードを持つ  

***インプットモード*** は、接続先の状態を受け入れるモードで、接続先が 0 になれば gpio も 0になる  
***アウトプットモード*** は、接続先の状態を変化させるモードで、gpio が 0 になればつながった先も 0 になる  
***アルタナティブファンクションモード*** は、gpio を単に接点の on/off に使うのではなく、別の機能（後に説明する i2c や SPI等）に割り当てているモードで、複数個の alt モードのアサインが可能なので `alt0` や `alt1` などと区別する。

混乱しているのが名前で、`BCM`、`wPi`、`Name`、`Physical` の ***４種類の名前*** が並記されている

`Phisical` は Pin 番号で、Raspberry Pi の場合は USB ポートがある方を下にして、トップ左を 1番、右を2番、下に下がって... と → ↓ の準に数える  

`Name` は、Raspberry Pi の GPIO番号か、もしくは ALT0モードの機能名

`wPi` は Raspberry Pi の GPIO番号と整合する GPIO 番号  
`BCM` は Broadcom2835内部の GPIO番号（後に説明する `/sys` に export する番号）  

### sysfs からの利用
Wiring Pi 等のライブラリがなにもインストールされていない Raspberry Pi であっても `/sys/class/gpio` を操作することで GPIO の読み書きは可能

1. gpio を/sys経由で使っていない場合、gpio フォルダは下記のようになっている
```
pi@gc1624:~ $ ls /sys/class/gpio
export  gpiochip0  gpiochip100  unexport
```

2. 例えば 27番のgpio(BCM27, wPi2, GPIO.0) を利用する場合、以下のように export させる
```
pi@gc1624:~ $ echo 27 > /sys/class/gpio/export
```
すると、27 が以下のように /sys/call/gpio に export される
```
pi@gc1624:~ $ ls /sys/class/gpio
export  gpio27  gpiochip0  gpiochip100  unexport
```

3. GPIO の値は value ファイルで参照できる

```
pi@gc1624:~ $ cat /sys/class/gpio/gpio27/value
0
```

この時、`gpio readall`で確認すると BCM27 は 0

<pre><code style="font-size: 75%">
pi@gc1624:~ $ gpio readall
 +-----+-----+---------+------+---+---Pi 3---+---+------+---------+-----+-----+
 | BCM | wPi |   Name  | Mode | V | Physical | V | Mode | Name    | wPi | BCM |
 +-----+-----+---------+------+---+----++----+---+------+---------+-----+-----+
 |     |     |    3.3v |      |   |  1 || 2  |   |      | 5v      |     |     |
 |   2 |   8 |   SDA.1 | ALT0 | 1 |  3 || 4  |   |      | 5V      |     |     |
 |   3 |   9 |   SCL.1 | ALT0 | 1 |  5 || 6  |   |      | 0v      |     |     |
 |   4 |   7 | GPIO. 7 |   IN | 1 |  7 || 8  | 1 | OUT  | TxD     | 15  | 14  |
 |     |     |      0v |      |   |  9 || 10 | 0 | OUT  | RxD     | 16  | 15  |
 |  17 |   0 | GPIO. 0 |   IN | 0 | 11 || 12 | 0 | IN   | GPIO. 1 | 1   | 18  |
 |  27 |   2 | GPIO. 2 |   IN | 0 | 13 || 14 |   |      | 0v      |     |     |
 |  22 |   3 | GPIO. 3 |   IN | 0 | 15 || 16 | 0 | IN   | GPIO. 4 | 4   | 23  |
 |     |     |    3.3v |      |   | 17 || 18 | 0 | IN   | GPIO. 5 | 5   | 24  |
 |  10 |  12 |    MOSI | ALT0 | 0 | 19 || 20 |   |      | 0v      |     |     |
 |   9 |  13 |    MISO | ALT0 | 0 | 21 || 22 | 0 | IN   | GPIO. 6 | 6   | 25  |
 |  11 |  14 |    SCLK | ALT0 | 0 | 23 || 24 | 1 | OUT  | CE0     | 10  | 8   |
 |     |     |      0v |      |   | 25 || 26 | 1 | OUT  | CE1     | 11  | 7   |
 |   0 |  30 |   SDA.0 |   IN | 1 | 27 || 28 | 1 | IN   | SCL.0   | 31  | 1   |
 |   5 |  21 | GPIO.21 |   IN | 1 | 29 || 30 |   |      | 0v      |     |     |
 |   6 |  22 | GPIO.22 |   IN | 1 | 31 || 32 | 0 | IN   | GPIO.26 | 26  | 12  |
 |  13 |  23 | GPIO.23 |   IN | 0 | 33 || 34 |   |      | 0v      |     |     |
 |  19 |  24 | GPIO.24 |   IN | 0 | 35 || 36 | 0 | IN   | GPIO.27 | 27  | 16  |
 |  26 |  25 | GPIO.25 |   IN | 0 | 37 || 38 | 0 | IN   | GPIO.28 | 28  | 20  |
 |     |     |      0v |      |   | 39 || 40 | 0 | IN   | GPIO.29 | 29  | 21  |
 +-----+-----+---------+------+---+----++----+---+------+---------+-----+-----+
 | BCM | wPi |   Name  | Mode | V | Physical | V | Mode | Name    | wPi | BCM |
 +-----+-----+---------+------+---+---Pi 3---+---+------+---------+-----+-----+
</code></pre>

4. GPIO の値を 1 に変更する。下記のようにエラーになる
```
pi@gc1624:~ $ echo 1 > /sys/class/gpio/gpio27/value
-bash: echo: write error: Operation not permitted
pi@gc1624:~ $ sh -c "echo 1 > /sys/class/gpio/gpio27/value"
sh: echo: I/O error
```

direction ファイルを確認すると in になっている

```
pi@gc1624:~ $ cat /sys/class/gpio/gpio27/direction
in
```

そこで、direction を out に変更して再挑戦。今度は正常に書き変わる

```
pi@gc1624:~ $ sh -c "echo out > /sys/class/gpio/gpio27/direction"
pi@gc1624:~ $ cat /sys/class/gpio/gpio27/direction
out
pi@gc1624:~ $ sh -c "echo 1 > /sys/class/gpio/gpio27/value"
pi@gc1624:~ $ cat /sys/class/gpio/gpio27/value
1
```

ここで、再度　`gpio readall` で確認すると、下記のように BCM27 が OUTPUT モードで 1 に変わっている

<pre><code style="font-size: 75%">
+-----+-----+---------+------+---+---Pi 3---+---+------+---------+-----+-----+
| BCM | wPi |   Name  | Mode | V | Physical | V | Mode | Name    | wPi | BCM |
+-----+-----+---------+------+---+----++----+---+------+---------+-----+-----+
|     |     |    3.3v |      |   |  1 || 2  |   |      | 5v      |     |     |
|   2 |   8 |   SDA.1 | ALT0 | 1 |  3 || 4  |   |      | 5V      |     |     |
|   3 |   9 |   SCL.1 | ALT0 | 1 |  5 || 6  |   |      | 0v      |     |     |
|   4 |   7 | GPIO. 7 |   IN | 1 |  7 || 8  | 1 | OUT  | TxD     | 15  | 14  |
|     |     |      0v |      |   |  9 || 10 | 0 | OUT  | RxD     | 16  | 15  |
|  17 |   0 | GPIO. 0 |   IN | 0 | 11 || 12 | 0 | IN   | GPIO. 1 | 1   | 18  |
|  27 |   2 | GPIO. 2 |  OUT | 1 | 13 || 14 |   |      | 0v      |     |     |
|  22 |   3 | GPIO. 3 |   IN | 0 | 15 || 16 | 0 | IN   | GPIO. 4 | 4   | 23  |
|     |     |    3.3v |      |   | 17 || 18 | 0 | IN   | GPIO. 5 | 5   | 24  |
|  10 |  12 |    MOSI | ALT0 | 0 | 19 || 20 |   |      | 0v      |     |     |
|   9 |  13 |    MISO | ALT0 | 0 | 21 || 22 | 0 | IN   | GPIO. 6 | 6   | 25  |
|  11 |  14 |    SCLK | ALT0 | 0 | 23 || 24 | 1 | OUT  | CE0     | 10  | 8   |
|     |     |      0v |      |   | 25 || 26 | 1 | OUT  | CE1     | 11  | 7   |
|   0 |  30 |   SDA.0 |   IN | 1 | 27 || 28 | 1 | IN   | SCL.0   | 31  | 1   |
|   5 |  21 | GPIO.21 |   IN | 1 | 29 || 30 |   |      | 0v      |     |     |
|   6 |  22 | GPIO.22 |   IN | 1 | 31 || 32 | 0 | IN   | GPIO.26 | 26  | 12  |
|  13 |  23 | GPIO.23 |   IN | 0 | 33 || 34 |   |      | 0v      |     |     |
|  19 |  24 | GPIO.24 |   IN | 0 | 35 || 36 | 0 | IN   | GPIO.27 | 27  | 16  |
|  26 |  25 | GPIO.25 |   IN | 0 | 37 || 38 | 0 | IN   | GPIO.28 | 28  | 20  |
|     |     |      0v |      |   | 39 || 40 | 0 | IN   | GPIO.29 | 29  | 21  |
+-----+-----+---------+------+---+----++----+---+------+---------+-----+-----+
| BCM | wPi |   Name  | Mode | V | Physical | V | Mode | Name    | wPi | BCM |
+-----+-----+---------+------+---+---Pi 3---+---+------+---------+-----+-----+
</code></pre>

### gpio コマンド
gpio コマンドは Wiring Pi のコマンドラインインターフェース。gpio に関して出来る事は基本的に `/sys/class/gpio` を操作する場合と原理的に同じなのだが、記述がシンプルになるので便利  


gpio コマンドをつかって実用的な ***漏水センサー*** システムをつくってみる  
仕組みは、隣接する GPIO.27 と GPIO.28 に平行線を接続し、以下のように設定する

- GPIO.28 output モード、1
- GPIO.27 input モード、0

平行線の先端に水滴があたれば、水滴がスイッチとなり両 GPIO が接触する。結果、GPIO.27 の値が 0 から 1に変化する

1. GPIO28 を `output モード、1` に設定する
<pre><code style="font-size: 75%">
pi@gc1624:~ $ gpio mode 28 out
pi@gc1624:~ $ gpio readall
 +-----+-----+---------+------+---+---Pi 3---+---+------+---------+-----+-----+
 | BCM | wPi |   Name  | Mode | V | Physical | V | Mode | Name    | wPi | BCM |
 +-----+-----+---------+------+---+----++----+---+------+---------+-----+-----+
 |     |     |    3.3v |      |   |  1 || 2  |   |      | 5v      |     |     |
 |   2 |   8 |   SDA.1 | ALT0 | 1 |  3 || 4  |   |      | 5V      |     |     |
 |   3 |   9 |   SCL.1 | ALT0 | 1 |  5 || 6  |   |      | 0v      |     |     |
 |   4 |   7 | GPIO. 7 |   IN | 1 |  7 || 8  | 1 | OUT  | TxD     | 15  | 14  |
 |     |     |      0v |      |   |  9 || 10 | 0 | OUT  | RxD     | 16  | 15  |
 |  17 |   0 | GPIO. 0 |   IN | 0 | 11 || 12 | 0 | IN   | GPIO. 1 | 1   | 18  |
 |  27 |   2 | GPIO. 2 |   IN | 0 | 13 || 14 |   |      | 0v      |     |     |
 |  22 |   3 | GPIO. 3 |   IN | 0 | 15 || 16 | 0 | IN   | GPIO. 4 | 4   | 23  |
 |     |     |    3.3v |      |   | 17 || 18 | 0 | IN   | GPIO. 5 | 5   | 24  |
 |  10 |  12 |    MOSI | ALT0 | 0 | 19 || 20 |   |      | 0v      |     |     |
 |   9 |  13 |    MISO | ALT0 | 0 | 21 || 22 | 0 | IN   | GPIO. 6 | 6   | 25  |
 |  11 |  14 |    SCLK | ALT0 | 0 | 23 || 24 | 1 | OUT  | CE0     | 10  | 8   |
 |     |     |      0v |      |   | 25 || 26 | 1 | OUT  | CE1     | 11  | 7   |
 |   0 |  30 |   SDA.0 |   IN | 1 | 27 || 28 | 1 | IN   | SCL.0   | 31  | 1   |
 |   5 |  21 | GPIO.21 |   IN | 1 | 29 || 30 |   |      | 0v      |     |     |
 |   6 |  22 | GPIO.22 |   IN | 1 | 31 || 32 | 0 | IN   | GPIO.26 | 26  | 12  |
 |  13 |  23 | GPIO.23 |   IN | 0 | 33 || 34 |   |      | 0v      |     |     |
 |  19 |  24 | GPIO.24 |   IN | 0 | 35 || 36 | 0 | IN   | GPIO.27 | 27  | 16  |
 |  26 |  25 | GPIO.25 |   IN | 0 | 37 || 38 | 0 | OUT  | GPIO.28 | 28  | 20  |
 |     |     |      0v |      |   | 39 || 40 | 1 | IN   | GPIO.29 | 29  | 21  |
 +-----+-----+---------+------+---+----++----+---+------+---------+-----+-----+
 | BCM | wPi |   Name  | Mode | V | Physical | V | Mode | Name    | wPi | BCM |
 +-----+-----+---------+------+---+---Pi 3---+---+------+---------+-----+-----+
 pi@gc1624:~ $ gpio write 28 1
 pi@gc1624:~ $ gpio readall
  +-----+-----+---------+------+---+---Pi 3---+---+------+---------+-----+-----+
  | BCM | wPi |   Name  | Mode | V | Physical | V | Mode | Name    | wPi | BCM |
  +-----+-----+---------+------+---+----++----+---+------+---------+-----+-----+
  |     |     |    3.3v |      |   |  1 || 2  |   |      | 5v      |     |     |
  |   2 |   8 |   SDA.1 | ALT0 | 1 |  3 || 4  |   |      | 5V      |     |     |
  |   3 |   9 |   SCL.1 | ALT0 | 1 |  5 || 6  |   |      | 0v      |     |     |
  |   4 |   7 | GPIO. 7 |   IN | 1 |  7 || 8  | 1 | OUT  | TxD     | 15  | 14  |
  |     |     |      0v |      |   |  9 || 10 | 0 | OUT  | RxD     | 16  | 15  |
  |  17 |   0 | GPIO. 0 |   IN | 0 | 11 || 12 | 0 | IN   | GPIO. 1 | 1   | 18  |
  |  27 |   2 | GPIO. 2 |   IN | 0 | 13 || 14 |   |      | 0v      |     |     |
  |  22 |   3 | GPIO. 3 |   IN | 0 | 15 || 16 | 0 | IN   | GPIO. 4 | 4   | 23  |
  |     |     |    3.3v |      |   | 17 || 18 | 0 | IN   | GPIO. 5 | 5   | 24  |
  |  10 |  12 |    MOSI | ALT0 | 0 | 19 || 20 |   |      | 0v      |     |     |
  |   9 |  13 |    MISO | ALT0 | 0 | 21 || 22 | 0 | IN   | GPIO. 6 | 6   | 25  |
  |  11 |  14 |    SCLK | ALT0 | 0 | 23 || 24 | 1 | OUT  | CE0     | 10  | 8   |
  |     |     |      0v |      |   | 25 || 26 | 1 | OUT  | CE1     | 11  | 7   |
  |   0 |  30 |   SDA.0 |   IN | 1 | 27 || 28 | 1 | IN   | SCL.0   | 31  | 1   |
  |   5 |  21 | GPIO.21 |   IN | 1 | 29 || 30 |   |      | 0v      |     |     |
  |   6 |  22 | GPIO.22 |   IN | 1 | 31 || 32 | 0 | IN   | GPIO.26 | 26  | 12  |
  |  13 |  23 | GPIO.23 |   IN | 0 | 33 || 34 |   |      | 0v      |     |     |
  |  19 |  24 | GPIO.24 |   IN | 0 | 35 || 36 | 0 | IN   | GPIO.27 | 27  | 16  |
  |  26 |  25 | GPIO.25 |   IN | 0 | 37 || 38 | 1 | OUT  | GPIO.28 | 28  | 20  |
  |     |     |      0v |      |   | 39 || 40 | 1 | IN   | GPIO.29 | 29  | 21  |
  +-----+-----+---------+------+---+----++----+---+------+---------+-----+-----+
  | BCM | wPi |   Name  | Mode | V | Physical | V | Mode | Name    | wPi | BCM |
  +-----+-----+---------+------+---+---Pi 3---+---+------+---------+-----+-----+
</code></pre>

2. コンソールで下記コマンドを実行すると、GPIO.27 が 0 から 1 に変化すると "water" と表示する
```
pi@gc1624:~ $ gpio wfi 27 rising; echo water!
```

3. 平行線の先端を水につける。"water!" と表示する

4. gpio の manpage で　wfi の意味を確認

```
pi@gc1624:~ $ man gpio

...
wfi <pin> <mode>
       This  set  the given pin to the supplied interrupt mode: rising,
       falling or both then waits for the interrupt to happen.  It's  a
       non-busy wait, so does not consume and CPU while it's waiting.
...
```

### python からの利用
python から gpio を利用する方法はいくつもある。ライブラリがいくつもあるし、そもそも上のように shell ベースの実装がすでにあるのであれば subprocess を使って python から実装できる

以下で、gpio コマンドを subprocess から利用する方法、rpi.GPIO パッケージを利用する方法の両方を紹介する

1. GPIO.28 が `output モード、1` になっていることを確認する。なっていなければそう設定する
<pre><code style="font-size: 75%">
pi@gc1624:~ $ gpio readall
 +-----+-----+---------+------+---+---Pi 3---+---+------+---------+-----+-----+
 | BCM | wPi |   Name  | Mode | V | Physical | V | Mode | Name    | wPi | BCM |
 +-----+-----+---------+------+---+----++----+---+------+---------+-----+-----+
 |     |     |    3.3v |      |   |  1 || 2  |   |      | 5v      |     |     |
 |   2 |   8 |   SDA.1 | ALT0 | 1 |  3 || 4  |   |      | 5V      |     |     |
 |   3 |   9 |   SCL.1 | ALT0 | 1 |  5 || 6  |   |      | 0v      |     |     |
 |   4 |   7 | GPIO. 7 |   IN | 1 |  7 || 8  | 1 | OUT  | TxD     | 15  | 14  |
 |     |     |      0v |      |   |  9 || 10 | 0 | OUT  | RxD     | 16  | 15  |
 |  17 |   0 | GPIO. 0 |   IN | 0 | 11 || 12 | 0 | IN   | GPIO. 1 | 1   | 18  |
 |  27 |   2 | GPIO. 2 |   IN | 0 | 13 || 14 |   |      | 0v      |     |     |
 |  22 |   3 | GPIO. 3 |   IN | 0 | 15 || 16 | 0 | IN   | GPIO. 4 | 4   | 23  |
 |     |     |    3.3v |      |   | 17 || 18 | 0 | IN   | GPIO. 5 | 5   | 24  |
 |  10 |  12 |    MOSI | ALT0 | 0 | 19 || 20 |   |      | 0v      |     |     |
 |   9 |  13 |    MISO | ALT0 | 0 | 21 || 22 | 0 | IN   | GPIO. 6 | 6   | 25  |
 |  11 |  14 |    SCLK | ALT0 | 0 | 23 || 24 | 1 | OUT  | CE0     | 10  | 8   |
 |     |     |      0v |      |   | 25 || 26 | 1 | OUT  | CE1     | 11  | 7   |
 |   0 |  30 |   SDA.0 |   IN | 1 | 27 || 28 | 1 | IN   | SCL.0   | 31  | 1   |
 |   5 |  21 | GPIO.21 |   IN | 1 | 29 || 30 |   |      | 0v      |     |     |
 |   6 |  22 | GPIO.22 |   IN | 1 | 31 || 32 | 0 | IN   | GPIO.26 | 26  | 12  |
 |  13 |  23 | GPIO.23 |   IN | 0 | 33 || 34 |   |      | 0v      |     |     |
 |  19 |  24 | GPIO.24 |   IN | 0 | 35 || 36 | 0 | IN   | GPIO.27 | 27  | 16  |
 |  26 |  25 | GPIO.25 |   IN | 0 | 37 || 38 | 1 | OUT  | GPIO.28 | 28  | 20  |
 |     |     |      0v |      |   | 39 || 40 | 1 | IN   | GPIO.29 | 29  | 21  |
 +-----+-----+---------+------+---+----++----+---+------+---------+-----+-----+
 | BCM | wPi |   Name  | Mode | V | Physical | V | Mode | Name    | wPi | BCM |
 +-----+-----+---------+------+---+---Pi 3---+---+------+---------+-----+-----+
</code></pre>

2. `if_gpio_led3.py` を実行する。緑の LED が消灯する。平行線の先端を水につけると緑の LED が点灯する

```
pi@gc1624:~/SCRIPT/slider $ python if_gpio_led3.py 27
waiting...
on
off
waiting...
```

3. `ctl + z` で終了

4. コードを確認する
```
pi@gc1624:~/SCRIPT/slider $ cat -n if_gpio_led3.py
     1	# coding:utf-8 Copy Right Atelier Grenouille © 2015 -
     2	import subprocess
     3	import importlib
     4	import led
     5
     6	import traceback
     7	import sys
     8	import getrpimodel
     9
    10	# RPi 3 は LED1(赤LED)を操作できない
    11	pi3 = True if getrpimodel.model() == "3 Model B" else False
    12
    13	l = led.LED()
    14	l.use(0) # green
    15	pi3 or l.use(1) # red
    16	l.off(0)
    17	pi3 or l.off(1)
    18	l_status = False
    19
    20	def get_gpio():
    21	  p = subprocess.call(gpio_str, stdout=subprocess.PIPE, shell=True)
    22	  return p.stdout.readline().strip()
    23
    24
    25	def wait(pin):
    26	  global l
    27	  while True:
    28	    try:
    29	      print "waiting..."
    30	      gpio_str = 'gpio wfi '+str(pin)+ ' rising'
    31	      p = subprocess.call(gpio_str, shell=True)
    32	      l.on(0)
    33	      pi3 or l.on(1)
    34	      print "on"
    35
    36	      gpio_str = 'gpio wfi '+str(pin)+ ' falling'
    37	      p = subprocess.call(gpio_str, shell=True)
    38	      l.off(0)
    39	      pi3 or l.off(1)
    40	      print "off"
    41
    42	    except:
    43	      info=sys.exc_info()
    44	      print "Unexpected error:"+ traceback.format_exc(info[0])
    45	      print traceback.format_exc(info[1])
    46	      print traceback.format_exc(info[2])
    47
    48	if __name__ == '__main__':
    49	  pin = 23
    50	  if (len(sys.argv) == 2):
    51	    pin = int(sys.argv[1])
    52	  print wait(pin)
pi@gc1624:~/SCRIPT/slider $
```
ポイントは
  - 2行目：subprocess パッケージ(pypi)のインポート
  - 4行目：RPi の led の乗っ取りパッケージ（ローカル）をインポート
  - 8行目：RPi のバージョン確認パッケージ(pypi)をインポート、RPi3 とそれ以外で乗っ取れる LED が違う
  - 20行-22行：subprocess の実体、受け取った文字列 `gpio_str` を shell script として実行して、結果文字列を返却
  - 30行,31行：GPIO が 1 になるのを wait
  - 36行,37行：GPIO が 1 になるのを wait

5. 再度、GPIO.28 が `output モード、1` になっていることを確認する。なっていなければそう設定した上で今度は `if_gpio_led3_pp.py` を同様に実行
```
pi@gc1624:~/SCRIPT/slider $ python if_gpio_led3_pp.py 16
waiting...
on
off
waiting...
```

ここでパラメタが、`27` ではなく `16` となっているのは、使っているライブラリの都合で Pin 番号の指定が wPi 番号ではなく BCM 番号だから。このあたりの混乱が全く無駄にややこしくなっている現実を楽しむ

6. `ctl + z` で終了して、コードを確認する
```
pi@gc1624:~/SCRIPT/slider $ cat -n if_gpio_led3_pp.py
     1	# coding:utf-8 Copy Right Atelier Grenouille © 2015 -
     2	#import subprocess
     3	import importlib
     4	import led
     5	import RPi.GPIO as GPIO
     6
     7	import traceback
     8	import sys
     9	import getrpimodel
    10
    11	# RPi 3 は LED1(赤LED)を操作できない
    12	pi3 = True if getrpimodel.model() == "3 Model B" else False
    13
    14	l = led.LED()
    15	l.use(0) # green
    16	pi3 or l.use(1) # red
    17	l.off(0)
    18	pi3 or l.off(1)
    19	l_status = False
    20
    21	# GPIO の設定
    22	GPIO.setmode(GPIO.BCM)
    23
    24	#def get_gpio():
    25	#  p = subprocess.call(gpio_str, stdout=subprocess.PIPE, shell=True)
    26	#  return p.stdout.readline().strip()
    27
    28
    29	def wait(pin):
    30	  global l
    31	  GPIO.setup(int(pin), GPIO.IN, pull_up_down=GPIO.PUD_DOWN)
    32	  while True:
    33	    try:
    34	      print "waiting..."
    35	      GPIO.wait_for_edge(int(pin), GPIO.RISING)
    36	      #gpio_str = 'gpio wfi '+str(pin)+ ' rising'
    37	      #p = subprocess.call(gpio_str, shell=True)
    38	      l.on(0)
    39	      pi3 or l.on(1)
    40	      print "on"
    41
    42	      GPIO.wait_for_edge(int(pin), GPIO.FALLING)
    43	      #gpio_str = 'gpio wfi '+str(pin)+ ' falling'
    44	      #p = subprocess.call(gpio_str, shell=True)
    45	      l.off(0)
    46	      pi3 or l.off(1)
    47	      print "off"
    48
    49	    except:
    50	      info=sys.exc_info()
    51	      print "Unexpected error:"+ traceback.format_exc(info[0])
    52	      print traceback.format_exc(info[1])
    53	      print traceback.format_exc(info[2])
    54
    55	if __name__ == '__main__':
    56	  pin = 23
    57	  if (len(sys.argv) == 2):
    58	    pin = int(sys.argv[1])
    59	  print wait(pin)
```

`if_gpio_led3.py` との違いは subprocess で shell を呼び出していたかわりに rpi.GPIO モジュールをつかっているだけ。コードにはあまり変化はない

### 実用的な漏水センサー
RPi の LED を on/off するだけでなく、その時の状況をサーバに送ったり、`twillio` のようなサービスをつかって管理者に電話をかけたりする処理を ***一行追加するだけで*** 実用的な漏水センサーシステムになってしまう

また、今回は漏水センサー（実体はただの平行線）をつかって GPIO を監視したが、GPIO に接続するセンサーを ***人感センサー*** や ***光センサー*** 等、***接点インターフェース*** の on/off で通知するセンサーに ***置き換えるだけ*** で監視の対象をなににでもかえることができる

1. 現在、一分おきに状況を撮影してる処理を crontab で行なっているので、コメントアウトする
```
crontab -e
```

末尾で read.py を 1分毎に起動するようにしているのでここを変更する

```
この行を　↓
*/1 * * * * sudo python /home/pi/SCRIPT/slider/read.py

先頭に # を追加してコメントアウト
＃*/1 * * * * sudo python /home/pi/SCRIPT/slider/read.py
```

`ctl-x`で保存すると read.py （センサを読んで送信する処理）が停止する

2. `if_gpio_rpi3_pp.py` を sudo 付きで (read.py が要sudoなので) 実行する
```
pi@gc1624:~/SCRIPT/slider $ sudo python if_gpio_rpi3_pp.py
waiting...
```

3. ブラウザで monitor を開く。平行線の先端を水につけるたびに撮影写真とその時のセンサーデータが送信されているのを確認

4. `ctl + z` で終了して、コードを確認する
```
pi@gc1624:~/SCRIPT/slider $ cat -n if_gpio_rpi3_pp.py
     1	# coding:utf-8 Copy Right Atelier Grenouille © 2015 -
     2	import importlib
     3	import led
     4	import RPi.GPIO as GPIO
     5
     6	import traceback
     7	import sys
     8	import getrpimodel
     9
    10	# RPi 3 は LED1(赤LED)を操作できない
    11	pi3 = True if getrpimodel.model() == "3 Model B" else False
    12
    13	l = led.LED()
    14	l.use(0) # green
    15	pi3 or l.use(1) # red
    16	l.off(0)
    17	pi3 or l.off(1)
    18	l_status = False
    19
    20	# GPIO の設定
    21	GPIO.setmode(GPIO.BCM)
    22
    23
    24	def wait(pin):
    25	  global l
    26	  GPIO.setup(int(pin), GPIO.IN, pull_up_down=GPIO.PUD_DOWN)
    27	  while True:
    28	    try:
    29	      print "waiting..."
    30	      GPIO.wait_for_edge(int(pin), GPIO.RISING)
    31	      reader = importlib.import_module("read")
    32	      reader.read()
    33	      l.on(0)
    34	      pi3 or l.on(1)
    35	      print "on"
    36
    37	      GPIO.wait_for_edge(int(pin), GPIO.FALLING)
    38	      l.off(0)
    39	      pi3 or l.off(1)
    40	      print "off"
    41
    42	    except:
    43	      info=sys.exc_info()
    44	      print "Unexpected error:"+ traceback.format_exc(info[0])
    45	      print traceback.format_exc(info[1])
    46	      print traceback.format_exc(info[2])
    47
    48	if __name__ == '__main__':
    49	  pin = 23
    50	  if (len(sys.argv) == 2):
    51	    pin = int(sys.argv[1])
    52	  print wait(pin)
```

`31行目` の import_module は、python の動的ロードで、名前で指定されたモジュールを動的にロードし、`32行目`でモジュールの read() を実行  
このように、なにかを通知する処理がパッケージやモジュールになっていれば数行で処理をシステムに Mix-in してしまうことができる

### コンピュータ間通信
2台のコンピュータの GPIO を直結し、コンピュータ間で通信をする  

1. 隣同士で、送信側と受信側に別れる、交互に変わっても良い
2. 温湿度センサーを確認する  
温湿度センサーの break out 基盤のマーク`+`の Pin が 3.3V電源、`-`の Pin が GND、`out` の Pin が Data

<img src="pic/ss.2017-03-23 17.59.11.png" width="30%">

それぞれ、以下のように接続されている

|線|機能|Physical Pin #|GPIO|
|:--:|:--:|:--:|:--:|
|+|3.3V|17||
|out|data|40|GPIO.29|
|-|GND|39||

<img src="pic/ss.2017-03-23 17.59.48.png" width="75%">

3. 送信側は、下記のようにセンサモジュールをケーブルから外す
<img src="pic/ss.2017-03-23 18.00.16.png" width="75%">

4. 受信側は RPi から温湿度センサモジュールのケーブルを外す（物理Pin番号17,39,40の３本を抜く）

5. 送信側には温湿度センサで使っていたケーブルが残る、それを使って送信側、受信側のそれぞれ下記のピンを直結する
  - 物理Pin39 (GND) 同士
  - 物理Pin40 (GPIO.29)同士

6. 送受信双方とも、GPIO.29 が input mode で値 0　であることを確認。ちがったらそのように設定
7. 受信側、`if_gpio_led3.py` で GPIO.29 が on になったら緑の LED が光るようにする
```
pi@gc1624:~/SCRIPT/slider $ python if_gpio_led3.py 29
waiting...
```
8. 送信側、gpio 29 を output mode の 1 にする
```
pi@gc1624:~ $ gpio mode 28 out
pi@gc1624:~ $ gpio write 28 1
```
9. 受信側、緑の LED が点灯している事と、GPIO.29 が 1になっていることを `gpio readall` で確認
10. 送信側、受信側ともに温湿度センサーモジュールの接続をもとに戻して RPi を再起動する
