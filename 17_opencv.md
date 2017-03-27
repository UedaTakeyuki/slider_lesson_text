# 17.OpenCVの利用

##<u>目的</u>
Raspberry Pi OpenCV を利用する方法を説明し、具体例をつかって実際につかってみる。実験や学習の目的で OpenCV の環境を自分の PC に用意しようとすると以外とめんどくさく、Raspberry Pi 上に簡単に環境が構築できればそれは有り難い  
また、Raspberry Pi を使った監視システムで、実際に OpenCV による画像処理が有効に利用できることを体験する  
尚、OpenCV は最新の 3 系を利用する

##<u>実習手順</u>
自身の gc16 に terminal でログインする

### インストール

1. OpenCV のインストールスクリプトは以下  
```
pi@gc1624:~/install $ cd
pi@gc1624:~ $ cat -n /home/pi/install/cv.setup.with_ffmpeg.sh
     1	# http://www.pyimagesearch.com/2016/04/18/install-guide-raspberry-pi-3-raspbian-jessie-opencv-3/
     2	c_path=`pwd`
     3
     4	sudo apt-get install libv4l-dev
     5	cd /usr/include/linux
     6	sudo ln -s ../libv4l1-videodev.h videodev.h
     7	cd ${c_path}
     8
     9	sudo apt-get install build-essential cmake pkg-config
    10	sudo apt-get install libjpeg-dev libtiff5-dev libjasper-dev libpng12-dev
    11	sudo apt-get install libavcodec-dev libavformat-dev libswscale-dev libv4l-dev
    12	sudo apt-get install libxvidcore-dev libx264-dev
    13	sudo apt-get install libgtk2.0-dev
    14	sudo apt-get install libatlas-base-dev gfortran
    15
    16	sudo apt-get install python2.7-dev python3-dev
    17
    18	sudo pip install numpy
    19
    20	wget -O opencv.zip https://github.com/Itseez/opencv/archive/3.1.0.zip
    21	unzip opencv.zip
    22	wget -O opencv_contrib.zip https://github.com/Itseez/opencv_contrib/archive/3.1.0.zip
    23	unzip opencv_contrib.zip
    24
    25	cd opencv-3.1.0
    26	mkdir build
    27	cd build
    28	export PYTHON_INCLUDE_DIRS=/usr/include/python2.7
    29	export PYTHON_LIBRARYS=/usr/lib/arm-linux-gnueabihf/libpython2.7.so
    30	if [ -f CMakeCache.txt ]; then
    31	  rm CMakeCache.txt
    32	fi
    33	cmake -D CMAKE_BUILD_TYPE=RELEASE \
    34	    -D CMAKE_INSTALL_PREFIX=/usr/local \
    35	    -D INSTALL_PYTHON_EXAMPLES=ON \
    36	    -D OPENCV_EXTRA_MODULES_PATH=${c_path}/opencv_contrib-3.1.0/modules \
    37	    -D BUILD_EXAMPLES=ON .. \
    38	    -D INSTALL_PYTHON_EXAMPLES=ON \
    39	    -D BUILD_EXAMPLES=ON \
    40	    -D BUILD_NEW_PYTHON_SUPPORT=ON ..
    41	make -j4
    42	sudo make install
    43	sudo ldconfig
```  
尚、ffmpeg のインストールスクリプトは以下  
```
pi@gc1624:~ $ cat -n /home/pi/install/ffmpeg.setup.sh
     1	c_path=`pwd`
     2	# x264
     3	git clone git://git.videolan.org/x264
     4	cd x264
     5	#./configure --prefix=${c_path}/output --enable-static  --disable-opencl
     6	#./configure --prefix=${c_path}/output --enable-shared --disable-opencl
     7	./configure --enable-shared --disable-opencl
     8	make -j4
     9	sudo make install
    10	cd ..
    11
    12	# ALSA
    13	#wget ftp://ftp.alsa-project.org/pub/lib/alsa-lib-1.1.1.tar.bz2
    14	#tar xjvf alsa-lib-1.1.1.tar.bz2
    15	#cd alsa-lib-1.1.1/
    16	#./configure --prefix=${c_path}/output
    17	#make -j4
    18	#make install
    19	#cd ..
    20
    21	# libfaac
    22	wget  http://downloads.sourceforge.net/project/faac/faac-src/faac-1.28/faac-1.28.tar.bz2
    23	tar  xvfj   faac-1.28.tar.bz2
    24	cd  faac-1.28
    25	#./configure   --prefix=${c_path}/output   --enable-static  --disable-shared  --with-mp4v2
    26	#./configure   --prefix=${c_path}/output   --with-mp4v2
    27	./configure   --with-mp4v2
    28	make -j4
    29	sudo make install
    30	cd ..
    31
    32	# ffmpeg
    33	sudo apt-get install pkg-config yasm lib-dev
    34	git clone https://git.ffmpeg.org/ffmpeg.git ffmpeg
    35	cd ffmpeg
    36	#./configure --enable-gpl --enable-libx264 --enable-nonfree\
    37	#            --enable-static\
    38	#            --extra-cflags="-I${c_path}/output/include"\
    39	#            --extra-ldflags="-L${c_path}/output/lib"\
    40	#            --extra-libs=-ldl
    41	#./configure --enable-gpl --enable-libx264 --enable-nonfree\
    42	#            --extra-cflags="-I${c_path}/output/include"\
    43	#            --extra-ldflags="-L${c_path}/output/lib"\
    44	#            --extra-libs=-ldl
    45	./configure --enable-gpl --enable-libx264 --enable-nonfree --extra-libs=-ldl
    46	make -j4
    47	sudo make install
```

