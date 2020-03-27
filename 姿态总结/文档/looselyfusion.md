
#松耦合的SLAM方案
在AR的应用方案中，如果采用松耦合的方案，基本思路和框架如下：
1.  ethasl_sensor_fusion 库的使用
该包的理论说明在文档《tephan Weiss. Vision Based Navigation for Micro Helicopters PhD Thesis》

1) 安装
　参考http://wiki.ros.org/ethzasl_sensor_fusion说明
　git https://github.com/ethz-asl/ethzasl_sensor_fusion.git
　使用的说明：
　http://wiki.ros.org/ethzasl_sensor_fusion/Tutorials
2) 关于坐标系的说明
![cooridnate](/home/ccdeng/project/notebook/framesetup.png)
![cooridnate](/home/ccdeng/project/notebook/structure.png)


其中：
  
　pw_i:是指IMU在世界坐标系中的位置，也就是imu相对于世界坐标系的位置
　qw_i:同上
    qi_c/pi_c: camera在imu坐标系中的位置（camera to imu）
qw_v:是指word frame 到vision frame的坐标变换，也就是word frame在vision frame中的位置。
观测方程中的
　pv_c:　表示vision坐标系到当前camera坐标系的位置，其中camera坐标系是随着摄像头的运动儿变化的,vision坐标系是固定在camera处于t0的坐标是时的位置，对于vision坐标系，如果使用loitor模组，vision 坐标系到c0(camera 在t0时的位置)
　   cv::Mat Rv2c0 = cv::Mat::eye(3,3,CV_32F); 
    　Rv2c0.at<float>(1,1) = -1; 
    　Rv2c0.at<float>(2,2) = -1;
　 　具体可以看如下示意图所示，其中vision frame 是camera 在t0时刻绕着x轴旋转180度。

其中：word frame 和vision frame是重合的，可以放置在一起。
　![cooridnate](/home/ccdeng/project/notebook/reason.jpg)




3)配置参数说明
   配置参数主要包括如下内容（loitor的参数配置如下）：
~~~
initialization of camera w.r.t. IMU 
init/q_ci/w: 0.0015391 
init/q_ci/x: -0.70681082 
init/q_ci/y: 0.707366 
init/q_ci/z: -0.0070313 

init/p_ci/x: -0.00734815    
init/p_ci/y: 0.04917667 
init/p_ci/z: -0.01987 

# initialization of world w.r.t. vision 
init/q_wv/w: 1.0 
init/q_wv/x: 0.0 
init/q_wv/y: 0.0 
init/q_wv/z: 0.0
~~~
4)初始化说明
　初始化时需要注意的事项：
　由于初始化的时候，需要首先初始化qv2c ，因此在系统启动之前，首先要将qv2c的初始姿态在进程运行之前进行初始化。
　具体流程和顺序如下：
　a.SLAM进程中先初始化qv2c
　~~~
　wihle(1)
　{
	….
	SLAM得到Tc02c姿 态；
　　　 计算Tv2c = Tc02c*Tv2c0
          // 必须先计算得到Tv2c, 然后再初始化EKF进程，以下顺序不能够颠倒。 
          PoseMeas.posehandler->measurementCallback(&vslampose); 
　　　if(!init_measurement) 
                            { 
                                PoseMeas.init_measurement(); 
                                init_measurement=true; 
                            }
 .	….
	….
}　
~~~
 b.然后进入imuThread
 
 ~~~
 
    while(1)
    {
   PoseMeas.posehandler->imuCallback(&imu_msg);
  }
~~~

5)使用说明
　目前主要的参数配置如下：
　
　~~~
　
init_filter   = true; 
scale_init    = 1; 
fixed_scale   = false; 
fixed_bias    = false; 
fixed_calib   = true; 
noise_acc     = 0.8;//0.5; 
noise_accbias = 0.02;//0.1;//0.02; 
noise_gyr     = 0.1; 
noise_gyrbias = 0.1;//0.1; 

~~~
　在眼镜上调试的时候发现，如果noise_acc以及noise_accbias如果比较小，会出现震荡，稍微变大一些的话才会收敛。