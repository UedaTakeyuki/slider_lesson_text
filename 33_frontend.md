# 33.front endの実装

##<u>目的</u>
柔軟かつシンプルに mobile first なフロントエンドを作るために Bootstrap と jQuerry Mobile の使い方を理解し、あわせて ajax API のブラウザからの呼び出し方法を理解する

##<u>実習手順</u>
自身の gc16 に terminal でログインする

### 準備
Bootstrap, jQuerry, jQuery-mobile 等の javascript ライブラリは通常だと `npm` コマンドでインストールするのだが、教室の環境は外部ネットワークにつながっていないので、既にインストール済みの別の Web アプリケーションの node_modules をコピーする

1. `/var/www/html/gpio/` に移動  
```
pi@gc1624:~ $ cd /var/www/html/gpio/
```

2. `monitor` の `node_modules` をコピー  
```
pi@gc1624:/var/www/html/gpio $ cp -rp ../SCRIPT/monitor/node_modules/ .
```

3. `BackupPi_2` の `node_modules` の中の `jquery-mobile` だけを、上でコピーした `node_modules` にコピー  
```
pi@gc1624:/var/www/html/gpio $ cp -rp ../SCRIPT/BackupPi_2/node_modules/jquery-mobile node_modules
```

4. コピーした node-modules の確認  
```
pi@gc1624:/var/www/html/gpio $ npm list
/var/www/html/gpio
├── bootstrap@3.3.7
├─┬ chart.js@2.4.0
│ ├─┬ chartjs-color@2.0.0
│ │ ├─┬ chartjs-color-string@0.4.0
│ │ │ └── color-name@1.1.1
│ │ └── color-convert@0.5.3
│ └── moment@2.17.1
├── jquery@1.12.4
├── jquery-mobile@1.4.1
└─┬ vue@1.0.28
  └─┬ envify@3.4.1
    ├─┬ jstransform@11.0.3
    │ ├── base62@1.1.2
    │ ├─┬ commoner@0.10.8
    │ │ ├─┬ commander@2.9.0
    │ │ │ └── graceful-readlink@1.0.1
    │ │ ├─┬ detective@4.3.2
    │ │ │ ├── acorn@3.3.0
    │ │ │ └── defined@1.0.0
    │ │ ├─┬ glob@5.0.15
    │ │ │ ├─┬ inflight@1.0.6
    │ │ │ │ └── wrappy@1.0.2
    │ │ │ ├── inherits@2.0.3
    │ │ │ ├─┬ minimatch@3.0.3
    │ │ │ │ └─┬ brace-expansion@1.1.6
    │ │ │ │   ├── balanced-match@0.4.2
    │ │ │ │   └── concat-map@0.0.1
    │ │ │ ├─┬ once@1.4.0
    │ │ │ │ └── wrappy@1.0.2
    │ │ │ └── path-is-absolute@1.0.1
    │ │ ├── graceful-fs@4.1.11
    │ │ ├── iconv-lite@0.4.15
    │ │ ├─┬ mkdirp@0.5.1
    │ │ │ └── minimist@0.0.8
    │ │ ├── private@0.1.6
    │ │ ├── q@1.4.1
    │ │ └─┬ recast@0.11.18
    │ │   ├── ast-types@0.9.2
    │ │   ├── esprima@3.1.2
    │ │   └── source-map@0.5.6
    │ ├── esprima-fb@15001.1001.0-dev-harmony-fb
    │ ├── object-assign@2.1.1
    │ └─┬ source-map@0.4.4
    │   └── amdefine@1.0.1
    └── through@2.3.8
```

