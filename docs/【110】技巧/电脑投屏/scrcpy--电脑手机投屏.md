### [scrcpy--电脑手机投屏](https://www.jianshu.com/p/72e5804eadfe)
#### 使用要求
安卓设备版本5.0以上
安卓设备要打开adb调试模式
[scrcpy官网地址](https://links.jianshu.com/go?to=https%3A%2F%2Fgithub.com%2FGenymobile%2Fscrcpy)
### Windows安装

[scrcpy-win64-v1.10.zip](https://github.com/Genymobile/scrcpy/releases/download/v1.10/scrcpy-win64-v1.10.zip)
### 启动
#### window下：
* 将下载的文件解压后放到合适的文件目录下
* 打开cmd进入放置该目录文件夹下  `H:\Program Files (x86)\scrcpy-win64-v1.10`
* 依次输入命令 
#### usb模式(wifi模式可参考官网)
```
adb usb
scrcpy
```
可能存在错误信息，如下：
![image.png](https://upload-images.jianshu.io/upload_images/2981395-073ea1797213da51.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
#### 可以将scrcpy带上参数比如：
```
scrcpy -m1920
```
### 录屏
 ```
scrcpy --record file.mp4
scrcpy -r file.mkv
```