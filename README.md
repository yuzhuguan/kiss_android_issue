# kiss_android_issue
开发android过程中的各种小问题，解决方案记录

### ubuntu16.04 无法启动模拟器
升级到android8.1的sdk后，发现模拟器竟然无法启动了
解决方法如下：
```
用命令行启动avd发现是Could not launch './qemu/linux-x86_64/qemu-system-i386': No such file or directory错误

cd sdk-path/emulator/lib64/libstdc++
mv libstdc++.so.6 libstdc++.so.6.bak
ln -s /usr/lib64/libstdc++.so.6 sdk-path/emulator/lib64/libstdc++  


```
