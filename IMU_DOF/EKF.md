[toc]

| 作者   | 版本 | 日期     | 联系方式          |
| ------ | ---- | -------- | ----------------- |
| yandq6 | 1.0  | 20201111 | sdydq1988@126.com |



# EKF algorithm

## 1.综述

IMU EKF算法主要是根据IMU输出的原始数据来计算IMU的姿态,基本原理是构建卡尔曼滤波模型,以陀螺的数据构建时间预测方程,以加速度的数据构建量测更新方程,从而实时估计出当前的状态.

## 2. 主要原理

   ### 2.1 时间更新

卡尔曼时间更新方程主要用于预测下一时刻的状态,建立的状态为7维度,四元数+ 零偏,主要的原理步骤为

1. 对于零偏建模为常数值:
   $$
   \dot{b} = 0   \tag{0}
   $$
   因此有:
   $$
   \hat{b}_{k+1|k} = \hat{b}_{k|k}   \tag{1}
   $$
   当有新的陀螺测量值$\omega_{k+1}$ ,则有:
   $$
   \hat{\omega}_{k+1|k} = \omega_{k+1} - \hat{b}_{k+1|k} \tag{3}
   $$
   

2. 有个初始状态四元数,对四元数积分,看做是时间更新方程:

$$
\overline{q}_G^L(t_{k+1}) = \Theta(t_{k+1},t_k)\overline{q}_G^L(t_k)  \tag{4}
$$

其中,
$$
\Theta(\Delta t) = \cos(\frac{\lvert \omega \rvert} 2\Delta t) \cdot I_{4\times4} + \frac {1}{\lvert \omega \rvert}\sin(\frac{\lvert \omega \rvert} 2\Delta t) \cdot \Omega(\omega)
$$

$$
\Omega(\omega) =\begin{bmatrix} 
0 \quad \omega_z  \quad -\omega_y  \quad \omega_x \\ 
-\omega_z \quad 0 \quad \omega_x \quad \omega_y  \\
\omega_y  \quad -\omega_x \quad 0 \quad \omega_z \\
-\omega_x \quad -\omega_y  \quad -\omega_z \quad 0
\end{bmatrix}
$$

3.  计算协方差矩阵更新:

$$
P_{k+1} = \Phi P_k \Phi^T + GQ_dG^T \tag{5}
$$

$$
\Phi = \left[\begin{matrix} \Theta & \Psi \\
0_{3 \times3 } & I_{3\times 3}
\end{matrix}\right]
$$

$$
\Theta = \cos(|\hat\omega| \Delta t)\cdot I_{3\times3} -
\sin(|\hat\omega| \Delta t)\cdot[\frac{\hat\omega}{|\hat\omega|}\times] + (1-\cos(|\hat\omega| \Delta t))\cdot \frac{\hat\omega}{|\hat\omega|}\frac{\hat\omega}{|\hat\omega|}^T
$$

$$
\Psi = -I_{3\times3}\Delta t + \frac{1}{|\hat\omega|^2}(1-\cos(|\hat\omega|\Delta t))[\hat\omega \times] -\frac{1}{|\hat\omega|^3}(|\hat\omega|\Delta t -sin(|\hat\omega|\Delta t))[\hat\omega \times]^2
$$

G矩阵:
$$
G= \left[\begin{matrix} -I_{3\times 3} & 0 \\ 0 & I_{3\times 3}
\end{matrix}\right]
$$
$Q$矩阵:
$$
Q_d = \left[\begin{matrix} Q_{11} & Q_{12}\\ Q_{12}^T & Q_{22} 
\end{matrix}\right]
$$

### 2.2 量测更新步骤

给定了时间的更新状态$\hat{\overline{q}}_{k+1|k}$和$\hat{b}_{k+1|k}$以及协方差矩阵$P_{k+1|k}$,目前的观测量$Z(k+1)$以及观测矩阵$H$,则可以进行量测更新.

计算残差 :
$$
r = z-\hat{z} \tag{6}
$$
由(6)式可以得到
$$
z-\hat{z} =  (I - [\delta \theta\times])C_n^bg^n -C_n^bg^n 
$$
因此 
$$
r =[C_n^bg^n\times] \delta \theta + 0 \cdot bias
$$
得到了H 矩阵为$([C_n^bg^n\times]\ \ 0)$

计算卡尔曼滤波增益:
$$
K = PH^T(HPH^T+R)^{-1} \tag{7}
$$
计算改正数:
$$
\Delta \hat{x}(+) = Kr \tag{8}
$$
计算更新的状态:
$$
x= \hat{x} \oplus \Delta \hat{x}(+) \tag{9}
$$
计算更新的协方差矩阵:
$$
P_{k+1|k+1} = (I_{6\times 6}-KH)P_{k+1|k}(I_{6 \times 6}-KH)^T + KRK^T \tag{10}
$$