5. index.php に jQuery, Bootstrap, jQuery-mobile を取り込む  
下記のように 6 - 14行目を `<titel></title>` の下に挿入
```
1	<!DOCTYPE html>
2	<html lang="ja">
3	<head>
4	    <meta charset="UTF-8">
5	    <title>Raspberry Pi GPIO</title>
6	    <script src="node_modules/chart.js/node_modules/moment/min/moment.min.js"></script>
7	    <script src="node_modules/chart.js/dist/Chart.min.js"></script>
8	    <link rel="stylesheet" href="node_modules/jquery-mobile/dist/jquery.mobile.min.css" />
9	    <script src="node_modules/jquery/dist/jquery.min.js"></script>
10	    <script src="node_modules/jquery-mobile/dist/jquery.mobile.min.js"></script>
11
12	    <link rel="stylesheet" href="node_modules/bootstrap/dist/css/bootstrap.min.css">
13	    <link rel="stylesheet" href="node_modules/bootstrap/dist/css/bootstrap-theme.min.css">
14	    <script src="node_modules/bootstrap/dist/js/bootstrap.min.js"></script>
15	</head>
16	<body>
17	<?php
18	  for ($i = 1; $i < 30; $i++){
19	    echo '<p>GPIO '.$i.' = '.rtrim(`sudo gpio read $i`).'</p>';
20	  }
21	?>
22	</body>
23	</html>
```  

### スマフォ風の UI に
1. jQuery mobile でヘッダとフッタを付ける  
下記のように 17 - 23行目、25 - 33行目を挿入して、jQuery-mobile のページ構成にする

```
1	<!DOCTYPE html>
2	<html lang="ja">
3	<head>
4	  <meta charset="UTF-8">
5	  <title>Raspberry Pi GPIO</title>
6	  <script src="node_modules/chart.js/node_modules/moment/min/moment.min.js"></script>
7	  <script src="node_modules/chart.js/dist/Chart.min.js"></script>
8	  <link rel="stylesheet" href="node_modules/jquery-mobile/dist/jquery.mobile.min.css" />
9	  <script src="node_modules/jquery/dist/jquery.min.js"></script>
10	  <script src="node_modules/jquery-mobile/dist/jquery.mobile.min.js"></script>
11
12	  <link rel="stylesheet" href="node_modules/bootstrap/dist/css/bootstrap.min.css">
13	  <link rel="stylesheet" href="node_modules/bootstrap/dist/css/bootstrap-theme.min.css">
14	  <script src="node_modules/bootstrap/dist/js/bootstrap.min.js"></script>
15	</head>
16	<body>
17	　<div data-role="page">
18	    <div data-role="header" data-position="fixed" data-theme="b" data-disable-page-zoom="false">
19	      <h1>GPIO status</h1>
20	    </div>
21
22	    <div data-roll="content" data-theme="c" class="no-cache">
23	<?php
24	      for ($i = 1; $i < 30; $i++){
25	        echo '<p>GPIO '.$i.' = '.rtrim(`sudo gpio read $i`).'</p>';
26	      }
27	?>
28	    </div>
29
30	    <div data-role="footer" data-position="fixed" class="no-cache" data-theme="b">
31	      <h4><?php echo "© Atelier UEDA" ?></h4>
32	    </div>
33	  </div>
34	</body>
35	</html>
```  
下記のような表示になる  
<img src="pic/ss.2017-04-05 18.36.34.png" width="75%">  

