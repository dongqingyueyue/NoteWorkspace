[toc]

# 预积分理论以及VINS公式推导

本文档主要是介绍预积分的基础理论推导以及相关SLAM比如VINS公式的推导.

## 1. 基础的数学理论

### 1.1矩阵李群和李代数

#### 李群

本部分主要介绍两种特殊的矩阵李群：用于表示旋转的特殊正交群（special orthogonal group）SO(3)和用于表示姿态的特殊欧几里得群（special Euclidean group）SE(3).

SO(3)对于加法并不封闭，用于表示旋转
$$
SO(3)=\{C\in R^{3\times3}|CC^T=I,detC = 1\}
$$
零矩阵也不是一个有效的旋转矩阵：$ 0 \notin SO(3)$ 如果没有这些性质，SO(3)无法构成一个向量空间。

SE(3)用于表示平移和旋转的集合
$$
SE(3) = \{T = \left[\begin{matrix} C&R\\0^T&1 \end{matrix}\right] \in R^{3\times3}|C\in SO(3),r\in R^{3\times3}\}
$$
矩阵李群中的每个元素都是矩阵形式，把它们组合起来的运算是矩阵乘法，而逆运算是矩阵求逆。SO(3)的逆从旋转矩阵可以很容易推算，SE(3)的逆：
$$
T^{-1}=\left[\begin{matrix} C&r\\0^T &1\end{matrix}\right]^{-1}=\left[\begin{matrix} C^T&-C^Tr\\0^T &1\end{matrix}\right]
$$

#### 李代数

每一个李群对应一个李代数，李代数的向量空间是一个切空间。

SO(3)对应的向量空间so(3)定义：
$$
so(3) = \{\Phi=\phi^\land\in R^{3\times3}|\phi\in R^3\}
$$
注意so(3)是一个$3\times3$的反对称矩阵，如下公式成立：
$$
\Phi=\phi^\land \Longrightarrow \phi=\Phi^\lor
$$
SE(3)对应的李代数se(3)有如下定义：
$$
se(3)=\{ \Xi=\xi^\land \in R^{4\times4}|\xi \in R^6\}
$$
其中 $\xi^\land=\left[ \begin{matrix} \rho \\\phi\end{matrix}\right]^\and = \left[ \begin{matrix} \phi^\land & \rho \\ 0^T&0\end{matrix}\right]$



#### 求导雅克比

在 $SO(3)$,定义加减法操作完全不同于实数空间,遵循自己的规则:TBD(补充)

###  1.2流型公式 

对于李代数$so(3)$中的向量$\phi \in R^{3 \times 3}$ ,使用hat($ ^\and$)操作表示如下计算,例如:
$$
s = \omega^\land = \left[ \begin{matrix} \omega_1 \\ \omega_2 \\ \omega3  \end{matrix} \right]^\land 
= \left[ \begin{matrix} 0 & -\omega_3 &  \omega_2 \\ \omega_3 & 0 & -\omega_1 \\ -\omega_2 & \omega_1 & 0  \end{matrix} \right] \in so(3)
$$
反对称矩阵的性质:
$$
a^\land b = - b^\land a
$$


hat($^\and$)的逆操作为vee($\vee$):
$$
\omega =  s^ \vee
$$
由李代数到李群$so(3) \rightarrow SO(3)$的指数映射公式为:
$$
exp(\phi^\wedge) = I + \frac {\sin(\lVert \phi \rVert)}{\lVert \phi \rVert} \phi^\wedge + \frac{1-\cos(\lVert \phi \rVert)}{\lVert \phi \rVert^2}(\phi^\wedge)^2 \tag{1}
$$
一阶近似则有:
$$
exp(\phi^\wedge) = I + \phi ^ \wedge
$$
对应公式(1)的逆操作为:
$$
\log(R) = \frac{\psi\cdot(R - R^T)}{2\sin(\psi)}
$$
其中:
$$
\psi = \cos^{-1} (\frac{tr(R)-1}{2})
$$
因此有:
$$
\log(R)^\vee = a\psi
$$

