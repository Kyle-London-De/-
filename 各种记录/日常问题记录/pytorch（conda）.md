# xcb
## 问题描述
在conda环境中使用pytorch时 xcb 报错
```shell
qt.qpa.plugin: Could not load the Qt platform plugin "xcb" in "/home/yulong/anaconda3/envs/yolo11/lib/python3.10/site-packages/cv2/qt/plugins" even though it was found.
This application failed to start because no Qt platform plugin could be initialized. Reinstalling the application may fix this problem.

Available platform plugins are: xcb, linuxfb, minimal, offscreen, vnc, webgl.
```
## 原因
qt相关库libqxcb.so未添加到环境变量中
## 解决方法1
在自己.py文件前面加
```python
envpath = '/home/yulong/anaconda3/envs/yolo11/lib/python3.10/site-packages/cv2/qt/plugins/platforms'
os.environ['QT_QPA_PLATFORM_PLUGIN_PATH'] = envpath
```
## 解决方法2（不一定）
根本原因应该是conda虚拟环境的python与系统默认python冲突导致的
在vsode中手动切换python解释器可解决
在vscode中“ctrl+shift+p”，输入`python selet interpreter`，选择想要使用的python版本
## 解决方法3
在虚拟环境中卸载pyqt5
```shell
conda list
```
查看pyqt5后面的build，来选择conda或者pip
```shell
conda uninstall pyqt5
pip uninstall pyqt5
```
正常需要重新使用pip安装pyqt5和opencv-python，但不适用qt也可以不安装
# torch.\_C
## 问题描述
在conda环境中使用pytorch时报错
```shell
File "/home/yulong/anaconda3/envs/yolo11/lib/python3.10/site-packages/torch/__init__.py", line 367, in <module>
from torch._C import *  # noqa: F403
ImportError: /home/yulong/anaconda3/envs/yolo11/lib/python3.10/site-packages/torch/lib/libtorch_cpu.so: undefined symbol: iJIT_NotifyEvent
```
## 原因
mkl与当前pytorch版本（新）不兼容
## 解决方法
```shell
pip install mkl==2024.0.0
```