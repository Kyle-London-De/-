ubuntu20.04+python3.8.10
官方要求c++11，一般都满足，可以再执行一遍
```shell
sudo apt-get install git gcc g++ vim make cmake
```
# 前置准备
## python版本（python2>=2.7，或较新的python3，参考3.8）
查看并切换python版本，参考[python版本切换](https://www.jb51.net/article/281665.htm)
```shell
# 根据具体版本跟准确
python --version
python -V
python2 --version
python3 --version

# 或者直接查看文件夹
ls /usr/bin/python*

# 将python3设置最高优先级（数字更大）
sudo update-alternatives --install /usr/bin/python python /usr/bin/python2 1
sudo update-alternatives --install /usr/bin/python python /usr/bin/python3 2
```
验证
```shell
python --version
```
另需要numpy
```shell
pip install numpy
```
## Pangolin=0.6
[Pangolin=0.6源码下载](https://github.com/stevenlovegrove/Pangolin/archive/refs/tags/v0.6.zip)
其他版本可能存在未知错误
```shell
unzip Pangolin-0.6
cd Pangolin-0.6
mkdir build
cd build
cmake -DCPP11_NO_BOOST=1 ..
make
sudo make install
```
## Eigen>=3.0
纯头文件库，建议源码安装
```shell
git clone https://github.com/eigenteam/eigen-git-mirror
cd eigen-git-mirror
mkdir build
cd build
cmake ..
make
sudo make install
```
 头文件安装在/usr/local/include/eigen3/中，需要复制一下
```shell
sudo cp -r /usr/local/include/eigen3/Eigen /usr/local/include
```
## Opencv>=3.2
ubuntu20.04自带4.2.0，可用
另[Opencv3.4.3安装参考](https://blog.csdn.net/m0_49066914/article/details/131785030)
## 其他第三方库
在ORB_SLAM3/Thirdparty中
分别安装三个包（DBoW2和g2o源码有，其实不用管，Sophus需要安装）
这里直接安装（sudo make install）可以避免编译过程中的常见找不到Sophus库的情况
```shell
mkdir build
cd build
cmake .. -DCMAKE_BUILD_TYPE=Release
make -j
sudo make install
```
# ORB-SLAM3（非ROS）
[ORB-SLAM3官网](https://github.com/UZ-SLAMLab/ORB_SLAM3?tab=readme-ov-file#readme)即源码下载地址
src中包含核心算法
evaluate中包含用于评估ORB-SLAM3系统的工具和脚本
Thirdparty中包含第三方库文件
Vocabulary中包含ORB-SLAM3使用的ORB特征词典，用于词袋模型的回环检测和场景识别
Examples和Example_old中存放着各种例子，Monocular单目，Stereo双目，Inertial Imu等，这些例子直接运行，与ROS无关
编译参考官方给的build.sh和build_ros.sh脚本，但不推荐直接运行
## 修改CMakeLists.txt
建议修改所有能找到的CMakeLists.txt
所有Opencv和Eigen修改为安装的现有版本
```txt
find_package(OpenCV 4.2.0)
find_package(Eigen3 REQUIRED)
```
## 解压ORBvoc.txt
```shell
cd Vocabulary
tar -xf ORBvoc.txt.tar.gz
```
## 编译
在ORB_SLAM3主目录下
```shell
mkdir build
cd build
cmake .. -DCMAKE_BUILD_TYPE=Release
make -j4
```
## 运行数据集（EuRoC）
参考[官方脚本（最新版中没有这两个文件）](https://gitcode.com/gh_mirrors/or/ORB_SLAM3_detailed_comments/tree/master)euroc_examples.sh、euroc_eval_examples.sh
以ORB-SLAM3官方推荐的[EuRoC数据集（ASL格式）](https://projects.asl.ethz.ch/datasets/doku.php?id=kmavvisualinertialdatasets#downloads)为例，[备用下载链接](http://robotics.ethz.ch/~asl-datasets/ijrr_euroc_mav_dataset/)
### 运行数据集
运行代码参考stereo_euroc.cc中主函数里的提示
```c++
"Usage: ./stereo_euroc path_to_vocabulary path_to_settings path_to_sequence_folder_1 path_to_times_file_1 (path_to_image_folder_2 path_to_times_file_2 ... path_to_image_folder_N path_to_times_file_N) (trajectory_file_name)"
```
运行（V1_02_m文件夹下是mav0）
```shell
./Examples/Stereo/stereo_euroc ./Vocabulary/ORBvoc.txt ./Examples/Stereo/EuRoC.yaml ./dataset_euroc/V1_02_m ./Examples/Stereo/EuRoC_TimeStamps/V102.txt dataset-V102_stereo
```
生成以下三个文件
f_dataset-V102_stereo.txt、KeyFrameTrajectory.txt、kf_dataset-V102_stereo.txt
### 评估ORB-SLAM3算法
#### 修改evaluate_ate_scale.py和associate.py
由于使用python3（3.8.10），而源码使用python2.7，所以需要修改以下代码
修改associate.py（可以保留源码，另存为associate_py3.py）
```python
#!/usr/bin/python3
# Software License Agreement (BSD License)
#
# License information omitted for brevity.

import argparse
import sys
import os
import numpy


def read_file_list(filename, remove_bounds=False):
    """
    Reads a trajectory from a text file. 
    
    File format:
    The file format is "stamp d1 d2 d3 ...", where stamp denotes the time stamp (to be matched)
    and "d1 d2 d3.." is arbitrary data (e.g., a 3D position and 3D orientation) associated to this timestamp. 
    
    Input:
    filename -- File name
    
    Output:
    dict -- dictionary of (stamp, data) tuples
    """
    with open(filename) as file:
        data = file.read()
    lines = data.replace(",", " ").replace("\t", " ").split("\n")
    if remove_bounds:
        lines = lines[100:-100]
    list_data = [[v.strip() for v in line.split(" ") if v.strip() != ""] for line in lines if len(line) > 0 and line[0] != "#"]
    list_data = [(float(l[0]), l[1:]) for l in list_data if len(l) > 1]
    return dict(list_data)


def associate(first_list, second_list, offset, max_difference):
    """
    Associate two dictionaries of (stamp, data). As the time stamps never match exactly, we aim 
    to find the closest match for every input tuple.
    
    Input:
    first_list -- first dictionary of (stamp, data) tuples
    second_list -- second dictionary of (stamp, data) tuples
    offset -- time offset between both dictionaries (e.g., to model the delay between the sensors)
    max_difference -- search radius for candidate generation

    Output:
    matches -- list of matched tuples ((stamp1, data1), (stamp2, data2))
    """
    first_keys = list(first_list.keys())  # 修改：将dict_keys转换为列表以允许remove操作
    second_keys = list(second_list.keys())  # 同上

    potential_matches = [(abs(a - (b + offset)), a, b) 
                         for a in first_keys 
                         for b in second_keys 
                         if abs(a - (b + offset)) < max_difference]
    potential_matches.sort()
    matches = []
    for diff, a, b in potential_matches:
        if a in first_keys and b in second_keys:
            first_keys.remove(a)
            second_keys.remove(b)
            matches.append((a, b))
    
    matches.sort()
    return matches

if __name__ == '__main__':
    # parse command line
    parser = argparse.ArgumentParser(description='''This script takes two data files with timestamps and associates them.''')
    parser.add_argument('first_file', help='first text file (format: timestamp data)')
    parser.add_argument('second_file', help='second text file (format: timestamp data)')
    parser.add_argument('--first_only', help='only output associated lines from first file', action='store_true')
    parser.add_argument('--offset', help='time offset added to the timestamps of the second file (default: 0.0)', default=0.0)
    parser.add_argument('--max_difference', help='maximally allowed time difference for matching entries (default: 0.02)', default=0.02)
    args = parser.parse_args()

    first_list = read_file_list(args.first_file)
    second_list = read_file_list(args.second_file)

    matches = associate(first_list, second_list, float(args.offset), float(args.max_difference))

    if args.first_only:
        for a, b in matches:
            print(f"{a} {' '.join(first_list[a])}")
    else:
        for a, b in matches:
            print(f"{a} {' '.join(first_list[a])} {b - float(args.offset)} {' '.join(second_list[b])}")
```
修改evaluate_ate_scale.py（如果保留源码另存为evaluate_ate_scale_py3.py，需要同时修改源码中associate为associate_py3）
```python
# Modified by Raul Mur-Artal
# Automatically compute the optimal scale factor for monocular VO/SLAM.

# Software License Agreement (BSD License)
# License and copyright information omitted for brevity.

import sys
import numpy
import argparse
import associate

def align(model, data):
    """Align two trajectories using the method of Horn (closed-form).
    
    Input:
    model -- first trajectory (3xn)
    data -- second trajectory (3xn)
    
    Output:
    rot -- rotation matrix (3x3)
    trans -- translation vector (3x1)
    trans_error -- translational error per point (1xn)
    """

    numpy.set_printoptions(precision=3, suppress=True)
    model_zerocentered = model - model.mean(1)
    data_zerocentered = data - data.mean(1)

    W = numpy.zeros((3, 3))
    for column in range(model.shape[1]):
        W += numpy.outer(model_zerocentered[:, column], data_zerocentered[:, column])
    U, d, Vh = numpy.linalg.svd(W.transpose())  # 修改：linalg.linalg.svd()改为linalg.svd()
    S = numpy.identity(3)  # 修改：numpy.matrix()改为直接numpy.identity()
    if numpy.linalg.det(U) * numpy.linalg.det(Vh) < 0:
        S[2, 2] = -1
    rot = U @ S @ Vh  # 修改：矩阵乘法使用@代替 *

    rotmodel = rot @ model_zerocentered
    dots = 0.0
    norms = 0.0

    for column in range(data_zerocentered.shape[1]):
        dots += numpy.dot(data_zerocentered[:, column].transpose(), rotmodel[:, column])
        normi = numpy.linalg.norm(model_zerocentered[:, column])
        norms += normi * normi

    s = float(dots / norms)
    transGT = data.mean(1) - s * rot @ model.mean(1)
    trans = data.mean(1) - rot @ model.mean(1)

    model_alignedGT = s * rot @ model + transGT
    model_aligned = rot @ model + trans

    alignment_errorGT = model_alignedGT - data
    alignment_error = model_aligned - data

    trans_errorGT = numpy.sqrt(numpy.sum(numpy.multiply(alignment_errorGT, alignment_errorGT), 0)).A[0]
    trans_error = numpy.sqrt(numpy.sum(numpy.multiply(alignment_error, alignment_error), 0)).A[0]
        
    return rot, transGT, trans_errorGT, trans, trans_error, s

def plot_traj(ax, stamps, traj, style, color, label):
    """
    Plot a trajectory using matplotlib. 
    
    Input:
    ax -- the plot
    stamps -- time stamps (1xn)
    traj -- trajectory (3xn)
    style -- line style
    color -- line color
    label -- plot legend
    """
    stamps.sort()
    interval = numpy.median([s - t for s, t in zip(stamps[1:], stamps[:-1])])
    x = []
    y = []
    last = stamps[0]
    for i in range(len(stamps)):
        if stamps[i] - last < 2 * interval:
            x.append(traj[i][0])
            y.append(traj[i][1])
        elif len(x) > 0:
            ax.plot(x, y, style, color=color, label=label)
            label = ""
            x = []
            y = []
        last = stamps[i]
    if len(x) > 0:
        ax.plot(x, y, style, color=color, label=label)

if __name__ == "__main__":
    # parse command line
    parser = argparse.ArgumentParser(description='''This script computes the absolute trajectory error from the ground truth trajectory and the estimated trajectory.''')
    parser.add_argument('first_file', help='ground truth trajectory (format: timestamp tx ty tz qx qy qz qw)')
    parser.add_argument('second_file', help='estimated trajectory (format: timestamp tx ty tz qx qy qz qw)')
    parser.add_argument('--offset', help='time offset added to the timestamps of the second file (default: 0.0)', default=0.0)
    parser.add_argument('--scale', help='scaling factor for the second trajectory (default: 1.0)', default=1.0)
    parser.add_argument('--max_difference', help='maximally allowed time difference for matching entries (default: 10000000 ns)', default=20000000)
    parser.add_argument('--save', help='save aligned second trajectory to disk (format: stamp2 x2 y2 z2)')
    parser.add_argument('--save_associations', help='save associated first and aligned second trajectory to disk (format: stamp1 x1 y1 z1 stamp2 x2 y2 z2)')
    parser.add_argument('--plot', help='plot the first and the aligned second trajectory to an image (format: png)')
    parser.add_argument('--verbose', help='print all evaluation data (otherwise, only the RMSE absolute translational error in meters after alignment will be printed)', action='store_true')
    parser.add_argument('--verbose2', help='print scale error and RMSE absolute translational error in meters after alignment with and without scale correction', action='store_true')
    args = parser.parse_args()

    first_list = associate.read_file_list(args.first_file, False)
    second_list = associate.read_file_list(args.second_file, False)

    matches = associate.associate(first_list, second_list, float(args.offset), float(args.max_difference))
    if len(matches) < 2:
        sys.exit("Couldn't find matching timestamp pairs between ground truth and estimated trajectory! Did you choose the correct sequence?")
    first_xyz = numpy.matrix([[float(value) for value in first_list[a][0:3]] for a, b in matches]).transpose()
    second_xyz = numpy.matrix([[float(value) * float(args.scale) for value in second_list[b][0:3]] for a, b in matches]).transpose()

    dictionary_items = second_list.items()
    sorted_second_list = sorted(dictionary_items)

    second_xyz_full = numpy.matrix([[float(value) * float(args.scale) for value in sorted_second_list[i][1][0:3]] for i in range(len(sorted_second_list))]).transpose()
    rot, transGT, trans_errorGT, trans, trans_error, scale = align(second_xyz, first_xyz)
    
    second_xyz_aligned = scale * rot @ second_xyz + trans
    second_xyz_notscaled = rot @ second_xyz + trans
    second_xyz_notscaled_full = rot @ second_xyz_full + trans
    first_stamps = list(first_list.keys())
    first_stamps.sort()
    first_xyz_full = numpy.matrix([[float(value) for value in first_list[b][0:3]] for b in first_stamps]).transpose()
    
    second_stamps = list(second_list.keys())
    second_stamps.sort()
    second_xyz_full = numpy.matrix([[float(value) * float(args.scale) for value in second_list[b][0:3]] for b in second_stamps]).transpose()
    second_xyz_full_aligned = scale * rot @ second_xyz_full + trans
    
    if args.verbose:
        print(f"compared_pose_pairs {len(trans_error)} pairs")
        print(f"absolute_translational_error.rmse {numpy.sqrt(numpy.dot(trans_error, trans_error) / len(trans_error))} m")
        print(f"absolute_translational_error.mean {numpy.mean(trans_error)} m")
        print(f"absolute_translational_error.median {numpy.median(trans_error)} m")
        print(f"absolute_translational_error.std {numpy.std(trans_error)} m")
        print(f"absolute_translational_error.min {numpy.min(trans_error)} m")
        print(f"absolute_translational_error.max {numpy.max(trans_error)} m")
        print(f"max idx: {numpy.argmax(trans_error)}")
    else:
        print(f"{numpy.sqrt(numpy.dot(trans_error, trans_error) / len(trans_error))},{scale},{numpy.sqrt(numpy.dot(trans_errorGT, trans_errorGT) / len(trans_errorGT))}")
    
    if args.verbose2:
        print(f"compared_pose_pairs {len(trans_error)} pairs")
        print(f"absolute_translational_error.rmse {numpy.sqrt(numpy.dot(trans_error, trans_error) / len(trans_error))} m")
        print(f"absolute_translational_errorGT.rmse {numpy.sqrt(numpy.dot(trans_errorGT, trans_errorGT) / len(trans_errorGT))} m")

    if args.save_associations:
        with open(args.save_associations, "w") as file:
            file.write("\n".join([f"{a} {x1} {y1} {z1} {b} {x2} {y2} {z2}" for (a, b), (x1, y1, z1), (x2, y2, z2) in zip(matches, first_xyz.transpose().A, second_xyz_aligned.transpose().A)]))
        
    if args.save:
        with open(args.save, "w") as file:
            file.write("\n".join([f"{stamp} " + " ".join([f"{d}" for d in line]) for stamp, line in zip(second_stamps, second_xyz_notscaled_full.transpose().A)]))

    if args.plot:
        import matplotlib
        matplotlib.use('Agg')
        import matplotlib.pyplot as plt
        fig = plt.figure()
        ax = fig.add_subplot(111)
        plot_traj(ax, first_stamps, first_xyz_full.transpose().A, '-', "black", "ground truth")
        plot_traj(ax, second_stamps, second_xyz_full_aligned.transpose().A, '-', "blue", "estimated")
        label = "difference"
        for (a, b), (x1, y1, z1), (x2, y2, z2) in zip(matches, first_xyz.transpose().A, second_xyz_aligned.transpose().A):
            ax.plot([x1, x2], [y1, y2], '-', color="red", label=label)
            label = ""
            
        ax.legend()
        ax.set_xlabel('x [m]')
        ax.set_ylabel('y [m]')
        plt.axis('equal')
        plt.savefig(args.plot, format="pdf")
```
运行评估代码
```shell
python evaluation/evaluate_ate_scale_py3.py evaluation/Ground_truth/EuRoC_left_cam/V102_GT.txt f_dataset-V102_stereo.txt --plot V102_stereo.pdf
```
生成误差pdf
## 使用realsense相机（D455）
需要安装realsense SDK（详见yolo11+realsD455配置记录）
以Example/Monocular为例
复制Realsense_D435i.yaml文件并重命名为RealSense_D455.yaml ，根据D455参数修改fx、fy、cx、cy、width、height等参数（realsense相机的参数获取可通过realsense-ros，查看ros节点camera_info）
运行代码参考mono_realsense_D435i.cc中主函数里的提示
```c++
"Usage: ./mono_realsense_D435i path_to_vocabulary path_to_settings (trajectory_file_name)"
```
使用D455运行Monocular时，借用mono_realsense_D435i.cc文件并不修改，不用重新编译，终端位于ORB_SLAM3主目录时，通过以下代码运行
```shell
./Examples/Monocular/mono_realsense_D435i Vocabulary/ORBvoc.txt ./Examples/Monocular/RealSense_D455.yaml 
```
加载需要一定时间
# ORB-SLAM（ROS）
源码在ORB_SLAM3/Examples_old/ROS/ORB_SLAM3中
## 修改CMakeLists.txt
所有Opencv和Eigen修改为安装的现有版本
```txt
find_package(OpenCV 4.2.0)
find_package(Eigen3 REQUIRED)
```
## 编译ORB-SLAM3-ROS
在ROS/ORB_SLAM3主目录下，遇到编译错误时，根据后面小节更改后，需要清空build文件夹内容，再重新cmake
```shell
mkdir build
cd build
cmake .. -DCMAKE_BUILD_TYPE=Release
make -j4
```

___--------以下操作建议报出对应错误后再进行--------___
## ROS环境环境报错
```shell
gedit ~/.bashrc
```
在最后插入
```shell
export ROS_PACKAGE_PATH=${ROS_PACKAGE_PATH}:~/catkin_ws/src/ORB_SLAM3/Examples_old/ROS
```
并
```shell
source ~/.bashrc
```
检查
```shell
echo $ROS_PACKAGE_PATH
```
## 找不到sophus/se3.hpp文件
在CMakeLists.txt中添加Sophus路径（前面直接安装了就不用添加）
```txt
include_directories(
${PROJECT_SOURCE_DIR}
${PROJECT_SOURCE_DIR}/../../../
${PROJECT_SOURCE_DIR}/../../../include
${PROJECT_SOURCE_DIR}/../../../include/CameraModels
${PROJECT_SOURCE_DIR}/../../../Thirdparty/Sophus
${Pangolin_INCLUDE_DIRS}
)
```
## ‘Sophus::SE3f’到‘cv::Mat’类型转换错误
### ROS/ORB_SLAM3/src/AR/ros_mono_ar.cc文件第151行
将
```c++
cv::Mat Tcw = mpSLAM->TrackMonocular(cv_ptr->image,cv_ptr->header.stamp.toSec());
```
  替换为
```c++
cv::Mat Tcw;
Sophus::SE3f Tcw_SE3f = mpSLAM->TrackMonocular(cv_ptr->image,cv_ptr->header.stamp.toSec());
Eigen::Matrix4f Tcw_Matrix = Tcw_SE3f.matrix();
cv::eigen2cv(Tcw_Matrix, Tcw);
```
### ROS/ORB_SLAM3/src/AR/ViewerAR.cc文件第409行
将
```c++
vPoints.push_back(pMP->GetWorldPos());
```
替换为
```c++
cv::Mat WorldPos;
cv::eigen2cv(pMP->GetWorldPos(), WorldPos);
vPoints.push_back(WorldPos);
```
### ROS/ORB_SLAM3/src/AR/ViewerAR.cc文件第539行
将
```c++
cv::Mat Xw = pMP->GetWorldPos();
```
替换为
```c++
cv::Mat Xw;
cv::eigen2cv(pMP->GetWorldPos(), Xw);
```
## 找不到‘eigen2cv’
在报错的文件前面加入
```c++
#include <Eigen/Dense>
#include <opencv2/core/eigen.hpp>
#include <opencv2/opencv.hpp>
```
## 运行数据集
以ORB-SLAM3官方推荐的[EuRoC数据集](https://projects.asl.ethz.ch/datasets/doku.php?id=kmavvisualinertialdatasets#downloads)为例，[备用下载链接](http://robotics.ethz.ch/~asl-datasets/ijrr_euroc_mav_dataset/)
使用ORB-SLAM-ROS时，下载rosbag版本（如V1_02_medium.bag ）
以Mono_Inertial为例
第一个终端，启动roscore
```shell
roscore
```
第二个终端，播放rosbag
```shell
rosbag play ~/ORB_SLAM3/dataset_euroc/V1_02_medium.bag /cam0/image_raw:=/camera/image_raw /imu0:=/imu
```
第三个终端，运行mono_inertial ROS节点
运行代码参考ros_mono_inertial.cc中主函数里的提示
```c++
"Usage: rosrun ORB_SLAM3 Mono_Inertial path_to_vocabulary path_to_settings [do_equalize]"
```
运行
```shell
rosrun ORB_SLAM3 Mono_Inertial ../../../../Vocabulary/ORBvoc.txt ../../../../Examples/Monocular-Inertial/EuRoC.yaml 
```
后续直接连接设备时，修改订阅节点即可