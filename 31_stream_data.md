# 31.ストリームデータ処理の実装

##<u>目的</u>
IoT サービスを実装する際、ストリームデータの取り扱いに工夫が必要になる
  - すぐに膨大なデータが溜まる
  - 直近のデータにはリアルタイムでの参照が必要  

このため、業務アプリのような DB スキーマの設計ではシステムのカットイン後にしばらくして破綻するという嫌らしい破綻の仕方をする  
直近のデータにたいしては定数時間でのアクセスが必要で、IoT サービスはみな何らかの方法でキャッシュをおこなっていて、その部分の工夫が売りになっている  
本章では monitor がどのように`直近データへの定数時間でのアクセス`をシンプルに実装しているかその工夫を参考までに紹介する

##<u>実習手順</u>
自身の gc16 に terminal でログインする

### 準備
  - top コマンド、tail コマンド:  
  `top` コマンド、`tail` コマンドはそれぞれテキストファイルの先頭もしくは末尾から一定数の行数だけ抜き出すコマンド
  - time コマンド:
  `time コマンド` で、コマンドの経過時間を表示するコマンド
  - wc コマンド:
  テキストファイルの文字数、単語数、行数を表示するコマンド  
  `wc -l` で行数（line #)を表示する

### 実験データのコピー
Web サーバー上の実験データ `sample` を自分の RPi に`rcp`コマンドを使ってコピーする  
```
pi@gc1624:~ $ rcp -rp gc1601.local:/home/pi/sample .
pi@gc1601.local's password:
stemp.csv.gz                                  100% 1075     1.1KB/s   00:00    
temp.csv.gz                                   100%  386KB 386.3KB/s   00:00    
10temp.csv.gz                                 100% 3863KB 772.6KB/s   00:05    
```  
フォルダ sample に移動し、中のファイルを `gunzip` コマンドで解凍する  
```
pi@gc1624:~ $ cd sample
pi@gc1624:~/sample $ ls
10temp.csv.gz  stemp.csv.gz  temp.csv.gz
pi@gc1624:~/sample $ gunzip *.gz
pi@gc1624:~/sample $ ls
10temp.csv  stemp.csv  temp.csv
```

### 実験
`tail` コマンドが、対象のテキストファイルのサイズによらず、ほぼ一定の時間で結果を返す事を確認する  
stemp.csv は、300行の csv ファイル  
stemp.csv の末尾10行を `tail -n 10` で取得し、その時間を `time` で計測する  
実行時間には揺らぎがあるので複数回ためす  
0.003s - 0.006s ぐらいで終了する  
```
pi@gc1601:~/sample $ wc -l stemp.csv
300 stemp.csv
pi@gc1601:~/sample $ time tail -n 10 stemp.csv
2017/04/04 09:05:16,21.9
2017/04/04 09:10:17,21.8
2017/04/04 09:15:15,21.8
2017/04/04 09:20:16,21.9
2017/04/04 09:25:16,21.9
2017/04/04 09:30:15,22.0
2017/04/04 09:35:17,21.8
2017/04/04 09:40:16,21.9
2017/04/04 09:45:18,21.9
2017/04/04 09:50:18,22.0

real	0m0.004s
user	0m0.010s
sys	0m0.000s
```  

同様に temp.csv の tail -n 10 を取得した際の実行時間を取得する  
temp.csv は 11万6千行の csv ファイルで、先ほどの stemp.csv の 400倍のサイズ  
```
pi@gc1601:~/sample $ wc -l temp.csv
116223 temp.csv

pi@gc1601:~/sample $ time tail -n 10 temp.csv
2017/04/04 09:05:16,21.9
2017/04/04 09:10:17,21.8
2017/04/04 09:15:15,21.8
2017/04/04 09:20:16,21.9
2017/04/04 09:25:16,21.9
2017/04/04 09:30:15,22.0
2017/04/04 09:35:17,21.8
2017/04/04 09:40:16,21.9
2017/04/04 09:45:18,21.9
2017/04/04 09:50:18,22.0

real	0m0.004s
user	0m0.000s
sys	0m0.000s
```  
実行時間はかわらない  

さらに 10temp.csv で試す、これは temp.csv の10倍、116万行の csv  
```
pi@gc1601:~/sample $ wc -l 10temp.csv
1162230 10temp.csv
pi@gc1601:~/sample $ time tail -n 10 10temp.csv
2017/04/04 09:05:16,21.9
2017/04/04 09:10:17,21.8
2017/04/04 09:15:15,21.8
2017/04/04 09:20:16,21.9
2017/04/04 09:25:16,21.9
2017/04/04 09:30:15,22.0
2017/04/04 09:35:17,21.8
2017/04/04 09:40:16,21.9
2017/04/04 09:45:18,21.9
2017/04/04 09:50:18,22.0

real	0m0.004s
user	0m0.000s
sys	0m0.000s
```  
やはり実行時間はかわらない

### monitor の実装
monitor では、csv ファイルの末尾 n 行を tail で取り出した物を元データとして json を作成してかえしており、これによって直近データへの実時間アクセスが実現できている  
```
pi@gc1624:/var/www/html/SCRIPT/monitor $ cat -n data.php
     1	<?php
     2	/**
     3	 * [API] Get fresh sensor data.
     4	 *
     5	 * Sensors shoud be specified by .dini settings for the account.
     6	 * Return data as JSON.
     7	 * Latest data than $_GET[ILTimes] if specified.
     8	 *
     9	 * Requires $_GET['serial_id']
    10	 * Return json[SENSOR_NAME]=array("datetime" => , "data" => )
    11	 *
    12	 * @author Dr. Takeyuki UEDA
    13	 * @copyright Copyright© Atelier UEDA 2016 - All rights reserved.
    14	 *
    15	 */
    16	 
...
...
    97	function get_latest_data(&$json, $name, $lasttime, $show_data_lows){
...
...
   104	  if (is_updated($csv_file_name, $lasttime)){
   105	    $result = array();
   106	    // 末尾n行だけの仮ファイルをつくり、そこから読む
   107	    $temp_cmd = "tail -n ".$show_data_lows." ".$csv_file_name." > tmp.csv";
   108	    `$temp_cmd`;
   109	    if (($handle = fopen("tmp.csv", "r")) !== FALSE) {      
   110	      while (($data = fgetcsv($handle)) !== FALSE) {
   111	        array_push($result, (array("datetime" => $data[0], "data" => (float)$data[1])));
   112	      }
   113	      fclose($handle);
   114	      $json[$name]=array_reverse($result);
   115	    }
   116	  }
   117	}
   118
   119	// '2015/10/26 14:40:13' 形式の文字列を unix time に変換
   120	function jdatetotime($str){
   121	  $ymdhms=explode(" ", $str);
   122	  $ymd = $ymdhms[0];
   123	  $hms = $ymdhms[1];
   124	  $ymd_dc = explode("/",$ymd);
   125	  $y = $ymd_dc[0];
   126	  $m = $ymd_dc[1];
   127	  $d = $ymd_dc[2];
   128	  $dtstr= $y . "-" . $m . "-" . $d . " " . $hms;
   129	  return strtotime($dtstr);
   130	}
   131	?>
```
