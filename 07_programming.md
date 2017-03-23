# 5.Raspberry Pi のプログラミング

##<u>目標</u>
bash と python のコーディングと実行を経験する
手順を一度体験しておけば、後はどうとでもなる

##<u>実習手順</u>
自身の gc16 に terminal でログインする

### Raspberry led の乗っ取り
Raspberry Pi のグリーンの LED は通常は SD カードへのアクセスで点滅する
そのように kernel で関連づけられているからそのような動作になっているのであって
その関連を切れば、`/sys/class/led` で自由に操作できる

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
上の手順をそのまま、ファイルに書いて bash のスクリプトにする。led の on と off の間に 1 秒間の sleep を、`sleep 秒数` コマンドで入れる。s

1. sleep コマンドの動作の確認
`sleep 3` を実行すると、３秒まってコマンドプロンプトに戻る

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
