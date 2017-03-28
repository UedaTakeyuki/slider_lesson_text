# 18.Raspberry Pi のバージョンチェック

##<u>目的</u>
Raspberry Pi の構成や挙動はハードウェアバージョンによって違いがある。その違いをアプリケーションで吸収するためにはソフトウェアからバージョンを知ることが必要になるので、その原理的な方法と、そのための便利な Python パッケージを紹介する

##<u>実習手順</u>
自身の gc16 に terminal でログインする

### getrpimodel
コマンドとして利用
```
pi@gc1624:~ $ python -m getrpimodel
3 Model B
```

もしくは python のモジュールとして利用

```
pi@gc1624:~ $ python
Python 2.7.9 (default, Sep 17 2016, 20:26:04)
[GCC 4.9.2] on linux2
Type "help", "copyright", "credits" or "license" for more information.
>>> import getrpimodel
>>> print getrpimodel.model()
3 Model B
```

### getrpimodel の仕組み
1. インストール済の pypi パッケージの一覧  
```
pi@gc1624:~ $ pip list
Adafruit-BBIO (1.0.0)
Adafruit-GPIO (1.0.1)
Adafruit-PureIO (0.2.1)
Adafruit-SSD1306 (1.6.1)
argparse (1.2.1)
chardet (2.3.0)
colorama (0.3.2)
configparser (3.5.0)
funcsigs (1.0.2)
funniest (0.1)
gaugette (1.2)
gdata (2.0.18)
getrpimodel (0.1.9)
gyp (0.1)
html5lib (0.999)
mock (2.0.0)
ndg-httpsclient (0.3.2)
numpy (1.11.2)
paho-mqtt (1.2)
pbr (1.10.0)
pip (1.5.6)
piserialnumber (0.1.1)
protobuf (3.0.0b2)
pyasn1 (0.1.7)
pyOpenSSL (0.13.1)
pyserial (3.2.1)
python-apt (0.9.3.12)
pytoml (0.1.11)
redis (2.10.5)
requests (2.4.3)
RPi.GPIO (0.6.2)
setuptools (5.5.1)
six (1.10.0)
smbus (1.1)
spidev (3.3)
subprocess32 (3.2.7)
tensorflow (0.10.0)
urllib3 (1.9.1)
wheel (0.24.0)
wiringpi (2.32.1)
wiringpi2 (1.1.1)
wsgiref (0.1.2)
pi@gc1624:~ $
```

2. getrpimodel の保存場所  
```
pi@gc1624:~ $ sudo find / | grep -e "getrpimodel"
/usr/local/lib/python2.7/dist-packages/getrpimodel
/usr/local/lib/python2.7/dist-packages/getrpimodel/__main__.pyc
/usr/local/lib/python2.7/dist-packages/getrpimodel/__init__.pyc
/usr/local/lib/python2.7/dist-packages/getrpimodel/getrpimodel.py
/usr/local/lib/python2.7/dist-packages/getrpimodel/getrpimodel.pyc
/usr/local/lib/python2.7/dist-packages/getrpimodel/__init__.py
/usr/local/lib/python2.7/dist-packages/getrpimodel/__main__.py
/usr/local/lib/python2.7/dist-packages/getrpimodel-0.1.9.egg-info
/usr/local/lib/python2.7/dist-packages/getrpimodel-0.1.9.egg-info/PKG-INFO
/usr/local/lib/python2.7/dist-packages/getrpimodel-0.1.9.egg-info/installed-files.txt
/usr/local/lib/python2.7/dist-packages/getrpimodel-0.1.9.egg-info/dependency_links.txt
/usr/local/lib/python2.7/dist-packages/getrpimodel-0.1.9.egg-info/SOURCES.txt
/usr/local/lib/python2.7/dist-packages/getrpimodel-0.1.9.egg-info/top_level.txt
```

