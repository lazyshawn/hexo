---
title: 基于Matlab的二指夹具在手操作仿真（一）
tags: [Matlab, Robot-hand]
categories: 术业专攻
date: 2020-03-03 12:34:53
---

> 二指夹具在重力作用下进行在手操作，通过控制二指夹具施加的加持力，让工具精确地下降指定距离。利用Matlab仿真验证控制方案的可行性。

<!--more-->

## 模型的建立
本研究中的二指夹具仅有一个自由度，在手指位置具有半球形的软垫，可以用来加持工具。记工具顶端到夹具中心位置的竖直距离为$h(t)$，工具在任意时刻的状态$x(t)=[h(t), \dot h(t)]^T$可以由传感器测量得到。假设工具竖直下落，不会产生转动。故夹具的加持动作只产生一个摩擦力，而不会产生摩擦力矩。我们的控制目标是让工具从开始位置$h_0(t_0)$，按期望的路径$h_d(t)$竖直下落到指定位置$h_d(t_f)$[<sup>[1]</sup>](#ref-anchor)。运动过程如下图所示：
![运动过程示意图](http://q789z4z6g.bkt.clouddn.com/sim1.png?center)

### 滑动摩擦力模型
考虑摩擦力由库伦摩擦力和黏滞摩擦力组成，即
$$f=-\mu sgn(\dot h)f_n - \sigma \dot h \tag{1}$$
式中$\mu$为库伦摩擦系数，$f_n$为手指施加给工具的正压力，$\dot h$为工具下落速度，$sgn(\cdot)$为符号函数，$\sigma$为黏滞摩擦系数。从式$(1)$中我们可以得到施加的正压力和摩擦力之间的关系。

### 动力学模型
由牛顿第二定律可以得到该系统的动力学方程为：
$$ m\ddot h = -mg + f \tag{2}$$
将式$(1)$带入上式得到如下的动力学模型：
$$m\ddot h = -mg - \mu sgn(\dot h)f_n - \sigma \dot h \tag{3}$$

## 自适应控制模型的设计
为了推到自适应控制方程，我们将式$(3)$描述的动力学模型改写为以$u_{f_n}$为输入信号的形式：
$$ a\ddot{h} + b\dot{h} + cg = u_{f_n} \tag{4}$$
式中$a = c = m/\mu$，$b = \sigma/\mu$。然后定义如下的跟踪误差：
$$s = \dot{\tilde{h}} + \lambda\tilde{h} \tag{5}$$
式中$\dot{\tilde{h}}=\dot{h}-\dot{h}_d$，$\tilde{h} = h-h_d$分别为位置和速度误差，$\lambda$为一常数。

这样我们就能写出标准的自适应控制方程[<sup>[2]</sup>](#ref-anchor)
$$u_{f_n} = \hat{a}\ddot{h}_r - ks + \hat{b}\dot{h} + \hat{c}g \tag{6}$$

该控制方程共由以下几部分组成：反馈加速度项$\hat{a}\ddot{h}_r$、跟踪误差项$ks$、速度误差项$\hat{b}\dot{h}$以及重力补偿项$\hat{c}g$。其中参考加速度$\ddot{h}_r = \ddot{h}_d - \lambda\dot{\tilde{h}}$，$k$是一个正的跟踪控制激励，$\hat{a}, \hat{b}, \hat{c}$分别为$a, b, c$的适应性估计值，根据下式进行调整：
$$\begin{array}{c}
\dot{\hat{a}} &=& -\alpha_a s\ddot{h}_r  \\
\dot{\hat{b}} &=& -\alpha_b s\dot{h}     \\
\dot{\hat{c}} &=& -\alpha_c sg \tag{7}
\end{array}$$
其中$\alpha_a, \alpha_b, \alpha_c$均为正的适应激励。值得注意的是式$(7)$给出的实时估计值并不能保证对应的参数收敛到真值，除非满足持续激励条件（persistent excitation conditions)。但这并不是我们考虑的问题，因为我们项目只需要保证工具的运动，而不是摩擦参数的估计值的准确性。这个控制方案也的确保证了跟踪误差$s$的收敛性，说明我们工具能准确运动到指定位置。


## 验证控制模型
由于我们的模型相对简单，只有一个自由度，外力只需要考虑垂直方向上的重力和摩擦力，所以可以很容易的通过编程来模拟运动过程，以此验证控制算法的准确性。同样用Matlab编程，程序如下：
```matlab
% 基于Matlab的二指夹具在手操作仿真
% --- 符号规定 ---
% t: 时间序列, tt: 时间间隔, h: 实际位置, hd: 目标位置, 
% hv: 位置误差, dhv: 速度误差, s: 跟踪误差,
% miu: 库伦摩擦系数, sigma: 黏滞摩擦系数, g: 重力加速度, 
% fn: 正压力, f:总摩擦力,
% lamb: 控制常量, k: 跟踪控制激励, 
% alpha_a, alpha_b, alpha_c: 控制方程调参系数,
% ae, be, ce: 控制方程系数的估计值,
clear;  clc;

% --- 控制参数调整
lamb = 1;
k = 10;
alpha_a = 4;
alpha_b = 4;
alpha_c = 4;
ae(1) = 0;
be(1) = 0;
ce(1) = 0;

% --- 模型常量估计值
g = 9.8;
miu = 0.5;
sigma = 0.5;
m = 1;

% --- 系统初值及参考模型
tt = 0.01;
t = 0: tt: 10;
N = length(t);
h0 = 0;
hd = -1*ones(N,1);

% --- 循环变量初始化
h = zeros(N,1);
dh = zeros(N,1);
ddh = zeros(N,1);

% For begining
h(1) = h0;
dh(1) = 0;
ddh(1) = 0;
dhv(1) = 0;

% Loop begin
for i = 2:N
    h(i) =  h(i-1) + dh(i-1)*tt + 1/2*ddh(i-1)*tt*tt;
    if (h(i)>h(i-1))
        h(i) = h(i-1);
    end
    
    dh(i) = (h(i) - h(i-1))/tt;
    dhd(i) = (hd(i) - hd(i-1))/tt;
    ddhd(i) = (dhd(i) - dhd(i-1))/tt;
    hr(i) = ddhd(i) - lamb*dhv;
    hv = h(i) - hd(i);
    dhv = dh(i) - dhd(i);
    s(i) = dhv + lamb*hv;
    
    ae(i) = ae(i-1) - alpha_a*s(i)*hr(i);
    be(i) = be(i-1) - alpha_b*s(i)*dh(i);
    ce(i) = ce(i-1) - alpha_c*s(i)*g;
    
    ufn(i) = ae(i)*hr(i) - k*s(i) + be(i)*dh(i) + ce(i)*g;
    fn(i) = ufn(i);
    if (fn(i) < 0)
        fn(i) = 0;
        f(i) = 0;
    else
        f(i) = miu*fn(i) - sigma*dh(i);
    end
    ddh(i) =  f(i)/m - g;
end

% 绘图
subplot(2,2,1)
plot(t, hd, ':');
hold on;
plot(t, h);
set(gca,'YLim',[1.1*min(h), 0]);
title('运动路径');
xlabel('t/s');
ylabel('h/m');
legend('Ref', 'Real');

subplot(2,2,2)
plot(t,fn);
hold on;
plot(t, f);
title('加持力&摩擦力');
xlabel('t/s');
ylabel('f/N');
legend('f_n','f');

subplot(2,2,3)
plot(t, dh)
title('运动速度')
xlabel('t/s');
ylabel('v/m\cdots^{-1}');

```
运行程序，观察生成的图像(如下图所示)，基本符合实际运动规律，可以认为控制模型合理。
![程序模拟的运动曲线](http://q789z4z6g.bkt.clouddn.com/sim2.svg?center)


## 相关链接
<div id="ref-anchor"></div>

[1].  Francisco, E. Vina B. , et al. "Adaptive control for pivoting with visual and tactile feedback." 2016 IEEE International Conference on Robotics and Automation (ICRA) IEEE, 2016. <br>
[2]. J.Slotine and W.Li, "Applied Nonlinear Control." Prentice Hall, 1991. <br>
