# 小萝卜的例子
希望具备自主运动能力，需要知道自身在什么地方（定位），需要知道周围长什么样子（建图），这是两个互相耦合的问题
# 传感器分类
安装于环境中的（缺点在于对环境有要求）
1. 二维码Marker
2. GPS
3. 导轨、磁条
携带于机器人本体上的（相对来说限制较小）
1. IMU
2. 激光
3. 相机
### 激光与相机
1. 激光更成熟
2. 相机更便宜轻便，信息更丰富，但有算力需求，且要求光线条件、无遮挡，有纹理
# 视觉SLAM
## 相机分类
1. 单目相机（Monocular）：没有深度，必须通过移动相机产生深度Moving View Stereo。相机运动起来时，看物体的相对运动就是进快远慢，可以借此推断距离
2. 双目相机（Stereo，立体）：通过视差计算深度。通过左右眼的差异，远处物体变化小，近处物体变化大，推算距离，计算量非常大
3. 深度相机（RGBD）：通过物理方法（ToF或者结构光）测量深度。主动测量，功耗大，深度值较准确，量程较小，易受干扰，室内有较好的表现
4. 其他：鱼眼 全景（Event Camera）
## 共同点
1. 利用图像和场景的几何关系，计算相机运动和场景结构Motion&Structure
2. 三维空间的运动和结构
3. 图像来自连续的视频
# 经典视觉SLAM框架
1. 传感器信息读取：利用传感器对外部信息的获取
2. 前端视觉里程计（Visual odometry）：通过逐步累加局部帧的运动状态，得到运动轨迹，特征点法vs直接法
3. 后端非线性优化（Optimization）：累加可能存在局部误差的放大，需要全局的非线性优化
4. 回环检测（Loop Closure Detection）：判断机器人是否到达先前到达过的位置。词袋模型
5. 建图（Mapping）：根据特征点地图，建立与任务要求对应的地图（度量地图vs拓扑地图，稀疏地图vs稠密地图）
# SLAM问题的数学表达
## 运动方程
 $$
x_k=f(x_{k-1},u_k,w_k)
$$
$x_k$是$k$时刻机器人的位置，需要把它们都看成随机变量，服从一定的概率分布
$u_k$是运动传感器的读数或者说输入（有些时候可能没有，比如纯视觉SLAM）
$w_k$是噪声
## 观测方程
$$
z_{k,j}=h(x_k,y_j,v_{k,j})
$$
$z_{k,j}$是机器人在$x_k$位置上观测路标点$y_j$的观测数据
$v_{k,j}$是噪声
由于在不同位置能够观察到的路标点是不定的，所以观测方程的数量也不定
# 实践部分
## g++
创建cpp文件
```Linux
cd slam14/learn-slam/ch2
touch main.cpp
gedit main.cpp
```
编写main.cpp
```c++
#include <iostream>

int main(int argc, char** argv)
{
	std::cout << "Hello SLAM!" << std::endl;
	return 0;
}
```
编译与运行
```Linux
g++ main.cpp -o HelloSLAM
./HelloSLAM
>Hello SLAM!
```
对于包含多个源文件的项目，使用g++命令是很不方便的，推荐使用CMake
## CMake
### 基本使用方法
创建CMakeLists.txt
```Linux
touch CMakeLists.txt
gedit CMakeLists.txt
```
CMakeLists.txt
```txt
// CMake版本信息
CMake_minimum_required(VERSION 3.0.2)

// 创建项目名称
project(helloSLAM)

// 添加可执行文件（可执行文件名 源文件）
add_executable(helloslam main.cpp)
```
创建CMake文件
```Linux
mkdir build //在build文件中编译，更加整洁方便
cd build
cmake ..
```
编译项目
在makefile目录下
```Linux
make
```
运行
```Linux
./helloSLAM
>Hello SLAM!
```
### 扩展调用
创建库libhello.cpp
```c++
#inculde <iostream>

void PrintHello()
{
	std::cout << "Hello Lib" << std::endl;
}
```
创建头文件libhello.h
```c++
#pragma once

void PrintHello();
```
创建主函数uselib.cpp
```c++
#include "libhello.h"

int main()
{
	PrintHello();
}
```
修改CMakeLists.txt
```txt
// 添加库
add_library(libhello libhello.cpp)
// 添加执行文件
add_executable(uselib uselib.cpp)
// 链接库
target_link_libraries(uselib libhello)
```
## IDE
```Linux
sudo apt-get install kdevelop
```
用法与VS类似