---
layout:     post
title:      "Paper recurrence:Discovering governing equations from data by sparse identification of nonlinear dynamical systems"
date:       2021-01-27 11:16:00
author:     "Alanby"
header-img: "img/post-seu-build01.jpg"
tags:
    - data driver
    - governing equation
    - nonlinear dynamical system
---

<script src="https://cdn.mathjax.org/mathjax/latest/MathJax.js?config=TeX-AMS-MML_HTMLorMML" type="text/javascript"></script>
<script type="text/x-mathjax-config">
  MathJax.Hub.Config({
    tex2jax: {
      inlineMath: [ ['$','$'], ["\\(","\\)"] ],
      processEscapes: true
    }
  });
</script>


# 数据驱动论文复现--基于稀疏辨识构建 非线性动力系统的控制方程

*Discovering governing equations from data by sparse identification of nonlinear dynamical systems*   

*Steven L. Bruntona,1, Joshua L. Proctorb , and J. Nathan Kutzc*



Qinyan Zhou, Zhen Li，Zhihai Bi



## 1. 背景

​		从数据中提取控制方程是一个很大的挑战。往往是数据很多，很难建立模型。前人有从 数据中确定非线性动力系统的方法，比如 Bongard 和 Lipson 等人提出了使用符号回归来寻 找非线性微分方程。但是符号回归代价较大，难以拓展到其他感兴趣的大型系统，容易过拟 合，总体效果一般。

​		本文将稀疏性技术、机器学习与非线性动力系统相结合，探讨从含有噪声的测量数据中 提取控制方程的方法。文中的假设是：如果只有几个重要的项目支配着动力学，那么方程可 能的函数空间中是稀疏的，这个假设在很多物理系统中都是成立的，结果表明该算法在效率 和鲁棒性上都表现得不错。





## 2. 意义

​		这项工作开发了一个新的框架，利用稀疏技术和机器学习的进步，简单地从数据测量中 发现动力系统背后的控制方程。由此产生的简洁的模型，平衡了模型的复杂性和描述能力， 同时避免了过度拟合。有许多关键的数据驱动问题，比如从神经记录中理解认知，推断气候 模式，确定金融市场的稳定性，预测和抑制疾病的传播，以及控制湍流，实现更绿色的交通 和能源等。





## 3. 稀疏辨识

​		非线性动力学的稀疏辨识(SINDy)：大部分的物理系统可以通过较少的参数来定义动力 学，构造高维非线性稀疏函数来确定物理系统，依赖于测量变量、数据质量和稀疏化函数基 的选择。根据这种形式的动力系统：
$$
\frac{d}{dt}x(t) = f(x(t))   \qquad \qquad (1)
$$
向量$x(t) \in R^n$ 表示系统在$t$ 时刻的状态，函数$f(x(t))$ 表示动力学系统的动态约束。在许多物理系统中，函数$f(x(t))$ 只需要较少参数，从而能在高维空间中符合稀疏条件，在压缩感知和解析回归的前提下，可以确定函数的非零项，因此能更好地利用稀疏辨识的方 法。这样就可以利用产生的稀疏模型高效处理凸优化问题，在平衡复杂性与准确性的同时避免了过拟合现象的发生。我们可以使用稀疏回归的方法来对观测到的物理量进行降噪处理，通过收集下$x(t)$状态的时间历史，并观测$\dot{x}(t)$ ,列出两矩阵：

