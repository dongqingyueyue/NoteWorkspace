[toc]



# 一. 姿态的表示方法总结

##  1. 旋转矩阵(方向余弦阵)

### 基础

假设参考系为$i$ ,载体坐标系为$v$  ,载体坐标系可以看成一个与载体固联的固定坐标系,载体坐标系的原点代表载体的位置.

假设一个向量有大小方向用$\vec{r}$来表示,分解到三维的坐标轴上分别为$\vec{i} \ \vec{j} \ \vec{k}$ 表示,那么载体坐标系在参考系的位置为
$$
\vec{r}_v \quad = \quad x\vec{i}+y\vec{j} + z\vec{k} \\
$$
写成矩阵相乘的形式为
$$
\vec{r}_v = \left[\begin{matrix} x & y & z \end{matrix}\right] \left[\begin{matrix} \vec{i} \\ \vec{j} \\ \vec{k} \end{matrix}\right]
$$
本文中默认所有的向量为列向量,那么有
$$
\vec{r}_v = r^T \vec{F}
$$


同样有
$$
\vec{r}_v = \left[\begin{matrix} \vec{i} & \vec{j}&\vec{k} \end{matrix}\right]
\left[\begin{matrix} x \\ y \\ z \end{matrix}\right]
$$
即:
$$
\vec{r}_v = {\vec{F}}^Tr
$$


那么两个向量的点积为:
$$
\vec{r}_v \cdot \vec{s}_v = r_xs_x + r_ys_y+r_zs_z
$$
两个向量的叉积为:
$$
\vec{r}_v \times \vec{s}_v =
\left[\begin{matrix} r_x & r_y & r_z \end{matrix}\right] 
\left[\begin{matrix} \vec{i} \\ \vec{j} \\ \vec{k} \end{matrix}\right] \times
\left[\begin{matrix} \vec{i} & \vec{j}&\vec{k} \end{matrix}\right]
\left[\begin{matrix} s_x \\ s_y \\ s_z \end{matrix}\right]\\
=\left[\begin{matrix} r_x & r_y & r_z \end{matrix}\right]
\left[\begin{matrix} \vec{i}\times \vec{i} & \vec{i}\times\vec{j} & \vec{i}\times\vec{k} \\ \vec{j}\times\vec{i} & \vec{j}\times\vec{j}&\vec{j}\times\vec{k} \\  \vec{z}\times\vec{i} & \vec{k}\times\vec{j}&\vec{k}\times\vec{k}\end{matrix}\right] 
\left[\begin{matrix} s_x \\ s_y \\ s_z \end{matrix}\right]
$$
如果使用右手法则的话 上式有:


$$
\vec{r}_v \times \vec{s}_v =
\left[\begin{matrix} r_x & r_y & r_z \end{matrix}\right] 
\left[\begin{matrix} \vec{i} \\ \vec{j} \\ \vec{k} \end{matrix}\right] \times
\left[\begin{matrix} \vec{i} & \vec{j}&\vec{k} \end{matrix}\right]
\left[\begin{matrix} s_x \\ s_y \\ s_z \end{matrix}\right]\\
=\left[\begin{matrix} r_x & r_y & r_z \end{matrix}\right]
\left[\begin{matrix} 0 & \vec{k} & -\vec{j} \\ -\vec{k} &0&\vec{i} \\  \vec{j} & - \vec{i}&0\end{matrix}\right] 
\left[\begin{matrix} s_x \\ s_y \\ s_z \end{matrix}\right] \\
=\left[\begin{matrix} \vec{i} & \vec{j}&\vec{k} \end{matrix}\right]
\left[\begin{matrix} 0 & -r_z & r_y \\ r_z &0&-r_x \\ -r_y  &r_x&0\end{matrix}\right] 
\left[\begin{matrix} s_x \\ s_y \\ s_z \end{matrix}\right]
$$

###  方向余弦引出

方向余弦矩阵的推导可以参考《states estimation for robotics》 和严老师的组合导航讲义p14页.

