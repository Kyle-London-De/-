# CMake
CMake 默认会根据系统的环境变量 PATH 来查找可用的编译器
通常情况下，在 *Linux* 或 macOS 上，默认的 *C 编译器是 gcc*，*C++ 编译器是 g++*
在 *Windows* 上，默认的编译器可能是 *MSVC*（如果安装了 Visual Studio），或者 *MinGW* 的 gcc 和 g++（如果安装了 MinGW）
CMake 会优先选择系统中最常用的编译器。如果你安装了多个编译器（例如 GCC 和 Clang），CMake 会选择第一个在 PATH 中找到的编译器
## 安装
[CMake官网下载](https://cmake.org/download/)
下载cmake-xxx-windows-x86_64.msi，双击安装
Add CMake to system PATH for all users
## 使用
-S：指定源代码目录
-B：指定构建目录
-D：修改CMake变量，如-DCMAKE_BUILD_TYPE=Release，修改构建类型
# MinGW-w64
[MinGW官网](https://www.mingw-w64.org/)
MinGW(全称为，Minimalist GNU for Windows)，它实际上是将经典的开源 C语言编译器 GCC 移植到了 Windows 平台下，并且包含了 Win32API ，因此可以将源代码编译为可在 Windows 中运行的可执行程序。而且还可以使用一些 Windows 平台不具备的，但是Linux平台具备的开发工具和API函数。用一句话来概括就是：*MinGW 就是 GCC 的 Windows 版本* 。
MinGW-w64原本是MinGW项目的分支，后来成为独立发展得项目，由于仅有MinGW-w64被GCC官方所支持, 而MinGW早已停止更新, 因此推荐使用MinGW-w64。
MinGW-w64 与 MinGW 的区别在于 MinGW 只能编译生成32位可执行程序，而 MinGW-w64 则可以编译生成 64位 或 32位 可执行程序
## exe安装
[exe安装](https://sourceforge.net/projects/mingw-w64/)
可能会由于网络（需要VPN）或者win11系统等问题无法安装，跳转至压缩包安装
指定安装设置，有如下设置需要指定
Version：GCC 版本，若无特殊要求，选择最新版本即可
Architechture：架构，*64系统 位选择 x86_64，32 位系统选择 i686*
Threads：接口，Windows 选择 win32，Linux、Mac OS 等其他操作系统选择 posix
Exception：异常机制
- SJLJ：支持 32/64 位系统
- DRARF：仅支持 32 位系统，性能优于 SJLJ
- SEH：仅支持 64 位系统，性能优于 SJLJ
## 压缩包安装
[在sourceforge托管的源码](https://sourceforge.net/projects/mingw-w64/files/mingw-w64/mingw-w64-release/)（解压完并不是直接的bin、lib、include，也不知道怎么处理）
[在github托管的源码](https://github.com/niXman/mingw-builds-binaries/releases)（解压完是直接的bin、lib、include）
下载选择：*x86_64-14.2.0-release-win32-seh-ucrt-rt_v12-rev1.7z*
GitHub 上较新版的压缩包的命名又分为 *msvcrt 和 ucrt*，MSVCRT（Microsoft Visual C++ Runtime）和 UCRT（Universal C Runtime）是 Microsoft Windows 上的两种 C 运行时库，MSVCRT 在所有 Windows 版本上均可用，从 Windows 10 起，支持 UCRT
*若支持 UCRT 则建议选择 UCRT*
## 配置环境变量
配置环境变量->系统变量->Path
添加MinGW的bin路径
## 验证
win+R->cmd->gcc -v