3. getrpimodel の実体
```
pi@gc1624:~ $ cat -n /usr/local/lib/python2.7/dist-packages/getrpimodel/__init__.py
     1	# -*- coding: utf-8 -*-
     2	#
     3	# © Takeyuki UEDA 2016 -.
     4
     5	#import getpirevision
     6	import re
     7	import sys
     8
     9
    10	usage = 'Usage: python -m getrpimodel [--s]'
    11
    12	# model definition table from revision info.
    13	# refer http://elinux.org/RPi_HardwareHistory
    14	model_a          = ["0007","0008","0009",]
    15	model_b          = ["0002","0004","0005","0006","000d","000e","000e",]
    16	model_b_beta     = ["Beta",]
    17	model_b_ECN0001  = ["0003",]
    18	model_cm         = ["0011","0014",]
    19	model_cm3        = ["a020a0",]
    20	model_a_plus     = ["0012","0015","900021",]
    21	model_b_plus     = ["0010","0013",]
    22	model_2b         = ["a01040","a01041","a21041",]
    23	model_2b_2837    = ["a22042",]
    24	model_3b         = ["a02082", "a22082","a32082",]
    25	model_zero       = ["900092","900093","920093",]
    26
    27	def revision():
    28	  revision = "unknown"
    29	  with open('/proc/cpuinfo', 'r') as f:
    30	    for line in f:
    31	      m = re.search('Revision.*: ([0123456789abcdef]*)', line)
    32	      if m:
    33	        revision = m.group(1)
    34	        return revision
    35
    36	def model_strict():
    37	#  rev = getpirevision.revision()
    38	  rev = revision()
    39	  if rev in model_a:
    40	    return "A"
    41	  elif rev in model_b:
    42	    return "B"
    43	  elif rev in model_b_beta:
    44	    return "B (Beta)"
    45	  elif rev in model_b_ECN0001:
    46	    return "B (ECN0001)"
    47	  elif rev in model_cm:
    48	    return "Compute Module"
    49	  elif rev in model_cm3:
    50	    return "Compute Module 3(and CM3 Lite)"
    51	  elif rev in model_a_plus:
    52	    return "A+"
    53	  elif rev in model_b_plus:
    54	    return "B+"
    55	  elif rev in model_2b:
    56	    return "2 Model B"
    57	  elif rev in model_2b_2837:
    58	    return "2 Model B (with BCM2837)"
    59	  elif rev in model_3b:
    60	    return "3 Model B"
    61	  elif rev in model_zero:
    62	    return "Zero"
    63	  else:
    64	    return rev
    65	#    return None
    66
    67	def model():
    68	#  rev = getpirevision.revision()
    69	  rev = revision()
    70	  if rev in model_a:
    71	    return "A"
    72	  elif rev in model_b + model_b_beta + model_b_ECN0001:
    73	    return "B"
    74	  elif rev in model_cm + model_cm3:
    75	    return "Compute Module"
    76	  elif rev in model_a_plus:
    77	    return "A+"
    78	  elif rev in model_b_plus:
    79	    return "B+"
    80	  elif rev in model_2b + model_2b_2837:
    81	    return "2 Model B"
    82	  elif rev in model_3b:
    83	    return "3 Model B"
    84	  elif rev in model_zero:
    85	    return "Zero"
    86	  else:
    87	    return rev
    88	#    return None
    89
    90	if __name__ == '__main__':
    91	  if len(sys.argv) == 1:
    92	    print (model())
    93	  elif len(sys.argv) == 2:
    94	    if sys.argv[1] == '--s':
    95	      print (model_strict())
    96	    else:
    97	      print usage
    98	  else:
    99	    print usage
```  
ポイント　　
  - 29行: /proc/cpuinfo を読む
  - 31行: "Revision" でパターンマッチして、続く文字列を値に
  - 67 - 87行: revision の値でバージョンをチェック

### /proc/cpuinfo の Revision
```
pi@gc1624:~ $ cat /proc/cpuinfo
processor	: 0
model name	: ARMv7 Processor rev 4 (v7l)
BogoMIPS	: 76.80
Features	: half thumb fastmult vfp edsp neon vfpv3 tls vfpv4 idiva idivt vfpd32 lpae evtstrm crc32
CPU implementer	: 0x41
CPU architecture: 7
CPU variant	: 0x0
CPU part	: 0xd03
CPU revision	: 4

processor	: 1
model name	: ARMv7 Processor rev 4 (v7l)
BogoMIPS	: 76.80
Features	: half thumb fastmult vfp edsp neon vfpv3 tls vfpv4 idiva idivt vfpd32 lpae evtstrm crc32
CPU implementer	: 0x41
CPU architecture: 7
CPU variant	: 0x0
CPU part	: 0xd03
CPU revision	: 4

processor	: 2
model name	: ARMv7 Processor rev 4 (v7l)
BogoMIPS	: 76.80
Features	: half thumb fastmult vfp edsp neon vfpv3 tls vfpv4 idiva idivt vfpd32 lpae evtstrm crc32
CPU implementer	: 0x41
CPU architecture: 7
CPU variant	: 0x0
CPU part	: 0xd03
CPU revision	: 4

processor	: 3
model name	: ARMv7 Processor rev 4 (v7l)
BogoMIPS	: 76.80
Features	: half thumb fastmult vfp edsp neon vfpv3 tls vfpv4 idiva idivt vfpd32 lpae evtstrm crc32
CPU implementer	: 0x41
CPU architecture: 7
CPU variant	: 0x0
CPU part	: 0xd03
CPU revision	: 4

Hardware	: BCM2709
Revision	: a32082
Serial		: 00000000fbaa1f70
``` 
