# YOLOv8
![](https://i-blog.csdnimg.cn/blog_migrate/04c2dd15fc4b3ff4d05f2e7cf01d9b9a.png)
## 基础设置
### 模型大小
1. n：最小的模型，最快的推理，最低的准确性
2. s
3. m
4. l
5. x
### 卷积层设置
1. k（kernel参数）：卷积核参数，代表卷积核的大小（k x k）
2. s（stride步幅）：滤波器在图像上滑动的步长
3. p（padding填充）：图像每一侧边缘添加的额外零边框
4. c（channel输入通道数）：输入的通道数
5. w（width multiple宽度倍数）：网络宽度控制参数，用于控制通道数（0.25、0.5、1.0等）
## YOLOv5和YOLOv8对比图
- YOLOv5
![](https://i-blog.csdnimg.cn/blog_migrate/f17ca55a95ab82154938a5e1c31a67ed.png)
- YOLOv8
![](https://i-blog.csdnimg.cn/blog_migrate/8e19632bfb69492bb49c998f1bf0fefc.png)
## Backbone
负责提取图像特征，如颜色、纹理、边缘等信息
由 5 个 Conv 模块、4 个 C2f 模块和 1 个 SPPF 模块
### CBS
Conv+BN+SiLU
SiLU激活函数，在0附近比ReLU更平滑，适用于深层网络
$$
f(x)=x\cdot\sigma(x)=\frac{x}{1+e^{-x}}
$$
![](https://i-blog.csdnimg.cn/direct/04575449ee8e4f0d8cb37dc74b699aa4.png)
### C3
![](https://i-blog.csdnimg.cn/blog_migrate/a3eb4ee42c32a5a8bd9b1e1b9930a31d.png)
这里的梯度流主分支为Bottleneck\*n（根据需要可以选择不同的主分支网络和不同的n），使用了3个CBS模块，通过concat沿通道拼接，保留部分浅层特征，减少信息损失
### C2f
YOLOv8 相比于 YOLOv5，使用 C2f 模块替换 C3 模块，网络进一步轻量化
![](https://i-blog.csdnimg.cn/blog_migrate/c4c5bd6344f4378419a92257f2ad767b.png)
一种另类的残差模块，split 模块将特征图沿通道方向一分为二，一半直接传递到 concat 模块，一半则经过 0~n 个 Bottleneck 模块后传递到 concat 模块中，拼接后的输出是 0.5(n+2) 倍的原通道数
C2f在YOLOv8中有两个不同网络设置
- C2f 的 Bottleneck 有一个 shortcut 的区别
![](https://i-blog.csdnimg.cn/blog_migrate/d3d252e106573da5454f17d14aaea2ce.png)
- C2f 的 Bottleneck 的 数量 n，由 $n=3\times d$ 得到，其中 d 是 depth multiple 深度倍数
### SPPF 空间金字塔池化快速（Spatial Pyramid Pooling - Fast）
![](https://i-blog.csdnimg.cn/blog_migrate/4928a0a8cae551e1775699a206338c15.png)
SPPF 模块（右）通过将0~3个 maxpooling 的输出拼接进行多尺度特征融合，提高对不同尺寸目标的检测能力，相比于SPP 模块（左），参数更少，训练速度更快
## Neck
负责多尺度特征融合，将来自Backbone不同阶段的特征图进行融合，增强特征表示能力
包含 SPPF 模块（与Backbone是同一个）和 FPN-PAN（也有叫2 个 PAN） 模块
![](https://i-blog.csdnimg.cn/blog_migrate/5a083ff0f6daa1930721e44ff544c0a9.png)
### FPN （特征金字塔网络 Feature Pyramid Network）
2017年提出，构建由不同层的多尺度特征图，将低分辨率高语义信息的高层特征、高分辨率低语义信息的低层特征融合，获得所有尺度下的特征都丰富的语义信息
从高层（深层）特征开始，逐层上采样，每层的上采样特征与低一层的特征图进行融合，补充空间信息和增强语义信息，并通过1x1Conv对齐通道数，逐层相加的方式进行特征融合
### PAN （路径聚合网络 Path Aggregation Network）
PAN 在 FPN 的基础上，进一步增强特征金字塔网络结构
从低层特征开始，逐层向高层传递，通过跨步卷积进行下采样提取的特征与高一层的特征图进行融合
### PAN-FPN
PAN+FPN 的组合，首先通过FPN构建自顶而下的特征金字塔，进行多尺度特征的初步融合，然后引入PAN的自底而上的特征融合，进一步丰富多尺度特征
不断使用上采样/跨步卷积和 concat 来融合不同尺度的特征，进行双向特征融合，确保每一层的特征都包含不同尺度的信息
最终的输出分别是80 x 80 x（min（256，mc）x w）、40 x 40 x（min（512，mc）x w）、20 x 20 x（min（1024，mc）x w）
## Head（损失函数）
负责将Neck部分输出的特征进一步处理，生成最终的检测结果，如类别、位置和置信度等
![](https://i-blog.csdnimg.cn/blog_migrate/6a5dd2e75058ee2f89ac390f8d57b6ed.png)
三个detect模块，输入来自不同尺度的特征图（20、40、80）分别用于检测大型物体、中型物体、小型物体
采用解耦式预测头（Decoupled Head）的设计，损失函数包括两个分支，分类分支（类别和置信度）和回归分支（回归框坐标），YOLOv8 使用 BCE 计算分类损失，使用 DFL Loss 和 CIoU Loss 计算回归损失，采用 Anchor-free 的设计，直接预测目标的边框位置
### PAA （Probabilistic Anchor Assignment）
YOLOv8中未使用，智能分配锚框，以优化正负样本的选择，提高模型的训练效率
### NMS 非极大值抑制（Non-Maximum Suppression）
将预测结果通过非极大值抑制，去除重复的检测框，保留置信度最高的边界框，并移除与之重叠度高的其他边界框，目的是确保每个目标只检测一次
### IoU 及其变体
#### IoU（Intersection over Union）
是一种衡量目标检测性能的评估指标，主要用于计算两个边界框之间的重叠度，也就是交并比
$$
IoU=\frac{|A\cap B|}{|A\cup B|}
$$
#### GIoU（Generalized IoU）
引入两个边界框的*最小外接矩形*，来获取两框在最小闭包区域的比重，从而解决了两框没有交集时梯度为0的问题
$$
GIoU=IoU-\frac{|C-(A\cup B)|}{|C|}
$$
#### DIoU（Distance IoU）
在IoU的基础上直接回归两个框中心点的欧式距离，DIoU的额外惩罚项是基于*中心点的距离和对角线距离的比值*，避免了在两框距离较远时Loss值过大
$$
DIoU=IoU-\frac{\rho^2(b,b^{gt})}{c^2}
$$
其中 $\rho$ 表示两个中心点的欧式距离， $c$ 为两框最小闭包区域的对角线距离，
#### CIoU（Complete IoU）
在DIoU的基础上又添加了长宽比的惩罚项
$$\begin{matrix}
CIoU=1-IoU+\frac{\rho^2(b,b^{gt})}{c^2}-\alpha v\\
\alpha=\frac{v}{(1-IoU)+v}\\
v=\frac{4}{\pi^2}\left(\arctan\frac{w^{gt}}{h^{gt}}-\arctan\frac{w}{h}\right)^2
\end{matrix}
$$
#### EIoU（Efficient IoU）
在CIoU的基础上，将长宽比的惩罚项拆开，分别计算长和宽，并加入了Focal聚焦优质的锚框，来解决CIoU中长宽比相同但值不同的问题
$$
EIoU=1-IoU+\frac{\rho^2(b,b^{gt})}{c^2}+\frac{\rho^2(w,w^{gt})}{c_w^2}+\frac{\rho^2(h,h^{gt})}{c_h^2}
$$
其中 $c_w$ 表示两框最小闭包区域的宽，$c_h$ 同理
#### Focal EIoU 
是利用 Focal Loss 函数对 EIoU 进行加权处理
$$
Focal\text{-}EIoU=IoU^\gamma*EIoU
$$
#### SIoU（Shape-Enhanced IoU）
引入角度对齐约束，提升方向一致性
$$
SIoU=1-IoU+\lambda_{angle}+\lambda_{dist}+\lambda_{shape}
$$

- 角度成本公式及直观图（其中 $\lambda_{angle}=1-2\sin^2(\theta)$ ，$\theta$ 为两框的角度差异）
![](https://s2.51cto.com/images/blog/202411/28113806_6747e59ea1f1870882.png?x-oss-process=image/watermark,size_16,text_QDUxQ1RP5Y2a5a6i,color_FFFFFF,t_30,g_se,x_10,y_10,shadow_20,type_ZmFuZ3poZW5naGVpdGk=/format,webp/resize,m_fixed,w_1184)
- 距离成本公式及直观图（角度成本在距离成本中被使用，所以总公式中不再出现角度成本）
![](https://s2.51cto.com/images/blog/202411/28113806_6747e59ede6cd63132.png?x-oss-process=image/watermark,size_16,text_QDUxQ1RP5Y2a5a6i,color_FFFFFF,t_30,g_se,x_10,y_10,shadow_20,type_ZmFuZ3poZW5naGVpdGk=/format,webp/resize,m_fixed,w_1184)
- 形状成本公式
![](https://s2.51cto.com/images/blog/202411/28113807_6747e59f156d896728.png?x-oss-process=image/watermark,size_16,text_QDUxQ1RP5Y2a5a6i,color_FFFFFF,t_30,g_se,x_10,y_10,shadow_20,type_ZmFuZ3poZW5naGVpdGk=/format,webp/resize,m_fixed,w_1184)
- *SIoU总公式*
$$
SIoU=1-IoU+\frac{\Delta+\Omega}{2}
$$
#### WIoU（Weighted IoU）
动态调整样本权重，关注困难样本
$$
WIoU=r\cdot(1-IoU),\quad r=\frac{e^{}(1-IoU)}{\gamma}
$$
其中 $r$ 为动态权重因子，$\gamma$ 为超参数
#### 对比

|  方法  |      核心       |      优点       |        缺点        |    应用     |
| :--: | :-----------: | :-----------: | :--------------: | :-------: |
| IoU  |    基础重叠度量     |    简单，计算快     | 无重叠时梯度消失，忽略形状和方向 |   初步框评估   |
| GIoU |   引入最小闭合区域    |   解决无重叠梯度问题   |     长宽比优化不足      |  一般目标检测   |
| CIoU | 增加中心点距离和长宽比约束 |    高精度框回归     |   计算复杂，长宽比依赖近似   |   小目标检测   |
| EIoU |  分解长宽比为独立约束   | 更直接地优化尺寸，计算高效 |     对极端长宽比敏感     |  人脸/文字检测  |
| SIoU |  对角对齐和方向一致性   |    提升方向敏感性    |      计算复杂度高      |  车辆/行人检测  |
| WIoU |    动态样本权重     | 关注困难样本，缓解不平衡  |  需要调参，可能过拟合困难样本  | 遮挡/复杂场景检测 |
计算复杂度：SIoU > CIoU > EIoU > GIoU > WIoU
### BCE、FL 及其变体
#### BCE 二值交叉熵（Binary Cross-Entropy）
是一种应用于二分类任务中的损失函数，用于衡量目标类别预测值与实际值之间的差距
$$
BCE(y,p)=-y\log(p)-(1-y)log(1-p)
$$
其中 $y$ 表示目标的实际类别，值为0或1，$p$ 为目标的预测类别，值为\[0，1\]，利用函数
$$
BCE(p_t)=-\log(p_t),\quad 
p_t=\begin{cases}
p,&y=1\\
1-p,&otherwise
\end{cases}
$$
针对多类别分类任务同样适用
#### FL（Focal Loss）
在 BCE Loss 的基础上添加权重系数解决以下问题
- 解决正负样本不平衡问题，目标检测任务中存在大量的背景，即负样本，而实际目标，即正样本，却占比较少，引入超参数 $\alpha\in(0,1)$
$$
\begin{aligned}
BCE(p_t)=-\alpha_t\log(p_t)\\
\alpha_t=\begin{cases}
\alpha&y=1\\
1-\alpha&otherwise
\end{cases}
\end{aligned}
$$
- 降低易分类样本的权重，即，使模型训练更加关注与困难样本，预测值越接近于1则说明该样本为简单样本，用 $1-p_t$ 降低权重
$$
FL(p_t)=-(1-p_t)^\gamma\log(p_t)
$$
最终 FL 可表示为
$$
FL(y,p)=
\begin{cases}
-\alpha(1-p)^\gamma\log(p)&y=1\\
-(1-\alpha)p^\gamma\log(1-p)&otherwise
\end{cases}
$$
#### QFL（Quality Focal Loss）
为了解决训练中定位质量（目标框质量评估）和分类得分独立训练，而测试中联合评估的不匹配问题，提出了定位分类联合表示的方法
![](https://i-blog.csdnimg.cn/blog_migrate/df72ec6f0c86b1378ed5eb9e0dd0535d.png#pic_center)
分类输出的标签 y 不再是传统的0或者1，而是来自预测框与真实框的交并比IoU的定位质量参数，取值0~1（使用sigmoid函数归一化），同时也将 $(1-p_t)^\gamma$ 修改为 $|y-\sigma|^\beta$ ，可以理解为预测值与质量参数的差距，$\beta$ 取2时最佳。（$\sigma$ 在这里可以简单等效为 $p_t$）
![](https://i-blog.csdnimg.cn/blog_migrate/10b93954d3d13d9037b759e0ee3f6caf.png)
$$
QFL=|y-\sigma|^\beta[-(1-y)\log(1-\sigma)-y\log(\sigma)]
$$
#### DFL（Distribution Focal Loss）
降低将边界框位置回归为狄拉克分布或高斯分布时的误差，转而学习边界框位置的一般分布的离散表示，$[y_0,y_1,\dots,y_n],\Delta=1$ ，并鼓励分布接近目标y，扩大 $y_i$ 和 $y_{i+1}$ 的概率（$y_i<y<y_{i+1}$），y的分布概率可以用n+1个单位的softmax函数得到，输出为 $S_i$
![](https://i-blog.csdnimg.cn/blog_migrate/10b93954d3d13d9037b759e0ee3f6caf.png)
$$
DFL(S_i,S_{i+1})=-(y_{i+1}-y)\log(S_i)-(y-y_i)\log(S_{i+1})
$$
直观来讲
$$
S_i=\frac{y_{i+1}-y}{y_{i+1}-y_i},S_{i+1}=\frac{y-y_i}{y_{i+1}-y_i}
$$
而回归目标 $\hat{y}$ 的正确性也得以证明
$$
\hat{y}=\sum^{n}_{j=0}P(y_i)y_i=S_iy_i+S_{i+1}y_{i+1}=\frac{y_{i+1}-y}{y_{i+1}-y_i}y_i+\frac{y-y_i}{y_{i+1}-y_i}y_{i+1}=y
$$
#### GFL（Generalized Focal Loss）
将QFL和DFL统一为一般形式，假设一个模型估计两个变量 $y_l,y_r$，其中 $y_l<y_r$，概率 $p_{y_l}>0,p_{y_r}>0,p_{y_l}+p_{y_r}=1$，期望 $\hat{y}=y_lp_{y_l}+y_rp_{y_r}$，以绝对距离 $|y-\hat{y}|^\beta$ 作为调制因子
$$
GFL(p_{y_l},p_{y_r})=|y-(y_lp_{y_l}+y_rp_{y_r})|^\beta[-(y_r-y)\log(p_{y_l})-(y-y_l)\log(p_{y_r})]
$$
在训练过程中的损失函数
$$
L=\frac{1}{N_{pos}}\sum_zL_Q+\frac{1}{N_{pos}}\sum_z1_{\{c_z^*>0\}}(\lambda_0L_B+\lambda_1L_D)
$$
$N_{pos}$ 为正样本数，$1_{\{c_z^*>0\}}$ 保证只在正样本时计入 $L_B,L_D$ ，$L_B$ 为 GIoU
*QFL、DFL、GFL 来自文献《Generalized Focal Loss: Learning Qualified and Distributed Bounding Boxes for Dense Object Detection——2020》*
#### VFL（Varifocal Loss）
变焦距损失函数来源于 Focal Loss，用于训练密集目标检测器，同 QFL 类似使用置信度与定位精度的联合表示，并且修改了 FL 的超参数设置，取消了正样本的超参数，也就是仅降低负样本的权重，而尽可能保留珍贵稀少的正样本，FL 中正负样本处理是对称的，而 VFL 中进行了非对称加权
$$
VFL(p,q)=\begin{cases}
q(-q\log(p)-(1-q)\log(1-p))&q>0\\
-\alpha p^\gamma\log(1-p)&q=0
\end{cases}
$$
其中 p 是由 IACS（IoU-aware Classification Score）得到的定位分类联合分数（在原有置信度 p 的基础上引入边框定位精度），q 在前景点（正样本）时是两框之间的IoU，背景点时为0
*VFL 来自文献《VarifocalNet: An IoU-aware Dense Object Detector——2021》*
### 正负样本分配
[参考](https://zhuanlan.zhihu.com/p/633094573)
YOLOv5 中使用静态分配策略（训练开始前根据经验分配），而 YOLOv8 中采用动态分配策略，直接使用 TOOD 的对齐分配器（Task-Aligned Assigner）匹配策略，根据分类与回归的分数加权计算选择正样本
$$
t=s^\alpha\times u^\beta
$$
其中 s 是 Cls Score 分类预测分数，u 是 Reg Score（CIoU）位置预测分数，$\alpha,\beta$ 为超参数，t 越接近于 1 越匹配
一张图片共生成 $20\times20+40\times40+80\times80=8400$ 个 Anchors（不同尺寸的特征点），每个预测框包含 $4\times reg\_max+cls\_num$ 个参数，reg_max 是 DFL 中一般分布的跨度参数
1. 初筛正样本：预测框中心点在真实框内的正样本
2. 精选正样本：根据对齐分数 t 进一步筛选前 k 个正样本，其余为负样本
3. 过滤正样本：若一个预测点匹配到多个真实框，选取 CIoU 最大的作为匹配正样本
# YOLOv8-VOS
IEEE Access 开源 2024
*改进的 VanillaNet 主干 + 挤压激励（SE）注意力 + ODConv + WIoU损失函数*
## Wise IoU
[文献参考](https://zhuanlan.zhihu.com/p/690012294)
使用 Wise IoU 替换 YOLOv8 的损失函数，Wise IoU 具有动态非单调焦点机制（FM，Focal Mechanism），有效减少简单示例对损失值的贡献，而使模型更多关注困难的样本
引入“离群度”的概念，计算锚框的离群度，来动态调整梯度增益，离群度大的锚框（低质量锚框）获得较大的梯度增益，是模型在训练过程中能够更专注于普通质量的锚框，提高整体性能
通过构建梯度增益计算方法结合焦点机制，更好地优化了训练过程，增强模型在轻量化设备的适用性
## VanillaNet
解决 ResNet 中 shortcut 操作会额外占用大量内存的问题，提出 VanillaNet，极简的设计，大大提高了运算效率，但存在弱非线性的问题，提出改进
1. 深度训练策略：训练早期使用两个卷积层和一个激活函数，并将激活函数训练成恒等映射（identity mapping），将两个卷积合并为一个，提高模型的非线性性
2. 堆叠已知激活函数：浅层网络性能不佳主要是非线性性不足引起的，有两种方法解决，堆叠非线性激活函数或者增强单个激活层的非线性性，VanillaNet 通过加权堆叠改善每个激活层的非线性
直接使用 VanillaNet 堆叠来替代 YOLOv8 的 backbone 会影响深度特征的原始准确性，引入 ODConv 模块来深化 VanillaNet 深度，学习更深入的特征
## ODConv 全维动态卷积（Omni-Dimensional Dynamic Convolution）
传统卷积运算使用静态卷积核（独立于输入样本，全数据集通用），注意力机制主要包括通道注意力机制和空间注意力机制
动态卷积的核心思想是对多个卷积核并进行线性加权，而权重 $\alpha_{w1}$ 则与输入数据有关，使用策略 $\pi_{wi}(x)$ 计算
$$
y=(\alpha_{w1}W_1+\dots+\alpha_{wn}w_n)*x
$$
ODConv 将动态属性扩展到了空间维度、输入通道和输出通道，称为全维动态卷积
通过跨四个维度的并行策略，引入多维度注意力机制（通道、滤波器、空间、卷积核）
## SE Attention 挤压奖励注意力机制（Squeeze-and-Excitation Attention）
1. Squeeze：全局平均池化，将每个通道的特征值减少一个，组成一个通道数长度的全局向量
2. Excitation：通过两个全连接层、一个 ReLU 和一个 Sigmoid 函数激活
3. Scale：得到的通道注意力权重与原始输入特征图相乘加权
# YOLOv8+CBAM
CBAM 卷积块注意力模块（Convolutional Block Attention Module）
通道注意力机制+空间注意力机制加入到卷积层中