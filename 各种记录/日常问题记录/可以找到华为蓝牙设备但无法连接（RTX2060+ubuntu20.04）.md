解决方法：
启动模块
```Linux
sudo apt install pulseaudio-module-bluetooth
pulseaudio -k
pulseaudio --start # 一定要启动这个
sudo pactl load-module module-bluetooth-discover # 这一行命令好像不成功也不影响
```
连接蓝牙（蓝牙地址在设置中可以直接查看）
```Linux
bluetoothctl    # 打开蓝牙的管理程序 
scan on
trust XX:XX:XX:XX:XX:XX    # 询问（yes/no）时，要输入完整的yes
pair XX:XX:XX:XX:XX:XX
connect XX:XX:XX:XX:XX:XX
```
