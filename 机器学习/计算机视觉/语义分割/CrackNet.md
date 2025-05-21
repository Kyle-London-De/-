Wang, Kelvin C. P. 美国 2013-2019
基于3D信息，深度图
# CrackNet
Computer-Aided Civil and Infrastructure Engineering 32 (2017) *高引*
没有池化层，从根本上保证像素级精度
特征提取器：用于从原始图像中生成特征图（36个方向、2个长度、5个宽度的过滤器），人工设计，使用过滤器匹配裂缝轮廓
![](D:/p/featureextractor.png)

![](D:/p/cracknet.png)
1. CrackNet的输入是由特征提取器的360个特征图
2. 使用50x50x360的卷积核
3. 全连接层：将一个像素的所有通道作为特征向量，输入全连接层，得到的输出空间与输入空间相同
4. Leaky-ReLU（实现非线性变换），提取像素周围的局部特征信息
5. 1x1卷积层，将通道压缩到1维，sigmoid函数输出（>0.5）
6. 100万个参数
7. 交叉熵损失学习，使用归一化初始化，dropout（0.5）
- 根据同一像素位置的多通道响应来学习像素级裂缝特征
- 单个通道的每个像素使用相同的权重，来实现空间不变性，达到特征的像素级定位
# CrackNet Ⅱ
Journal of Computing in Civil Engineering, 2018 *高引*
四个卷积层，五个通道1x1卷积层，一个输出层
去掉特征提取器，特征提取器人工设计限制了网络的学习能力
CrackNet Ⅱ 使用更多的隐藏层，但使用更少的参数（维度），更精准的概括裂缝检测的复杂性，性能提升5倍
逐层提高卷积核大小，有助于深层网络对更大的上下文的把握
![](D:/p/cracknet2.png)
*使用掩蔽过滤消除路面的全局不平整性*
![](D:/p/flatten.png)
# CrackNet V
IEEE TRANSACTIONS ON INTELLIGENT TRANSPORTATION SYSTEMS 2020 *高引*
结合VGG网络，一个前处理层，8个卷积层，一个输出层，64113个参数
![](D:/p/cracknetv.png)
预处理使用中值滤波器，并在输入时采用四张预处理图像（不同尺寸的中值滤波器）
3D数据集包含了裂缝的深度信息，该信息在卷积后可能存在巨大差异，而Leaky ReLU 激活函数会保留这一差异，导致网络专注于学习裂纹的深度范围，但当考虑噪声时，裂缝的深度范围的特征可能并不是裂缝独有的，更希望网络关注裂缝的形态信息这一唯一性更强的信息，甚至因此忽略了一下浅层裂缝，因此使用 *Leaky Rectified Tanh 作为隐藏层的激活函数*
$$
\sigma(x)=\max\left(1.7159\times\tanh\left(\frac{2}{3}x\right),0\right)+\min(0.1x,0)
$$
![](D:/p/leakyrectifiedtanh.png)
使用 *LeCun Tanh 输出层激活函数*，代替sigmoid函数。LeCun Tanh 的范围 Sigmoid 的大，可学习的参数有更多的空间可以反向传播调整；同时拥有正负输出，适用于二元分类问题；一阶导更大，梯度反向传播更好
$$
\sigma(x)=1.7159\times\tanh(\frac{2}{3}x)
$$
![](D:/p/lecuntanh.png)
交叉熵损失函数和L2权重结合，快速训练的同时，防止过拟合