title: Mac/windows下nexus系列产品的刷机官方指南
categories: 
 - 综合
 - 其他
tags:
  - Mac
  - Nexus
date: 2016-07-30 15:26:44
author: mzeht
avatar: /images/favicon.png
---
#注意事项
1.仅针对官方镜像  
2.需要下载官方镜像，需要翻墙或者国内寻找搬运资源  
3.nexus刷入新系统后，开机需要网络验证激活（部分可以跳过），需要网络环境能够翻墙，请自行解决  

<!-- more -->

#镜像
1.前往[镜像官网](https://developers.google.com/android/nexus/images)  
2.![](http://7xqtsx.com1.z0.glb.clouddn.com/16-7-30/11576201.jpg)  
右侧选择对应的设备,同意相关协议，选择下载镜像  
![](http://7xqtsx.com1.z0.glb.clouddn.com/16-7-30/27674143.jpg)  
这里以6p为例子，镜像列表从上到下，从旧到新

#环境准备
1.fastboot工具  
 
To flash a device using one of the system images below (or one of your own), you need the latest fastboot tool. You can get it from one of the sources below.

* From a compiled version of the [Android Open Source Project](https://source.android.com/).  

* From the `platform-tools/` directory in the Android SDK. Be sure that you have the latest version of the `Android SDK Platform-tools` from the [SDK Manager](https://developer.android.com/studio/intro/update.html).  

Once you have the `fastboot tool`, add it to your `PATH` environment variable (the flash-all script below must be able to find it). Also be certain that you've set up USB access for your device, as described in the Using [Hardware Devices guide](https://developer.android.com/studio/run/device.html).  

Caution: Flashing a new system image deletes all user data. Be certain to first backup any personal data such as photos.

简单来说就是下载fastboot工具 并且添加到系统变量，注意用户数据会被清空

#刷入步骤  
1. Download the appropriate system image for your device below, then unzip it to a safe directory.  

2. Connect your device to your computer over USB.  

3. Start the device in fastboot mode with one of the following methods:
	* Using the `adb tool`: With the device powered on, execute:
	
	```
	adb reboot bootloader
	```


	* Using a key combo: Turn the device off, then turn it on and immediately hold down the relevant`key combination` for your device. For example, to put a Nexus 5 ("hammerhead") into fastboot mode, press and hold Volume Up + Volume Down + Power as the device begins booting up.  
	
4. If necessary, unlock the device's bootloader by running:

```
fastboot flashing unlock
```
or, for older devices, running:

```
fastboot oem unlock
```
The target device will show you a confirmation screen. (This erases all data on the target device.)  

  
5. Open a terminal and navigate to the unzipped system image directory.  

6. Execute the flash-all script. This script installs the necessary bootloader, baseband firmware(s), and operating system.  

Once the script finishes, your device reboots. You should now lock the bootloader for security:


1. Start the device in fastboot mode again, as described above.
2. Execute:
fastboot flashing lock
or, for older devices, running:
fastboot oem lock


Locking bootloader will wipe the data on some devices. After locking the bootloader, if you want to flash the device again, you must run `fastboot oem unlock again`, which will wipe the data.



#结语
偷懒，直接上官方指南了，一些细节问题，（比如怎么下载fastboot，怎么配置系统变量）,相信能看到这篇文章和有这个意向的人都能解决。  

通过此方法完美利用mac给nexus5手机和nexus7wifi刷好系统,前者网络验证可以跳过，后者不行，采用可以刷入开源固件的路由器，刷入openwrt，设置好socks代理（付费版），即可无线翻墙，顺利激活，windows上可以电脑翻墙后共享热点，期间还尝试了`Privoxy`将shadowsocks的socks5代理转为http代理,然后验证阶段，连接普通wifi,设备和mac处在同一局域网下，设置代理地址为mac上的socks代理，但是不是很理想，还是硬件翻墙来的粗暴

