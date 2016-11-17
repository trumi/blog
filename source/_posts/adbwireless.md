---
title: Android无线调试
date: 2015-05-04 20:00:49
tags: [Android调试 , Android]
---
厌倦了连接数据线的真机调试吗？或者你的笔记本已经没有多余的usb接口留给调试用了，这该怎么办？下面介绍一种炫酷而又便捷的连接方式，无线调试

### 准备工作
* [adbWireless app](http://down11.zol.com.cn/suyan/adbWireless1.5.4.apk)
* 无线路由器一台
* root过的手机

<!--more-->

### 开始
* 首先在手机上记得打开usb调试，相信大家都会。将手机与调试的电脑连入同一无线路由器
* 安装好 adbWireless后，保证授予其root权限，否则无法使用。完成授权后，打开app，点击正中间的按钮，按钮变绿后，可以在按钮下方看到手机的ip地址
* 操作调试的电脑，Windows系统用户用cmd命令行进入adb工具的目录，一般为sdk目录/platform-tools，Linux用户直接打开终端。输入
```
adb connect [手机的IP地址]
```

![connect](/images/adbwirelessPictures/connect.png)

* 输入命令
{% codeblock %}
adb devices
{% endcodeblock %}

![devices](/images/adbwirelessPictures/devices.png)

查看设备列表里是否有自己的设备，如有，恭喜你，手机已通过无线接入。

### 优缺点分析
##### 优点
* 抛弃数据线，连接更加优雅
* 解放宝贵的笔记本IO接口
* 可以将调试版本安装在任何连接在同一路由器的手机下，范围大至整个办公室，团队调试的福音

##### 缺点
* 比有线连接费电
* 必须使用root过后的手机
* 传输速度不高，大约700 kb/s，体积较大的app不适合采用此方案