2. レスポンシブデザイン
mobile first でレスポンシブな表示にするために BootStrap のグリッドシステムを使う  
24行、33行の `<class="row">` class と 28行、30行の `<class="col-xx-xx">` を追加した  
```
1	<!DOCTYPE html>
2	<html lang="ja">
3	<head>
4	  <meta charset="UTF-8">
5	  <title>Raspberry Pi GPIO</title>
6	  <script src="node_modules/chart.js/node_modules/moment/min/moment.min.js"></script>
7	  <script src="node_modules/chart.js/dist/Chart.min.js"></script>
8	  <link rel="stylesheet" href="node_modules/jquery-mobile/dist/jquery.mobile.min.css" />
9	  <script src="node_modules/jquery/dist/jquery.min.js"></script>
10	  <script src="node_modules/jquery-mobile/dist/jquery.mobile.min.js"></script>
11
12	  <link rel="stylesheet" href="node_modules/bootstrap/dist/css/bootstrap.min.css">
13	  <link rel="stylesheet" href="node_modules/bootstrap/dist/css/bootstrap-theme.min.css">
14	  <script src="node_modules/bootstrap/dist/js/bootstrap.min.js"></script>
15	</head>
16	<body>
17	　<div data-role="page">
18	    <div data-role="header" data-position="fixed" data-theme="b" data-disable-page-zoom="false">
19	      <h1>GPIO status</h1>
20	    </div>
21
22	    <div data-roll="content" data-theme="c" class="no-cache">
23
24	      <div class="row">
25
26	<?php
27	        for ($i = 1; $i < 30; $i++){
28	          echo '<div class="col-md-2 col-sm-3 col-xs-6">';
29	          echo '<p>GPIO '.$i.' = '.rtrim(`sudo gpio read $i`).'</p>';
30	          echo '</div>';
31	        }
32	?>
33	      </div><!-- <div class="row"> -->
34	    </div>
35
36	    <div data-role="footer" data-position="fixed" class="no-cache" data-theme="b">
37	      <h4><?php echo "© Atelier UEDA" ?></h4>
38	    </div>
39	  </div>
40	</body>
41	</html>
```  
28行目の意味は、「普通のサイズの場合は幅の 2/12 を使って表示、小さいサイズの場合は幅の 3/12、凄く小さいサイズの場合は 6/12」で、それぞれ 6段表示、4段表示、 2段表示になる  

ブラウザのサイズに合わせて下記のような表示になる
<img src="pic/ss.2017-04-05 18.39.17.png" width="75%">  
<img src="pic/ss.2017-04-05 18.39.28.png" width="75%">  
<img src="pic/ss.2017-04-05 18.39.40.png" width="75%">  

3. 静的な文字列は静的なままに
上のコードで、28行と30行は、dom の静的な要素であったはずの <div> を echo で動的に生成している  
最終的にできあがる HTML はかわらないのだが静的なものは静的にしておきたいので、下記のように変更することができる  
29行と31行は php の for loop によって繰り返し生成される  
```
1	<!DOCTYPE html>
2	<html lang="ja">
3	<head>
4	  <meta charset="UTF-8">
5	  <title>Raspberry Pi GPIO</title>
6	  <script src="node_modules/chart.js/node_modules/moment/min/moment.min.js"></script>
7	  <script src="node_modules/chart.js/dist/Chart.min.js"></script>
8	  <link rel="stylesheet" href="node_modules/jquery-mobile/dist/jquery.mobile.min.css" />
9	  <script src="node_modules/jquery/dist/jquery.min.js"></script>
10	  <script src="node_modules/jquery-mobile/dist/jquery.mobile.min.js"></script>
11
12	  <link rel="stylesheet" href="node_modules/bootstrap/dist/css/bootstrap.min.css">
13	  <link rel="stylesheet" href="node_modules/bootstrap/dist/css/bootstrap-theme.min.css">
14	  <script src="node_modules/bootstrap/dist/js/bootstrap.min.js"></script>
15	</head>
16	<body>
17	　<div data-role="page">
18	    <div data-role="header" data-position="fixed" data-theme="b" data-disable-page-zoom="false">
19	      <h1>GPIO status</h1>
20	    </div>
21
22	    <div data-roll="content" data-theme="c" class="no-cache">
23
24	      <div class="row">
25
26	<?php
27	        for ($i = 1; $i < 30; $i++){
28	?>
29	          <div class="col-md-2 col-sm-3 col-xs-6">
30	<?php
31	          echo '<p>GPIO '.$i.' = '.rtrim(`sudo gpio read $i`).'</p>';
32	?>
33	          </div>
34	<?php
35	        }
36	?>
37	      </div><!-- <div class="row"> -->
38	    </div>
39
40	    <div data-role="footer" data-position="fixed" class="no-cache" data-theme="b">
41	      <h4><?php echo "© Atelier UEDA" ?></h4>
42	    </div>
43	  </div>
44	</body>
45	</html>
```

