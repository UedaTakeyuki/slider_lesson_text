# 7.Raspberry Pi のプログラミング

##<u>目標</u>
bash と python のコーディングと実行を経験する  

##<u>実習手順</u>
自身の gc16 に terminal でログインする

### Raspberry led の乗っ取り
Raspberry Pi のグリーンの LED は SD カードへのアクセスに応じて点滅する  
そのように kernel で関連づけられているからそのように動作しているのであって  
`sys`ファイルを操作して関連を切ってしまえば自由に操作できるようになる

1. `cat /sys/class/leds/led0/trigger` コマンドで、現在の led0 のトリガーを確認
```
pi@gc1624:~ $ cat /sys/class/leds/led0/trigger
[none] kbd-scrollock kbd-numlock kbd-capslock kbd-kanalock kbd-shiftlock kbd-altgrlock kbd-ctrllock kbd-altlock kbd-shiftllock kbd-shiftrlock kbd-ctrlllock kbd-ctrlrlock mmc0 mmc1 timer oneshot heartbeat backlight gpio cpu0 cpu1 cpu2 cpu3 default-on input rfkill0 rfkill1
```

2. `/sys/class/leds/led0/trigger` に `none` と書いてトリガーをなくす  
```
pi@gc1624:~ $ sudo sh -c "echo none > /sys/class/leds/led0/trigger"
```  
緑の led は消灯する

3. led の on
```
sudo sh -c "echo 1 > /sys/class/leds/led0/brightness"
```

4. led の off
```
sudo sh -c "echo 0 > /sys/class/leds/led0/brightness"
```

### 上記、led の乗っ取りをプログラムする
上の手順をそのまま、ファイルに書いて bash のスクリプトにする。led の on と off の間に 1 秒間の sleep を、`sleep 秒数` コマンドで入れる

1. sleep コマンドの動作の確認  
コンソールで `sleep 3` を実行すると、３秒まってコマンドプロンプトに戻る

2. `nano led.sh` で（vi使いの方は vim でもいいです）エディタを開き、下記のように編集して保存
する  
```
echo none > /sys/class/leds/led0/trigger
echo 1 > /sys/class/leds/led0/brightness
sleep 1
echo 0 > /sys/class/leds/led0/brightness
```  
nano エディタの場合　`CTL+x`　で下記のように保存する確認してくるので `y`  
<img src="pic/ss.2017-03-22 22.03.30.png" width="75%">  
さらに、ファイル名を確認してくるので、そのままでよければ `enter`  
<img src="pic/ss.2017-03-22 22.04.52.png" width="75%">  
`ls` で、led.sh が出来ていることを確認  
  
3. 実行権限の付与  
`chmod a+x led.sh`

4. root 権限で実行  
緑の led が 1秒間点灯する  
`sudo ./led.sh`

### python のインタラクティブな実行
スクリプトファイルを指定せずに python を起動することでインタラクティブな実行が可能  
このモードで python のインタプリタは一行の入力を受け付け、評価し、評価結果を出力する事を繰り返す（read-eval-print loop とか REPL とか呼ぶ）

Raspberry Pi の緑の LED をつかって、モールス信号を発信するプログラムを作る

1. 利用するモジュール(led.py)のあるフォルダに移動
```
pi@gc1624:~ $ cd /home/pi/SCRIPT/slider/
pi@gc1624:~/SCRIPT/slider $
```

2. python のインタプリタをインタラクティブに実行
```
Python 2.7.9 (default, Sep 17 2016, 20:26:04)
[GCC 4.9.2] on linux2
Type "help", "copyright", "credits" or "license" for more information.
```

3. led モジュールをインポート
```
>>> import led
```

4. led の初期化
```
>>> l = led.LED()
>>> l.use(0) # green
```

5. 点灯、消灯、短く点滅、長く点滅を試す
```
>>> l.on(0)
>>> l.off(0)
>>> l.short(0)
>>> l.long(0)
```

6. `SOS` を発信
```
>>> l.S(0)
>>> l.O(0)
>>> l.S(0)
```
### Python のプログラムファイルを作成する
上記の手続きをプログラムファイルに実装する。処理は、SOSを繰り返し発信するようにする。

