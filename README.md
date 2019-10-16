# pi-face-box-door
刷脸开门模型设计

### 一、项目环境搭建流程 ####

##### 1.安装`Opencv`所需的依赖环境 #####
利用ssh工具或图形界面的方式打开`Raspberry pi`终端更新后，复制下面的代码回车下载.
```
// 项目均在`Raspberry pi`环境搭建，编译
sudo apt-get update
sudo apt-get install -y build-essential
cmake libgtk2.0-dev pkg-config python-numpy
python-dev libavcodec-dev libavformat-dev
libswscale-dev libjpeg-dev libpng-dev
libtiff-dev libjasper-dev
```
下面是`OpenCV`人脸识别库所需的依赖包
```
sudo apt-get -qq install libopencv-dev
build-essential checkinstall cmake pkg-config
yasm libjpeg-dev libjasper-dev libavcodec-dev
libavformat-dev libswscale-dev libdc1394-22-dev
libxine-dev libgstreamer0.10-dev libgstreamer-plugins-base0.10-dev
libv4l-dev python-dev python-numpy libtbb-dev libqt4-dev libgtk2.0-dev libmp3lame-dev
libopencore-amrnb-dev libopencore-amrwb-dev libtheora-dev libvorbis-dev libxvidcore-dev
x264 v4l-utils
```
下载 opencv-2.4.10
```
wget http://downloads.sourceforge.net/project/opencvlibrary/opencv-unix/2.4.10/opencv-2.4.10.zip
```
对下载包进行解包
```
unzip opencv-2.4.10.zip
cd opencv-2.4.10 # 进入解压后的OpenCV目录
```
编译和安装
```
cmake -D CMAKE_BUILD_TYPE=RELEASE
-D CMAKE_INSTALL_PREFIX=/usr/local
-D WITH_TBB=ON -D BUILD_NEW_PYTHON_SUPPORT=ON
-D WITH_V4L=ON -D INSTALL_C_EXAMPLES=ON
-D INSTALL_PYTHON_EXAMPLES=ON
-D BUILD_EXAMPLES=ON -D WITH_QT=ON
-D WITH_OPENGL=ON .
```
```
make -j4 #Raspberry pi 四核全开,提高编译时间
sudo make install
```
此过程所需时间大概要 5 个小时左右,所以可以在编译期间去做任何事。
##### 2.人脸识别所需的`python`依赖环境 #####
#### 二、模型训练 ####
利用软件中的`capture-positives.py`脚本生成能够打开门的图像,在`raspberrypi`中运行此脚本将拍摄的图片写入`training/positive`子目录中。`raspberry pi`上电后,`ssh`连接到树莓派,导航到脚本所在的目录,执行下面的命令:
```
sudo python captuure-positives.py
```
捕获正面训练图像,键入`c`(然后回车)捕获图像,`Ctrl-C`退出。
捕获期间拍摄不同的脸部照片,在不同光照下捕获不同的照片会提高各种场合的识别能力。下面是我捕获的图像。
`Ctrl -C`退出`capture-positives.py`脚本,运行以下脚本来训练面部识别模型:
```
python train.py
```
等待一段时间后训练数据存储在`training.xml`,用于配置软件加载配置面部识别模型。
私服电机的配置调整
利用管理员权限进入`python shell`控制台 :
```
sudo python
```
此步骤用于确定盒子的锁定和解锁伺服位置的脉冲宽度值。
```
>>>from RPIO import PWM # 导入伺服电机调整所需的包
>>>servo = PWM.Servo()
>>> servo.set_servo(18,1500) # 此命令用调整伺服电机的位置,他会移动到中心位置(或者说在1000~2000 之间)
>>> quit()
```
#### 三.项目测试
最后进行项目的测试,确保已经完成硬件组装和模型培训,运行以下的命令:
```
sudo python box.py
```
你可以看到这样的输出,说明已经成功了:
```
Running box...
Press button or type c to lock (if unlocked), or unlock if the correct face is detecte
Press Ctrl-C to quit.
```
在键盘上键入`C`用于锁定和解锁,成功解锁及锁定。
