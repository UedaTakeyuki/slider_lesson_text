# SDカードの特徴

##<u>目的</u>
Raspberry Pi は BCM

##<u>実習手順</u>
グラフィカルなパーティションエディタである gparted を使い、Raspberry Pi の SD カードの構造を理解するとともに、gparted を使ったパーティションの操作を実習する。  

### gparted の起動
1. SD カードを ***gc15*** で Raspberry Pi を起動する
2. USB 外付け SD カードリーダーを使って Jessie Lite の SD カードを Raspberry Pi の USB ポートに装着する
3. RemoteDesktop で自身の Raspberry Pi に接続し、[Raspberry ICON] - [Preferences] - [GParted] を起動  
<img src="pic/ss.2017-03-10 14.52.00.png" width="75%">
4. ログインが求められるので gc15 の pi アカウントでログイン（初期PW: gc15pw）  
<img src="pic/ss.2017-03-10 14.52.25.png" width="75%">
5. デフォルトで、起動中のSDカードのパーティションが表示される
<img src="pic/ss.2017-03-10 14.52.53.png" width="75%">  
  - 起動中のSDカードは /dev/mmcblk0 にマウントされている
  - USB外付けのSDカードは /dev/sda にマウントされている
  - 起動中のSDカードは unmount できないので操作はできない
6. 右上のストレージ選択メニューで /dev/sda を選択すると、USB 外付けSDカードのパーティションが下記のように表示される
<img src="pic/ss.2017-03-10 14.53.09.png" width="75%">  
  - 二つのパーティションと前後のギャップから構成されている
    - 最初のパーティションは FAT32でフォーマットされた 63MByte
    - 次のパーティションは ext4 でフォーマットされた 1.79GByte
  - 後ろの16MByte のギャップは、普通に Raspberry Pi の起動 SD カードを用意すると発生しない。バックアップ&リストアの利便性のために弊所では常に ***意図的に*** このギャップを入れている

###（任意）パーティションのサイズの変更
Jessie lite の ext4 のパーティションはまだ空きがあるのでサイズを少し小さくする

1. 先の課題の続き、***gc15*** で起動し、jessie liet の SD を USB でマウントし、gparted で /dev/sda が選択されている状態からスタート

2. partition /dev/sda2 を選択して右クリック、現れたポップアップメニューから "Unmount" を選択し、ext4 の /dev/sda2 を unmount する
<img src="pic/ss.2017-03-10 18.17.03.png" width="75%">  

3. 再度 /dev/sda2 を選択して右クリック、ポップアップメニューから "Resize/Move" を選択
<img src="pic/ss.2017-03-10 18.17.29.png" width="75%">  

4. サイズ変更のダイアログが表示されるので、"Free Space Following" を少しふやしてダイアログ右下の "Resize/Move" ボタンをクリック
<img src="pic/ss.2017-03-10 18.18.23.png" width="75%">  

5.緑色のチェックマークのボタンをクリックしてリサイズを実行。処理に少し時間がかかるがリサイズが実行される
<img src="pic/ss.2017-03-10 18.18.46.png" width="75%">  


### (任意) parted
gparted の CUI版である parted も利用できる
```
man parted
```
で使い方を確認し、起動中の SD カードの情報を確認し、Jessie Lite の SD カードのパーティションを操作してみる


##<u>ポイント解説</u>
1. 因に、gparted の "g" は Gnome の G