2. opencv のインストール先は以下  
```
pi@gc1624:~ $ ls /home/pi/install/opencv-3.1.0/
3rdparty  cmake           CONTRIBUTING.md  doc      LICENSE  platforms  samples
apps      CMakeLists.txt  data             include  modules  README.md
```

3. チュートリアル、サンプル  
```
pi@gc1624:~ $ ls install/opencv-3.1.0/doc/tutorials/
calib3d     gpu      imgproc       ml         tutorials.markdown
core        highgui  introduction  objdetect  video
features2d  images   ios           photo      viz
pi@gc1624:~ $ ls install/opencv-3.1.0/samples/
android         cpp   directx  hal   opencl  python  va_intel  winrt_universal
CMakeLists.txt  data  gpu      java  opengl  tapi    winrt     wp8
```

4. カスケードファイル  
```
pi@gc1624:~ $ ls install/opencv-3.1.0/data
CMakeLists.txt  haarcascades_cuda  lbpcascades  vec_files
haarcascades    hogcascades        readme.txt
```

### 顔認識
1. 自分の顔の写った写真を webcam で撮影する。slider によって毎分 `/boot/DATA/video0/` の今日の日付のフォルダの下に保存される撮影写真を利用してもよいし、`fswebcam` で撮影してもよい

2. /boot/DATA 配下に移動  
```
pi@gc1624:~ $ cd /boot/DATA
pi@gc1624:/boot/DATA $
```

3. 撮影した写真で facedetect.py を実行  
```
pi@gc1624:/boot/DATA $ sudo python /home/pi/SCRIPT/cv/facedetect.py 顔写真.jpg
get face
```

4. 顔が認識できていれば /boot/DATA に detectedface.jpg というファイルができている。RPi をshutdown して SD カードを RPi から抜き USB SD reader/writer で PC に挿す

5. PC で /boot/DATA/detectedface.jpg を開く