4. 制御構造に関する別の構文(Alternative syntax for control structures)
php のブロック `{` と html のタグ `<` がまじって構造が見にくくくなることを避けるために  
`別の構文`で書き換えるとかなりリーダブルなコードになる
```
1	<!DOCTYPE html>
2	<html lang="ja">
3	<head>
4	  <meta charset="UTF-8">
5	  <title>Raspberry Pi GPIO</title>
6	  <script src="node_modules/chart.js/node_modules/moment/min/moment.min.js"></script>
7	  <script src="node_modules/chart.js/dist/Chart.min.js"></script>
8	  <link rel="stylesheet" href="node_modules/jquery-mobile/dist/jquery.mobile.min.css" />
9	  <script src="node_modules/jquery/dist/jquery.min.js"></script>
10	  <script src="node_modules/jquery-mobile/dist/jquery.mobile.min.js"></script>
11
12	  <link rel="stylesheet" href="node_modules/bootstrap/dist/css/bootstrap.min.css">
13	  <link rel="stylesheet" href="node_modules/bootstrap/dist/css/bootstrap-theme.min.css">
14	  <script src="node_modules/bootstrap/dist/js/bootstrap.min.js"></script>
15	</head>
16	<body>
17	　<div data-role="page">
18	    <div data-role="header" data-position="fixed" data-theme="b" data-disable-page-zoom="false">
19	      <h1>GPIO status</h1>
20	    </div>
21
22	    <div data-roll="content" data-theme="c" class="no-cache">
23
24	      <div class="row">
25
26	        <?php for ($i = 1; $i < 30; $i++): ?>
27	          <div class="col-md-2 col-sm-3 col-xs-6">
28	            <p>GPIO <?= $i ?> = <?= rtrim(`sudo gpio read $i`) ?></p>
29	          </div>
30	        <?php endfor ?>
31
32	      </div><!-- <div class="row"> -->
33	    </div>
34
35	    <div data-role="footer" data-position="fixed" class="no-cache" data-theme="b">
36	      <h4><?php echo "© Atelier UEDA" ?></h4>
37	    </div>
38	  </div>
39	</body>
40	</html>
```  
ポイント
  - 26行: 開きカッコの代わりに、python 風の `:`  
  - 28行: echo 分の代わりに、<?= 値 ?>
  - 30行: 閉じカッコの代わりに `endfor;`

5. php によって生成される html の確認
php がどのように展開しているかは、index.php ファイルを php インタプリタで実行することでみることができる  
```
php index.php
```  
上の二つのスクリプトは同じ html に展開される  

 .php ファイルの構文エラーでブラウザの画面が真っ白になった場合 nginx のエラーログ `/var/log/error/nginx/error.log` をしらべるのだが、単に
php インタプリタで .php ファイルを実行してエラーメッセージを表示させるのも有益  

### SPA(Single Page Application) と ajax
この Web アプリケーションはリロードするたびに GPIO の現在の値の一覧を表示する  
これを GPIO の値に変化があった時に自動的に反映されるように変更する  
方法は簡単で  
  - 表示する値に ID を付ける
  - interval loop から ajax で値を取得、ID で取得した dom 要素を書き換える