假设有两个原点相同的参考坐标系$F_1$和$F_2$,那么同一个向量$\vec{r}$在这两个参考系下表示为:
$$
\vec{r} = \vec{F}_1^Tr_1 = \vec{F}_2^Tr_2
$$
继续推导:
$$
\vec{r} = \vec{F}_1^Tr_1 = \vec{F}_2^Tr_2 \\
\vec{F}_2 \cdot \vec{F}_1^Tr_1 = \vec{F}_2 \cdot \vec{F}_2^Tr_2 \\
r_2 = \vec{F}_2 \cdot \vec{F}_1^Tr_1 = C_{21}r_1
$$
因此:
$$
C_{21} =\vec{F}_2 \cdot \vec{F}_1^T
$$
$C_{21}$称为方向余弦阵,其中$\vec{F_1}$和$\vec{F_2}$为参考系的单位向量.

方向余弦的坐标变化：在坐标系$b$中的坐标为$[x_b   y_b  z_b ]^T$,在坐标系$i$中的坐标为$[x_i   y_i  z_i ]^T$,使用旋转矩阵表示为：
$$
r^b=C_i^br^i
$$

以上是从静态转换的角度来说明旋转矩阵的由来.
$$
C^i_b = \begin{bmatrix} i_i\cdot i_b&i_i\cdot j_b & i_i\cdot k_b \\j_i\cdot i_b& j_i\cdot j_b&j_i\cdot k_b\\k_i\cdot i_b&k_i\cdot j_b&k_i\cdot k_b\end{bmatrix}
$$
表示的是将b坐标系的坐标转化到i坐标系的坐标；

可以从点积的物理意义来说明旋转矩阵的意义,尤其是绕着某个轴进行旋转的时候更容易理解.

每一行表示的是i坐标系的每一个向量在b坐标系的三个坐标轴的夹角；

如果表示坐标系的运动的话，表示的将i坐标系运动到b坐标系；而对应的四元数：由i运动到b的四元数为$q_b^i$,旋转矢量则是有i到b的旋转矢量。

如果由A运动到B再运动到c，旋转矩阵有：
$$
C_C^A=C^A_B\cdot C_C^B
$$
同理四元数也有这样的关系：
$$
q_C^A=q_B^A\otimes q_C^B
$$
需要注意的是下面的过程刚好相反

假设将坐标系A绕z轴逆时针旋转$\alpha$，可以得到B，那么得到的旋转矩阵为：
$$
C_A^B= \begin{bmatrix} cos\alpha&sin\alpha&0\\-sin\alpha&cos\alpha&0\\0&0&1\end{bmatrix}
$$

### 旋转运动学-角速度与旋转矩阵

假设观察者在参考坐标系$F_1$下,参考坐标系$F_2$相对于参考坐标系$F_1$的旋转角速度为$\omega^{21}$,如果旋转角速度投影在$F_2$坐标系下,可以得到泊松公式:
$$
\dot{C_{21}} = -(\omega_2^{21}\times )C_{21}
$$
如果是将其转换到IMU相关的坐标系下,对应的公式是:
$$
\dot{C_i^b} = -(\omega^b_{ib}\times)C_i^b
$$
其中,$\omega_{ib}^b$表示的是载体坐标系相对于惯性坐标系的旋转角速度在b坐标下的投影,与上述公式表示的不一样. 
$$
\dot{C_i^b} =C_i^b C_b^i (-\omega^b_{ib} \times) C_i^b \\
\dot{C_i^b} =C_i^b (\omega^i_{bi}\times)
$$
同理可以得到:
$$
\dot{C_b^i} =C_b^i (\omega^b_{ib}\times)
$$
泊松公式推导见 state estimation for  robotics 6.2.4章节.

后面公式的推导见严老师讲义.

### 加速度运动学公式--Paul Groves

运动学研究的是物体本身的运动，不考虑导致运动的原因。研究物体的运动涉及到三种坐标系：

1. 描述载体的运动，即载体坐标系--$\alpha$

   载体坐标系一般的是IMU坐标系或者以载体质心形成的坐标系。载体坐标系与载体固联，将载体看成刚体，载体坐标系的运动就是刚体的运动。

