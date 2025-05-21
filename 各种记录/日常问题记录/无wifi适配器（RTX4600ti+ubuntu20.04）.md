[首次安装参考](https://blog.csdn.net/coco_1998_2/article/details/141208959?spm=1001.2101.3001.6650.2&utm_medium=distribute.pc_relevant.none-task-blog-2%7Edefault%7EYuanLiJiHua%7ECtr-2-141208959-blog-130720017.235%5Ev43%5Epc_blog_bottom_relevance_base1&depth_1-utm_source=distribute.pc_relevant.none-task-blog-2%7Edefault%7EYuanLiJiHua%7ECtr-2-141208959-blog-130720017.235%5Ev43%5Epc_blog_bottom_relevance_base1&utm_relevant_index=5)
[已成功安装，突然不能使用](https://blog.csdn.net/weixin_43293404/article/details/136356257)
执行以下代码报错
```shell
sudo modprobe 8852be
```
执行
```shell
cd rtl8852be
sudo make uninstall
sudo make clean 
make  
sudo make install
sudo modprobe 8852be
```