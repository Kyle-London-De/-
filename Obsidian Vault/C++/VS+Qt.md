## 描述：
当编译器从mingw（Qt Creator）换成MSVC（VS），重新编译的时候，出现中文乱码或者不识别的字符。
## 原因：
Qt Creator 保存文件使用的是UTF-8编码，MSVC虽然可以正常编译带BOM的UTF-8编码的源文件(所以这里将源文件改为带BOM的UTF-8,比如可以使用NodePad++设置UTF-8)，但是生成的可执行文件的编码是Windows本地字符集，比如GB2312。在可执行文件中，字符串是以GB2312编码的，当执行到这条语句时，对这个字符串是以UTF-8解码的，所以会出现乱码。
## 解决：
1. 在用到中文的地方使用：
```c++
QString str = QStringLiteral("中文内容");
```
2. 在每个用到中文的文件开头加上语句：
```c++
#if _MSC_VER >=1600
#pragma execution_character_set("utf-8")
#endif
```


