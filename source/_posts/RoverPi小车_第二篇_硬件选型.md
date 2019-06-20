---
title: RoverPi小车 第二篇：硬件选型和组装
date: 2019-05-14 12:00:27
tags:
---

<img src="https://www.dropbox.com/s/k6q54nuwmnef8q2/IMG_20190512_204935.jpg?raw=1" alt="venv" style="height:300px;" align="middle">

这一篇主要解决几个问题：

１．计算单元
２．运动单元驱动
３．供电
４．机械固定

也就是说，除了小车车身，Lidar和Web Camera全部是来自新的组件．其实，仅就这几样而言也说不上很满意，只是为了尽可能得把现有组件利用起来．

<!--more-->

- 车身就是普通的铝合金底盘和四个轮子，在某宝或者亚马逊逊上可以找到很多替代；
- Lidar看起来是个仿品，山寨的一代RPLidar，建议大家直接使用原版，毕竟有充分的官方软件支持。我后面会介绍到的SLAM程序也是参考自RPLidar ；
- Ｗeb Camera也可以用Raspberry Pi Camera替代.


题外话，原本的众筹项目简直就是诈骗项目在 Kickstarter 上被当成诈骗项目了。骗钱是实打实了，可以长达7个月没有更新。

## 概述

下表是我所使用的机械和硬件单元：

