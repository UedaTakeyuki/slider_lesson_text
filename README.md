# Introduction

このドキュメントは講座 `IoT基礎` の実習テキストです。`IoT基礎` では、Raspberry Pi と、Atelier UEDA が著作権をもつアプリケーション [slider](https://github.com/UedaTakeyuki/slider)、[monitor](https://github.com/UedaTakeyuki/monitor)、[BackupPi_2](https://github.com/UedaTakeyuki/BackupPi_2)、などのオープンソースのプロダクトを利用して IoT システムの仕組みを理解するとともに Raspberry Pi をつかって情報システムのプロトタイプを作るためのノウハウを学びます  



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
gitbook pdf
```

このドキュメントは[源真ゴシック](http://jikasei.me/font/genshin/) を利用させていただき、体裁を確認させていただきました。[こちらの手順](http://backport.net/blog/2016/09/06/pdf_embedded_japanese_font/)を参照させていただき、[cloud9](https://c9.io)に源真ゴシックをインストールして pdf を作成しました
