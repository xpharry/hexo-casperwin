---
title: RoverPi小车 第三篇：系统安装和网络设置
date: 2019-05-16 14:30:27
tags:
---

<img src="https://cdn.instructables.com/FPN/U31F/HBNXVG6G/FPNU31FHBNXVG6G.LARGE.jpg" alt="venv" style="height:300px;" align="middle">

<!--more-->

## 在Raspberry Pi上安装ROS

参考了下面这篇博文．

[INSTALL ROS AND OPENCV IN RASPBERRY PI(RASPBIAN STRETCH)](https://neverbenever.wordpress.com/2017/12/20/install-ros-and-opencv-in-raspberry-piraspbian-stretch/)

系统安装这方面不需要耗费太多精力，尤其有了一次经验以后，重复这样无聊和漫长的过程是实在是浪费生命．可以直接利用网上现成的安装好ROS的镜像，一次烧录到位．

这个链接有长期更新的镜像，值得拥有．

https://downloads.ubiquityrobotics.com/pi.html

## 网络设置

**设置首次启动自动连接WIFI**

我们可以创建一个特别的文件，用来在首次启动时连接wifi．更详细的介绍可以看[这个链接](https://raspberrypi.stackexchange.com/questions/10251/prepare-sd-card-for-wifi-on-headless-pi)，这里我们可以看一下步骤．


- 打开文本编辑器：在Linux上是 `gedit`．在Linux上是Notepad． 在Mac上是TextEdit．
- 粘贴并编辑下面的内容，以匹配你的WIFI:

```
country=US
ctrl_interface=DIR=/var/run/wpa_supplicant GROUP=netdev
update_config=1

network={
    ssid="<your network name>"
    psk="<your password>"
}
```

替换 `<your network name>` 为你的WIFI名称，并保留引号． 替换 `<your password>` 为你的WIFI登录密码，包含在引号里．如果你希望在这里的密码是被加密的状态，可以在你启动树莓派并登录以后[修改这个文件](https://unix.stackexchange.com/questions/278946/hiding-passwords-in-wpa-supplicant-conf-with-wpa-eap-and-mschap-v2)．


- 保存这个文件到boot分区的根目录下，文件名为`wpa_supplicant.conf`．第一次启动之后，这个文件会被转移到`/etc/wpa_supplicant/wpa_supplicant.conf`，之后可以在这里继续编辑．

激活[**SSH**](https://www.raspberrypi.org/documentation/remote-access/ssh/)

从2016年11月的版本开始，Raspbian默认把SSH关闭．可以通过如下步骤手动激活SSH：


1. 从`Preferences`菜单启动 `Raspberry Pi Configuration`
2. 导航到 `Interfaces` 条目
3. 选择 `SSH`旁边的`Enabled`
4. 点击 `OK`

## ROS Master 设置

**设置 ROS IP: 机器人小车端**

把下列变量输出到你的机器人小车的工作空间脚本文件．

```
> echo export ROS_MASTER_URI=http://localhost:11311 >> ~/.bashrc
> echo export ROS_HOSTNAME=IP_OF_RoverPi >> ~/.bashrc
```

**设置ROS IP：远程上位机**

把下列变量输出到你的上位机的工作空间脚本文件．注意master是机器人小车RoverPi.

```
> echo export ROS_MASTER_URI=http://IP_OF_RoverPi:11311 >> ~/.bashrc
> echo export ROS_HOSTNAME=IP_OF_PC >> ~/.bashrc
```

这个过程可以参考如下图所示的TurtleBot设置过程，只需要把TurtleBot换成RoverPi Car即可。

![](http://emanual.robotis.com/assets/images/platform/turtlebot3/software/network_configuration.png)

可以做一个简单的测试，看网络是否设置成功．

在机器人小车的一端先开启roscore再开启一个新窗口运行

```
rostopic pub -r10 /hello std_msgs/String "hello"
```

在上位机运行

```
rostopic echo /hello
```

这个信息＂hello＂将会在１秒内被打印10次．如果没有，检查你的ROS_HOSTNAME设置．

## 参考

- https://neverbenever.wordpress.com/2017/12/20/install-ros-and-opencv-in-raspberry-piraspbian-stretch/
- https://raspberrypi.stackexchange.com/questions/10251/prepare-sd-card-for-wifi-on-headless-pi
- https://downloads.ubiquityrobotics.com/pi.html
- http://wiki.ros.org/Robots/TurtleBot/Network%20Setup