#### 1.2.1 重要公式一

$$
Exp(\phi + \delta\phi) \approx  Exp(\phi)Exp(J_r(\phi)\delta \phi) \tag{2}
$$

其中$J_r$为流型中右雅克比,有
$$
J_r(\phi) = I -  \frac{1-\cos(\lVert \phi \rVert)}{\lVert \phi \rVert^2}\phi^\wedge +\frac {\lVert \phi \rVert-sin(\lVert \phi \rVert)}{\lVert \phi^3 \rVert}   (\phi^\wedge)^2   \tag{3}
$$
相同的一阶近似(令 $ J_r^{-1}(\phi)\delta \phi$替代$\delta\phi$),有
$$
\log({Exp(\phi)Exp(\delta\phi)}) \approx \phi + J_r^{-1}(\phi)\delta\phi
$$








## 2. 预积分理论





## 3. VINS预积分的推导 

1.1 离散状态下预积分方程:

(1) [VINS-Mono: A Robust and Versatile Monocular Visual-Inertial State Estimator, Tong Qin, Peiliang Li, Zhenfei Yang, Shaojie Shen (techincal report)](https://github.com/HKUST-Aerial-Robotics/VINS-Mono/blob/master/support_files/paper/tro_technical_report.pdf)

(2) [Solà J. Quaternion kinematics for the error-state KF[\]// Surface and Interface Analysis. 2015.](http://www.iri.upc.edu/people/jsola/JoanSola/objectes/notes/kinematics.pdf)

关于这部分的论文和代码中的推导，可以参考文献[2]中Appendx部分“A Runge-Kutta numerical integration methods”中的欧拉法和中值法。

$$w_{k}^{{}'}=\frac{w_{k+1}+w_{k}}{2}-b_{w}$$

$q _{i+1}=q _{i}\otimes \begin{bmatrix}1\\0.5w_{k}^{{}'}\delta t\end{bmatrix}$

$a_{k}^{{}'}=\frac{q_{k}(a_{k}+n_{a0}-b_{a_{k}})+q_{k+1}(a_{k+1}++n_{a1}-b_{a_{k}})}{2}$
$\alpha _{i+1}=\delta\alpha _{i}+\beta _{i}t+0.5a_{k}^{{}'}\delta t^{2}$
$\beta _{i+1}=\delta\beta _{i}+a_{k}^{{}'}\delta t$
1.2 离散状态下误差状态方程

论文中Ⅱ.B部分的误差状态方程是连续时间域内，在实际代码中需要的是离散时间下的方程式，而且在前面的预积分方程中使用了中值法积分方式。所以在实际代码中和论文是不一致的。在推导误差状态方程式的最重要的部分是对$\delta \theta _{k+1}$ 部分的推导。

由泰勒公式可得：
$\delta \theta _{k+1} = \delta \theta _{k}+\dot{\delta \theta _{k}}\delta t(2.1)$
依据参考文献[2]中 "5.3.3 The error-state kinematics"中公式(222c)及其推导过程有：
$\dot{\delta \theta _{k}}=-[w_{m}-w_{b}]_{\times }\delta \theta _{k}-\delta w_{b}-w_{n}$
对于中值法积分下的误差状态方程为：
$\dot{\delta \theta _{k}}=-[\frac{w_{k+1}+w_{k}}{2}-b_{g_{k}}]_{\times }\delta \theta _{k}-\delta b_{g_{k}}+\frac{n_{w0}+n_{w1}}{2}(2.2)$
将式(2.2)带入式(2.1)可得：

$\delta \theta _{k+1} =(I-[\frac{w_{k+1}+w_{k}}{2}-b_{g_{k}}]_{\times }\delta t) \delta \theta _{k} -\delta b_{g_{k}}\delta t+\frac{n_{w0}+n_{w1}}{2}\delta t$
这部分也可以参考，文献[2]中“7.2 System kinematics in discrete time”小节。
  接下来先推导 $\delta \beta _{k+1}$ 部分，再推导$\delta \alpha _{k+1}$部分。$\delta \beta _{k+1}$部分的推导也可以参考文献[2]中“5.3.3 The error-state kinematics”公式(222b)的推导。将式(1.5)展开得到：
$\delta\beta _{i+1}=\delta\beta _{i}+\frac{q_{k}(a_{k}+n_{a0}-b_{a_{k}})+q_{k+1}(a_{k+1}++n_{a1}-b_{a_{k}})}{2}\delta t$
即，
$\delta\beta _{i+1}=\delta\beta _{i}+\dot{\delta\beta_{i}}\delta t$
文献[2]中
$\dot{\delta v}=-R[a_{m}-a_{b}]_{\times}\delta \theta-R\delta a_{b}+\delta g-Ra_{n}$
对于中值法积分下的误差状态方程为

$\begin{align} \notag \dot{\delta\beta_{i}} =&-\frac{1}{2}q_{k}[a_{k}-b_{a_{k}}]_{\times}\delta \theta-\frac{1}{2}q_{k+1}[a_{k+1}-b_{a_{k}}]_{\times}\delta \theta _{k+1} -\delta b_{g_{k}}\delta t+\frac{n_{w0}+n_{w1}}{2}\delta t)\delta \theta \\\notag  &-\frac{1}{2}q_{k}\delta b_{a_{k}}-\frac{1}{2}q_{k+1}\delta b_{a_{k}}-\frac{1}{2}q_{k}n_{a0}-\frac{1}{2}q_{k}n_{a1} \end{align} $

将式(2.3)带入式(2.5)可得

$\begin{align}\notag\dot{\delta\beta_{i}} =&-\frac{1}{2}q_{k}[a_{k}-b_{a_{k}}]_{\times}\delta \theta-\frac{1}{2}q_{k+1}[a_{k+1}-b_{a_{k}}]_{\times}((I-[\frac{w_{k+1}+w_{k}}{2}-b_{g_{k}}]_{\times }\delta t) \delta \theta _{k} \delta b_{g_{k}}\delta t\\\notag&+\frac{n_{w0}+n_{w1}}{2}\delta t)  -\frac{1}{2}q_{k}\delta b_{a_{k}}-\frac{1}{2}q_{k+1}\delta b_{a_{k}}-\frac{1}{2}q_{k}n_{a0}-\frac{1}{2}q_{k}n_{a1}\end{align}$

同理，可以计算出$\delta \alpha _{k+1}$ ，可以写为：
$\delta\alpha _{i+1}=\delta\alpha _{i}+\dot{\delta\alpha_{i}}\delta t\tag{2.7}$

$\begin{align}\notag\dot{\delta\alpha_{i}} =&-\frac{1}{4}q_{k}[a_{k}-b_{a_{k}}]_{\times}\delta \theta\delta t-\frac{1}{4}q_{k+1}[a_{k+1}-b_{a_{k}}]_{\times}((I-[\frac{w_{k+1}+w_{k}}{2}-b_{g_{k}}]_{\times }\delta t) \delta \theta _{k} -\delta b_{g_{k}}\delta t \\\notag&+\frac{n_{w0}+n_{w1}}{2}\delta t)\delta t -\frac{1}{4}q_{k}\delta b_{a_{k}}\delta t-\frac{1}{4}q_{k+1}\delta b_{a_{k}}\delta t-\frac{1}{4}q_{k}n_{a0}\delta t-\frac{1}{4}q_{k}n_{a1}\delta t\end{align}$

最后是加速度计和陀螺仪bias的误差状态方程，
$\delta b_{a_{k+1}}=\delta b_{a_{k}}+n_{ba}\delta t$

$$\delta b_{w_{k+1}}=\delta b_{w_{k}}+n_{bg}\delta t$$

综合式(2.3)等误差状态方程，将其写为矩阵形式，

$\begin{align}\notag\begin{bmatrix}\delta \alpha_{k+1}\\\delta \theta  _{k+1}\\\delta \beta _{k+1} \\\delta b _{a{}{k+1}} \\\delta b _{g{}{k+1}}\end{bmatrix}&=\begin{bmatrix}I & f_{01} &\delta t  & -\frac{1}{4}(q_{k}+q_{k+1})\delta t^{2} & f_{04}\\0 & I-[\frac{w_{k+1}+w_{k}}{2}-b_{wk}]_{\times }\delta t & 0 &  0&-\delta t \\0 &  f_{21}&I  &  -\frac{1}{2}(q_{k}+q_{k+1})\delta t & f_{24}\\0 &  0&  0&I  &0 \\ 0& 0 & 0 & 0 & I\end{bmatrix}\begin{bmatrix}\delta \alpha_{k}\\\delta \theta  _{k}\\\delta \beta _{k} \\\delta b _{a{}{k}} \\\delta b _{g{}{k}}\end{bmatrix} \\\notag&+\begin{bmatrix} \frac{1}{4}q_{k}\delta t^{2}&  v_{01}& \frac{1}{4}q_{k+1}\delta t^{2} & v_{03} & 0 & 0\\0& \frac{1}{2}\delta t & 0 & \frac{1}{2}\delta t &0  & 0\\\frac{1}{2}q_{k}\delta t&  v_{21}& \frac{1}{2}q_{k+1}\delta t & v_{23} & 0 & 0 \\0 & 0 & 0 & 0 &\delta t  &0 \\0& 0 &0  & 0 &0  & \delta t\end{bmatrix}\begin{bmatrix}n_{a0}\\n_{w0}\\n_{a1}\\n_{w1}\\n_{ba}\\n_{bg}\end{bmatrix}\end{align}$



其中，

$\begin{align}\notag f_{01}&=-\frac{1}{4}q_{k}[a_{k}-b_{a_{k}}]_{\times}\delta t^{2}-\frac{1}{4}q_{k+1}[a_{k+1}b_{a_{k}}]_{\times}(I-[\frac{w_{k+1}+w_{k}}{2}-b_{g_{k}}]_{\times }\delta t)\delta t^{2} \\\notag f_{21}&=-\frac{1}{2}q_{k}[a_{k}-b_{a_{k}}]_{\times}\delta t-\frac{1}{2}q_{k+1}[a_{k+1}-b_{a_{k}}]_{\times}(I-[\frac{w_{k+1}+w_{k}}{2}-b_{g_{k}}]_{\times }\delta t)\delta t \\\notag f_{04}&=\frac{1}{4}(-q_{k+1}[a_{k+1}-b_{a_{k}}]_{\times}\delta t^{2})(-\delta t) \\\notag f_{24}&=\frac{1}{2}(-q_{k+1}[a_{k+1}-b_{a_{k}}]_{\times}\delta t)(-\delta t) \\\notag v_{01}&=\frac{1}{4}(-q_{k+1}[a_{k+1}-b_{a_{k}}]_{\times}\delta t^{2})\frac{1}{2}\delta t \\\notag v_{03}&=\frac{1}{4}(-q_{k+1}[a_{k+1}-b_{a_{k}}]_{\times}\delta t^{2})\frac{1}{2}\delta t \\\notag v_{21}&=\frac{1}{2}(-q_{k+1}[a_{k+1}-b_{a_{k}}]_{\times}\delta t^{2})\frac{1}{2}\delta t \\\notag v_{23}&=\frac{1}{2}(-q_{k+1}[a_{k+1}-b_{a_{k}}]_{\times}\delta t^{2})\frac{1}{2}\delta t\end{align}$
将式(2.11)简写为，

$\delta z_{k+1} = F\delta z_{k}+VQ$

最后得到系统的雅克比矩阵$J_{k+1}$ 和协方差矩阵 $P_{k+1}$，初始状态下的雅克比矩阵和协方差矩阵为单位阵和零矩阵，即
$\notag J_{k}=I \\\notag  P_{k}=0$

$J_{k+1}=FJ_{k}$

$P_{k+1}=FP_{k}F^{T}+VQV_{T}$



