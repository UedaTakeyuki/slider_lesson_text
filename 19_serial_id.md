# 19.Raspberry Pi の Serial ID の確認

## <u>目的</u>
Raspberry Pi のハードウェアはそれぞれ固有の ID をもつ。ソフトウェアから Raspberry Pi の　Serial ID を参照する原理的な方法と、そのための便利な Python パッケージを紹介する

## <u>実習手順</u>
自身の gc16 に terminal でログインする

### getrpimodel
コマンドとして利用
```
pi@gc1624:~ $ python -m piserialnumber
00000000fbaa1f70
```

もしくは python のモジュールとして利用

```
pi@gc1624:~ $ python
Python 2.7.9 (default, Sep 17 2016, 20:26:04)
[GCC 4.9.2] on linux2
Type "help", "copyright", "credits" or "license" for more information.
>>> import piserialnumber
>>> piserialnumber.serial()
'00000000fbaa1f70'
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
pi@gc1624:~ $ sudo find / | grep -e "piserialnumber"
/usr/local/lib/python2.7/dist-packages/piserialnumber-0.1.1.egg-info
/usr/local/lib/python2.7/dist-packages/piserialnumber-0.1.1.egg-info/PKG-INFO
/usr/local/lib/python2.7/dist-packages/piserialnumber-0.1.1.egg-info/installed-files.txt
/usr/local/lib/python2.7/dist-packages/piserialnumber-0.1.1.egg-info/dependency_links.txt
/usr/local/lib/python2.7/dist-packages/piserialnumber-0.1.1.egg-info/SOURCES.txt
/usr/local/lib/python2.7/dist-packages/piserialnumber-0.1.1.egg-info/top_level.txt
/usr/local/lib/python2.7/dist-packages/piserialnumber
/usr/local/lib/python2.7/dist-packages/piserialnumber/__main__.pyc
/usr/local/lib/python2.7/dist-packages/piserialnumber/__init__.pyc
/usr/local/lib/python2.7/dist-packages/piserialnumber/__init__.py
/usr/local/lib/python2.7/dist-packages/piserialnumber/__main__.py
```

3. getrpimodel の実体
```
pi@gc1624:~ $ cat -n pi@gc1624:~ $ cat -n /usr/local/lib/python2.7/dist-packages/piserialnumber/__init__.py
     1	# -*- coding: utf-8 -*-
     2	#
     3	# © Takeyuki UEDA 2016 -.
     4
     5	#import getpirevision
     6	import re
     7
     8
     9	def read_info(index):
    10	  revision = "unknown"
    11	  with open('/proc/cpuinfo', 'r') as f:
    12	    for line in f:
    13	      m = re.search('{}.*: ([0123456789abcdef]*)'.format(index), line)
    14	      if m:
    15	        value  = m.group(1)
    16	        return value
    17
    18	def serial():
    19	  return read_info("Serial")
```  
ポイント　　
  - 11行: /proc/cpuinfo を読む
  - 13行: "Serial" でパターンマッチして、続く文字列を返却

### /proc/cpuinfo の Serial
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
