---
layout:     post
title:      "Ubuntu上安装AndroidStudio"
subtitle:   " \"好难安装\""
date:       2018-12-8 18:06:00
author:     "wanglilong"
header-img: "img/post-bg-2015.jpg"
catalog: true
tags:
    - Android
---
# Ubuntu上安装AndroidStudio
## 一、下载安装
去[安卓开发者中心](https://developer.android.google.cn/)或[安卓论坛](http://www.android-studio.org/)下载`AndroidStudio`。

## 二、启动Studio
### 跳过启动页
在安装目录下的bin目录下的`idea.properties`文件里添加
```vim
disable.android.first.run=true
```
### 在Studio里下载SDK
只要下载最基本的就可以，这个过程会很慢。尽量选择急需的
![](/img/post-ubuntu-android/sdkplatforms.png)
![](/img/post-ubuntu-android/sdktool.png)

## 三、配置gradle
比较恶心的是gradle老是下载失败，所以选用本地的
![](/img/post-ubuntu-android/gradle.png)

## 四、在虚拟机上运行
### 开启Intel虚拟机运行

创建虚拟机后，运行时出现错误对话框，错误内容如下：
```vim
KVM is required to run this AVD.
/dev/kvm is not found.
Enable VT-x in your BIOS security settings,
 ensure that your Linux distro has working KVM module.
```
解决方法：
重启电脑进入系统固件设置（BIOS）界面，使用左右光标键移动至至“Security”页，用上下光标键移动至“Virtualization”项，按Enter键，再用上下光标键移动至“Intel (R) Virtualization Technology”项，按Enter键，选择“Enabled”选项，按F10键保存退出，重启操作系统，问题解决。

## 五、在真机上运行
### 手机设置
1. 手机打开开发者选项
2. 允许USB调试
3. 连接了手机选择进行”文件管理“

### 无法发现设备
#### 1. 看usb连接列表
使用命令
```shell
lsusb
```
如果是华为手机能直接看到如下样子的设备
```vim
Bus 001 Device 006: ID 12d1:107e Huawei Technologies Co., Ltd.
```
如果是小米的需要通过拔插数据线，对比usb列表

#### 2. 编辑usb识别规则文件
android.rules文件名字前缀随意
```shell
sudo gedit /etc/udev/rules.d/70-android.rules
```
#### 3. 添加语句
指定usb可以识别
```vim
SUBSYSTEM=="usb",ATTR{idVendor}=="12d1",ATTRS{idProduct}=="107e",MODE="0666"
SUBSYSTEM=="usb", ENV{DEVTYPE}=="usb_device", MODE="0666"
```
#### 4. 文件提权
```shell
sudo chmod a+rx /etc/udev/rules.d/70-android.rules
```
#### 5. 重启设备管理器
```shell
sudo /etc/init.d/udev restart
```
#### 6. 重启server
```shell
sudo ./adb kill-server
sudo ./adb start-server
```
#### 7.看连接列表
```shell
sudo ./adb devices
```