2. `nano morse_code.py` で（vi使いの方は vim でもいいです）エディタを開き、下記のように編集して保存
する  
```
import led
import time
l = led.LED()
l.use(0)
while True:
  l.S(0)
  time.sleep(1)
    l.O(0)
  time.sleep(1)
  l.S(0)
  time.sleep(3)
```

3. morse_code.py を実行する  
```
pi@gc1624:~ $ python morse_code.py
  File "morse_code.py", line 7
    l.O(0)
    ^
IndentationError: unexpected indent
```  
このようにエラーになる。python では `字下げ` が単なる字句ではなく制御ブロックを表すので、字下げのレベルがおかしいと構文エラーを報告する。再度、morse_code.py をエディタで開き、字下げを合わせる  
```
import led
import time
l = led.LED()
l.use(0)
while True:
  l.S(0)
  time.sleep(1)
  l.O(0)
  time.sleep(1)
  l.S(0)
  time.sleep(3)
```  
これを実行すると、RPi の緑の LED が SOS を発信し続ける。`CNTL+Z`で終了する  


### Tensor Flow を使ったプログラムの実行
Tensor Flow のサンプルプログラムを実行する  
Google の TF のページのサンプルプログラム程度であれば Raspberry Pi で十分に動作する

1. 以下は、Google の　TF のページのサンプルプログラムを少し修正したものである（Google のサイト上のサンプルプログラムは実は今の TF とバージョンがあっておらず(!)、そのままでは動かない）  
```
pi@gc1624:~ $ cat -n /home/pi/tf.py
     1	import tensorflow as tf
     2	import numpy as np
     3
     4	# Create 100 phony x, y data points in NumPy, y = x * 0.1 + 0.3
     5	x_data = np.random.rand(100).astype(np.float32)
     6	y_data = x_data * 0.1 + 0.3
     7
     8	# Try to find values for W and b that compute y_data = W * x_data + b
     9	# (We know that W should be 0.1 and b 0.3, but TensorFlow will
    10	# figure that out for us.)
    11	W = tf.Variable(tf.random_uniform([1], -1.0, 1.0))
    12	b = tf.Variable(tf.zeros([1]))
    13	y = W * x_data + b
    14
    15	# Minimize the mean squared errors.
    16	loss = tf.reduce_mean(tf.square(y - y_data))
    17	optimizer = tf.train.GradientDescentOptimizer(0.5)
    18	train = optimizer.minimize(loss)
    19
    20	# Before starting, initialize the variables.  We will 'run' this first.
    21	#init = tf.global_variables_initializer()
    22	init = tf.initialize_all_variables()
    23
    24	# Launch the graph.
    25	sess = tf.Session()
    26	sess.run(init)
    27
    28	# Fit the line.
    29	for step in range(201):
    30	    sess.run(train)
    31	    if step % 20 == 0:
    32	        print(step, sess.run(W), sess.run(b))
    33
    34	# Learns best fit is W: [0.1], b: [0.3]
```  
ランダムな 100個 の x と、y = 0.1x + 0.3 の組を教師として、直線の傾きと原点を予測するという少々自明な問題  

2. 実行してみる  
```
pi@gc1624:~ $ python /home/pi/tf.py
(0, array([ 0.10128424], dtype=float32), array([ 0.39288887], dtype=float32))
(20, array([ 0.08956077], dtype=float32), array([ 0.30521467], dtype=float32))
(40, array([ 0.09698179], dtype=float32), array([ 0.30150768], dtype=float32))
(60, array([ 0.09912737], dtype=float32), array([ 0.3004359], dtype=float32))
(80, array([ 0.09974769], dtype=float32), array([ 0.30012605], dtype=float32))
(100, array([ 0.09992705], dtype=float32), array([ 0.30003646], dtype=float32))
(120, array([ 0.09997892], dtype=float32), array([ 0.30001056], dtype=float32))
(140, array([ 0.0999939], dtype=float32), array([ 0.30000305], dtype=float32))
(160, array([ 0.09999822], dtype=float32), array([ 0.30000091], dtype=float32))
(180, array([ 0.09999949], dtype=float32), array([ 0.30000028], dtype=float32))
(200, array([ 0.09999985], dtype=float32), array([ 0.3000001], dtype=float32))
```  
少し時間がかかるが、正解に収束していくのが見える  

3. RPi への Tensor Flow ライブラリのインストールスクリプトは以下  
`/home/pi/install/gc_setups/tensorflow.setup.sh`
