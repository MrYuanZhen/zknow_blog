---
title: 探索智能家居实现系列(一)-初探ali-Haas100开发板
date: 2021/05/26
updated: 2021/05/26
comments: true
tags: 
- Iot
- Haas100
categories:
- 智能家居
- 边缘开发
---

# 系列文章介绍
&emsp;&emsp;目前新房子正在装修，有一些智能家居的打算，也了解一些比如说云起(lifesmart)、aqara、欧瑞博(ORVIBO)等解决方案，这些智能家居解决方案提供商的方案都非常好，但是有个问题，每一家厂商都在做自己的解决方案，正好我们准备使用Bose的一套家庭影院，这台家庭影院使用蓝牙协议，想要实现智能控制，遗憾的是每一家的解决方案都不能实现这个需求。于是诞生了一个想法，自己开发一套解决方案，如果能整合目前已有的标准协议(蓝牙，WIFI，Zigbee，MQTT)的设备到一个APP里面来统一操作，就很方便了。抱着学习的态度(是的，随时可能放弃)诞生了这一系列的文章，作者不保证能持续更新，哈哈哈，目前智能家居解决方案基本已经成熟了，此系列文章也就只是抱着学习，了解目前Iot智能解决方案的态度去做的。话不多说，接下来记录一下系列文档的开篇-初探ali-Haas100开发板。

---
# Haas 100开发板介绍
>* 以下介绍大部分参考阿里Haas100官方介绍

Haas100是由阿里云IoT团队推出的高性价比开发板，硬件配置非常适合智能IoT的应用，并且有40个GPIO引脚，可以接入各种外设，同时支持以下协议：
  
* Wi-Fi 802.11 a/b/g/n
* 支持2.4GHz和5GHz
* 支持20MHz和40MHz带宽
* 支持蓝牙5.0双模
* 支持A2DP V1.3/AVRCP V1.5/HFP V1.6
* 支持Wi-Fi和蓝牙共存

硬件上面采用了以下配置：
* 双核Cortex-M33，主频是300MHz
* Flash 16MB
* 内存 2.5M的SRAM 16MB的PSRAM

硬件则有以下内容：
* 1个USB 2.0
* 3个6Mbps UART
* 2个50Mbps SPI，可以支持LCD
* 2个 1.4Mbps I2C master
* 4-ch I2S/8-ch TDM
* 4个PWM

&emsp;&emsp;总之，以上配置是非常符合我们这种初学者的功能要求的，更多的配置和关于haas 100开发板的信息，大家可以去阿里云IoT官网了解，这里就不做过多介绍了；