1. 表示する値に ID を付ける
gpio の値を javascript から操作できるようにするため `<span>`要素にして、`id`を持たせる  
変更箇所は 28行目で、表示される gpio の値を`<span id="gpio_1"></span>`で囲む  
```
1	<!DOCTYPE html>
2	<html lang="ja">
3	<head>
4	  <meta charset="UTF-8">
5	  <title>Raspberry Pi GPIO</title>
6	  <script src="node_modules/chart.js/node_modules/moment/min/moment.min.js"></script>
7	  <script src="node_modules/chart.js/dist/Chart.min.js"></script>
8	  <link rel="stylesheet" href="node_modules/jquery-mobile/dist/jquery.mobile.min.css" />
9	  <script src="node_modules/jquery/dist/jquery.min.js"></script>
10	  <script src="node_modules/jquery-mobile/dist/jquery.mobile.min.js"></script>
11
12	  <link rel="stylesheet" href="node_modules/bootstrap/dist/css/bootstrap.min.css">
13	  <link rel="stylesheet" href="node_modules/bootstrap/dist/css/bootstrap-theme.min.css">
14	  <script src="node_modules/bootstrap/dist/js/bootstrap.min.js"></script>
15	</head>
16	<body>
17	　<div data-role="page">
18	    <div data-role="header" data-position="fixed" data-theme="b" data-disable-page-zoom="false">
19	      <h1>GPIO status</h1>
20	    </div>
21
22	    <div data-roll="content" data-theme="c" class="no-cache">
23
24	      <div class="row">
25
26	        <?php for ($i = 1; $i < 30; $i++): ?>
27	          <div class="col-md-2 col-sm-3 col-xs-6">
28	            <p>GPIO <?= $i ?> = <span id="gpio_<?= $i ?>"><?= rtrim(`sudo gpio read $i`) ?></span></p>
29	          </div>
30	        <?php endfor ?>
31
32	      </div><!-- <div class="row"> -->
33	    </div>
34
35	    <div data-role="footer" data-position="fixed" class="no-cache" data-theme="b">
36	      <h4><?php echo "© Atelier UEDA" ?></h4>
37	    </div>
38	  </div>
39	</body>
40	</html>
```  

