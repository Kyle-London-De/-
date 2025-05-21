# 配置
[配置参考](https://blog.csdn.net/lixushi/article/details/122578852?ops_request_misc=%257B%2522request%255Fid%2522%253A%25229e16a727b5858ad3f65e1db0c5ec3f83%2522%252C%2522scm%2522%253A%252220140713.130102334..%2522%257D&request_id=9e16a727b5858ad3f65e1db0c5ec3f83&biz_id=0&utm_medium=distribute.pc_search_result.none-task-blog-2~all~sobaiduend~default-1-122578852-null-null.142^v101^pc_search_result_base7&utm_term=kuka_experimental&spm=1018.2226.3001.4187)
根据ubuntu版本修改melodic->noetic，使用kuka-kr16为例
```shell
git clone https://github.com/ros-industrial/kuka_experimental.git

sudo apt-get install ros-noetic-ros-control ros-noetic-ros-controllers ros-noetic-gazebo-ros ros-noetic-gazebo-ros-control ros-noetic-hector-gazebo-plugins 
sudo apt-get install ros-noetic-joint-trajectory-controller
sudo apt-get install ros-noetic-moveit*
```
# urdf生成及解析
复杂结构的urdf一般通过sw插件sw_urdf_exporter生成，*最好通过坐标系生成，并且添加配合使机械臂固定为初始姿态，建议使用一般DH规律建系，尤其注意基座和末端的坐标系*
[参考1](https://www.mahaofei.com/post/7fec171b.html)
[参考2](https://blog.csdn.net/qq_45672222/article/details/128161189?spm=1001.2101.3001.6650.3&utm_medium=distribute.pc_relevant.none-task-blog-2%7Edefault%7ECTRLIST%7ERate-3-128161189-blog-108015605.235%5Ev43%5Epc_blog_bottom_relevance_base9&depth_1-utm_source=distribute.pc_relevant.none-task-blog-2%7Edefault%7ECTRLIST%7ERate-3-128161189-blog-108015605.235%5Ev43%5Epc_blog_bottom_relevance_base9)
***在 URDF 文件中，每行的缩进并不直接影响功能性，因为 XML 格式对缩进和空格不敏感***
## robot
包裹整个机器人模型
### link
定义机器人中一个刚体部分
```xml
<link name="base_link">
	<inertial> <!-- 质量属性 -->
		<origin rpy="0 0 0" xyz="0 0 0" /> <!-- 质量中心位置和方向 -->
		<mass value="2" /> <!-- 质量（单位kg） -->
		<inertia ixx="0.01" ixy="0" ixz="0" iyy="0.01" iyz="0" izz="0.01" /> <!-- 惯性张量（惯性矩） -->
	</inertial>
	<visual> <!-- 视觉 -->
		<origin rpy="0 0 0" xyz="0 0 0" />
		<geometry> <!-- 几何模型 -->
			<mesh filename="package://kuka_kr16_support/meshes/kr16_2/visual/base_link.dae" />
		</geometry>
	</visual>
	<collision> <!-- 碰撞 -->
		<origin rpy="0 0 0" xyz="0 0 0" />
		<geometry>
			<mesh filename="package://kuka_kr16_support/meshes/kr16_2/collision/base_link.stl" />
		</geometry>
	</collision>
</link>
```
### joint
定义两个链接之间的运动约束（关节）
```xml
<joint name="joint_a1" type="revolute">
	<origin rpy="0 0 0" xyz="0 0 0.675" /> <!-- 初始位置和方向 -->
	<parent link="base_link" /> <!-- 父链接 -->
	<child link="link_1" /> <!-- 子链接 -->
	<axis xyz="0 0 -1" /> <!-- 旋转轴/滑动轴 -->
	<limit effort="0" lower="-3.2288591161895095" upper="3.2288591161895095" velocity="2.722713633111154" /> <!-- 关节关节 最大力矩/运动限制/速度限制 -->
</joint>
```
### transmission
定义关节驱动方式（传动器），通常用于与控制器交互
```xml
<transmission name="trans_joint_a1">
	<type>transmission_interface/SimpleTransmission</type> <!--传动类型-->
	<joint name="joint_a1"> <!-- 指定驱动关节 -->
		<hardwareInterface>hardware_interface/PositionJointInterface </hardwareInterface> <!-- 硬件接口 力力矩/位置/速度控制 -->
	</joint>
	<actuator name="joint_a1_motor"> <!-- 驱动器 -->
		<hardwareInterface>hardware_interface/PositionJointInterface </hardwareInterface>
		<mechanicalReduction>1</mechanicalReduction> <!-- 减速比 -->
	</actuator>
</transmission>
```
### gazebo
定义一个gazebo特定的配置块，连接ROS和gazebo
```xml
<gazebo>
	<plugin name="gazebo_ros_control" filename="libgazebo_ros_control.so"> <!-- 插件 名称和（标准）动态库 -->
		<robotNamespace>/</robotNamespace> <!-- 指定命名空间 -->
	</plugin>
</gazebo>
```
# moveit生成及修改
## moveit助手
启动moveit助手，参考[moveit+gazebo](https://blog.csdn.net/lixushi/article/details/122578852?ops_request_misc=%257B%2522request%255Fid%2522%253A%25229e16a727b5858ad3f65e1db0c5ec3f83%2522%252C%2522scm%2522%253A%252220140713.130102334..%2522%257D&request_id=9e16a727b5858ad3f65e1db0c5ec3f83&biz_id=0&utm_medium=distribute.pc_search_result.none-task-blog-2~all~sobaiduend~default-1-122578852-null-null.142^v101^pc_search_result_base7&utm_term=kuka_experimental&spm=1018.2226.3001.4187)生成moveit-config文件
```shell
roslaunch moveit_setup_assistant setup_assistant.launch
```
## moveit-config修改
### config/gazebo_xxx.urdf
#### 添加世界坐标系固定机械臂
```xml
<link name="world"/>
<joint name="fixed" type="fixed">
	<parent link="world"/>
	<child link="base_link"/>
</joint>
```
#### 修改质量
修改所有link的质量，这里质量的取值与后面的力以及pid参数有关，最直接的影响就是机械臂在gazebo中的抖动幅度，传统机械臂位控时可以参考以下取值
```xml
<link name="xxx">
	<inertial>
		...
		<mass value="2" />
		...
	</inertial>
</link>
```
#### 修改力
修改所有joint的力（limit其他参数如果在前面生成urdf时未设置，这里也可以修改）
```xml
<joint name="xxx" type="revolute">
	...
	<limit lower="-3.14" upper="3.14" effort="50" velocity="1" />
	...
</joint>
```
#### 修改控制方式
修改所有传动器transmission的控制方式
```xml
<transmission name="trans_joint_xxx">
	...
	<joint name="joint_xxx">
		<hardwareInterface>hardware_interface/PositionJointInterface
		</hardwareInterface>
	</joint>
	<actuator name="joint_xxx_motor">
		<hardwareInterface>hardware_interface/PositionJointInterface
		</hardwareInterface>
		...
	</actuator>
	...
</transmission>
```
### ros_controllers.yaml
***yaml文件对缩进和空格敏感，注意对齐（尽量在原有的基础上修改）***
```yaml
arm_position_controller:
	type: position_controllers/JointTrajectoryController
	joints:
		- joint1
		- joint2
		- joint3
		- joint4
		- joint5
		- joint6
	gains:
		joint1:
			p: 100.0
			d: 0.01
			i: 10.0
		joint2:
			p: 100.0
			d: 0.01
			i: 10.0
		joint3:
			p: 100.0
			d: 0.01
			i: 10.0
		joint4:
			p: 100.0
			d: 0.01
			i: 10.0
		joint5:
			p: 100.0
			d: 0.01
			i: 10.0
		joint6:
			p: 100.0
			d: 0.01
			i: 10.0
```
# 启动
```shell
source catkin_xxx/devel/setup.bash
roslaunch xxx_moveit_config demo_gazebo.launch 
```
通过代码直接控制参考[moveit官方代码](https://moveit.github.io/moveit_tutorials/doc/move_group_python_interface/move_group_python_interface_tutorial.html)
# kuka机械臂-空间姿态xyzabc
a、b、c 分别代表绕z、y、x的旋转角度，即欧拉角中的yaw、pitch、roll
内旋、绕自身、右乘（由左向右）：$R_1=R_z(a)R_y(b)R_x(c)$
外旋、绕定轴、左乘（由右向左）：$R_2=R_z(a)R_y(b)R_x(c)$
$R_1=R_2$，zyx的顺序左乘（内旋）等价于xyz的顺序左乘（外旋）
*向量绕某一个坐标轴进行旋转时，相当于坐标轴进行反方向的旋转*，所以只需要将旋转的角度取负值代入对应的坐标轴旋转矩阵即可