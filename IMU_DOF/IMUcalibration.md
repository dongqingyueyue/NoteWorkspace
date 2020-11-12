[toc]

# IMU标定

## 1. G2的标定-6面法

简要:glass中IMU的标定主要是标定IMU的零偏,陀螺的零偏以及IMU与glass之间的安装角度.

求零偏的基本原理: 

1. 先求加速度计的零偏.假设我有个加速度计x坐标轴向上,  静止状态下  $\hat a = a_{true} + bias+n$,如果旋转180°向下,则有$\hat b= -a_{true} + bias + n$,

   则$bias = (\hat a + \hat b)/ 2$ ,因为IMU有三个坐标轴,因此需要六位置法来求得各个加速度坐标轴的零偏.

2. 陀螺的零偏则是静止情况下陀螺输出的数据.
3. 假设加速度计和陀螺的坐标轴重合,代表IMU的坐标轴.则加速度计和glass的夹角就代表安装角度.

## 2. newG的标定 -12面法

加速度计的建模公式如下:

![image-20201111174540696](/home/lenovo/Noteworkspace/IMU_DOF/imuadd.png)

![image-20201111174704922](/home/lenovo/Noteworkspace/IMU_DOF/Ka.png)

![image-20201111174741683](/home/lenovo/Noteworkspace/IMU_DOF/ba.png)

![image-20201111174852486](/home/lenovo/Noteworkspace/IMU_DOF/acc.png)

对IMU标定本质上是求上述$T^a$ ,$K^a$和$b^a$ 的值,主要使用的原理是优化算法,其中残差就是用静止状态下加速度的模值与重力加速度的差值,求导的过程使用的是ceres-solver的自动求导过程.

![image-20201111175042755](/home/lenovo/Noteworkspace/IMU_DOF/modle.png)

之所以是12面就是尽可能的使用各个方向的观测值来保证参数的精确性.