展開された html には以下のように `gpio_1`, `gpio_2`, ... という固有の ID が付く  
```
pi@gc1624:/var/www/html/gpio $ php index.php | cat -n
     1	<!DOCTYPE html>
     2	<html lang="ja">
     3	<head>
     4	  <meta charset="UTF-8">
     5	  <title>Raspberry Pi GPIO</title>
     6	  <script src="node_modules/chart.js/node_modules/moment/min/moment.min.js"></script>
     7	  <script src="node_modules/chart.js/dist/Chart.min.js"></script>
     8	  <link rel="stylesheet" href="node_modules/jquery-mobile/dist/jquery.mobile.min.css" />
     9	  <script src="node_modules/jquery/dist/jquery.min.js"></script>
    10	  <script src="node_modules/jquery-mobile/dist/jquery.mobile.min.js"></script>
    11
    12	  <link rel="stylesheet" href="node_modules/bootstrap/dist/css/bootstrap.min.css">
    13	  <link rel="stylesheet" href="node_modules/bootstrap/dist/css/bootstrap-theme.min.css">
    14	  <script src="node_modules/bootstrap/dist/js/bootstrap.min.js"></script>
    15	</head>
    16	<body>
    17	　<div data-role="page">
    18	    <div data-role="header" data-position="fixed" data-theme="b" data-disable-page-zoom="false">
    19	      <h1>GPIO status</h1>
    20	    </div>
    21
    22	    <div data-roll="content" data-theme="c" class="no-cache">
    23
    24	      <div class="row">
    25
    26	                  <div class="col-md-2 col-sm-3 col-xs-6">
    27	            <p>GPIO 1 = <span id="gpio_1">0</span></p>
    28	          </div>
    29	                  <div class="col-md-2 col-sm-3 col-xs-6">
    30	            <p>GPIO 2 = <span id="gpio_2">0</span></p>
    31	          </div>
    32	                  <div class="col-md-2 col-sm-3 col-xs-6">
    33	            <p>GPIO 3 = <span id="gpio_3">0</span></p>
    34	          </div>
    35	                  <div class="col-md-2 col-sm-3 col-xs-6">
    36	            <p>GPIO 4 = <span id="gpio_4">0</span></p>
    37	          </div>
    38	                  <div class="col-md-2 col-sm-3 col-xs-6">
    39	            <p>GPIO 5 = <span id="gpio_5">0</span></p>
    40	          </div>
    41	                  <div class="col-md-2 col-sm-3 col-xs-6">
    42	            <p>GPIO 6 = <span id="gpio_6">0</span></p>
    43	          </div>
    44	                  <div class="col-md-2 col-sm-3 col-xs-6">
    45	            <p>GPIO 7 = <span id="gpio_7">1</span></p>
    46	          </div>
    47	                  <div class="col-md-2 col-sm-3 col-xs-6">
    48	            <p>GPIO 8 = <span id="gpio_8">1</span></p>
    49	          </div>
    50	                  <div class="col-md-2 col-sm-3 col-xs-6">
    51	            <p>GPIO 9 = <span id="gpio_9">1</span></p>
    52	          </div>
    53	                  <div class="col-md-2 col-sm-3 col-xs-6">
    54	            <p>GPIO 10 = <span id="gpio_10">1</span></p>
    55	          </div>
    56	                  <div class="col-md-2 col-sm-3 col-xs-6">
    57	            <p>GPIO 11 = <span id="gpio_11">1</span></p>
    58	          </div>
    59	                  <div class="col-md-2 col-sm-3 col-xs-6">
    60	            <p>GPIO 12 = <span id="gpio_12">0</span></p>
    61	          </div>
    62	                  <div class="col-md-2 col-sm-3 col-xs-6">
    63	            <p>GPIO 13 = <span id="gpio_13">0</span></p>
    64	          </div>
    65	                  <div class="col-md-2 col-sm-3 col-xs-6">
    66	            <p>GPIO 14 = <span id="gpio_14">0</span></p>
    67	          </div>
    68	                  <div class="col-md-2 col-sm-3 col-xs-6">
    69	            <p>GPIO 15 = <span id="gpio_15">1</span></p>
    70	          </div>
    71	                  <div class="col-md-2 col-sm-3 col-xs-6">
    72	            <p>GPIO 16 = <span id="gpio_16">0</span></p>
    73	          </div>
    74	                  <div class="col-md-2 col-sm-3 col-xs-6">
    75	            <p>GPIO 17 = <span id="gpio_17">0</span></p>
    76	          </div>
    77	                  <div class="col-md-2 col-sm-3 col-xs-6">
    78	            <p>GPIO 18 = <span id="gpio_18">1</span></p>
    79	          </div>
    80	                  <div class="col-md-2 col-sm-3 col-xs-6">
    81	            <p>GPIO 19 = <span id="gpio_19">0</span></p>
    82	          </div>
    83	                  <div class="col-md-2 col-sm-3 col-xs-6">
    84	            <p>GPIO 20 = <span id="gpio_20">0</span></p>
    85	          </div>
    86	                  <div class="col-md-2 col-sm-3 col-xs-6">
    87	            <p>GPIO 21 = <span id="gpio_21">1</span></p>
    88	          </div>
    89	                  <div class="col-md-2 col-sm-3 col-xs-6">
    90	            <p>GPIO 22 = <span id="gpio_22">1</span></p>
    91	          </div>
    92	                  <div class="col-md-2 col-sm-3 col-xs-6">
    93	            <p>GPIO 23 = <span id="gpio_23">0</span></p>
    94	          </div>
    95	                  <div class="col-md-2 col-sm-3 col-xs-6">
    96	            <p>GPIO 24 = <span id="gpio_24">0</span></p>
    97	          </div>
    98	                  <div class="col-md-2 col-sm-3 col-xs-6">
    99	            <p>GPIO 25 = <span id="gpio_25">0</span></p>
   100	          </div>
   101	                  <div class="col-md-2 col-sm-3 col-xs-6">
   102	            <p>GPIO 26 = <span id="gpio_26">0</span></p>
   103	          </div>
   104	                  <div class="col-md-2 col-sm-3 col-xs-6">
   105	            <p>GPIO 27 = <span id="gpio_27">0</span></p>
   106	          </div>
   107	                  <div class="col-md-2 col-sm-3 col-xs-6">
   108	            <p>GPIO 28 = <span id="gpio_28">0</span></p>
   109	          </div>
   110	                  <div class="col-md-2 col-sm-3 col-xs-6">
   111	            <p>GPIO 29 = <span id="gpio_29">1</span></p>
   112	          </div>
   113	        
   114	      </div><!-- <div class="row"> -->
   115	    </div>
   116
   117	    <div data-role="footer" data-position="fixed" class="no-cache" data-theme="b">
   118	      <h4>© Atelier UEDA</h4>
   119	    </div>
   120	  </div>
   121	</body>
   122	</html>
```

