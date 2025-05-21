# 安装与运行
[Github pytorch-unet](https://github.com/milesial/Pytorch-UNet)
直接下载数据失败
```shell
bash scripts/download_data.sh
```
去官网下载[kaggle比赛数据](https://www.kaggle.com/competitions/carvana-image-masking-challenge/data)，根据.sh文件步骤放置文件
图像分辨率 1918x1280，针对数据集的分割效果良好，但*对于与训练样本差距较大的图像分割效果显著降低*
# Seg-Net （提出编码器解码器模型）
![](https://ask.qcloudimg.com/http-save/yehe-781483/gv2qar7tb5.jpeg)
# U-Net
![](https://i-blog.csdnimg.cn/direct/1974f2c5c0f249739f9594a42dd27630.png#pic_center)
跳跃连接，用于特征融合，将编码阶段的遗失的高分辨率特征信息，传递到对应层的解码阶段中（concat，延通道维度拼接），也有助于梯度传播，缓解深度网络梯度消失
上采样多种，反卷积、双线性插值、最近邻插值
不同深度网络用于不同检测对象
## Dice系数
衡量两个区域重叠程度
$$
Dice=\frac{2|X\cap Y|}{|X|+|Y|}
$$
$$
Dice\_Loss=1-Dice
$$
# DeepCrack
DeepCrack: Learning Hierarchical
Convolutional  Features for Crack Detection——IEEE TRANSACTIONS ON IMAGE PROCESSING, VOL. 28, NO. 3, MARCH 2019

DeepCrack 基于 Seg-Net 的编码器解码器的结构，通过在 VGG-16基础上集成批归一化层和侧网络预测融合技术，显著提高了 FCN 在不同场景下的裂缝检测精度。
分层卷积特征的融合对从图像背景中推断出裂缝非常有效
![](https://i-blog.csdnimg.cn/blog_migrate/55830fb8bfd34ab60af3611c764ca5c9.png)
上采样unpooling，与max pooling结合，同时利用稀疏特征图和连续特征图
跳跃连接：1x1x1 conv，将多通道转换到一个通道上，反卷积到原始大小，多层再卷积融合，输出最终特征图
![](https://i-blog.csdnimg.cn/blog_migrate/6333b22586cea2b887abbcccd3199be4.png)
损失函数不再需要使用softmax多分类器，使用交叉熵损失逐像素计算概率
![](https://i-blog.csdnimg.cn/blog_migrate/69c3029618aa19439e9c6bf5acc9cd88.png)
编码-解码结构得到薄裂纹图
![](https://i-blog.csdnimg.cn/blog_migrate/886e22ebfc20ef303f4943ec5cb3c0eb.png)

Chen和 Jahanshahi 借鉴主动旋转滤波器对DeepCrack进行了改进，增强了网络对旋转不变裂缝的识别性能。
# CrackU-Net
Struct Control Health Monit. 2020
*被引用237*
基于UNet架构，固定卷积层数4层，针对裂缝特征
使用固定的卷积核，因为裂缝局部相似性
使用Max Pooling，因为裂缝往往与背景高对比度
使用交叉熵损失函数，因为裂缝像素占比小
![](D:/p/crackunet.png)
# DAU-Net 密集注意力U-Net
2021 IEEE Intelligent Transportation Systems Conference (ITSC)
密集编码器+注意力解码器
![](D:/p/daunet.png)
密集编码器（dense encoder）：提高提取上下文特征的能力，设计多级密集块的编码器，密集块中，后面的卷积层输入是前面特征的融合（16->12，12+16=28->12，28+12=40->12）
![](D:/p/denseblock.png)
注意力解码器（CAB，通道注意力模块）：
1. 全局注意力池化：解码器转置卷积得到的特征图->压缩通道数为1->特征向量->softmax->每个像素的注意力权重，编码器特征图->卷积一次转化到特征空间->多通道特征向量，将编码器和解码器得到的特征图进行融合，得到通道层面的全局上下文特征图（Cx1x1）
2. 瓶颈转换：通过降维->卷积->升维的操作，加速提取全局上下文特征（Cx1x1）
3. 加法特征融合：使用加法融合函数，全局上下文特征权重加到编码器特征图上，用来屏蔽不相关的特征和噪声，来相对增强相关特征
4. 编码器提供特征图，解码器提供通道注意力权重
![](D:/p/cab.png)
疤痕等容易误导裂缝分割模型