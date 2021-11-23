---
title: 探索智能家居实现系列(二)-ali-Haas100对接阿里云生活物联网平台
date: 2021/07/06
updated: 2021/07/06
comments: true
tags: 
- Iot
- Haas100
categories:
- 智能家居
- 边缘开发
---

# 背景介绍
&emsp;&emsp;时隔几个月，系列文章第二篇姗姗来迟，在上一期文章中，已经初步探索了Haas100开发版的一些功能，并且介绍了基本的编译烧录方法，在本期文章中，我们将会将Haas100开发版发不到阿里云生活物联网平台，并通过一个公用APP，实现远程APP控制小灯开关的一个小功能；

# 实现步骤概述

* 云端配置，此步骤中需要配置阿里云生活物联网产品的配置；
* 编辑代码，编辑APP控制小灯的逻辑代码，然后编译/烧录到Haas100开发版；
* Haas100开发版联网，连接到阿里云生活物联网平台；
* 下载APP，控制小灯；

# 云端配置
> 此步骤中，我们将在阿里云生活物联网平台创建项目，并获取三元组信息，以供Haas100开发版连接；

&emsp;&emsp;阿里云生活物联网地址：https://living.aliyun.com/ 未注册阿里云账户的用户，需要先完成账户注册。


## 创建项目

1、首先需要新建一个自由品牌项目，天猫精灵生态项目需要公司认证；

