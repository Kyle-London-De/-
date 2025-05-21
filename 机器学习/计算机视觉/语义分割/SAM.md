[SAM官网](https://github.com/facebookresearch/segment-anything?tab=readme-ov-file)
[SAM2官网](https://github.com/facebookresearch/sam2)
[安装依赖](https://blog.csdn.net/yinweimumu/article/details/136432874)
创建conda环境（SAM：python>=3.8，SAM2：python>=3.10 pytorch>=2.5.1）
```shell
conda create -n seganything python=3.10
```
参考官网安装
模型是针对图像的分割，分割所有前景（和背景？），想要对目标进行分割需要prompt，提示点或提示区域（需要每次手动提示？）