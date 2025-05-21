```shell
conda config --set auto_activate_base false
```
由于conda base环境的自启动（终端前面有（base）字样），在使用python编辑时可能找到的是conda环境下的python，导致找不到python3-empy包等问题
可以取消conda环境的自启动，取消后仍然可以正常启动自己的venv
```shell
conda activate yolo11
```