2. interval loop から ajax で値を取得、ID で取得した dom 要素を書き換える  
16行 - 33行目の`script`を追加した  
18行目以下の $.ajax で、先ほど作成した ajax.php を呼び出し、gpio の値を取得  
26行目で<span></span> を getElementById で取得し、ajax で取得した値で書き換える  
```
1	<!DOCTYPE html>
2	<html lang="ja">
3	<head>
4	  <meta charset="UTF-8">
5	  <title>Raspberry Pi GPIO</title>
6	  <script src="node_modules/chart.js/node_modules/moment/min/moment.min.js"></script>
7	  <script src="node_modules/chart.js/dist/Chart.min.js"></script>
8	  <link rel="stylesheet" href="node_modules/jquery-mobile/dist/jquery.mobile.min.css" />
9	  <script src="node_modules/jquery/dist/jquery.min.js"></script>
10	  <script src="node_modules/jquery-mobile/dist/jquery.mobile.min.js"></script>
11
12	  <link rel="stylesheet" href="node_modules/bootstrap/dist/css/bootstrap.min.css">
13	  <link rel="stylesheet" href="node_modules/bootstrap/dist/css/bootstrap-theme.min.css">
14	  <script src="node_modules/bootstrap/dist/js/bootstrap.min.js"></script>
15
16	  <script>
17	    var iv = setInterval( function() {
18	      $.ajax({
19	        type: "GET",
20	        url:  "ajax.php",
21	      })
22	      .then(
23	        function(data, dataType){
24	          for(var pin in data) {
25	            var value = data[pin];
26	            document.getElementById(pin).innerHTML = String(value);
27	          }
28	        },
29	        function(XMLHttpRequest, textStatus, errorThrown){
30	          console.log('Error : ' + errorThrown);
31	      })
32	    }, 1000 );
33	  </script>
34	</head>
35	<body>
36	　<div data-role="page">
37	    <div data-role="header" data-position="fixed" data-theme="b" data-disable-page-zoom="false">
38	      <h1>GPIO status</h1>
39	    </div>
40
41	    <div data-roll="content" data-theme="c" class="no-cache">
42
43	      <div class="row">
44
45	        <?php for ($i = 1; $i < 30; $i++): ?>
46	          <div class="col-md-2 col-sm-3 col-xs-6">
47	            <p>GPIO <?= $i ?> = <span id="gpio_<?= $i ?>"><?= rtrim(`sudo gpio read $i`) ?></span></p>
48	          </div>
49	        <?php endfor ?>
50
51	      </div><!-- <div class="row"> -->
52	    </div>
53
54	    <div data-role="footer" data-position="fixed" class="no-cache" data-theme="b">
55	      <h4><?php echo "© Atelier UEDA" ?></h4>
56	    </div>
57	  </div>
58	</body>
59	</html>
```  

`自分のgc16のホスト名.local/gpio`をブラウザで再読み込みして、下記のように gpio コマンドで gpio の値をいろいろ変えてみる  
変更が自動的に反映される  
```
pi@gc1624:/var/www/html/gpio $ gpio mode 1 out
pi@gc1624:/var/www/html/gpio $ gpio write 1 1
pi@gc1624:/var/www/html/gpio $ gpio write 1 0
```
