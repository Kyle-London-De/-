函数的作用：避免代码代码
分为声明和定义，声明写在头文件中，定义写在cpp中
# 函数
## 有输入输出
```c++
int Multiply(int a, int b)
{
	return a * b;
}
```
## 没有输入输出
```c++
void Multiply()
{
	std::cout << 5 * 8 << std::endl;
}
```
## 使用例子
```c++
#include <iostream>

int Multiply(int a, int b)
{
	return a * b;
}

int main()         // 主函数是特殊的，int main()默认return 0
{
	int result = Multiply(2, 4);
	std::cout << result << std::endl;
	std::cin.get();
}
```
*函数的使用存在内存的跳转和栈的使用，会拖慢程序运行的速度，函数使用应适当*