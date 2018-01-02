### Table of Contents
**[kiss_android_issue](#kiss_android_issue)**<br>
**[Android_Threading_Performance](#Android_Threading_Performance)**<br>
**[ubuntu16.04_无法启动模拟器](#ubuntu16.04_无法启动模拟器)**<br>

## kiss_android_issue

拥抱android各种技术，再来一个深深的kiss

## Android_Threading_Performance

[![alt text](https://vthumb.ykimg.com/0541040856CD0EEC6A0A490451CEE5A5)](http://player.youku.com/embed/XMTQ4MDU3Nzc3Mg==)

## ubuntu16.04_无法启动模拟器

升级到android8.1的sdk后，发现模拟器竟然无法启动了
解决方法如下：
```
用命令行启动avd发现是Could not launch './qemu/linux-x86_64/qemu-system-i386': No such file or directory错误

cd sdk-path/emulator/lib64/libstdc++
mv libstdc++.so.6 libstdc++.so.6.bak
ln -s /usr/lib64/libstdc++.so.6 sdk-path/emulator/lib64/libstdc++  


```