| 组件                                                                                   | 价格      | 来源                                                                                                                                                                                                                                                                                                                                                               |
| ------------------------------------------------------------------------------------ | ------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| 四轮小车底盘，包含２个直流电机带编码器，２个履带，４个车轮                                                        | $20（估计） | 既有<br>可替换为：任意机器人小车底盘，大多不超过$20．给一个例子：https://www.amazon.com/Rubber-Aluminum-Chassis-Graduation-Project/dp/B07B8Z7J8J/ref=sr_1_18?crid=338XA4RRM8NAW&keywords=robot+car+chassis&qid=1557703804&s=gateway&sprefix=robot+car+c%2Caps%2C198&sr=8-18                                                                                                                   |
| 2D Lidar                                                                             | $99（估计） | 既有，不知名<br>可替换为：[**RPLiDAR A1M8 360 Degree Laser Scanner Kit - 12M Range**](https://www.amazon.com/RPLiDAR-A1M8-Degree-Laser-Scanner/dp/B07H7X3SFF/ref=sr_1_3?keywords=rplidar&qid=1557703713&s=gateway&sr=8-3)**,** [$99.00](https://www.amazon.com/RPLiDAR-A1M8-Degree-Laser-Scanner/dp/B07H7X3SFF/ref=sr_1_3?keywords=rplidar&qid=1557703713&s=gateway&sr=8-3) |
| Web Camera                                                                           | $14（估计） | 既有，不知名<br>可替换为：Raspberry Pi Camera                                                                                                                                                                                                                                                                                                                               |
| 树莓派３B+                                                                               | $38.10  | https://www.amazon.com/gp/product/B07BDR5PDW/ref=ppx_yo_dt_b_asin_title_o04_s00?ie=UTF8&psc=1                                                                                                                                                                                                                                                                    |
| SanDisk Extreme 32GB microSDHC UHS-I Card with Adapter                               | $14.49  | https://www.amazon.com/gp/product/B01HU3Q6F2/ref=ppx_yo_dt_b_asin_title_o02_s00?ie=UTF8&psc=1                                                                                                                                                                                                                                                                    |
| L298N电机驱动板                                                                           | $6.99   | https://www.amazon.com/gp/product/B014KMHSW6/ref=ppx_yo_dt_b_asin_title_o00_s00?ie=UTF8&psc=1                                                                                                                                                                                                                                                                    |
| 电池盒                                                                                  | $4.99   | https://www.amazon.com/gp/product/B07DBM11LF/ref=ppx_yo_dt_b_asin_title_o01_s00?ie=UTF8&psc=1                                                                                                                                                                                                                                                                    |
| 树莓派供电模块Li电池                                                                          | $17.89  | https://www.amazon.com/gp/product/B06W9FWDSP/ref=ppx_yo_dt_b_asin_title_o04_s00?ie=UTF8&psc=1                                                                                                                                                                                                                                                                    |
| eBoot 180 Pieces Male Female Hex Brass Spacer Standoff Screw Nut Assortment Kit (M3) | $10.99  | https://www.amazon.com/gp/product/B0756CW6Y2/ref=ppx_yo_dt_b_asin_title_o07_s00?ie=UTF8&psc=1                                                                                                                                                                                                                                                                    |
| 树莓派扩展板                                                                               | $6.19   | https://www.amazon.com/gp/product/B074T7D1V5/ref=ppx_yo_dt_b_asin_title_o07_s00?ie=UTF8&psc=1                                                                                                                                                                                                                                                                    |
| JST PH 2.0MM连接线                                                                      | $7.69   | https://www.amazon.com/gp/product/B07FBDZHNX/ref=ppx_yo_dt_b_asin_title_o00_s00?ie=UTF8&psc=1                                                                                                                                                                                                                                                                    |
| 电路连接跳线                                                                               | $6.78   | https://www.amazon.com/gp/product/B01LZF1ZSZ/ref=ppx_yo_dt_b_asin_title_o08_s00?ie=UTF8&psc=1                                                                                                                                                                                                                                                                    |



## 计算单元

其实这一部分没有什么悬念，小车底盘的尺寸以及功能，用Raspberry Pi非常适合，这里选用最新的版本，Raspberry Pi 3 B+．如果还会做下一代的话，我想我会有兴趣尝试Nvidia Jetson Nano，并且尝试AI级别的运算．


![Raspberry Pi 3](https://www.raspberrypi.org/magpi/wp-content/uploads/2017/04/Raspberry-Pi-3.jpg)



## L298N电机驱动板


![L298N 电机驱动板](http://fritzing.org/media/fritzing-repo/projects/w/working-with-l298n-dc-motor-driver/images/Dual-H-Bridge-DC-Stepper-Motor-Drive-Controller-Board-Module-Arduino-L298N-HE2.jpg)


L298N驱动板算是一个比较常规的用于驱动两路直流电机的板子，与树莓派的配合使用也很常见．网上用很多教程，这里参考了这一篇博文：


https://www.electronicshub.org/raspberry-pi-l298n-interface-tutorial-control-dc-motor-l298n-raspberry-pi/



## 供电方案

自己设计的项目，坑很多，很多问题是一边想一边解决一边又遇到新的问题．

L298N本身需要6-12V的供电，我一开始就想到用一个装４个５号电池的电池盒为其供电．那么，从L298N还可以引出5V的电压，似乎正好可以解决树莓派的供电．但实验发现不可行，电机的供电是可以满足，但无法同时驱动树莓派甚至Lidar.

所以，特意搜寻了可以给树莓派供电又容易叠加安装固定的电池，最后选择了这款带扩展板的Li电池，令人满意．


![](https://paper-attachments.dropbox.com/s_F2F5BA21703E8CFE0F69C6C645220F346DE5C1561E56AE695BBFC0012CFC9BA8_1557718528861_718PGO1TDgL._SL1500_.jpg)



## 机械固定

原本CrazyPi的设计中，负责固定电路板的金属柱加入了磁性，为的是方便随时安装和拆卸，但对我现在的搭配，这个磁吸设计就是败笔．单单电池盒的重量，就使得磁力无法负荷基本的稳定要求，还经常连同连接的线一起车下来，非常恼人．

既然原来的CrazyPi电路板已经被我放弃，我就把它放到车身下面，用于固定金属柱．L298N也同样固定在车身下方，便于连接电机．


![](https://www.dropbox.com/s/k6q54nuwmnef8q2/IMG_20190512_204935.jpg?raw=1)


最后小车完成图如下：

![](https://paper-attachments.dropbox.com/s_F2F5BA21703E8CFE0F69C6C645220F346DE5C1561E56AE695BBFC0012CFC9BA8_1557718939652_IMG_1659.JPG)



## 后记

上图的设计应当说还是有一些待改进的地方甚至缺陷，有些坑还没有踩透。


- RoverPi小车电机驱动的测试一开始只做了悬空的测试，基本的向前向后和转向远动是听从指令的。一旦放到地面上，转向是死活也转不动。一方面是来自于车体重量大幅加重了，尤其是干电池组。另一方面，L298N是一个比较老的电机驱动板了，看起来需要更高电压和更大电池容量的组合。不同于两轮的差分运动(Differential Steer)，四轮的差分运动（英文叫Skid Steer）需要克服更大的滑动阻力。
- 另一个设计上的缺陷是，树莓派和电池组的叠加，完全挡住了2D Lidar的部分扫描范围。这个Lidar我没有官方的参数表，如果是比照RPLidar A1的话，它的扫描范围应该是360度和12米。现在生生被挡掉了大约45度角。
- 摄像头的摆放比起原来的设计是降级了，而且摆放位置也很尴尬。原本有两个舵机可以实现2个自由度的角度变换。现在直接固定在树莓派上，如果把Lidar的位置作为车头，摄像头放在后面就好比把眼睛放在后脑勺了。暂时没有找到合适的方案来解决机械上的限制。也许也可以考虑针对不同任务，改变小车前后的定义。
