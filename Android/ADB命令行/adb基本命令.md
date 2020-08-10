# ADB基本命令
基本语法
```
adb [-d|-e|-s <serialNumber>] <command>
/**
* -d:当前唯一的USB连接的Android设备为命令目标
* -e:当前唯一运行的模拟器为命令目标
* -s<serialNumber>:指定相应的serialNumer设备为命令目标
**/
```

1. 查看当前连接设备
```
    adb devices//查看当前设备
    adb -s 设备号 其他指令//指定设备执行任务
```
2. 查看adb版本
```
    adb version
```
3. 安装apk文件
```
    adb install xxx.apk //如果apk已经安装则无法再安装
    adb install -r xxx.apk //覆盖安装
```
4. 卸载APP
```
    adb uninstall com.xxx.app //卸载APP不保留数据
    abd uninstall -k com.xxx.app //卸载APP但保留数据
```
5. 传递文件
```
    adb push 文件名 手机SD卡路径 //传递文件到手机
    adb pull 文件路径 //从手机中下载数据
```
6. 查看手机所以安装的APP包名
```
    adb shell pm list packages [-f] [-d] [-e] [-s] [-3] [-i] [-u] [--user USER_ID] [FILTER]
    /**
    * -f:显示应用关联的apk文件
    * -d:只显示disabled的应用
    * -e:只显示enabled的应用
    * -s:只显示系统应用
    * -3:只显示第三方应用
    * -i:只显示应用installer
    * -u:包含已卸载的应用
    * <FILTER>:包名包含 <FILTER> 字符串
    **/
```
7. 启动Acitivy
```
    adb shell am start 包名/Activity完整路径
```
8. 发送广播
```
    adb shell am broadcast -a "broadcastactionfilter"
    //这类用法在测试的时候很实用，比如某个广播的场景很难制造，可以考虑通过这种方式来发送广播。
```
9. 启动服务
```
    adb shell am startservice 包名/Service完整路径
```
10. 屏幕截图
```
    adb shell screencap 截取的图片存放路径（xxx/xxx.png）
```
11. 录制视频
```
    adb shell screenrecord 录制的视频存放路径（xxx/xxx.mp4)
```

[其他命令参考](https://www.wanandroid.com/blog/show/2310)