2. 运动所参照的坐标系，即参考坐标系---$\beta$

   参考坐标系可能是上一时刻的运动状态，也可以是绝对的东北天或者惯性坐标系。

3. 用于表现运动的坐标轴组成的坐标系，即投影坐标系---$\gamma$

   刚体相对于参考坐标系的运动本身就是一个矢量，这个矢量可以在投影坐标系里分解描述；投影坐标系可以是载体坐标系，也可以是参考坐标系，也可以是第三方坐标系。无论怎么描述，矢量的幅度是不变的。

基础公式关系：
$$
\omega_{\beta\alpha}^\gamma=\omega_{\beta\delta}^\gamma+\omega_{\delta\alpha}^\gamma
$$

$$
\omega_{\beta\alpha}^\gamma=C_\gamma^\delta\omega_{\beta\alpha}^\gamma
$$

上述公式表示的角速度矢量，如果将角速度矢量用斜对称矩阵$\Omega_{\beta\alpha}^\gamma$表示，用到矩阵的相似变换公式：
$$
\Omega_{\beta\alpha}^\delta=C_\gamma^\delta\Omega_{\beta\alpha}^\gamma C_\delta^\gamma
$$
方向余弦阵的微分公式：
$$
\dot{C_\beta^\alpha}  =  -\Omega_{\beta\alpha}^\alpha C_\beta^\alpha \\  = -C_\beta^\alpha\Omega_{\beta\alpha}^\beta\\ = \Omega_{\alpha\beta}^\alpha C_\beta^\alpha \\  = C_\beta^\alpha\Omega_{\alpha\beta}^\beta
$$
速度表示的是载体坐标系原点的位置相对于参考坐标系原点和坐标轴的变化率。

$\alpha$坐标系相对于$\beta$坐标系的加速度在$\gamma$坐标系中的投影为：
$$
a_{\beta\alpha}^\gamma=C_\beta^\gamma \ddot{r}_{\beta\alpha}^\beta
$$
由于投影坐标系$\gamma$相对于参考坐标系$\beta$可能存在转动，加速度并不简单的等同于$v_{\beta\alpha}^\gamma$的时间导数或者位置$r_{\beta\alpha}^\gamma$二次时间导数。
$$
\dot{v}_{\beta\alpha}^\gamma=\dot{C}_\beta^\gamma \dot{r}_{\beta\alpha}^\beta+a_{\beta\alpha}^\gamma
$$

$$
\ddot{r}_{\beta\alpha}^\gamma=\ddot{C}_\beta^\gamma r_{\beta\alpha}^\beta+\dot{C}_\beta^\gamma\dot{r}_{\beta\alpha}^\beta+\dot{v}_{\beta\alpha}^\gamma\\=\ddot{C}_\beta^\gamma r_{\beta\alpha}^\beta+2\dot{C}_\beta^\gamma\dot{r}_{\beta\alpha}^\beta+a_{\beta\alpha}^\gamma
$$

$$
\ddot{C}_\beta^\gamma r_{\beta\alpha}^\beta=(\Omega_{\beta\gamma}^\gamma\Omega_{\beta\gamma}^\gamma-\dot{\Omega}_{\beta\gamma}^\gamma)r_{\beta\alpha}^\gamma
$$

$$
\dot{C}_\beta^\gamma\dot{r}_{\beta\alpha}^\beta=-\Omega_{\beta\gamma}^\gamma C_\beta^\gamma\dot{r}_{\beta\alpha}^\beta\\=\Omega_{\beta\gamma}^\gamma(\dot{C}_\beta^\alpha r_{\beta\alpha}^\beta-\dot{r}_{\beta\alpha}^\gamma)\\=-\Omega_{\beta\gamma}^\gamma\Omega_{\beta\gamma}^\gamma r_{\beta\alpha}^\gamma-\Omega_{\beta\gamma}^\gamma\dot{r}_{\beta\alpha}^\gamma
$$

