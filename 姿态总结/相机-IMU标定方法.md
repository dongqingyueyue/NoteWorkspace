Camera IMU联合标定方法
---------------

本文档介绍标定开源工具Kalibr的编译安装，以及联想设备camera-IMU的联合标定方法。相机标定的程序和IMU的参数标定方法不包含在此文档中。

[TOC]
------



## 1.编译安装标定kalibr

### 首先安装ROS 

具体版本可参考下表：

| ROS发布日期 | ROS版本      | 对应Ubutnu版本                           |
| ------- | ---------------- | ---------------------------------------- |
| 2016.3  | ROS Kinetic Kame | Ubuntu 16.04 (Xenial) / Ubuntu 15.10 (Wily) |
| 2015.3  | ROS Jade Turtle  | Ubuntu 15.04 (Wily) / Ubuntu LTS 14.04 (Trusty) |
| 2014.7  | ROS Indigo Igloo | Ubuntu 14.04 (Trusty)                    |
| ...     | ...              | ...                                      |

- ROS Indigo，安装步骤：http://wiki.ros.org/indigo/Installation/Ubuntu

- ROS Kinetic，安装步骤：http://wiki.ros.org/kinetic/Installation/Ubuntu

以下安装过程都以Ubuntu 16.04 LTS + ROS Kinetic为例，如果是其他版本则把出现kinetic的地方替换成对应版本代号indigo, lunar等。

首先执行：

```
$ sudo sh -c 'echo "deb http://packages.ros.org/ros/ubuntu $(lsb_release -sc) main" > /etc/apt/sources.list.d/ros-latest.list'
```

设置秘钥:

```
$ sudo apt-key adv --keyserver hkp://ha.pool.sks-keyservers.net:80 --recv-key 0xB01FA116
```

确保系统软件处于最新版

```
$ sudo apt-get update
```

然后安装 ROS ， ROS kinetic 也有很多版本，例如全功能版 -desktop-full， 安装命令：

```
$ sudo apt-get install ros-kinetic-desktop-full
```

或者按照https://github.com/ethz-asl/kalibr/wiki/installation 安装desktop版本：

```
$ sudo apt-get install ros-kinetic-desktop python-rosinstall python-rosdep -y
```

安装完成后，可以用下面的命令来查看可使用的包：

```
$ apt-cache search ros-kinetic
```

初始化 rosdep：

```
$ sudo rosdep init
$ rosdep update
```

然后初始化环境变量:

```
$ echo "source /opt/ros/kinetic/setup.bash" >> ~/.bashrc
$ source ~/.bashrc
```

上面两句非常重要，使用中找不到 Package,或node, 很多情况下都是没有添加source。

安装插件：

```
$ sudo apt-get install python-rosinstall
```

测试是否安装成功，启动ROS环境

  ```
$ roscore
  ```

如果显示 started core service [/rosout]，表示安装成功。

安装更新常用插件，包括catkin等 ：

```
$ sudo apt-get install python-setuptools python-rosinstall ipython libeigen3-dev libboost-all-dev doxygen libopencv-dev ros-kinetic-vision-opencv ros-kinetic-image-transport-plugins ros-kinetic-cmake-modules python-software-properties software-properties-common libpoco-dev python-matplotlib python-scipy python-git python-pip ipython libtbb-dev libblas-dev liblapack-dev python-catkin-tools libv4l-dev 

sudo pip install python-igraph --upgrade
```
如果python-igraph安装不成功，则可能要先更新pip：
```
$ pip install --upgrade pip
```
### 然后编译安装kalibr

源码工程在 https://github.com/ethz-asl/kalibr，编译方法参见https://github.com/ethz-asl/kalibr/wiki/installation。关键步骤如下：

1.先建workspace:

```
$ mkdir -p ~/kalibr_workspace/src 
$ cd ~/kalibr_workspace 
$ source /opt/ros/kinetic/setup.bash 
$ catkin init 
$ catkin config --extend /opt/ros/kinetic 
$ catkin config --cmake-args -DCMAKE_BUILD_TYPE=Release
```

