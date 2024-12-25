# ImuPreintegration
## ImuPreintegration类
1. 订阅后端的*增量里程计*（激光），通过imu原始数据进行因子图优化，得到每一激光帧的*imu偏置*。（使用增量里程计进行因子图优化，是因为增量里程计不会有大幅的位姿跳跃，适用于因子图中对预积分的约束。）由于imu频率远高于增量里程计，得到新的imu偏置后，还需要更新当前激光帧已经计算的imu预积分器
2. 订阅imu原始数据，用不断更新的imu偏置计算*imu里程计*（使用imu预积分器）并发布
## TransformFusion类
1. 在imu里程计的基础上，用激光里程计的位姿替换掉增量里程计的位姿，新的imu里程计用于Rviz展示
# ImageProjection
1. 通过imu原始数据获取*旋转增量*，通过imu里程计获取*平移增量*（移动速度较慢的行人等，可以忽略平移），对原始雷达数据进行*运动畸变校正*，将当前激光帧点云都校正到第一个激光点时刻
2. 发布校正后的有效点云用于Rviz展示；发布校正后的各项点云信息，用于FeatureExtraction
3. 从*imu原始数据*中提取激光帧起始时刻的*姿态角*
	1. 用于第一帧雷达帧的初始姿态角
	2. 用于scan-to-map之后的姿态角融合
4. 从*imu里程计*中提取激光帧起始时刻的*位姿*
	1. 用于非第一帧雷达帧的初始位姿
# FeatureExtraction
1. 对运动畸变校正后的点云进行*特征提取*，通过计算激光点曲率判断*角点*和*平面点*
2. 发布当前激光帧角点点云、平面点点云，用于Rviz展示；发布特征提取后的各项点云信息，用于MapOptimization
# MapOptimization
1. scan-to-map（特征点匹配）
	1. 第一帧使用*imu原始数据*初始化，非第一帧使用*imu里程计*初始化（基于前一帧优化后的位姿）
	2. 提取最近一帧*一定时间空间*内的关键帧组成局部地图，并提取特征点
	3. 通过当前帧与局部地图的特征点，进行匹配优化，将优化后的位姿与imu原始数据的姿态角进行加权融合，得到scan-to-map后的位姿
2. 闭环线程
	1. 在历史关键帧中寻找与当前帧在*空间上较近、时间上较远*的关键帧，作为候选闭环帧
	2. 使用scan-to-map的*icp方法*，通过当前帧与闭环帧相邻关键帧特征点匹配，得到闭环因子所需的数据
3. 因子图优化
	1. 添加新的关键帧，需要当前帧与前一帧的的位姿变化大于阈值才添加
	2. 添加激光里程计因子（当前帧与前一帧的位姿变换）
	3. 添加GPS因子，需要与上一次添加间隔一段距离，且位姿协方差较大才添加
	4. 添加闭环因子
	5. 因子图优化，更新*最优的位姿*
4. 发布激光里程计（*scan-to-map+因子图优化*，非关键帧则没有因子图优化），最优位姿；发布增量里程计（*仅scan-to-map*），当前帧与前一帧的位姿变换