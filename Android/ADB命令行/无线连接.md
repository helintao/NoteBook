环境：android设备与电脑在同一个局域网

步骤：

1. Android设备与电脑USB连接，连接成功后，让设备在5555端口监听TCP/IP连接：
```
adb tcpip 5555
``` 
2. 端口USB连接后，开始无线连接
```
adb connect Android设备的IP地址和端口
```