2.Clone 源码到src  
```
$ cd ~/kalibr_workspace/src 
$ git clone https://github.com/ethz-asl/Kalibr.git
```
3.Build工程，使用 -j4还是 -j2取决于内存大小 

```
$ cd ~/kalibr_workspace 
$ catkin build -DCMAKE_BUILD_TYPE=Release -j4
```
4. 编译结束后初始化环境(在新打开的终端窗口中执行kalibr程序时，都需要先执行它。或者一劳永逸把这个命令加到 ~/.bashrc文件最后)

```
$ source ~/kalibr_workspace/devel/setup.bash
```
37个程序编译成功后可以查看其中calibrate_imu标定程序的帮助：
```
$ kalibr_calibrate_imu_camera -h
```



## 2.Camera-IMU标定方法

标定的具体方法参见：https://github.com/ethz-asl/kalibr/wiki/camera-imu-calibration

- 数据下载页面 https://github.com/ethz-asl/kalibr/wiki/downloads，下载[IMU-camera calibration](https://drive.google.com/file/d/0B0T1sizOvRsUcGpTWUNTRC14RzA/edit?usp=sharing)数据。


- 进入数据目录/dynamic，执行标定程序(可先执行kalibr_calibrate_imu_camera -h 查看使用说明)：

```
$ kalibr_calibrate_imu_camera --target april_6x6.yaml --cam camchain.yaml --imu imu_adis16448.yaml --bag dynamic.bag --bag-from-to 5 45
```

### 四个输入参数文件说明

