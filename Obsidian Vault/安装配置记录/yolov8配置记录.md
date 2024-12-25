python=3.10
cuda=11.3
cudnn=8.9.7.29
# 1.安装pycharm和anaconda
略
# 2.配置anaconda
安装后添加环境变量
```
D:\anaconda3
D:\anaconda3\Library\bin
D:\anaconda3\Scripts
```
启动 anaconda prompt 终端
创建 yolov8 环境
```
conda create -n yolov8 python==3.10
```
启动 yolov8 环境
```
conda activate yolov8
```
为 conda 更换国内镜像源
```
conda config --add channels https://mirrors.tuna.tsinghua.edu.cn/anaconda/pkgs/free/
conda config --add channels https://mirrors.tuna.tsinghua.edu.cn/anaconda/pkgs/main/
conda config --add channels https://mirrors.tuna.tsinghua.edu.cn/anaconda/cloud/pytorch/
conda config --set show_channel_urls yes
conda config --add channels https://mirrors.ustc.edu.cn/anaconda/pkgs/main/
conda config --add channels https://mirrors.ustc.edu.cn/anaconda/pkgs/free/
conda config --add channels https://mirrors.ustc.edu.cn/anaconda/cloud/conda-forge/
conda config --add channels https://mirrors.ustc.edu.cn/anaconda/cloud/msys2/
conda config --add channels https://mirrors.ustc.edu.cn/anaconda/cloud/bioconda/
conda config --add channels https://mirrors.ustc.edu.cn/anaconda/cloud/menpo/
conda config --set show_channel_urls yes
```
# 3.安装pytorch
在[pytorch官网](https://pytorch.org/get-started/previous-versions/)找到匹配的下载代码（提前确定 python 和 cuda 的版本，cudatoolkit 版本要比 cuda 版本低，最好pytorch>=1.12.0，1.11有bug）
```
conda install pytorch==1.12.0 torchvision==0.13.0 torchaudio==0.12.0 cudatoolkit=11.3
```
# 4.安装cuda和cudnn
通过 nvidia control panel->帮助->系统信息->组件，查看 cuda 版本
通过[cuda下载](https://developer.nvidia.com/cuda-toolkit-archive)找到对应版本后，选择 windows->x86_64->10->exe(local)
如果有旧版本应用需要卸载（nvidia cuda 开头，版本号结尾）
安装时选择自定义安装，安装所有插件
确认系统变量 CUDA_PATH、CUDA_PATH_V11_3、NVCUDASAMPLES_ROOT、NVCUDASAMPLES11_0_ROOT
命令行中查看 cuda 版本
```
nvcc -V
```
通过[cudnn官网](https://developer.nvidia.com/rdp/cudnn-archive)下载对应版本的 cudnn
将文件中 bin/lib/include 文件夹复制到 cuda 文件夹下
# 5.制作数据集
下载 labelimg
```
pip install labelimg -i https://pypi.tuna.tsinghua.edu.cn/simple
```
创建 labelimg 环境
```
conda create -n labelimg python=3.9.16
```
激活 labelimg 环境
```
conda activate labelimg
```
打开 labelimg
```
labelimg
```
open dir 为原始图片位置，change save dir 为标签位置，标记时切换到 yolo 模式，标记完成后点击保存
测试可使用coco128进行训练[coco数据集下载](https://ultralytics.com/assets/coco128.zip)
# 6.配置pycharm
下载yolov8源码[github官网](https://github.com/ultralytics/ultralytics)
下载预训练权重[yolo官网](https://docs.ultralytics.com/tasks/detect/)，yolov8n.pt
在 pycharm 中新建项目，编译器选择 conda 中新建的 yolov8 编译器anaconda3/envs/yolov8/python.exe，载入该源码，此时右下角编译器显示为 yolov8 环境
将下载的 yolov8n.pt 复制到根目录下
修改终端，file->settings->tools->terminal->shell path 修改为 anaconda prompt *快捷方式*的目标内 cmd.exe 及之后的内容（也可不修改，原终端为原始 cmd.exe）
在修改后的终端中激活 yolov8 环境
```
conda activate yolov8
```
安装相关库
```
pip install -r requirsments.txt
pip install ultralytics
pip install yolo
```
测试环境
```
yolo predict model=yolov8n.pt source='ultralytics/assets/bus.jpg'
```
预测结果在 runs->predict 中
可能存在问题及解决方法（目前仅在python=3.10出现）
[ImportError: cannot import name ‘Callable‘ from ‘collections‘](https://blog.csdn.net/weixin_45662399/article/details/134501374?spm=1001.2014.3001.5502)\
[Error: No such command ‘predict‘](https://blog.csdn.net/weixin_46030156/article/details/132962611?ops_request_misc=%257B%2522request%255Fid%2522%253A%2522170044606216800192299672%2522%252C%2522scm%2522%253A%252220140713.130102334..%2522%257D&request_id=170044606216800192299672&biz_id=0&utm_medium=distribute.pc_search_result.none-task-blog-2~all~sobaiduend~default-1-132962611-null-null.142%5Ev96%5Epc_search_result_base2&utm_term=No%20such%20command%20predict&spm=1018.2226.3001.4187)
# 7.配置系统环境
训练使用 GPU 可能存在内存不足的情况，可以适当提高内存分配
搜索高级系统设置，高级->性能->设置->高级->更改，可使用系统管理的大小或自定义大小（4000-6000）（如果发现 windows 外观舒适度降低适当调整视觉效果，或勾选调整为最佳外观）
# 8.训练数据集
将 ultralytics/cfg/datasets 中的 coco128.yaml复制到自制数据集目录下（与 images 同级）修改名称和path、train、val、test、calsses 等内容（path 最好使用绝对路径，train 和 val 为相对于 path 的相对路径）
若自制数据集，文件结构仿照 coco128
在终端输入训练代码
```
 yolo detect train data=datasets/coco128/coco128.yaml model=yolov8n.yaml pretrained=ultralytics/yolov8n.pt epochs=100 batch=4 lr0=0.01 resume=True
```
训练结果在 runs->detect 中