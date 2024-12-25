# 标准DH法（SDH，Standard DH）和改进DH法（Modified DH）
## 使用DH方法的条件
1. *使用DH方法的必要条件：$X_i$垂直于$Z_{i-1}$或$X_i$ 与$Z_{i-1}$，千万不能平行。*
2. 可以通过调整坐标系或者加入额外坐标系的方法达成必要条件
3. 如果仅仅改变x轴方向无法达成，则需要加入额外坐标系，根据旋转副和移动副的不同，*旋转副分旋转、移动两步，移动副分移动、旋转两步*，因为旋转和移动的表示都是相对于前一个z轴的
## 标准DH法
### 坐标系
坐标系建在杆件的远端
1. $Z_{i-1}$轴：与$A_i$关节轴重合，方向任意
2. $X_{i-1}$轴：$Z_{i-1}$轴与$Z_i$的公法线方向，方向由$Z_{i-1}$指向$Z_i$
3. $Y_{i-1}$轴：右手定则
### DH参数
1. $d_i$=沿$z_{i-1}$轴，从$x_{i-1}$移动到$x_i$的距离
2. $\theta_i$=绕$z_{i-1}$轴，从$x_{i-1}$旋转到$x_i$的角度
3. $a_i$=沿$x_i$轴，从$z_{i-1}$轴移动到$z_i$轴的距离
4. $\alpha_i$=绕$x_i$轴，从$z_{i-1}$轴旋转到$z_i$轴的角度

## 改进DH法
### 坐标系
坐标系建在杆件的远端
1. $Z_i$轴：与$A_i$关节轴重合，方向任意
2. $X_i$轴：$A_i$关节轴与$A_{i+1}$的公法线方向，方向由$A_1$指向$A_{i+1}$
3. $Y_i$轴：右手定则
### DH参数
1. $a_{i-1}$=沿$x_{i-1}$轴，从$z_{i-1}$轴移动到$z_i$轴的距离
2. $\alpha_{i-1}$=绕$x_{i-1}$轴，从$z_{i-1}$轴旋转到$z_i$轴的角度
3. $d_i$=沿$z_i$轴，从$x_{i-1}$移动到$x_i$的距离
4. $\theta_i$=绕$z_i$轴，从$x_{i-1}$旋转到$x_i$的角度

| i   | alpha | a   | d    | theta |
| --- | ----- | --- | ---- | ----- |
| 1   | 0     | 0   | 1685 | 0     |
| 2   | -pi/2 | 270 | 0    | pi/2  |
| 3   | -pi/2 | 128 | 0    | -pi/2 |
| 4   | 0     | 0   | 1025 | 0     |
| 5   | 0     | 0   | 1020 | 0     |
# Matlab Robtics Toolbox 中由DH参数创建机械臂
在matlab中，由于d、theta是转动副或移动副的变量，对于转动副，theta值在DH中的输入不起作用；对于移动副，d的值在输入时不起作用
可以通过修改qlim和arm.plot的参数修正
## 例子
```MATLAB
%% 机械臂可达空间动画求解
clc;
clear;
close;

% [theta, d, a, alpha, sigma]
L(1) = Link([pi/2,0,0,0,1],"modified");
L(2) = Link([pi/2,0,0,pi/2,0],"modified");
L(3) = Link([pi/2,0,270,pi/2,0],"modified");
L(4) = Link([0,1025,128,pi/2,1],"modified");
L(5) = Link([0,0,0,0,0],"modified");

L(1).qlim=[1685, 1145+1685];
L(2).qlim=[-pi*7/24,pi*31/24];
L(3).qlim=[pi*4/9,pi*5/9];
L(4).qlim=[1025+1020, 1035+1025+1020];
L(5).qlim=[-pi,pi];

arm5 = SerialLink(L,'name','arm5');

teach(arm5); 
arm5.plot([1685 pi/2 pi/2 1025+1020 0])
hold on;

%% 工作空间
N=60000;    %随机次数

%关节角度限制
limitmax_1 = 1145+1685;
limitmin_1 = 1685;
limitmax_2 = 232.5;
limitmin_2 = -52.5;
limitmax_3 = 100.0;
limitmin_3 = 80.0;
limitmax_4 = 1035+1025+1020;
limitmin_4 = 1025+1020;
limitmax_5 = 180.0;
limitmin_5 = -180.0;

d1=(limitmin_1+(limitmax_1-limitmin_1)*rand(N,1)); %关节1限制
theta2=(limitmin_2+(limitmax_2-limitmin_2)*rand(N,1))*pi/180; %关节2限制
theta3=(limitmin_3+(limitmax_3-limitmin_3)*rand(N,1))*pi/180; %关节3限制
d4=(limitmin_4+(limitmax_4-limitmin_4)*rand(N,1)); %关节4限制
theta5=(limitmin_5+(limitmax_5-limitmin_5)*rand(N,1))*pi/180; %关节5限制

qq=[d1,theta2,theta3,d4,theta5];

Mricx=arm5.fkine(qq);
X=zeros(N,1);
Y=zeros(N,1);
Z=zeros(N,1);
for n=1:1:N
    X(n)=Mricx(n).t(1);
    Y(n)=Mricx(n).t(2);
    Z(n)=Mricx(n).t(3);
end
plot3(X,Y,Z,'b.','MarkerSize',0.5);%画出落点
hold on;
```