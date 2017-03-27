# 16.USB WebCamの利用

##<u>概要</u>
Raspberry Pi では財団公式の高速高画質なカメラを利用することも、特殊な Serial Camera を利用することも、安価な USB WebCam を利用することもできる。USB WebCam と Linux 間のインターフェースは USB のデバイスクラスの一つ UVC（USB Video Class) を介して Linux 上の標準のドライバでおこなわれる。そのため、USB WebCam の利用には特別なドライバや設定は不要で、単に USB にカメラを挿せば自動的に認識され、`/dev/video0` 等のデバイスとしてすぐに利用できる  

##<u>実習手順</u>
