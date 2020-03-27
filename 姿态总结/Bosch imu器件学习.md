# Bosch imu器件学习

1. 有的IMU芯片是经过标定的，我们使用的BOSCH芯片是已经将标定结果写入内置寄存器还是仍然需要我们自己标定零偏非正交性等相关参数？

我们的IMU 在零偏，非正交性等参数上的校准，其校准精度请参考datasheet 的上的参数标准。所以严格的讲，从应用level 的要求来看，还是需要在整机level 进行相应的校准操作，然后将校准值进行保存，并应用到sensor 的raw data 中。目前sensor 寄存器是可以存储零偏参数的，但是其他参数需要存入系统其他掉电不丢失的存储空间内。

至于对器件如何配置，需要结合具体的应用，比如应用是希望data noise 小一些，还是希望data rate 高一下，data 延时小一些。应用需求不同，sensor的推荐配置不同。

噪声和延时是两个相悖的参数，如果想要获得较小的噪声，势必会增加延时，反之亦然。
如果希望将BMI160 ODR 设置为400HZ 的话，可以通过设置bandwidth (osr setting) 来改善噪声表现，osr 越大，噪声越小。

在normalfilter mode下，加速度计和陀螺仪的noise和bandwidth（3Db cutoff frequency）正相关，bandwidth越大，noise越大。

陀螺仪的noise可以用这个公式计算：0.007 * sqrt(bw*1.6)，bw即为bandwidth，计算结果的单位为°/srms。注意：Noise变小，相应的group delay会变大。

1.	output nosie和带宽是有关系，BMI160 如果知道带宽，如何计算noise？直接用datasheet 值乘以带宽吗？
   output RMS noise 和带宽是有关系的，noise density 是确定的，可以通过公式 RMS noise = noise density *sqrt(1.6*bandwidth) 来计算 
   2.Zero-g offset temperature drift的意思是以25度为基准，增加的度数乘以datasheet中给的数值吗？
   TCO 乘以温度和25度的差值即为温漂。



2.Offset 是指sensor的零偏。Datasheet 里边描述的是在不同的情况下offset 的spec.

OffA, INT 表示sensor 出厂时最初的offset spec, 是component level
OffA, board 表示sensor 在贴到PCB 上的offset spec, 是board level
OffA, MSL 表示sensor 在经历MSL1 precondition 的条件后再焊接到板子上的offset spec
OffA, life 表示sensor 在整个life time 内保证的offset spec, 但是如果机器结构设计不理想，导致PCB 板存在比较大的弯折，sensor offset 会受到比较大的影响，那这种情况就需要特别讨论，而不能直接参考这个spec.

3.我如果想用这款芯片做AR的3DOF和SLAM应用，您接触过的案例有没有推荐配置？

一般SLAM应用会采用6/9轴数据，对应我们的BMI和BMX产品。下表是我们主流BMI参数的对比，AR/VR产品对

   bias instability, TCO, TCS, offset这几项指标非常重视。BMI085噪声更低，这款应该能适合您。BMI088更突出的优势是抗震动性能。BMI085/BMI088这两款是高性能的，注重性能指标的客户首选，BMI160性能比上两款稍差，

   价格便宜些。 如果还需要地磁数据，推荐BMX055。

 

![](file:///C:/Users/yangdq6/AppData/Local/Temp/msohtmlclip1/01/clip_image002.gif)

 

4.您说的带宽指的是采样频率还是低通滤波的带宽？这两者之间有关系吗？

带宽是bandwidth（3Dbcutoff frequency），低通滤波的带宽。采样频率为ODR，下表为采样频率和滤波器带宽的关系。

![](file:///C:/Users/yangdq6/AppData/Local/Temp/msohtmlclip1/01/clip_image004.gif)

 

5.假设我采样400HZ，还需要配置什么寄存器来降低噪声呢？

以BMI088为例，采样频率为400Hz,滤波为normal模式，滤波器带宽为145Hz, OSR2模式带宽为75Hz,OSR4模式为40Hz。

 

![](file:///C:/Users/yangdq6/AppData/Local/Temp/msohtmlclip1/01/clip_image006.gif)

