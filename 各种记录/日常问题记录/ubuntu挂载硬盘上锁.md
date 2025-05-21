查看硬盘
```shell
sudo fdisk -l
```
取消磁盘锁
```shell
sudo hdparm -r0 /dev/xxx
```
**更有可能Windows使用后没有正常关闭**，切换Windows检查一下（Word、SolidWorks等）