推出最后的公式：
$$
\ddot{r}_{\beta\alpha}^\gamma=-\Omega_{\beta\gamma}^\gamma\Omega_{\beta\gamma}^\gamma r_{\beta\alpha}^\gamma-2\Omega_{\beta\gamma}^\gamma\dot{r}_{\beta\alpha}^\gamma-\dot{\Omega}_{\beta\gamma}^\gamma r_{\beta\alpha}^\gamma+a_{\beta\alpha}^\gamma
$$
等号右边的前三项分别为离心力、哥氏力和欧拉虚力（当参考坐标系相对于惯性坐标系存在角加速度的时候产生的力）

比力是相对于惯性坐标系中除了引力以外的其他力，因此：
$$
f_{ib}^\gamma=a_{ib}^\gamma-\gamma_{ib}^\gamma
$$
上述就是在惯性坐标系下的加速度关系式。

## 2. 四元数

### 2.1四元数与坐标旋转

#### 2.1-1四元数的定义与运算法则

四元数分为两个定义方式：Hamilton四元数和JPL四元数，其根本的不同在于虚数运算$ij=k$或者$ij=-k$的区别。以下所推导的四元数都是标量在前，矢量在后；JPL的四元数的推导为个人手工推导，如有谬误，请指出；

两种四元数的定义规范都满足加法、减法，并且都满足交换律和结合律；

Hamilton四元数的乘法：
$$
\overline{p}\otimes\overline{q}=(p_0+p_1i+p_2j+p_3k)(q_0+q_1i+q_2j+q_3k) \\=(p_0q_0-p_1q_1-p_2q_2-p_3q_3)\\+(p_0q_1+p_1q_0+p_2q_3-p_3q_2)i\\+(p_0q_2+p_2q_0-p_1q_3+p_3q_1)j\\+(p_0q_3+p_3q_0+p_1q_2-p_2q_1)k
$$
写成矩阵的形式则有：
$$
\overline{p}\otimes\overline{q}=\begin{bmatrix}p_0&-p_1&-p_2&-p_3\\p_1&p_0&-p_3&p_2\\p_2&p_3&p_0&-p_1\\p_3&-p_2&p_1&p_0\end{bmatrix}\begin{bmatrix}q_0\\q_1\\q_2\\q_3\end{bmatrix}
$$

JPL四元数的乘法：
$$
\overline{p}\otimes\overline{q}=(p_0+p_1i+p_2j+p_3k)(q_0+q_1i+q_2j+q_3k) \\=(p_0q_0-p_1q_1-p_2q_2-p_3q_3)\\+(p_0q_1+p_1q_0-p_2q_3+p_3q_2)j\\+(p_0q_2+p_2q_0+p_1q_3-p_3q_1)j\\+(p_0q_3+p_3q_0-p_1q_2+p_2q_1)k
$$
写成矩阵的形式则有：
$$
\overline{p}\otimes\overline{q}=\begin{bmatrix}p_0&-p_1&-p_2&-p_3\\p_1&p_0&p_3&-p_2\\p_2&-p_3&p_0&p_1\\p_3&p_2&-p_1&p_0\end{bmatrix}\begin{bmatrix}q_0\\q_1\\q_2\\q_3\end{bmatrix}
$$
举个例子说明：

假设一个坐标系先绕着z轴瞬时转90 再绕着x瞬时转90

Hamilton四元数：

绕z轴旋转：$\begin{bmatrix}\sqrt{2}/2&0&0&-\sqrt{2}/2\end{bmatrix}$

绕x轴旋转：$\begin{bmatrix}\sqrt{2}/2&-\sqrt{2}/2&0&0\end{bmatrix}$

最后得到的四元数：

$\begin{bmatrix}1/2&-1/2&1/2&-1/2\end{bmatrix}$

JPL四元数：

绕z轴旋转：$\begin{bmatrix}\sqrt{2}/2&0&0&\sqrt{2}/2\end{bmatrix}$

绕x轴旋转：$\begin{bmatrix}\sqrt{2}/2&-\sqrt{2}/2&0&0\end{bmatrix}$

最后得到的四元数：

$\begin{bmatrix}1/2&-1/2&1/2&1/2\end{bmatrix}$

