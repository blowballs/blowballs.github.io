---
typora-root-url: 笔记图像
---

## MSCKF(Multi-State Constraint Kalman Filter) 论文总结（一）

论文：Mourikis A I , Roumeliotis S I . A Multi-State Constraint Kalman Filter for Vision-Aided Inertial Navigation[J]. 2007.
下面主要对该论文的重点内容进行记录、总结，可能会涉及部分自己的理解。

### 摘要
作者提出了一种基于扩展卡尔曼（EKF）的滤波算法来实现实时的视觉辅助惯性导航。文章的主要贡献工作是推导了多相机位姿观测同一个静态特征时形成的地理约束的测量模型。该测量模型不需要在EKF的状态向量里包含3D特征点位置，而且在线性化误差范围里是最优的。该视觉惯性导航算法的时间复杂度只与特征点的数量成线性关系，而且能够在大尺度实际场景中提供高精度位姿估计。作者在实际城市区域进行了大量的实验对算法进行验证。

### 1 状态估计算法

文章提出的基于EKF的估计器目标是跟踪IMU固连坐标系（*I*系）相对全局参考坐标系（*G*系）的3D位姿。为了简化地球自转角速度对IMU测量的影响，文章中的全局坐标系选为地心固定坐标系（ECEF系）。算法的整体介绍见**Algorithm 1**。

![](/../MSCKF(Multi-State%20Constraint%20Kalman%20Filter)%20%E8%AE%BA%E6%96%87%E6%80%BB%E7%BB%93%EF%BC%88%E4%B8%80%EF%BC%89.assets/Algorithm%201.PNG)

如上图所见，IMU的测量每次都只要可用便即可处理，在EKF里基于此进行状态量和协方差的预测更新（见1.2）。另一方面，每次当一张图像被记录时，在状态向量里增加当前的相机位姿估计量（见1.3）。对状态量进行增广对于处理视觉特征测量非常有必要，因为当EKF更新时，每一个跟踪到的视觉特征测量都会被用来在观测到这些视觉特征的所有相机位姿间建立约束。因此，在任何时刻，EKF的状态向量包含 1）IMU状态，$X_{IMU}$；2）最大$N_{max}$个过去的相机位姿。

#### 1.1 EKF状态向量的结构

IMU的状态向量展开如下所示：

<img src="/../MSCKF(Multi-State%20Constraint%20Kalman%20Filter)%20%E8%AE%BA%E6%96%87%E6%80%BB%E7%BB%93%EF%BC%88%E4%B8%80%EF%BC%89.assets/1568014948362.png" alt="1568014948362" style="zoom: 67%;" />

上式中，q是单位四元数，描述了*G*系到*I*系得旋转，$p_I^G$和$v_I^G$表示IMU相对于*G*系的位置和速度。$b_g$和$b_a$是3*1的向量，分别表示影响陀螺仪和加速度计测量的零偏。IMU的零偏建模为随机游走过程，通过高斯白噪声向量$n_{wg}$和$n_{wa}$进行推导。同时，可得IMU的误差状态可定义为：

<img src="/../MSCKF(Multi-State%20Constraint%20Kalman%20Filter)%20%E8%AE%BA%E6%96%87%E6%80%BB%E7%BB%93%EF%BC%88%E4%B8%80%EF%BC%89.assets/1568017155218.png" alt="1568017155218" style="zoom:67%;" />

对于位置、速度、和零偏，使用标准的相加误差定义（误差$\breve{x} = 变量x - 估计值\hat{x}$）。但是，对于四元数误差定义是不同的。特殊的是，$q = \delta q \bigotimes \hat{q}$， *q*是四元数，$\delta q$是四元数的估计值误差， $\hat{q}$是四元数的估计值。该表达中，$\bigotimes$表示四元数乘法。误差四元数可定义为”

<img src="/../MSCKF(Multi-State%20Constraint%20Kalman%20Filter)%20%E8%AE%BA%E6%96%87%E6%80%BB%E7%BB%93%EF%BC%88%E4%B8%80%EF%BC%89.assets/1568018218590.png" alt="1568018218590" style="zoom:67%;" />

直观可见，四元数误差表示了真实姿态到估计的姿态间的微小旋转。由于姿态对应三个自由度，使用$\delta\theta$可以对姿态误差进行最小维度描述。

假设在*k*时刻， 有*N*个相机位姿包含在EKF的状态向量里，则该向量具有下面的格式：

<img src="/../MSCKF(Multi-State%20Constraint%20Kalman%20Filter)%20%E8%AE%BA%E6%96%87%E6%80%BB%E7%BB%93%EF%BC%88%E4%B8%80%EF%BC%89.assets/1568018476379.png" alt="1568018476379" style="zoom:67%;" />

上式中，$\hat{q}和\hat{p}$分别表示*i = 1...N*个序列的相机姿态和位置估计值。则EKF的误差状态向量同时可以表示为：

<img src="/../MSCKF(Multi-State%20Constraint%20Kalman%20Filter)%20%E8%AE%BA%E6%96%87%E6%80%BB%E7%BB%93%EF%BC%88%E4%B8%80%EF%BC%89.assets/1568018626110.png" alt="1568018626110" style="zoom:67%;" />