6. SD カードを RPi に戻し、facedetect のコードを参照  
```
pi@gc1624:~ $ cat -n /home/pi/SCRIPT/cv/facedetect.py
     1	#coding:utf-8
     2
     3	import cv2
     4	import sys
     5
     6	#顔探索用のカスケード型分類器を取得
     7	#haarcascade_frontalface_default.xmlのパスを渡す
     8	face_cascade = cv2.CascadeClassifier("/home/pi/install/opencv-3.1.0/data/haarcascades/haarcascade_frontalface_default.xml")
     9
    10	file = sys.argv[1]
    11
    12	img = cv2.imread(file)
    13
    14	#読み込んだ画像をグレースケールに変換
    15	gray = cv2.cvtColor(img,cv2.COLOR_RGB2GRAY)
    16
    17	#分類器で顔を認識する
    18	face = face_cascade.detectMultiScale(gray,1.3,5)
    19
    20	if 0 < len(face):
    21
    22	    print "get face"
    23
    24	    for (x,y,w,h) in face:
    25		#the input to cv.HaarDetectObjects was resized, scale the
    26	      	#bounding box of each face and convert it to two CvPoints
    27	      	pt1 = (int(x), int(y))
    28	      	pt2 = (int((x + w)), int((y + h)))
    29	      	cv2.rectangle(img, pt1, pt2, (255, 0, 0), 3, 8, 0)
    30	    cv2.imwrite("detectedface.jpg",img)
    31	else:
    32
    33	    print "no face"
```  
ポイントは以下  
  - 8行: 正面顔のカスケードファイルを読み込む
  - 15行: 対象の写真を白黒に変換
  - 18行: 顔認識の実体、見つかった数だけ顔の座標情報がリストとして face に保存される
  - 29行: 顔の周りを青い四角で囲う
  - 30行: "detectface.jpg" ファイルに結果を出力


### 顔にモザイクを掛ける
顔が認識できる写真は個人情報保護法の個人識別符号そのものなので、その保管には個人情報保護法の求める安全管理措置等の各種責務が発生する。そこで、個人を特定できないように撮影画像の公開前に機会的にモザイクをかる  
尚、モザイクを掛ける事を "pixelate" という  
また、opencv に pixcelate の関数は用意されていないので、工夫して実装する

1. /boot/DATA 配下に移動して、先程の撮影した自分の写真で再度、以下のコマンドを実行する  
```
pi@gc1624:~ $ cd /boot/DATA
pi@gc1624:/boot/DATA $
```

2. 撮影した写真で facepixelate.py を実行  
```
pi@gc1624:/boot/DATA $ sudo python /home/pi/SCRIPT/cv/facepixelate.py 顔写真.jpg
get face
```

3. 顔が認識できていれば /boot/DATA に mosaic.jpg というファイルができている。RPi をshutdown して SD カードを RPi から抜き USB SD reader/writer で PC に挿し、PC で開く

4. SD カードを RPi に戻し、facedetect のコードを参照  
```
pi@gc1624:~ $ cat -n /home/pi/SCRIPT/cv/facepixelate.py
     1	#coding:utf-8
     2	# http://qiita.com/lrf141/items/ff1462c5c6b7b3207775
     3
     4	import numpy as np
     5	import cv2
     6	import sys
     7
     8	#顔探索用のカスケード型分類器を取得
     9	#haarcascade_frontalface_default.xmlのパスを渡す
    10	face_cascade = cv2.CascadeClassifier("/home/pi/install/opencv-3.1.0/data/haarcascades/haarcascade_frontalface_default.xml")
    11
    12	file = sys.argv[1]
    13
    14	img = cv2.imread(file)
    15	result = cv2.imread(file)
    16
    17	#読み込んだ画像をグレースケールに変換
    18	gray = cv2.cvtColor(img,cv2.COLOR_RGB2GRAY)
    19
    20	#分類器で顔を認識する
    21	face = face_cascade.detectMultiScale(gray,1.3,5)
    22
    23	if 0 < len(face):
    24
    25	    print "get face"
    26
    27	    for (x,y,w,h) in face:
    28
    29	        #顔の部分だけ切り抜いてモザイク処理をする
    30	        cut_img = img[y:y+h,x:x+w]
    31	        cut_face = cut_img.shape[:2][::-1]
    32	        #10分の1にする
    33	        cut_img = cv2.resize(cut_img,(cut_face[0]/10, cut_face[0]/10))
    34	        #画像を元のサイズに拡大
    35	        cut_img = cv2.resize(cut_img,cut_face,interpolation = 0)
    36
    37	        #モザイク処理した部分を重ねる
    38	        result[y:y+h,x:x+w] = cut_img
    39	        cv2.imwrite("mosaic.jpg",result)
    40	else:
    41
    42	    print "no face"
```  
基本的な構造は先程の facedetect.py と同じ  
ポイントは以下  
  - 32 - 38行: 顔部分の画像サイズを 1/10 に縮小し、それを元のサイズに拡大することでモザイクを実装