按照四元数的表示方法同一个旋转，只有Z轴的符号有差异结论得证。

四元数的乘法满足结合律和分配律：
$$
(p\otimes q)\otimes r=p\otimes(q\otimes r)
$$

$$
p\otimes(q+r)=p\otimes q +p\otimes r
$$

另外四元数相乘表示成矩阵的形式有：
$$
p\otimes q=[p]_Lq=[q]_Rp
$$
特别需要注意的是Hamilton 四元数的矩阵形式与JPL的矩阵形式恰好相反。

四元数的共轭与求逆的运算规律与矩阵的运算规律保持一致的。

 #### 2.1-2四元数与运动、坐标旋转的关系

***JPL、Hamilton四元数以及旋转之间相互的关系推导及说明***

个人认为Hamilton四元数与JPL四元数之前的关系完全由坐标系决定。尤其是四元数$i,j,k$所确定的坐标系关系。以陀螺仪为例，旋转矢量的三个分解轴在陀螺仪的三个轴，同时也是$i,j,k$所确定的坐标系的轴。

如下例子假设Hamilton四元数的初始坐标系是东北天，而对应的JPL四元数的初始坐标系为西南地：

Hamilton四元数：

假设物体由$b_k$时刻参考坐标系转到$bk+1$时刻，如果用向量${\omega}$表示这种运动，则会有如下的坐标旋转关系：
$$
r^{b_k}=Q^{b_k}_{b_k+1}*r^{b_k+1}*(Q^{b_k}_{b_k+1})^{-1}
$$
由此可见，$\omega$表示${\omega}^{b_k}_{b_k+1}$,即$b_{k+1}$时刻相对于$b_k$时刻的运动，由$\omega$组成的四元数为$Q^{bk}_{bk+1}$ 

假设物体由$b_k$时刻参考坐标系转到$bk+1$时刻,运动对应的坐标旋转矩阵是$C_{bk}^{bk+1}$

  ***四元数微分方程及求解***

Hamliton四元数连续形式下的旋转状态推导如下：
$$
q_{b_{k+1}}^w =q_{b_k}^w\otimes\int_{t\in[k,k+1]}\dot{q_t}dt
$$
注意以上的形式是右乘，$\dot{q_t}$(四元数的导数)如下：
$$
\dot{q_t}=\lim_{\delta t \to0}\frac1 {\delta t}(q_{t+\delta t}-q_t)\\=\lim_{\delta t \to0}\frac1 {\delta t}(q_t\otimes q_{t+\delta t}^t-q_t\otimes\left[\begin{matrix} 0\\1\end{matrix}\right])\\=\lim_{\delta t \to0}\frac1 {\delta t}(q_t\otimes \left[\begin{matrix} \hat ksin\frac\theta 2\\cos\frac\theta 2\end{matrix}\right]-q_t\otimes\left[\begin{matrix} 0\\1\end{matrix}\right])\\=\lim_{\delta t \to0}\frac1 {\delta t}(q_t\otimes \left[\begin{matrix} \hat k\frac\theta 2\\1\end{matrix}\right]-q_t\otimes\left[\begin{matrix} 0\\1\end{matrix}\right])
$$
将上述公式写成矩阵的形式：

JPL四元数：

根据物体的相对关系,得到：
$$
r^{b_k+1}= Q_{b_k}^{b_k+1}\otimes r^{b_k} \otimes Q_{b_k+1}^{b_k}
$$


四元数的导数
$$
\dot{Q}_b^i =1/2*Q_b^i*\omega_{ib}^b
$$

$$
\dot{Q}_b^i=1/2*\omega_{ib}^i*Q_b^i
$$

$$
\dot{Q}_i^b=1/2*Q_i^b*\omega_{bi}^i
$$

四元数的指数形式
$$
e^q=e^{qw+qv}=e^{qw}e^{qv}=e^{qw}[||cos(qv)||\quad qv/||qv||sin|||qv|]
$$
四元数的对数形式
$$
log(q)=log(||q||*  q/||q||)=log||q||+u\theta=[log||q||\quad u\theta]
$$


## 