![](https://zknow-1256858200.cos.ap-guangzhou.myqcloud.com/%E5%85%AC%E4%BC%97%E5%8F%B7-%E6%8E%A2%E7%B4%A2%E6%99%BA%E8%83%BD%E5%AE%B6%E5%B1%85%E7%B3%BB%E5%88%97/Haas100_2/01CreateProject.jpg)

2、项目创建完成后，需要新建一个产品；
![](https://zknow-1256858200.cos.ap-guangzhou.myqcloud.com/%E5%85%AC%E4%BC%97%E5%8F%B7-%E6%8E%A2%E7%B4%A2%E6%99%BA%E8%83%BD%E5%AE%B6%E5%B1%85%E7%B3%BB%E5%88%97/Haas100_2/02CreateProduct.jpg)

3、输入产品信息，此处我们以小灯为例创建，需要以下信息：

* 产品名称：Haas100_LED
* 所属品类：电工照明/灯
* 节点类型：设备
* 是否接入网关：否
* 联网方式：WIFI(此处后面设备会使用WIFI联网)
* 数据格式：ICA标准数据格式
* 使用 ID² 认证：否(ID²为设备上云提供了双向身份认证能力，建立轻量化的安全链路（iTLS）来保障数据的安全性,需要单独购买)

创建完成后即可进入到详细产品信息配置页面

![](https://zknow-1256858200.cos.ap-guangzhou.myqcloud.com/%E5%85%AC%E4%BC%97%E5%8F%B7-%E6%8E%A2%E7%B4%A2%E6%99%BA%E8%83%BD%E5%AE%B6%E5%B1%85%E7%B3%BB%E5%88%97/Haas100_2/03CreateProductInfo.jpg)

## 配置产品详情
> 此处是配置产品的详细信息，包括功能定义、人机交互、设备调试、批量投产几个模块；

### 功能定义
&emsp;&emsp;默认创建好的电工照明/灯品类会有很多的标准功能定义，在本文章中，由于使用的是默认的LED小灯，所以像灯模式、色温灯功能都不需要，因此全部删除，只保留开关功能；

![](https://zknow-1256858200.cos.ap-guangzhou.myqcloud.com/%E5%85%AC%E4%BC%97%E5%8F%B7-%E6%8E%A2%E7%B4%A2%E6%99%BA%E8%83%BD%E5%AE%B6%E5%B1%85%E7%B3%BB%E5%88%97/Haas100_2/04DeleteFunction.jpg)

### 人机交互
> 此部分有很多内容，主要是定义人机交互的内容，其中完善左侧惊叹号提示的配置设置，对于已经有默认数值的按默认确认即可。

1、产品展示
> 此部分配置APP展示的产品名称
![](https://zknow-1256858200.cos.ap-guangzhou.myqcloud.com/%E5%85%AC%E4%BC%97%E5%8F%B7-%E6%8E%A2%E7%B4%A2%E6%99%BA%E8%83%BD%E5%AE%B6%E5%B1%85%E7%B3%BB%E5%88%97/Haas100_2/05FunctionInfo.jpg)

2、设备面板
> 此部分配置APP的控制面板，注意：此处默认的面板没有需求本文章中的模版，因此在本文章中会新建一个自定义面板

![](https://zknow-1256858200.cos.ap-guangzhou.myqcloud.com/%E5%85%AC%E4%BC%97%E5%8F%B7-%E6%8E%A2%E7%B4%A2%E6%99%BA%E8%83%BD%E5%AE%B6%E5%B1%85%E7%B3%BB%E5%88%97/Haas100_2/06CreatePanel.jpg)

![](https://zknow-1256858200.cos.ap-guangzhou.myqcloud.com/%E5%85%AC%E4%BC%97%E5%8F%B7-%E6%8E%A2%E7%B4%A2%E6%99%BA%E8%83%BD%E5%AE%B6%E5%B1%85%E7%B3%BB%E5%88%97/Haas100_2/07CustomPanel.jpg)

* 保存自定义后选择自定义的面板；

![](https://zknow-1256858200.cos.ap-guangzhou.myqcloud.com/%E5%85%AC%E4%BC%97%E5%8F%B7-%E6%8E%A2%E7%B4%A2%E6%99%BA%E8%83%BD%E5%AE%B6%E5%B1%85%E7%B3%BB%E5%88%97/Haas100_2/08CreateCustomPanel.jpg)

3、配网引导
> 此步骤中设置设备的配网发现方式；注意：此步骤决定APP是否能够发现设备；默认的配网方式为标准配网，此处我们需要修改为自定义配网；

* 修改自定义配网

![](https://zknow-1256858200.cos.ap-guangzhou.myqcloud.com/%E5%85%AC%E4%BC%97%E5%8F%B7-%E6%8E%A2%E7%B4%A2%E6%99%BA%E8%83%BD%E5%AE%B6%E5%B1%85%E7%B3%BB%E5%88%97/Haas100_2/09Network.jpg)


* 选择一键配网方式

![](https://zknow-1256858200.cos.ap-guangzhou.myqcloud.com/%E5%85%AC%E4%BC%97%E5%8F%B7-%E6%8E%A2%E7%B4%A2%E6%99%BA%E8%83%BD%E5%AE%B6%E5%B1%85%E7%B3%BB%E5%88%97/Haas100_2/10NetworkConfig.jpg)

4、多语言管理
> 此步骤中配置多语言环境，本文章中默认就好，保存后感叹号就会消失

![](https://zknow-1256858200.cos.ap-guangzhou.myqcloud.com/%E5%85%AC%E4%BC%97%E5%8F%B7-%E6%8E%A2%E7%B4%A2%E6%99%BA%E8%83%BD%E5%AE%B6%E5%B1%85%E7%B3%BB%E5%88%97/Haas100_2/11LanguageConfig.jpg)

### 设备调试
> 此部分内容中，创建设备信息；

1、选择未认证设备

![](https://zknow-1256858200.cos.ap-guangzhou.myqcloud.com/%E5%85%AC%E4%BC%97%E5%8F%B7-%E6%8E%A2%E7%B4%A2%E6%99%BA%E8%83%BD%E5%AE%B6%E5%B1%85%E7%B3%BB%E5%88%97/Haas100_2/12DeviceSelect.jpg)

2、新增测试设备
* 在此步骤中，会生成四元组信息，需要记录下来ProductKey、DeviceName、DeviceSecret、Product Secret(在页面右上方)

![](https://zknow-1256858200.cos.ap-guangzhou.myqcloud.com/%E5%85%AC%E4%BC%97%E5%8F%B7-%E6%8E%A2%E7%B4%A2%E6%99%BA%E8%83%BD%E5%AE%B6%E5%B1%85%E7%B3%BB%E5%88%97/Haas100_2/13CreateDevice.jpg)

![](https://zknow-1256858200.cos.ap-guangzhou.myqcloud.com/%E5%85%AC%E4%BC%97%E5%8F%B7-%E6%8E%A2%E7%B4%A2%E6%99%BA%E8%83%BD%E5%AE%B6%E5%B1%85%E7%B3%BB%E5%88%97/Haas100_2/14DeviceInfo.jpg)

![](https://zknow-1256858200.cos.ap-guangzhou.myqcloud.com/%E5%85%AC%E4%BC%97%E5%8F%B7-%E6%8E%A2%E7%B4%A2%E6%99%BA%E8%83%BD%E5%AE%B6%E5%B1%85%E7%B3%BB%E5%88%97/Haas100_2/15ProductSecret.jpg)

### 批量投产
> 此部分为发布产品和下载APP的相关信息

1、发布产品

![avatar](https://zknow-1256858200.cos.ap-guangzhou.myqcloud.com/%E5%85%AC%E4%BC%97%E5%8F%B7-%E6%8E%A2%E7%B4%A2%E6%99%BA%E8%83%BD%E5%AE%B6%E5%B1%85%E7%B3%BB%E5%88%97/Haas100_2/16Release.jpg)

2、下载APP

![avatar](https://zknow-1256858200.cos.ap-guangzhou.myqcloud.com/%E5%85%AC%E4%BC%97%E5%8F%B7-%E6%8E%A2%E7%B4%A2%E6%99%BA%E8%83%BD%E5%AE%B6%E5%B1%85%E7%B3%BB%E5%88%97/Haas100_2/17DowAPP.jpg)

## 云端配置完成

&emsp;&emsp; 截止到本部分，云端配置已经全部完成，接下来需要进行设备端配置，包括代码修改、编译、烧录部分；

---

# 设备端配置
> 此部分包括代码开发、编译烧录的内容；

## 代码开发

修改application/example/linkit_demo/linkkit_example_solo.c代码

1、添加四元组信息，此步骤在云端配置的设备调试步骤中可以获取，四元组包括ProductKey、DeviceName、DeviceSecret、Product Secret

```C
// 四元组信息
#define PRODUCT_KEY      "a1zvLWQJXKy"
#define PRODUCT_SECRET   "AkzPfL15TfEL5VsD"
#define DEVICE_NAME      "Haas100"
#define DEVICE_SECRET    "6bf4e9d44c394a5f72eb36763a8295c5"
```

2、增加LED控制部分代码
```C
//增加头文件定义

#include "led.h"
```

```C
//修改user_property_set_event_handler()函数

/** recv event post response message from cloud **/
static int user_property_set_event_handler(const int devid, const char *request, const int request_len)
{
    cJSON *root = NULL, *LightSwitch_Value = NULL;
    root = cJSON_Parse(request);

    LightSwitch_Value = cJSON_GetObjectItem(root, "powerstate");  //开关属性
    if (LightSwitch_Value != NULL)
    {
    
        if(LightSwitch_Value->valueint)
        {
    
            printf("\r\n Turn on power \r\n"); 
            led_switch(1, LED_ON);
        }
        else
        {
    
            printf("\r\n Turn off power \r\n"); 
            led_switch(1, LED_OFF);
        }
    }    

    cJSON_Delete(root);
    return 0;
}
```

## 编译烧录
&emsp;&emsp;详细编译烧录信息请查看历史文章《探索智能家居实现系列(一)-初探ali-Haas100开发板》

```shell
//进入`AliOS-Things`顶层目录
cd AliOS-Things

//配置
aos make linkkit_demo@haas100 -c config	

//编译
aos make

//烧录
aos upload linkkit_demo@haas100

```

编译成功截图
![](https://zknow-1256858200.cos.ap-guangzhou.myqcloud.com/%E5%85%AC%E4%BC%97%E5%8F%B7-%E6%8E%A2%E7%B4%A2%E6%99%BA%E8%83%BD%E5%AE%B6%E5%B1%85%E7%B3%BB%E5%88%97/Haas100_2/18BuildSee.jpg)

烧录成功截图
![](https://zknow-1256858200.cos.ap-guangzhou.myqcloud.com/%E5%85%AC%E4%BC%97%E5%8F%B7-%E6%8E%A2%E7%B4%A2%E6%99%BA%E8%83%BD%E5%AE%B6%E5%B1%85%E7%B3%BB%E5%88%97/Haas100_2/19UploadSee.jpg)

## 设备端开发完成
&emsp;&emsp;至此，烧录完成后设备会自动重启，代码开发部分完成，接下来需要将设备配网；

# 设备端配网
> 此部分配置设备联网，将设备连接至互联网；

## 配置WIFI网络
1、进入串口终端，此步骤以MAC OS为例：
```shell

// 连接串口设备
aos monitor /dev/cu.usbserial-144310 1500000

// 回车显示#号，注意：此步骤可能终端一直在刷一些信息，可以直接输入命令后回车

// 打开连网成功后会自动保存AP信息的功能
netmgr -t wifi -a 1

//连接网络，ssid和pass根据实际WIFI信息填写
netmgr -t wifi -c {ssid} {password}
```

![](https://zknow-1256858200.cos.ap-guangzhou.myqcloud.com/%E5%85%AC%E4%BC%97%E5%8F%B7-%E6%8E%A2%E7%B4%A2%E6%99%BA%E8%83%BD%E5%AE%B6%E5%B1%85%E7%B3%BB%E5%88%97/Haas100_2/20netmgr_shell01.jpg)

终端打印以下信息证明配网成功
```
9018/wifi_conn | connect to ssid: 2L_NomoreSuper_WIFI
9018/wifi_conn | bwifi_add_config ssid:2L_NomoreSuper_WIFI, passwd:xxxxxxx, hidden:0
9019/wifi_conn | _bwifi_connect
9019/wifi_conn | [WIFI] change status:IDLE->IDLE
9019/wifi_conn | [WIFI] change status:IDLE->CONNECTING
9019/wifi_conn | wpa_supplicant_ssid_bss_match 861.
9019/wifi_conn | wpas_select_network_from_last_scan 1720.
9147/temp_main | ######### switch channle ,nw last temp= 657,  0x1700
9499/wifi_conn | bwifi send event, id:1
9499/wifi_even | [WIFI] status CONNECTING, event 1
9499/wifi_even | wifi connecting in state 0
9499/wifi_even | on_wifi_connect event_id:0
```

![](https://zknow-1256858200.cos.ap-guangzhou.myqcloud.com/%E5%85%AC%E4%BC%97%E5%8F%B7-%E6%8E%A2%E7%B4%A2%E6%99%BA%E8%83%BD%E5%AE%B6%E5%B1%85%E7%B3%BB%E5%88%97/Haas100_2/21WIFI_connect.jpg)

终端打印以下信息说明连接云端成功
```

[Jan 01 00:00:10.611]<I>MQTT Downstream Payload:
1;33m
{
"code":200, 
"data":{ 
 },
 "id":"0", 
"message":"success", 
"method":"thing.deviceinfo.update", 
"version":"1.0" 
}
```

云端显示以下信息说明设备连接成功

![](https://zknow-1256858200.cos.ap-guangzhou.myqcloud.com/%E5%85%AC%E4%BC%97%E5%8F%B7-%E6%8E%A2%E7%B4%A2%E6%99%BA%E8%83%BD%E5%AE%B6%E5%B1%85%E7%B3%BB%E5%88%97/Haas100_2/22connectsee.jpg)

## 连接网络完成
&emsp;&emsp;至此，我们的Haas100开发版已经成功连接阿里云生活物联网平台，接下来，我们将进行在线测试和APP测试控制；


# 测试
> 此部分分为两个部分，阿里云生活物联网平台提供了在线测试设备功能，我们将在线测试功能是否可用和测试手机APP端的远程控制功能；

## 在线测试
进入设备详情，选择在线调试；

![](https://zknow-1256858200.cos.ap-guangzhou.myqcloud.com/%E5%85%AC%E4%BC%97%E5%8F%B7-%E6%8E%A2%E7%B4%A2%E6%99%BA%E8%83%BD%E5%AE%B6%E5%B1%85%E7%B3%BB%E5%88%97/Haas100_2/23DeviceOnDebug.jpg)


```
//输入调试命令：
{
  "powerstate": 1
}
```
预期结果：

```
//终端日志打印

[Jan 01 00:08:52.957]<I>MQTT Downstream Payload:

␛[1;33m
{
"method":"thing.service.property.set", 
"id":"1774327511", 
"params":{ 
   "powerstate":1 
},
"version":"1.0.0" 
}
```

1号灯成功点亮

![](https://zknow-1256858200.cos.ap-guangzhou.myqcloud.com/%E5%85%AC%E4%BC%97%E5%8F%B7-%E6%8E%A2%E7%B4%A2%E6%99%BA%E8%83%BD%E5%AE%B6%E5%B1%85%E7%B3%BB%E5%88%97/Haas100_2/31LED_on.jpg)

## APP测试
下载并安装APP(下载二维码在阿里云生活物联网平台批量生产页面)

1、下载完成后注册账号登录，选择添加设备即可自动发现设备；

![](https://zknow-1256858200.cos.ap-guangzhou.myqcloud.com/%E5%85%AC%E4%BC%97%E5%8F%B7-%E6%8E%A2%E7%B4%A2%E6%99%BA%E8%83%BD%E5%AE%B6%E5%B1%85%E7%B3%BB%E5%88%97/Haas100_2/29AppUI02.jpg)

2、添加完成后即可进入之前的自定义控制面板操作小灯的开关；

![](https://zknow-1256858200.cos.ap-guangzhou.myqcloud.com/%E5%85%AC%E4%BC%97%E5%8F%B7-%E6%8E%A2%E7%B4%A2%E6%99%BA%E8%83%BD%E5%AE%B6%E5%B1%85%E7%B3%BB%E5%88%97/Haas100_2/28AppUI01.jpg)

预期结果
```
//终端日志打印
[Jan 01 00:15:24.249]<I>MQTT Downstream Payload:

␛[1;33m
"method":"thing.service.property.set", 
"id":"684794469", 
"params":{ 
   "powerstate":1 
},
"version":"1.0.0" 
}

```

手机上操作开关，小灯可以自由控制
![](https://zknow-1256858200.cos.ap-guangzhou.myqcloud.com/%E5%85%AC%E4%BC%97%E5%8F%B7-%E6%8E%A2%E7%B4%A2%E6%99%BA%E8%83%BD%E5%AE%B6%E5%B1%85%E7%B3%BB%E5%88%97/Haas100_2/29LED_off.jpg)

![](https://zknow-1256858200.cos.ap-guangzhou.myqcloud.com/%E5%85%AC%E4%BC%97%E5%8F%B7-%E6%8E%A2%E7%B4%A2%E6%99%BA%E8%83%BD%E5%AE%B6%E5%B1%85%E7%B3%BB%E5%88%97/Haas100_2/31LED_on.jpg)



# 总结
&emsp;&emsp; 经过此文章的演示，已经可以成功的在APP上实现远程控制小灯的开关了，当然这个只是初步的尝试，距离真正的连接其他设备实现智能家居还早呢，通过阿里云生活物联网平台的加持，整个步骤还是非常简单方便的。

# 下期预告
&emsp;&emsp; 虽然不知道下期是什么时候，但是下期应该会有一些GPIO的控制场景，比如说对于普通家用灯泡，是不是可以通过GPIO控制继电器的通断来进行控制。或者是连接蓝牙协议的一些设备，控制蓝牙设备，可能会在这两个主题其中选择一个吧。



# 参考链接

* 基于HaaS 100搭建智能家居应用 https://help.aliyun.com/document_detail/184190.html?spm=a2c4g.11186623.6.726.64f580dd0q9O1S#G6aFP