# 先把灯点亮
&emsp;&emsp;拿到开发版后，板子上有6颗LED灯，右边一排从上到下的编号是0,1,2左边一排的编号是3,4,5。既然是入门嘛，首先尝试本地点亮几颗灯再说；
![avatar](https://zknow-1256858200.cos.ap-guangzhou.myqcloud.com/%E5%85%AC%E4%BC%97%E5%8F%B7-%E6%8E%A2%E7%B4%A2%E6%99%BA%E8%83%BD%E5%AE%B6%E5%B1%85%E7%B3%BB%E5%88%97/%E7%AC%AC%E4%B8%80%E7%AB%A0-led%E5%B1%95%E7%A4%BA.jpeg)

## 开发环境安装
>* Haas 100开发板使用了AliOS Things系统，官方给了很多种开发方式，包括命令行、Studio IDE开发、Keil MDK开发、IAR IDE开发等方式，本文中使用命令行的方式开发，想要详细了解的小伙伴可以查看官方AliOS Things 文档库：https://help.aliyun.com/document_detail/213703.html?spm=a2c4g.11186623.6.541.7b4932055GH27n

Mac环境命令行的AliOS-Things开发环境的搭建主要分为两部分：
* pip和git安装
* 基于pip安装aos-cube及相关的依赖包
```
# 安装pip
$ sudo easy_install pip

# 安装依赖库和aos-cube，步骤如下：
$ python -m pip install setuptools wheel aos-cube

# 未安装homebrew，需安装homebrew
$ /usr/bin/ruby -e "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/master/install)"

# 安装git
$ brew install git
```
安装完成后检查aos是否安装成功
```
$ aos --version
0.5.11
```
* 以上是Mac环境下AliOS-Things开发环境的搭建，如果遇到问题或者是其他环境的部署，可以参考AliOS-Things官方文档：https://help.aliyun.com/document_detail/161037.html?spm=a2c4g.11186623.6.545.19743fd8nmH5yR

## 点亮个灯
&emsp;&emsp;作为第一次使用，我们先验证一下开发环境是否可用，以及熟悉一下编译/烧录的过程;

### 下载AliOS-Things代码
```
gitlab仓库
git clone https://github.com/alibaba/AliOS-Things.git -b dev_3.1.0_haas

国内客户可以使用gitee仓库
git clone https://gitee.com/alios-things/AliOS-Things.git -b dev_3.1.0_haas

```

### 修改代码
修改application/example/helloworld_demo/appdemo.c文件中的代码如下：
```
/*
 * Copyright (C) 2015-2020 Alibaba Group Holding Limited
 */

#include <stdio.h>
#include <stdlib.h>
#include <aos/kernel.h>
#include "aos/init.h"
#include "board.h"
#include <k_api.h>
#include "led.h"       ##引用方法

int application_start(int argc, char *argv[])
{
    aos_msleep(1000);         ##此处延时1秒执行，避免引导时打开led的开关
    printf("open led 3,4,5\r\n");
    led_switch(3,LED_ON);     ##打开3号灯
    led_switch(4,LED_ON);     ##打开4号灯
    led_switch(5,LED_ON);     ##打开5号灯
}
```

### 执行编译
```
# 进入AliOS-Things顶级目录执行编译命令
$ aos make helloworld_demo@haas100 -c config
$ aos make
```

### 进行烧录
>* 烧录的方式有很多种，本文章使用命令行的方式进行烧录；

```
# 进入AliOS-Things顶级目录执行烧录命令
$ aos upload
```
执行命令后会提示选择所用的串口地址，选择相应的串口地址进行烧录；如果执行烧录过程中提示以下日志：
```
Please reboot the board manually
```
则需要按一下开发板上的复位按钮，继续进行烧录。
复位按键在这：
![avatar](https://zknow-1256858200.cos.ap-guangzhou.myqcloud.com/%E5%85%AC%E4%BC%97%E5%8F%B7-%E6%8E%A2%E7%B4%A2%E6%99%BA%E8%83%BD%E5%AE%B6%E5%B1%85%E7%B3%BB%E5%88%97/%E7%AC%AC%E4%B8%80%E5%BC%A0-%E5%A4%8D%E4%BD%8D%E6%8C%89%E9%94%AE.png)

当打印以下信息的时候，证明烧录完成：
```
Burn "[('/Users/zhen/ali_haas100/AliOS-Things/platform/mcu/haas1000/release/write_flash_gui/ota_bin/ota_rtos.bin', '0')]" success.
---host_os:OSX
[INFO]: Firmware upload succeed!
```

烧录完成后开发板会自动重启；

### 点亮成功
&emsp;&emsp;烧录完成后开发板自动重启，重启完成后可以看到3、4、5号灯成功点亮，0号灯为电源指示灯，无法熄灭；
![avatar](https://zknow-1256858200.cos.ap-guangzhou.myqcloud.com/%E5%85%AC%E4%BC%97%E5%8F%B7-%E6%8E%A2%E7%B4%A2%E6%99%BA%E8%83%BD%E5%AE%B6%E5%B1%85%E7%B3%BB%E5%88%97/%E7%AC%AC%E4%B8%80%E7%AB%A0-%E6%88%90%E5%8A%9F%E7%82%B9%E4%BA%AE.jpeg)

使用aos自带的命令，可以进入串口查看开发板输出，可以看到，已经成功输出我们在代码中打印的信息了；
```
#aos进入串口命令行
$ aos monitor /dev/cu.xxxxxxxxx 1500000
```
![avatar](https://zknow-1256858200.cos.ap-guangzhou.myqcloud.com/%E5%85%AC%E4%BC%97%E5%8F%B7-%E6%8E%A2%E7%B4%A2%E6%99%BA%E8%83%BD%E5%AE%B6%E5%B1%85%E7%B3%BB%E5%88%97/%E7%AC%AC%E4%B8%80%E7%AB%A0-%E4%B8%B2%E5%8F%A3%E8%BE%93%E5%87%BA.jpg)


# 总结
&emsp;&emsp;在初探ali-Haas100开发板的过程中不难看出来，这块Haas100开发板门槛还是不算特别高，加上aos-cube集成开发工具的支持，从编译到烧录都非常简单，如果使用Studio IDE开发则更加简单，使用Studio IDE动动鼠标就可以完成开发板的编译与烧录了；关于Haas100的更多用途，我尽量多保持以下三分钟热度，多更新几个系列文章，哈哈哈哈；

&emsp;&emsp; 美中不足的是，目前可能还在生态建立初期，除了官方文档和一些案例以外，无论google还是baidu都基本找不到其他文档了(也不排除我没找到的可能性)所以目前的社区生态可能还需要广大爱好者的支持；

&emsp;&emsp;可能会问我为啥不用小米的生态开发，emmm，因为小米目前不面向个人开发者，网上也买不到对应的生态开发板，所以选择了ali-Haas100先学习一下；

# 下期预告
&emsp;&emsp;在本期中初探了ali-Haas100开发板的基本编译/烧录的过程，在下一期中，将会将ali-Haas100开发板与阿里云IoT云服务连接，通过手机控制LED灯的开关；

# 参考链接
* HaaS100快速开始:https://help.aliyun.com/document_detail/184184.html?spm=a2c4g.11186623.6.701.66e73fd8UYfbIV
* Mac开发环境安装:https://help.aliyun.com/document_detail/161039.html?spm=a2c4g.11186623.6.547.29066e1dJGk50c
* AliOS Things 文档库:https://help.aliyun.com/document_detail/213703.html?spm=a2c4g.11186623.6.541.42383fd8f4qC9h
