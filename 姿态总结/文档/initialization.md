##1.IMU初始化##
IMU初始化和紧耦合的论文的参考文献如下：
[1]VINS-Mono: A Robust and Versatile Monocular Visual-Inertial State Estimator
[2]visual inertial monocular slam with map reuse
[3]ON-Manifold preintegration for real-time visual-inertial odometry
IMU初始化主要初始化bias,重力和尺度。
其中gyro 的bias 初始化通过图像得到。
VINS中的预计分计算方法同[3]中的方法有较大的不同。VINS中采用error-state model.计算得到系统的状态方程(p,q,v,ba,bg)，并且得到系统整体相对于时间t的雅克比方程，针对系统中的其他参数对ba,bg的雅克比方程是间接计算得到的。
而[3]中是直接结算雅克比方程。

IMU 紧耦合初始化流程：
**初始化Gyro Bias**
**初始化尺度和重力**


在VINS 中初始化gyro bias代码如下:
~~~c

void solveGyroscopeBias(map<double, ImageFrame> &all_image_frame, Vector3d* Bgs)
{
    Matrix3d A;
    Vector3d b;
    Vector3d delta_bg;
    A.setZero();
    b.setZero();
    map<double, ImageFrame>::iterator frame_i;
    map<double, ImageFrame>::iterator frame_j;
    for (frame_i = all_image_frame.begin(); next(frame_i) != all_image_frame.end(); frame_i++)
    {
        frame_j = next(frame_i);
        MatrixXd tmp_A(3, 3);
        tmp_A.setZero();
        VectorXd tmp_b(3);
        tmp_b.setZero();
        Eigen::Quaterniond q_ij(frame_i->second.R.transpose() * frame_j->second.R);
        tmp_A = frame_j->second.pre_integration->jacobian.template block<3, 3>(O_R, O_BG);
        tmp_b = 2 * (frame_j->second.pre_integration->delta_q.inverse() * q_ij).vec();
        A += tmp_A.transpose() * tmp_A;
        b += tmp_A.transpose() * tmp_b;

    }
    **$H deltax =b$**
    delta_bg = A.ldlt().solve(b);
    ROS_WARN_STREAM("gyroscope bias initial calibration " << delta_bg.transpose());

    for (int i = 0; i <= WINDOW_SIZE; i++)
        Bgs[i] += delta_bg;

    for (frame_i = all_image_frame.begin(); next(frame_i) != all_image_frame.end( ); frame_i++)
    {
        frame_j = next(frame_i);
        frame_j->second.pre_integration->repropagate(Vector3d::Zero(), Bgs[0]);
    }
}`
~~~
在ygz中初始化gyro bias代码如下
~~~
 Vector3d Optimizer::OptimizeInitialGyroBias(const vector<SE3d> &vTwc, const vector<IMUPreintegrator> &vImuPreInt) {
        int N = vTwc.size();
        cout<<"vTwc.size():\t"<< N<<'\t'<<vImuPreInt.size()<<std::endl;
        std::cout<< "IMUPreintegration:\t"<<vImuPreInt[0].getDeltaP()<<endl<<vImuPreInt[1].getDeltaP()<<endl<<vImuPreInt[2].getDeltaP()<<endl;

        if (vTwc.size() != vImuPreInt.size()) 
             cout << "vTwc.size()!=vImuPreInt.size()" << endl;

        Matrix4d Tbc = ConfigParam::GetEigTbc();
        Matrix3d Rcb = Tbc.topLeftCorner(3, 3).transpose();

        // Setup optimizer
        g2o::SparseOptimizer optimizer;
        g2o::BlockSolverX::LinearSolverType *linearSolver;

        linearSolver = new g2o::LinearSolverEigen<g2o::BlockSolverX::PoseMatrixType>();

        g2o::BlockSolverX *solver_ptr = new g2o::BlockSolverX(linearSolver);

        g2o::OptimizationAlgorithmGaussNewton *solver = new g2o::OptimizationAlgorithmGaussNewton(solver_ptr);
        //g2o::OptimizationAlgorithmLevenberg* solver = new g2o::OptimizationAlgorithmLevenberg(solver_ptr);
        optimizer.setAlgorithm(solver);
   
        #if 1
        g2o::VertexGyrBias2 *vBiasg = new g2o::VertexGyrBias2();
       // Vector3d data= Eigen::Vector3d::Zero();
         Eigen::Matrix<double, 3, 1> data;
        data<<0,0,0; 
        vBiasg->setEstimate(data);
        vBiasg->setId(0);
        //ccdeng
     //   vBiasg->setMarginalized(true);    
       // vBiasg->setFixed(true);

        optimizer.addVertex(vBiasg);

        #endif
        #if 0
        // Add vertex of gyro bias, to optimizer graph
        g2o::VertexGyrBias *vBiasg = new g2o::VertexGyrBias();
       // Vector3d data= Eigen::Vector3d::Zero();
         Eigen::Matrix<double, 3, 1> data;
        data<<0,0,0; 
        vBiasg->setEstimate(data);
        vBiasg->setId(0);
        //ccdeng
     //   vBiasg->setMarginalized(true);    
        vBiasg->setFixed(true);

        optimizer.addVertex(vBiasg);
        #endif
        #if 1
        // Add unary edges for gyro bias vertex
        //for(std::vector<KeyFrame*>::const_iterator lit=vpKFs.begin(), lend=vpKFs.end(); lit!=lend; lit++)
        for (int i = 0; i < N; i++) {
            // Ignore the first KF
            if (i == 0)
                continue;

            const SE3d &Twi = vTwc[i - 1];    // pose of previous KF
           // cout<< "Twi:\t"<<Twi<<endl;
            Matrix3d Rwci = Twi.rotationMatrix();
            const SE3d &Twj = vTwc[i];        // pose of this KF
            Matrix3d Rwcj = Twj.rotationMatrix();
            const IMUPreintegrator &imupreint = vImuPreInt[i];

            g2o::EdgeGyrBias2 *eBiasg = new g2o::EdgeGyrBias2();
            eBiasg->setVertex(0, dynamic_cast<g2o::OptimizableGraph::Vertex *>(optimizer.vertex(0)));
            // measurement is not used in EdgeGyrBias
　　　　　　　 利用图像的R来矫正gyro 的bias，观测方程为残差
            eBiasg->dRbij = imupreint.getDeltaR();
            eBiasg->J_dR_bg = imupreint.getJRBiasg();
            eBiasg->Rwbi = Rwci * Rcb;
            eBiasg->Rwbj = Rwcj * Rcb;
            eBiasg->setInformation(Eigen::Matrix3d::Identity());
            optimizer.addEdge(eBiasg);
        }
     #endif 
        // It's actualy a linear estimator, so 1 iteration is enough.
        optimizer.setVerbose(true);
        
        optimizer.initializeOptimization();
        optimizer.optimize(1);
      
       g2o::VertexGyrBias2 *vBgEst = dynamic_cast<g2o::VertexGyrBias2 *>(optimizer.vertex(0));//dynamic_cast
        //Eigen::Matrix<double, 3, 1> data2;
      
       Eigen::Vector3d data2= Eigen::Vector3d::Zero();

       data2= vBgEst->estimate().cast<double>();
       cout<<"estimate:\t"<<data2<<endl;

       //cout<<"estimate: "<<vBgEst->estimate()<<endl;
       
        //return  vBgEst->estimate();
        
        return data2;
      // return Eigen::Vector3d::Zero();
    }
~~~


ygz中使用的g2o优化单向图的形式进行bias的初始化过程，采用的是关键帧，在g2o的使用中，需要定义节点，节点表示的是状态量，是需要求解的内容，在这里表示gyro bias的三个维度。　在global BA中，节点包括三维地图点和摄像机的6Dof姿态（SE(3)）.




##2 梯度下降法##

回顾梯度下降法的基本数学模型:
$$ 
minF(x)
$$
$$
subject: x
$$
梯度下降法：
首先求得
$x=x-deltax$$



##3 非线性优化问题##
e(x)表示残差函数
这里需要注意的是：J是谁对谁的函数，大部分时候并不是残差相对于时间t的函数，而可能是残差相对于SE(3)或者se(3)的函数，或者bias的函数。
$$e(x+)=e(x)+J$$




##4 G2o 的使用示例##
 
 一个例子：
拟合
$$
y= Asin(Bx)+ Ccos(Dx)
$$
上述问题可以写成：
$$
min_{A,B,C,D}\sum_{k=1}^n|y-Asin(Bx)-Ccos(Dx)| 
$$
在上述例子中，待优化的向量为$A,B,C,D$ , 残差函数（目标函数为上述cost funtion）.在使用G2o作为优化工具时，需要定义一元边（Unary Edge）.

*** 在选择单向边还是双向边的时候需要注意：在本例子中，可以把`A,B,C,D`作为一个顶点，也可以分成$A,B$和$C,D$组成二元边*，但是在Global BA的时候，在相机姿态SE(3)和地图点时，不能够使用单向边，因为SE(3)的加法和`R^3`上的定义是完全不同的，因此他们的微分也不同。使用单向边的前提是单向边内部的状态向量是在一个空间内的元素，在BA中，姿态SE(3)和空间中的特征点是不同空间中的元素，因此必须使用二元边，如果涉及到更多的不同种类的空间元素，那么需要使用多向边.***


整个图中只有一个顶点，即待优化变量V，连接只有一个顶点的边即一元边（Unary Edge），使用g2o主要有以下几个步骤：
1. 定义顶点和边的类型；
2. 选择优化算法；
3. 构建图；
4. 调用 g2o 进行优化，返回结果。


由于g2o中没有本例类型顶点和边，因此首先需要自己定义。节点和边定义的方式，都是继承BaseVertex和BaseEdge，可以参考优化相机位姿和3D点坐标常用几个类型VertexSE3Expmap、VertexSBAPointXYZ、EdgeSE3ProjectXYZ、EdgeSE3ProjectXYZOnlyPose定义代码，熟悉这种这种模板定义的方式。
~~~
// 曲线模型的顶点(要优化的元素)
// vertex dimension: 4
// vertex type: Vector4d
class CurveFittingVertex: public g2o::BaseVertex<4, Vector4d>
{
public:
    EIGEN_MAKE_ALIGNED_OPERATOR_NEW

    CurveFittingVertex();

    virtual void setToOriginImpl()  // 重置
    {
        _estimate << 0, 0, 0, 0;
    }

    virtual void oplusImpl(const double* update_);// 更新

    //输入输出可以不定义
    bool read(std::istream& is) {}
    bool write(std::ostream& os) const {}
};

// 顶点构造函数
CurveFittingVertex::CurveFittingVertex() : BaseVertex<4, Eigen::Vector4d>() 
{    
}

// 顶点更新函数
void CurveFittingVertex::oplusImpl(const double* update_)   
{
    Eigen::Map<const Vector4d> up(update_);
    _estimate += up;
}
~~~
    

顶点的定义主要需要定义_estimate的更新函数oplusImpl和setToOriginImpl。
estimate数值是状态量或者待优化的变量。
~~~
// 曲线模型的边(要优化的误差)
// unary edge to optimize error
// error vector dimensition: 1
// measurement type: double
// Vertex type: CurveFittingVertex
class CurveFittingEdge: public g2o::BaseUnaryEdge<1, double, CurveFittingVertex>
{
public:
    EIGEN_MAKE_ALIGNED_OPERATOR_NEW
    CurveFittingEdge();

    // 计算曲线模型误差
    void computeError();

    virtual void linearizeOplus();

    bool read(std::istream& is) {}
    bool write(std::ostream& os) const {}

public:
    double _x;
};

// 边的构造函数
CurveFittingEdge::CurveFittingEdge() : g2o::BaseUnaryEdge<1, double, CurveFittingVertex>()
{

}
// 误差函数
void CurveFittingEdge::computeError()
{
    const CurveFittingVertex* v = static_cast<const CurveFittingVertex*>( _vertices[0]);
    const Vector4d abcd = v->estimate();
    double A = abcd[0], B = abcd[1], C = abcd[2], D = abcd[3];
    _error(0,0) = _measurement - (A * sin(B*_x) + C * cos(D*_x));  // 误差函数：观测量减去估计量
}

// Jacobin
void CurveFittingEdge::linearizeOplus()
{
    CurveFittingVertex *vi = static_cast<CurveFittingVertex *>(_vertices[0]);
    Vector4d abcd = vi->estimate();
    double A = abcd[0], B = abcd[1], C = abcd[2], D = abcd[3];
    // 误差项对待优化变量的Jacobin
    _jacobianOplusXi(0,0) = -sin(B*_x);
    _jacobianOplusXi(0,1) = -A*_x*cos(B*_x);
    _jacobianOplusXi(0,2) = -cos(D*_x);
    _jacobianOplusXi(0,3) = C*_x*sin(D*_x);
}
~~~


 









 
 