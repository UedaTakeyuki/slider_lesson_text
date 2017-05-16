# Introduction

このドキュメントは IoT の基礎を学ぶための実習例題集です  
Raspberry Pi、安価なセンサや表示デバイス、及び Raspberry Pi 上のアプリケーションソフトウェア [slider](https://github.com/UedaTakeyuki/slider)、[monitor](https://github.com/UedaTakeyuki/monitor)、[BackupPi_2](https://github.com/UedaTakeyuki/BackupPi_2)、等を使って
IoT システムの仕組みを理解し、並びに IoT にとどまらず Raspberry Pi をつかったいろいろな情報システムのプロトタイプを安価かつ迅速に作るためのノウハウを提供します 



## The body of this documents.
ドキュメントの本体は[こちら](SUMMARY.md)になります  


## How to make a PDF contents of this documents.

このドキュメントは gitbook を使って以下の手順で PDF を作成することができます  

1. [gitbook](https://www.gitbook.com)のコマンドラインツールをインストール
```
sudo npm install gitbook-cli -g
```

2. [calibre](http://calibre-ebook.com/download_linux)をインストール
```
sudo -v && wget -nv -O- https://download.calibre-ebook.com/linux-installer.py | sudo python -c "import sys; main=lambda:sys.stderr.write('Download failed\n'); exec(sys.stdin.read()); main()"
```

3. [このプロジェクト](https://github.com/UedaTakeyuki/slider_lesson_text)を clone
```
git clone git@github.com:UedaTakeyuki/slider_lesson_text.git
```

4. gitbook で book.pdf を作成
```
cd slider_lesson_text
make
```

このドキュメントは[源真ゴシック](http://jikasei.me/font/genshin/) を利用させていただき、体裁を確認させていただきました  
[こちらの手順](http://backport.net/blog/2016/09/06/pdf_embedded_japanese_font/)を参照させていただき、[cloud9](https://c9.io)に源真ゴシックをインストールして pdf を作成しました

## From where can I download the PDF.

上記、ビルド済 PDF の release version は[こちら](https://github.com/UedaTakeyuki/slider_lesson_text/releases)からダウンロードが可能です