![image](https://github.com/Alanaab/Alanaab.github.io/raw/master/img/Paper_data_driven_01.png)

构造备选函数组合库$\theta(x)$, 线性函数与非线性函数

![image](https://github.com/Alanaab/Alanaab.github.io/raw/master/img/Paper_data_driven_02.png)

备选函数库$\theta(x)$中的每一列的非零项都代表函数$f(x(t))$中的实际参数，构建动力学系统的动态约束。因为每个方程只有少数非线性是活跃的，所以可以建立稀疏回归矩阵问题 来确定稀疏向量中的$\Xi = [\xi_1 \quad \xi_2 \quad ...\quad \xi_n]$ 的活跃系数。
$$
\dot{X} = \theta(X)\Xi
$$
这决定了稀疏向量$\Xi$中的每一列$\xi_k$， 从而可以构建出每一行的控制方程：
$$
\dot{x}_k = f_k(x) = \theta(x^T)\xi_k
$$
$\theta(x^T)$是x元素的符号函数向量
$$
\dot{x} = f(x) = \Xi^T(\theta(x^T))^T
$$
每一列$\xi_k$都需要一个特定优化来找到第K行方程稀疏的稀疏向量。实际观测过程中，观测到的$X$ 和$\dot{X}$ 都是被噪声污染的，因此上述等式应变为
$$
\dot{X} = \theta(X)\Xi + \eta Z
$$
因此，我们寻求具有噪声的超定系统的稀疏解，通过回归算法促进稀疏性来处理这类数据。当只能观测到 X ，而不能观测到 $\dot{X}$时，可以通过微分得到 $\dot{X}$ ，使用总变分正则化去噪。 如果面对的时间状态维数 n 极大的系统，我们可以在低维流形或吸引子演化，使用降维技术如正交分解（POD），将高维系统演化为低维系统。如果未能预先知道正确变量，可以通过时滞坐标提供有用变量。根据物理学基础知识提出的稀疏识别模型，其稀疏性和准确性可以提供关于正确测量坐标和基础的有价值的诊断信息。





## 4. 全变分正则化

#### 4.1 差分存在的问题

​		在很多情况下，实验测得的时间序列数据 X 是可以直接得到的，但是$\dot{X}$需要通过计算得到。一般的计算方法是利用带有时间序列的数据，前后差分代替：

![image](https://github.com/Alanaab/Alanaab.github.io/raw/master/img/Paper_data_driven_03.png)

考虑虑到通常实验中采集的数据是带有噪声的，用这种简单的方式求导很可能会出现很大的偏差，例如：实验中的数据理论上是以 x 等于 0.5 为对称轴，斜率为 1 的绝对值函数。但是实际采集到的数据有很多毛刺，如下图所示：

![image](https://github.com/Alanaab/Alanaab.github.io/raw/master/img/Paper_data_driven_04.png)

这时候如果还用简单的差分求导，会将原本的噪声放大，求导结果如下图所示，红线 表示期望的斜率值，黑线是噪声利用简单差分后求出来的斜率值。



#### 4.2 全变分正则化

​		为了消除上述的误差，文中引用了另外一篇文章的作者 Rick Chartrand 的方法，用全 变分正则化的方法去求 dx，结果显示全变分约束降噪可以很好的消除噪声。具体的数学推导没有完全明白，尝试梳理一下。		

​		首先作者在文章中直接给出了一个定义在[0,L]上的平滑函数的形式。它由两项组成： 第一项是对微分求导的绝对值求积分和，表示如果对原函数求出来的微分不平滑，毛刺很 多的话，这一项数值上会比较大，如果平滑，趋向于常数，那么第一项趋向于 0；Au 表示 根据微分反推出原函数 f，与期望的原函数 f 做差，表示与原来数据的相似程度，也就是保 正度，值越小，相似程度越大。这两项既保证了与原来数据的相似程度，也保证了数据求 出的微分的平滑程度。

![image](https://github.com/Alanaab/Alanaab.github.io/raw/master/img/Paper_data_driven_05.png)

接下来的目的就是要最小化这个函数，理论上就能求出理想的 dx。文中提出了利用梯度下 降的思想。文中提出用拉格朗日乘子法将上式的偏微分方程表示成了以下形式：

![image](https://github.com/Alanaab/Alanaab.github.io/raw/master/img/Paper_data_driven_06.png)

推到过程没有完全明白，看了其他资料，当$u(x)$满足欧拉-拉格朗日函数时，泛函取极 值。想到可能是用梯度下降前，为了保证迭代能收敛，目标函数要有极小值。在 Matlab 代 码中，利用 pcg 方法迭代收敛后求出来的 u 就是我们所需要的 dx。

**注：全变分正则化原理的具体参考资料看最后补充信息**





## 5. Lorenz混沌系统复现

1963 年，Lorenz 发现了第一个混沌吸引子——Lorenz 系统，从此揭开了混沌研究的序 幕。人们不断发现新的混沌奇异性，不断地加深与统一对混沌的理解。混沌系统是指在一个 确定性系统中，存在着貌似随机的不规则运动，其行为表现为不确定性、不可重复、不可预 测，这就是混沌现象。混沌是非线性动力系统的固有特性，是非线性系统普遍存在的现象。 Lorenz 系统是数值试验中最早发现的呈现混沌运动的耗散系统，其状态方程为：
$$
\dot{x} = \sigma(y-x) \\
\dot{y} = x(\rho - z) -y \\
\dot{z} = xy - \beta z 
$$
该系统的一个简单物理实现是流体在下方加热上方冷却的热对流管中的环流。对于本 例，我们使用标准参数𝜎 = 10, 𝛽 = 8/3, 𝜌 = 28，初始条件为$[𝑥 \quad 𝑦 \quad 𝑧]^T  = [-8 \quad 7 \quad 27]^T$。从$t=0$到$t=100$ 收集数据，时间步长为$\Delta t = 0.001$ 。待选基函数上限为5阶。<u>**（为什么是5阶呢，这个候选参数要如何去选择，是关键的地方，具体参考论文补充材料）**</u> 

![image](https://github.com/Alanaab/Alanaab.github.io/raw/master/img/Paper_data_driven_07.png)

为了探讨加噪导数对系统的影响，我们在状态变量的导数中加入方差为  的零均值高 斯测量噪声。对于两个不同的噪声值 $\eta$=0:01 和 $\eta$=10，分别短时间（t=0 到 t=20）和长时 间（t=0 到 t=250）进行系统重构，重构结果和原系统的对比如图 3 所示。

![image](https://github.com/Alanaab/Alanaab.github.io/raw/master/img/Paper_data_driven_08.png)

为了探讨随着时间推移，重构系统与真实系统的误差的变化趋势，我们绘制了重构前 后状态变量随时间的变化图，并在两个不同的噪声值η=0:01 和 η=10 下进行对比，对比结 果如下图所示：

![image](https://github.com/Alanaab/Alanaab.github.io/raw/master/img/Paper_data_driven_09.png)

这里有个问题，为什么随着时间的推移，误差会越来越大呢？其实是洛伦兹系统是混沌系统的原因，具体需要看看混沌系统的概念。图 5 显示了在不同噪声的方差 $\eta$ 的情况下，误差与时间的关系。尽管对于较大的噪声 值，重构系统与真实系统的误差也会更大，但方程的形式都能被准确地构建。

![image](https://github.com/Alanaab/Alanaab.github.io/raw/master/img/Paper_data_driven_10.png)

接下来，我们探讨了当只有带噪的状态x可观测时，Lorenz 方程上的 SINDy 算法。在图 6 中，将方差为 $\eta$ =1.0 的高斯噪声添加到 x 中，并使用全变分正则化方法计算导数。可得即使对于相对较大的噪声幅度，该方法仍然可以辨识出系统。

![image](https://github.com/Alanaab/Alanaab.github.io/raw/master/img/Paper_data_driven_11.png)

图 7 展示了当只有带噪的状态 x 可观测时，$\eta$变化下误差与时间的关系。同样，由于 洛伦兹吸引子的混沌性质，误差迅速地随着时间增长。然而，即使对于较大的噪声量级， 但方程的形式依然能被准确地构建。

![image](https://github.com/Alanaab/Alanaab.github.io/raw/master/img/Paper_data_driven_12.png)





## 6. 结论

​		本文主要介绍了一种基于稀疏辨识构建非线性动力系统的控制方程的方法 sparse identification of nonlinear dynamics (SINDy)，并复现论文中 Lorenz 混沌系统的案例。后面还有几个难一点的例子，没有深究，因为主要是关注方法。由结果所示，该方法在已知状态变量和状态变量的微分时，可以还原原系统，若状态变量微分未知，还可以通过全变分正则化方法计算导数，从而减小噪声，重构已知状态变量的系统。但该方法仍然高度依赖于测量变量、数据质量和稀 疏化函数基的选择，在应用中需根据实际情况调整和选择。



## 7. 主要代码

思路如下：得知实验测量数据X

情况一：若已知$\dot{X}$，则利用X算出所有可能的theta，然后利用$\dot{X}$去筛选theta，得到预测的Xi；

情况二：未知$\dot{X}$, 先利用全微分正则化去估算$\dot{X}$，目的时为了去噪,  准确估计，其他步骤完全一样。

```matlab
%========================================= 情况一 =========================================================
%Copyright 2015, All Rights Reserved
% Code by Steven L. Brunton
% For Paper, "Discovering Governing Equations from Data: 
%        Sparse Identification of Nonlinear Dynamical Systems"
% by S. L. Brunton, J. L. Proctor, and J. N. Kutz
% Last edit by alanby 2021/1/27

clear all, close all, clc
figpath = '../figures/';
addpath('./utils');

%% generate Data
polyorder = 5;   % the max ploy order，but why 5? how to determine this parameter is the key
usesine = 0;     % use sin or cos to predict the function

sigma = 10;  % three Lorenz's parameters (chaotic)
beta = 8/3;  
rho = 28;

n = 3;  %the column of input data（X）

x0=[-8; 7; 27];  % Initial condition

% Integrate
tspan=[.001:.001:100]; % time of lorenz 
N = length(tspan);     % discrete signal 
options = odeset('RelTol',1e-12,'AbsTol',1e-12*ones(1,n));  % a option parameter to define to accuracy of lorenz
[t,x]=ode45(@(t,x) lorenz(t,x,sigma,beta,rho),tspan,x0,options); % ode45, a function to slove the differential equation

%% compute Derivative
eps = 0.01;  % the amplify of noise
for i=1:length(x)
    dx(i,:) = lorenz(0,x(i,:),sigma,beta,rho);
end
dx = dx + eps*randn(size(dx));   % randn the noise of Gaussian distribution？

%% pool Data  (i.e., build library of nonlinear time series)
Theta = poolData(x,n,polyorder,usesine);  % build the library, the key !!!
m = size(Theta,2); 

%% compute Sparse regression: sequential least squares
lambda = 0.025; % lambda is our sparsification knob. how to choose the lambda?
Xi = sparsifyDynamics(Theta,dx,lambda,n) % regression, this function is kind of hard to understand
poolDataLIST({'x','y','z'},Xi,n,polyorder,usesine); 

%% FIGURE 1:  LORENZ for T\in[0,20]
tspan = [0 20];
[tA,xA]=ode45(@(t,x)lorenz(t,x,sigma,beta,rho),tspan,x0,options);   % true model
[tB,xB]=ode45(@(t,x)sparseGalerkin(t,x,Xi,polyorder,usesine),tspan,x0,options);  % approximate

figure
subplot(1,2,1) %(row,colunm ,num)
dtA = [0; diff(tA)];
color_line3(xA(:,1),xA(:,2),xA(:,3),dtA,'LineWidth',1.5); % too hard!!!
view(27,16) % set the angle of the view from which an observer sees the current 3-D plot.
grid on
xlabel('x','FontSize',13)
ylabel('y','FontSize',13)
zlabel('z','FontSize',13)
set(gca,'FontSize',13) % set fontsize
subplot(1,2,2)
dtB = [0; diff(tB)];
color_line3(xB(:,1),xB(:,2),xB(:,3),dtB,'LineWidth',1.5);
view(27,16)
grid on

% Lorenz for t=20, dynamo view
figure
subplot(1,2,1)
plot(tA,xA(:,1),'k','LineWidth',1.5), hold on
plot(tB,xB(:,1),'r--','LineWidth',1.5)
grid on
xlabel('Time','FontSize',13)
ylabel('x','FontSize',13)
set(gca,'FontSize',13)
subplot(1,2,2)
plot(tA,xA(:,2),'k','LineWidth',1.5), hold on
plot(tB,xB(:,2),'r--','LineWidth',1.5)
grid on


%% FIGURE 2:  LORENZ for T\in[0,250]
tspan = [0 250];
options = odeset('RelTol',1e-6,'AbsTol',1e-6*ones(1,n));
[tA,xA]=ode45(@(t,x)lorenz(t,x,sigma,beta,rho),tspan,x0,options);   % true model
[tB,xB]=ode45(@(t,x)sparseGalerkin(t,x,Xi,polyorder,usesine),tspan,x0,options);  % approximate

figure
subplot(1,2,1)
dtA = [0; diff(tA)];
color_line3(xA(:,1),xA(:,2),xA(:,3),dtA,'LineWidth',1.5);
view(27,16)
grid on
xlabel('x','FontSize',13)
ylabel('y','FontSize',13)
zlabel('z','FontSize',13)

subplot(1,2,2)
dtB = [0; diff(tB)];
color_line3(xB(:,1),xB(:,2),xB(:,3),dtB,'LineWidth',1.5);
view(27,16)
grid on
xlabel('x')
ylabel('y')
zlabel('z')
```



```matlab
% ============================================== 情况二 ===================================================  
%Copyright 2015, All Rights Reserved
% Code by Steven L. Brunton
% For Paper, "Discovering Governing Equations from Data: 
%        Sparse Identification of Nonlinear Dynamical Systems"
% by S. L. Brunton, J. L. Proctor, and J. N. Kutz

% Note, for larger error terms, it helps to remove constant terms in
% "poolData"
% Last edit by alanby 2021/1/27

clear all, close all, clc
figpath = '../figures/';
addpath('./utils');

%% generate Data
polyorder = 5;
usesine = 0;
sigma = 10;  % Lorenz's parameters (chaotic)
beta = 8/3;
rho = 28;
n = 3;
x0=[-8; 7; 27];  % Initial condition

% Integrate
dt = 0.001;
tspan=[dt:dt:50];
N = length(tspan);
options = odeset('RelTol',1e-12,'AbsTol',1e-12*ones(1,n));
[t,x]=ode45(@(t,x) lorenz(t,x,sigma,beta,rho),tspan,x0);
xclean = x;
% add noise
eps = .01;
x = x + eps*randn(size(x));
% compute clean derivative  (just for comparison!)
for i=1:length(x)
    dxclean(i,:) = lorenz(0,xclean(i,:),sigma,beta,rho);
end

%%  Total Variation Regularized Differentiation
dxt(:,1) = Total_Variation_Reg( x(:,1), 10, .00002,  1e12, dt, 1, 1 );
% [                   输入向量、10此迭代、正则项系数、迭代初始化值、项数较少、
% 1e12: Parameter for avoiding division by zero, ]
hold on
plot(dxclean(:,1),'r')
xlim([5000 7500])
figure
dxt(:,2) = TVRegDiff( x(:,2), 10, .00002, 1e12, dt, 1, 1 );
hold on
plot(dxclean(:,2),'r')
xlim([5000 7500])
figure
dxt(:,3) = TVRegDiff( x(:,3), 10, .00002,  1e12, dt, 1, 1 );
hold on
plot(dxclean(:,3),'r')
xlim([5000 7500])

%
xt(:,1) = cumsum(dxt(:,1))*dt;
xt(:,2) = cumsum(dxt(:,2))*dt;
xt(:,3) = cumsum(dxt(:,3))*dt;
xt(:,1) = xt(:,1) - (mean(xt(1000:end-1000,1)) - mean(x(1000:end-1000,1)));
xt(:,2) = xt(:,2) - (mean(xt(1000:end-1000,2)) - mean(x(1000:end-1000,2)));
xt(:,3) = xt(:,3) - (mean(xt(1000:end-1000,3)) - mean(x(1000:end-1000,3)));
xt = xt(1000:end-1000,:);
dxt = dxt(1000:end-1000,:);  % trim off ends (overly conservative)

%% pool Data  (i.e., build library of nonlinear time series)
Theta = poolData(xt,n,polyorder,usesine);
m = size(Theta,2);

%% compute Sparse regression: sequential least squares
lambda = 0.02;      % lambda is our sparsification knob.
Xi = sparsifyDynamics(Theta,dxt,lambda,n)

%% FIGURE 1:  LORENZ for T\in[0,20]
tspan = [0 20];
[tA,xA]=ode45(@(t,x)lorenz(t,x,sigma,beta,rho),tspan,x0,options);   % true model
[tB,xB]=ode45(@(t,x)sparseGalerkin(t,x,Xi,polyorder,usesine),tspan,x0,options);  % approximate

figure
subplot(1,2,1)
dtA = [0; diff(tA)];
color_line3(xA(:,1),xA(:,2),xA(:,3),dtA,'LineWidth',1.5);
view(27,16)
grid on
xlabel('x','FontSize',13)
ylabel('y','FontSize',13)
zlabel('z','FontSize',13)
set(gca,'FontSize',13)
subplot(1,2,2)
dtB = [0; diff(tB)];
color_line3(xB(:,1),xB(:,2),xB(:,3),dtB,'LineWidth',1.5);
view(27,16)
grid on

% Lorenz for t=20, dynamo view
figure
subplot(1,2,1)
plot(tA,xA(:,1),'k','LineWidth',1.5), hold on
plot(tB,xB(:,1),'r--','LineWidth',1.5)
grid on
xlabel('Time','FontSize',13)
ylabel('x','FontSize',13)
set(gca,'FontSize',13)
subplot(1,2,2)
plot(tA,xA(:,2),'k','LineWidth',1.5), hold on
plot(tB,xB(:,2),'r--','LineWidth',1.5)
grid on


%% FIGURE 1:  LORENZ for T\in[0,250]
tspan = [0 250];
options = odeset('RelTol',1e-6,'AbsTol',1e-6*ones(1,n));
[tA,xA]=ode45(@(t,x)lorenz(t,x,sigma,beta,rho),tspan,x0,options);   % true model
[tB,xB]=ode45(@(t,x)sparseGalerkin(t,x,Xi,polyorder,usesine),tspan,x0,options);  % approximate

figure
subplot(1,2,1)
dtA = [0; diff(tA)];
color_line3(xA(:,1),xA(:,2),xA(:,3),dtA,'LineWidth',1.5);
view(27,16)
grid on
xlabel('x','FontSize',13)
ylabel('y','FontSize',13)
zlabel('z','FontSize',13)

subplot(1,2,2)
dtB = [0; diff(tB)];
color_line3(xB(:,1),xB(:,2),xB(:,3),dtB,'LineWidth',1.5);
view(27,16)
grid on
xlabel('x')
ylabel('y')
zlabel('z')
```

注：*全部代码实现请参考原论文作者附录资料*
