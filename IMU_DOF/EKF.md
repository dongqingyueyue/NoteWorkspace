[toc]

# EKF algorithm

## 1.综述

IMU EKF算法主要是根据IMU输出的原始数据来计算IMU的姿态,基本原理是构建卡尔曼滤波模型,以陀螺的数据构建时间预测方程,以加速度的数据构建量测更新方程,从而实时估计出当前的状态.

## 2. 主要原理

   ### 2.1 时间更新

1. 假设有个初始状态四元数,对四元数积分:

$$
\overline{q}_G^L(t_{k+1}) = \Theta(t_{k+1},t_k)\overline{q}_G^L(t_k)
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

2. 计算矩阵更新:

$$
P_{k+1} = \Phi P_k \Phi^T + GQ_dG^T
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
r = z-\hat{z}
$$
计算卡尔曼滤波增益:
$$
K = PH^T(HPH^T+R)^{-1}
$$
计算改正数:
$$
\Delta \hat{x}(+) = Kr
$$
计算更新的状态:
$$
x= \hat{x} \oplus \Delta \hat{x}(+)
$$
计算更新的协方差矩阵:
$$
P_{k+1|k+1} = (I_{6\times 6}-KH)P_{k+1|k}(I_{6 \times 6}-KH)^T + KRK^T
$$