(1).april_6x6.yaml是打印的标定板上Apriltag的描述(图案下载[Calibration targets](https://github.com/ethz-asl/kalibr/wiki/downloads))：
 ```
  target_type: 'aprilgrid'    #gridtype
  tagCols: 6                  #apriltags列数
  tagRows: 6                  #apriltags行数
  tagSize: 0.088              #一个apriltag的边长[m]
  tagSpacing: 0.3             #两个tag之间的空白和tag边长之比
 ```

(2).camchain.yaml 是相机的内参(针对不同设备，内参可以由另外的程序给出，比如调用cv::fisheye::calibrate函数)。

(3).imu_adis16448.yaml文件是IMU的参数描述，有些是出厂标定的，有些则需要自己标定(阿兰方差等标定方法另文给出)：

```
rostopic: /imu0
update_rate: 200.0 #Hz

accelerometer_noise_density: 0.01 #continous
accelerometer_random_walk: 0.0002 
gyroscope_noise_density: 0.005 #continous
gyroscope_random_walk: 4.0e-06
```

(4).dynamic.bag是图像序列文件的ROS包，图像和IMU数据压包和解包的命令分别是(参见https://github.com/ethz-asl/kalibr/wiki/bag-format)：

```
$ kalibr_bagcreater 
$ kalibr_bagextractor 
```

### 三个输出参数文件说明

(1).camchain-imucam-dynamic.yaml，存储多个相机各自的内参(intrinsics)，畸变函数(distortion_coeffs)，与IMU的空间关系(T_cam_imu)。以及相机之间的外参(T_cn_cnm1)：

```
cam0:
  T_cam_imu:
  - [0.016802056935728393, 0.9998586414621272, -0.0006228773858554766, 0.06847911180258197]
  - [-0.9998587108544209, 0.016802362616013256, 0.0004888139270591795, -0.014728977934088073]
  - [0.0004992106407396166, 0.0006145763006084033, 0.9999996865423043, -0.0037698791245958743]
  - [0.0, 0.0, 0.0, 1.0]
      cam_overlaps: [1, 3]
        camera_model: pinhole
        distortion_coeffs: [-0.0016509958435871643, 0.02437222940989351, -0.03582816956989852,
    0.019860839087717054]
  distortion_model: equidistant
  intrinsics: [461.487246372674, 460.1113992557959, 356.39105303227853, 231.15719697054647]
  resolution: [752, 480]
  rostopic: /cam0/image_raw
cam1:
  T_cam_imu:
  - [0.01561258484767336, 0.9998767634499093, -0.0016447210426646455, -0.04170223058316992]
  - [-0.9998773625268496, 0.015614571576060277, 0.0012021068366715106, -0.015134694060546424]
  - [0.0012276403076154584, 0.0016257513432484337, 0.9999979249137694, -0.0035685370710342766]
  - [0.0, 0.0, 0.0, 1.0]
      T_cn_cnm1:
  - [0.9999987703316412, 0.00118910902935917, -0.0010224259962834177, -0.11016759824092018]
  - [-0.0011883808160330714, 0.9999990400088908, 0.0007125533866886034, -0.0003216647632433766]
  - [0.0010232723184295653, -0.0007113374790444546, 0.9999992234560732, 0.00012078907255410462]
  - [0.0, 0.0, 0.0, 1.0]
      cam_overlaps: [0, 3]
        camera_model: pinhole
        distortion_coeffs: [-0.0009362378060020789, 0.018833308358932984, -0.030558453797100132,
    0.01955083559432553]
  distortion_model: equidistant
  intrinsics: [462.4318044040118, 461.1780497604126, 377.0119530476368, 226.49966248854923]
  resolution: [752, 480]
  rostopic: /cam1/image_raw
```



(2).results-imucam-dynamic.txt，除了camera和IMU的空间关系，cam和IMU的时间延迟timeshift，还有标定残差：

```
Calibration results
===================
Normalized Residuals
----------------------------
Reprojection error (cam0):     mean 0.142975135476, median 0.13069970904, std: 0.0818146701439
Reprojection error (cam1):     mean 0.139121604028, median 0.129092886162, std: 0.0768636278959
Gyroscope error (imu0):        mean 0.0953367763832, median 0.0828976336076, std: 0.0612208496161
Accelerometer error (imu0):    mean 0.362877924339, median 0.285766577555, std: 0.341494488515

Residuals
----------------------------
Reprojection error (cam0) [px]:     mean 0.142975135476, median 0.13069970904, std: 0.0818146701439
Reprojection error (cam1) [px]:     mean 0.139121604028, median 0.129092886162, std: 0.0768636278959
Gyroscope error (imu0) [rad/s]:     mean 0.0067413281077, median 0.00586174788682, std: 0.00432896779135
Accelerometer error (imu0) [m/s^2]: mean 0.0513186882086, median 0.0404134969652, std: 0.0482946137133

Transformation (cam0):
-----------------------
T_ci:  (imu0 to cam0): 
[[ 0.01680206  0.99985864 -0.00062288  0.06847911]
 [-0.99985871  0.01680236  0.00048881 -0.01472898]
 [ 0.00049921  0.00061458  0.99999969 -0.00376988]
 [ 0.          0.          0.          1.        ]]

T_ic:  (cam0 to imu0): 
[[ 0.01680206 -0.99985871  0.00049921 -0.0158756 ]
 [ 0.99985864  0.01680236  0.00061458 -0.06821963]
 [-0.00062288  0.00048881  0.99999969  0.00381973]
 [ 0.          0.          0.          1.        ]]

timeshift cam0 to imu0: [s] (t_imu = t_cam + shift)
0.0

Transformation (cam1):
-----------------------
T_ci:  (imu0 to cam1): 
[[ 0.01561258  0.99987676 -0.00164472 -0.04170223]
 [-0.99987736  0.01561457  0.00120211 -0.01513469]
 [ 0.00122764  0.00162575  0.99999792 -0.00356854]
 [ 0.          0.          0.          1.        ]]

T_ic:  (cam1 to imu0): 
[[ 0.01561258 -0.99987736  0.00122764 -0.01447738]
 [ 0.99987676  0.01561457  0.00162575  0.04193921]
 [-0.00164472  0.00120211  0.99999792  0.00351813]
 [ 0.          0.          0.          1.        ]]

timeshift cam1 to imu0: [s] (t_imu = t_cam + shift)
0.0

Baselines:
----------
Baseline (cam0 to cam1): 
[[ 0.99999877  0.00118911 -0.00102243 -0.1101676 ]
 [-0.00118838  0.99999904  0.00071255 -0.00032166]
 [ 0.00102327 -0.00071134  0.99999922  0.00012079]
 [ 0.          0.          0.          1.        ]]
baseline norm:  0.110168134052 [m]

Gravity vector in target coords: [m/s^2]
[-0.06327332  8.78210015 -4.36338587]

Calibration configuration
=========================
cam0
-----
  Camera model: pinhole
  Focal length: [461.487246372674, 460.1113992557959]
  Principal point: [356.39105303227853, 231.15719697054647]
  Distortion model: equidistant
  Distortion coefficients: [-0.0016509958435871643, 0.02437222940989351, -0.03582816956989852, 0.019860839087717054]
  Type: aprilgrid
  Tags: 
    Rows: 6
    Cols: 6
    Size: 0.088 [m]
    Spacing 0.0264 [m]

cam1
-----
  Camera model: pinhole
  Focal length: [462.4318044040118, 461.1780497604126]
  Principal point: [377.0119530476368, 226.49966248854923]
  Distortion model: equidistant
  Distortion coefficients: [-0.0009362378060020789, 0.018833308358932984, -0.030558453797100132, 0.01955083559432553]
  Type: aprilgrid
  Tags: 
    Rows: 6
    Cols: 6
    Size: 0.088 [m]
    Spacing 0.0264 [m]


IMU configuration
=================
IMU0:
----------------------------
  Model: calibrated
  Update rate: 200.0
  Accelerometer:
    Noise density: 0.01 
    Noise density (discrete): 0.141421356237 
    Random walk: 0.0002
  Gyroscope:
    Noise density: 0.005
    Noise density (discrete): 0.0707106781187 
    Random walk: 4e-06
  T_i_b
    [[ 1.  0.  0.  0.]
     [ 0.  1.  0.  0.]
     [ 0.  0.  1.  0.]
     [ 0.  0.  0.  1.]]
  time offset with respect to IMU0: 0.0 [s]
```

(3).report-imucam-dynamic.pdf，除了results-imucam-dynamic.txt的内容，还有时变IMU参数bias在视频序列上的变化等曲线。

- 如果执行时遇到缺库的问题：

```
$ ImportError: No module named wxversion
```
可以通过查找并安装相应的python-wxgtk包来解决： 
```
$ apt-cache search python-wx
$ sudo apt-get install python-wxgtk3.0
```



## 3.ORB-YGZ-SLAM使用的标定文件格式

orb-ygz版本中Mono_VIO程序执行命令：

```
./Examples/Monocular/mono_euroc_vins ./Vocabulary/ORBvoc.bin ./camera_imu_config.yaml cam_image_folder cam_image_timestamp.txt imu_data.csv
```

以联想设备(单目IR+IMU)为例：

```
#LENOVO DEVICE vio
./Examples/Monocular/mono_euroc_vins ./Vocabulary/ORBvoc.bin /home/wangl/Projects/slam_data/20170912/poc025_0613_A3StereofromCV.yaml /home/wangl/Projects/slam_data/20170912/02/cam0 /home/wangl/Projects/slam_data/20170912/02/loop02.txt /home/wangl/Projects/slam_data/20170912/02/data.csv
```

其中*poc0250613A3StereofromCV.yaml*的格式，其中IMU 的参数包括IMU和camera的时延 Timestamp shift，IMU和camera的空间关系Camera.Tbc，还有相机内外参等(和ORBSLAM原版相同)，都是来自对应的camera-IMU标定文件results-imucam-data.txt：

```
%YAML:1.0

# 1: realtime, 0: non-realtime
test.RealTime: 1
# Time for visual-inertial initialization
test.VINSInitTime: 10
bUseIMU: 1
# Modify test.InitVIOTmpPath and bagfile to the correct path
# Path to save tmp files/results
test.InitVIOTmpPath: "/home/ccdeng/ygzgit/ORG-YGZ-SLAM/temp/"

## For good initialization (no movement at the beginning for some bag)
test.DiscardTime: 0

#######################################
imutopic: "/imu0"
imagetopic: "/cam0/image_raw"

# Timestamp shift. Timage = Timu + image_delay
Camera.delaytoimu: 0.0

# acc=acc*9.8, if below is 1
IMU.multiplyG: 0

# camera-imu frame transformation, Pi = Tic * Pc
Camera.Tbc:
 [0.99957686,  0.01329456,  0.025872,   -0.06864479,
  0.01427233, -0.99917648, -0.03798228, -0.01810569,
  0.02534574,  0.03833547, -0.99894343, -0.01553862,
  0.0, 0.0, 0.0, 1.0]

# Local Window size
LocalMapping.LocalWindowSize: 20
#-----------------------------------------------------------------
# Auto generated for OpenCV fisheye camera model.
#-----------------------------------------------------------------
Camera.fx: 208.758
Camera.fy: 208.758
Camera.cx: 315.601
Camera.cy: 238.318

Camera.fovx: 113.755
Camera.fovy: 97.9632

Camera.k1 : 0.0
Camera.k2 : 0.0
Camera.p1 : 0.0
Camera.p2 : 0.0

Camera.width : 640
Camera.height : 480

Camera.fps: 27.0

Camera.bf: 16.9276

Camera.RGB: 0

ThDepth: 35

LEFT.height: 480
LEFT.width: 640
LEFT.D: !!opencv-matrix
  rows: 4
  cols: 1
  dt: d
  data: [ 0.0157751,-0.0196731,0.0203775,-0.00720542 ]
LEFT.K: !!opencv-matrix
  rows: 3
  cols: 3
  dt: d
  data: [ 276.337,0,317.925,0,277.192,235.087,0,0,1 ]
LEFT.R: !!opencv-matrix
  rows: 3
  cols: 3
  dt: d
  data: [ 0.99947,0.0147838,0.0289865,-0.0150287,0.999853,0.00824817,-0.0288603,-0.00867943,0.999546 ]
LEFT.P: !!opencv-matrix
  rows: 3
  cols: 4
  dt: d
  data: [ 208.758,0,315.601,0,0,208.758,238.318,0,0,0,1,0 ]
RIGHT.height: 480
RIGHT.width: 640
RIGHT.D: !!opencv-matrix
  rows: 4
  cols: 1
  dt: d
  data: [ 0.006943,-0.00668118,0.0109503,-0.00444057 ]
RIGHT.K: !!opencv-matrix
  rows: 3
  cols: 3
  dt: d
  data: [ 276.839,0,320.313,0,278.25,242.433,0,0,1 ]
RIGHT.R: !!opencv-matrix
  rows: 3
  cols: 3
  dt: d
  data: [ 0.999518,-0.0203163,-0.0234841,0.0201168,0.99976,-0.00870221,0.0236553,0.00822559,0.999686 ]
RIGHT.P: !!opencv-matrix
  rows: 3
  cols: 4
  dt: d
  data: [ 208.758,0,315.601,-16.9276,0,208.758,238.318,0,0,0,1,0 ]
#-----------------------------------------------------------------
# ORB Parameters
#-----------------------------------------------------------------
# ORB Extractor : Number of features per image
ORBextractor.nFeatures : 1000

# ORB Extractor : Scale factor between levels in the scale pyramid
ORBextractor.scaleFactor : 1.2

# ORB Extractor : Number of levels in the scale pyramid
ORBextractor.nLevels : 8

# ORB Extractor : Fast threshold
ORBextractor.iniThFAST: 20
ORBextractor.minThFAST : 7

#-----------------------------------------------------------------
# Viewer Parameters
#-----------------------------------------------------------------
Viewer.KeyFrameSize: 0.05
Viewer.KeyFrameLineWidth : 1
Viewer.GraphLineWidth : 0.9
Viewer.PointSize : 2
Viewer.CameraSize : 0.08
Viewer.CameraLineWidth : 3
Viewer.ViewpointX : 0
Viewer.ViewpointY : -0.7
Viewer.ViewpointZ : -1.8
Viewer.ViewpointF : 500
```
