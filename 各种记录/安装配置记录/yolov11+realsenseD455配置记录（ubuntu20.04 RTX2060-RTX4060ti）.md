*提前根据 pytorch 版本确定 python 和 cudatoolkit 的版本，cudatoolkit 版本不能比 cuda 版本高，pytorch 版本根据显卡的 cuda 决定，越高越好，最好pytorch>=1.12.0，1.11有bug*
参考：rtx2060，cudatoolkit=11.3（rtx4060ti，cuda=12.2，cudatoolkit=12.1）
# 1 安装anaconda
[官网下载](https://www.anaconda.com/download/success)
## 1.1 windows系统安装
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
## 1.2 Linux系统安装
[ubuntu安装详见](https://blog.csdn.net/weixin_44955407/article/details/139725961)
# 2 安装pytorch
在[pytorch官网](https://pytorch.org/get-started/previous-versions/)找到匹配的下载代码，建议使用conda install（仅安装在conda的虚拟环境中），使用pip install可能无法正确安装 
```
conda install pytorch==1.12.0 torchvision==0.13.0 torchaudio==0.12.0 cudatoolkit=11.3
```
# 3 安装cuda、cudnn
[安装参考](https://blog.csdn.net/KRISNAT/article/details/134870009?ops_request_misc=%257B%2522request%255Fid%2522%253A%2522D5D0EED6-3F1F-4EA7-9928-C33F344E9A0F%2522%252C%2522scm%2522%253A%252220140713.130102334..%2522%257D&request_id=D5D0EED6-3F1F-4EA7-9928-C33F344E9A0F&biz_id=0&utm_medium=distribute.pc_search_result.none-task-blog-2~all~top_positive~default-4-134870009-null-null.142^v100^pc_search_result_base7&utm_term=ubuntu%E5%AE%89%E8%A3%85cuda&spm=1018.2226.3001.4187)
## 3.1 cuda安装
通过[cuda下载](https://developer.nvidia.com/cuda-toolkit-archive)找到对应版本，使用runfile安装，且安装时不勾选driver和Kernel Objects（nvidia-fs）[安装注意事项详见](https://blog.csdn.net/level_code/article/details/126398286)（.deb安装会因nvidia driver 465无法安装报错）
检查
```shell
nvcc -V
```

## 3.2 cudnn安装 
通过[cudnn下载](https://developer.nvidia.com/rdp/cudnn-archive)下载对应版本的Local Installer for Linux x86_64(Tar)（cudnn用于深度学习，无需求不用下）
```shell
tar -xvf cudnn-linux-x86_64-8.9.7.29_cuda11-archive.tar.xz
cd cudnn-linux-x86_64-8.9.7.29_cuda11-archive/
sudo cp include/cudnn*.h /usr/local/cuda-11.3/include
sudo cp lib/libcudnn* /usr/local/cuda-11.3/lib64
sudo chmod a+r /usr/local/cuda-11.3/include/cudnn*.h
sudo chmod a+r /usr/local/cuda-11.3/lib64/libcudnn*
```
检查
```shell
cat /usr/local/cuda-11.3/include/cudnn_version.h | grep CUDNN_MAJOR -A 2
```
# 4 配置yolo
以下方式最好二选一，否则可能重复安装不同版本的包，产生问题
## 4.1 直接安装ultralytics
*适用于terminal直接运行.py时使用*
在conda环境中使用pip安装
```shell
pip install ultralytics
pip install yolo
```
检查
```shell
yolo predict model=yolov8n.pt source='https://ultralytics.com/images/bus.jpg
```
## 4.2 下载源码ultralytics
*适用于IDE编译*
源码[github官网](https://github.com/ultralytics/ultralytics)
# 5 配置realsense环境
[参考文档](https://www.mahaofei.com/post/cdee659e.html)
*两种安装方式相互独立*
## 5.1 安装Realsense SDK
*适用于使用realsense-viewer*
注册服务器的公钥
```shell
sudo apt-key adv --keyserver keyserver.ubuntu.com --recv-key F6E65AC044F831AC80A06380C8B3A55A6F3EFCDE || sudo apt-key adv --keyserver hkp://keyserver.ubuntu.com:80 --recv-key F6E65AC044F831AC80A06380C8B3A55A6F3EFCDE
```
如果仍然无法检索公钥，请检查并指定代理设置
```shell
export http_proxy="http://<proxy>:<port>"
```
将服务器添加到存储库列表中
```shell
sudo add-apt-repository "deb https://librealsense.intel.com/Debian/apt-repo $(lsb_release -cs) main" -u
```
安装库
```shell
sudo apt-get install librealsense2-dkms
sudo apt-get install librealsense2-utils
sudo apt-get install librealsense2-dev  
sudo apt-get install librealsense2-dbg
```
验证安装
```shell
realsense-viewer
```
## 5.2 安装realsense-ROS包
*适用于使用ros*
```shell
mkdir -p ~/catkin_ws/src
cd ~/catkin_ws/src
git clone -b ros1-legacy https://github.com/IntelRealSense/realsense-ros.git
cd ~/catkin_ws && catkin_make
```
一定要注意clone的分支，默认是ros2，如果是ros1环境需要指定分支
验证安装
```shell
source ~/catkin_ws/devel/setup.bash
roslaunch realsense2_camera rs_camera.launch
```
# 6 realsense相机调用
## 6.1 使用roslaunch调用
根据realsense相机型号及需求修改rs_camera.launch文件（启动红外相机）
```launch
  <arg name="enable_infra"        default="true"/>
  <arg name="enable_infra1"       default="true"/>
  <arg name="enable_infra2"       default="true"/>
```
启动相机
```shell
roslaunch realsense2_camera rs_camera.launch
```
查看图像
```shell
rviz
```
add->by topic
或
```shell
rqt_image_view
```
## 6.2 使用python代码调用
需要下载pyrealsense2
```shell
conda install pyrealsense2
```
import pyrealsense2时，由于版本问题，可能报错
ImportError: /lib/x86_64-linux-gnu/libstdc++.so.6: version ‘GLIBCXX_3.4.29' not found (required by /home/yulong/anaconda3/envs/yolo11/lib/python3.10/site-packages/pyrealsense2.cpython-310-x86_64-linux-gnu.so)
[解决方法](https://blog.csdn.net/BetrayFree/article/details/137690412)
代码示例
```python
import pyrealsense2 as rs
import socket
import cv2
import numpy as np
import struct
 
# 配置 RealSense 相机
pipeline = rs.pipeline()
config = rs.config()
config.enable_stream(rs.stream.color, 640, 480, rs.format.bgr8, 30)
 
# 开始捕获视频
pipeline.start(config)
 
# 配置 UDP 服务器
UDP_IP = "127.0.0.1"  # 目标计算机的 IP 地址
UDP_PORT = 5005
sock = socket.socket(socket.AF_INET, socket.SOCK_DGRAM)
 
while True:
    # 获取帧
    frames = pipeline.wait_for_frames()
    color_frame = frames.get_color_frame()
 
    # 将帧转换为 OpenCV 图像
    color_image = np.asanyarray(color_frame.get_data())
 
    # 显示图像
    cv2.imshow('RealSense server', color_image)
 
    # 将图像转换为字符串并发送到目标计算机
    # data = color_image.tostring()
 
    # 将图像转换为JPEG格式
    _, jpeg_image = cv2.imencode('.jpg', color_image)
    data = jpeg_image.tobytes()
    # 【定义文件头、数据】
    fread = struct.pack('i', len(data))
    # 【发送文件头、数据】
    sock.sendto(fread, (UDP_IP, UDP_PORT))
    # 每次发送x字节，计算所需发送次数
    pack_size = 65507
    send_times = len(data) // pack_size + 1
    for count in range(send_times):
        if count < send_times - 1:
            sock.sendto(data[pack_size * count:pack_size * (count + 1)], (UDP_IP, UDP_PORT))
        else:
            sock.sendto(data[pack_size * count:],(UDP_IP, UDP_PORT))
    # sock.sendto(data, (UDP_IP, UDP_PORT))
 
    # 按下 ESC 键退出循环
    if cv2.waitKey(1) == 27:
        break
 
# 停止捕获视频并关闭窗口
pipeline.stop()
cv2.destroyAllWindows()
```