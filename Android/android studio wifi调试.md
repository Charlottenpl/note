1. 首先添加adb环境变量
2. 刷新环境变量`source ~/.bash_profile`
3. 数据线链接手机
4. adb devices可以看到当前连接到电脑的设备
5. adb shell ifconfig wlan0 查看设备ip地址192.168.146.173（或者直接手机Wi-Fi，找到已连接的Wi-Fi点查看详情就可以看到ip地址）
6. adb tcpip 5555
7. adb connect 192.168.146.173
8. 链接成功，断开数据线即可