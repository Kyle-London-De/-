# 一般控制台程序封装
## 新建项目
1. vs2022，新建项目c++->windows->库，动态链接库（DLL）
2. 项目右键属性->配置属性->C/C++->预编译头，改为不使用预编译头
3. 清空头文件和源文件
## 编写动态链接库
xxdll_global.h
```c++
#ifndef __XXDLL_GLOBAL_H_              //防止重复编译
#define __XXDLL_GLOBAL_H_
// 或者
#pragma once                           //现在大多编译器普遍支持

#ifdef XXDLL_GLOBAL_EXPORTS            //判断导入还是导出函数
#define XXDLL_GLOBAL_API __declspec(dllexport)
#else
#define XXDLL_GLOBAL_API __declspec(dllimport)
#endif

#if __cplusplus                        //防止c++文件函数名混乱
extern "C" {
#endif

	XXDLL_GLOBAL_API void __stdcall Initxx();   //格式参考

#if __cplusplus
}
#endif

#endif
```
xxdll.cpp
```c++
#include "xxdll_global.h"
#include <iostream>

#define XXDLL_EXPORTS                  //库内cpp文件需要定义

void __stdcall InitXX()
{
	std::cout << "Hello" << std::endl;
}
```
选择对应环境平台编译（debug，x64）
在项目文件的debug文件中，找到xxdll.dll文件和xxdll.lib文件
## 调用动态链接库
1. 打开调用动态链接库的项目
2. 将xxdll.dll文件复制到可执行文件目录下（debug或者release）
3. 将xxdll.lib文件复制到源码文件目录下
4. 将xxdll.h添加到项目中
5. 修改需要使用动态链接库的文件，添加以下内容
```c++
#include "xxdll.h"
#pragma comment(lib,"xxdll.lib")
```
随后正常调用库中函数
# Qt（VS2022）项目封装
## 创建动态链接库dll
1. 创建动态链接库项目，项目模板Qt Class Library（Qt的动态链接库），解决方案处选择“添加到解决方案”，方便调试
2. 将Qt项目中的头文件、cpp文件、ui文件（qrc文件如果使用自定义图标也需要）复制到动态链接库项目中，并添加，ui文件添加到资源文件中，并配置好相应环境（如在项目属性的Qt Project Settings中，配置Qt版本和模块等）
3. 修改主要头文件，在该头文件中include项目自动生成的xxx_global.h文件，并修改类定义
xxx_global.h
```c++
#pragma once

#include <QtCore/qglobal.h>

#ifndef BUILD_STATIC
# if defined(XXX_LIB)
#  define XXX_EXPORT Q_DECL_EXPORT
# else
#  define XXX_EXPORT Q_DECL_IMPORT
# endif
#else
# define XXX_EXPORT
#endif
```
修改类定义
```c++
class QtReadFBGData : public QDialog
{

}
// 修改为
class XXX_EXPORT QtReadFBGData : public QDialog
{

}
```
4. 生成
## 调用测试
1. 创建调用测试项目Qt Widgets Application（也可以创建一般控制台程序，但需要额外添加Qt的库），解决方案处选择“添加到解决方案”，方便调试
2. 将动态链接库项目中的头文件和Debug/uic中的ui_xxx.h头文件复制到测试项目中（可以统一存放在项目下的include文件中方便管理，这样就需要在项目设置->C/C++->常规->附加包含目录./include），将生成的dll文件和lib文件复制到测试项目中（lib文件可以存放在lib文件中，同理，需要在项目属性->链接器->常规->附加库目录./lib），在链接器->输入中添加xxx.lib，配置相应环境同上
3. 修改main.cpp
```c++
#include "testdemo.h"
#include <QtWidgets/QApplication>
#include "xxx.h"                       //主要的头文件
int main(int argc, char* argv[])
{
	QApplication a(argc, argv);
	xxx w;                             //定义的类
	w.show();
	return a.exec();
}
```
4. 生成