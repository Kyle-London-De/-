# 介绍
MATLAB Coder可从 MATLAB 代码生成适用于各种硬件平台的 C 和 C++ 代码。它支持大多数 MATLAB 语言和广泛的工具箱。您可以将生成的代码作为源代码、静态库或动态库集成到您的工程中。生成的代码是可读且可移植的。您可以将它与现有 C 和 C++ 代码及库的关键部分结合使用。您还可以将生成的代码打包为 MEX 函数，用于在MATLAB环境中进行验证或加速。
# 配置C/C++编译器
## 方法一
下载Visual Studio，下载完成后基本无需配置
## 方法二
下载C语言编译器MinGW-w64
## 验证
MATLAB命令行输入
```matlab
mex -setup %默认C
mex -setup c++
```
若报错未检测到支持的编译器，需要手动输入MinGW的位置
```matlab
setenv('MW_MINGW64_LOC','D:\mingw64')
```
# 代码准备
简单例子：
mul.m文件。function函数内即为需要移植C/C++的内容，需要将其封装为单/多输入、单/多输出的一个函数。
```matlab
function c = mul(a,b)
	c = a * b;
```
main.m文件。一个简单的例子，调用function函数以验证。
```matlab
x=2;
y=3;
mul(x,y)
```
# MATLAB ->C(C++)
## MATLAB Coder
菜单栏APP中下拉找到MATLAB Coder或命令行输入
```matlab
coder
```
共包含六个步骤，分别为：
1. *Select* (Select Source File)
2. *Review* (Review Code Generation)
3. *Define* (Define Input Types)
4. *Check* (Check for Tun-Time Issues)
5. *Generate* (Generate Code)
6. *Finish* (Finish Workflow)
### *Select* (Select Source File)
选择需要转换成C/C++的文件名，如*mul.m*
在*Entry-Point Functions*下方输入*mul*，即Funciton的函数名
点击右下角*Next*进入下一步
### *Review* (Review Code Generation)
上一步点击*Next*后自动隐藏该步骤
### *Define* (Select Source File)
为每个入口点函数定义输入类型，有主函数直接选择主函数*main.m*即可，MATLAB会自动推导出该函数的输入类型
在方框中输入*main*，点击*Autodefine Input Types*
在下方仍可调整输入类型，如批量处理的函数，可改为n行或n列
点击右下角*Next*进入下一步
### *Check* (Check For Run-Time Issues)
检查运行问题，点击右侧*Check for Issues*，MATLAB会自动检查是否存在故障
若不存在，则*Generation trial code，Building MEX，Running test file with MEX*上方均显示√
点击右下角*Next*进入下一步
### *Generate* (Generate Code)
选择*C/C++*，其他选项一般默认，点击*Generate*生成代码
点击右下角*Next*进入下一步
### *Finish* (Finish Workflow)
在完成界面可查看代码生成位置，在*C Code*处为C/C++代码保存位置，一般为MATLAB当前运行文件夹下*codegen/lib/mul*
## Visual Studio
### 新建项目
新建一个空项目，将*codegen/lib/mul*下*mul.cpp,mul.h,rtwtypes.h*文件复制到项目目录下，将MATLAB安装目录*extern/include*下*tmwtypes.h*文件复制到项目目录下
在头文件中添加所有.h文件，源文件中添加所有.cpp文件
### 编写main.cpp
在资源文件中新建*main.cpp*文件，编写一个如main.m文件的调用例子，如：
```c++
#include <stdio.h>
#include <iostream>
#include "mul.h"
int main()
{
	int x = 2, y = 3, z;
	z = mul(x, y);
	printf("z=%d\n", z);
	std::cin.get();
}
```
### 运行
点击运行，控制台输出“*z=6*”
移植成功