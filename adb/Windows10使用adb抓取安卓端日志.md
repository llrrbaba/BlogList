### Windows10使用adb抓取安卓端日志

#### 一.电脑端准备

##### 1.1 安装adb

按照链接进行下载安装即可

https://www.charlesproxy.com/assets/release/4.5.4/charles-proxy-4.5.4-win64.msi

##### 1.2 配置环境变量

安装完第一步下载的文件后，默认会存储在这里

![image.png](https://i.loli.net/2019/11/12/lkmiLyseoVrM3x6.png)

将这个路径配置到环境变量中

![image.png](https://i.loli.net/2019/11/12/LtY4NplWJDgdckf.png)

##### 1.3 确认安装&环境变量配置均成功

![image.png](https://i.loli.net/2019/11/12/wYbNxmfTgPXRjzq.png)

#### 二.手机端准备(因为我是小米手机MIUI11，就直说小米了，别的手机自行参考)

##### 2.1 打开开发者模式

具体路径就是：设置--我的设备--全部参数--MIUI版本，多点几次MIUI版本，直到提示“已进入开发模式”类似描述

##### 2.2 USB设置

具体路径就是：设置--更多设置--开发者选项--USB调试勾选--USB安装勾选--USB调试(安全设置)勾选--启用MIUI优化取消勾选

#### 三.连接

##### 3.1 确保手机和电脑使用同一个无线，并用USB连接上电脑

##### 3.2 adb查看设备

![image.png](https://i.loli.net/2019/11/12/KVJqecrvbTyu76B.png)

显示为device的，即为已经连接的设备

##### 3.3 查看日志

因为Windows不支持grep这种管道操作，如果想要根据关键字查看日志的话，可以使用以下两种方法：

> adb -s 65d27f5a logcat | findstr vote/xxx

或者

> adb -s 65d27f5a shell "logcat | grep vote/xxx"
