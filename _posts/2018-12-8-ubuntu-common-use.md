---
layout:     post
title:      "ubuntu上常用软件"
subtitle:   " \"记录软件安装\""
date:       2018-12-08 15:53:00
author:     "wanglilong"
header-img: "img/post-bg-2015.jpg"
catalog: true
tags:
    - Linux
---
# ubuntu上常用软件
>安装的是简化版的`ubuntu`，只安装了必备的软件。 <br />
>美化软件`gnome-tweak-tool`，但由于对计算机的性能有一定影响，决定弃用，简简单单挺好。 <br />
>输入法之前安装过搜狗,但是需要安装一堆东西，没必要，系统的输入法足够用了。 <br />
>音乐播放器安传网易音乐，但是不用电脑播放音乐，不需要安装。 <br />

## 一、多媒体播放器
### VLC
世界上用户数目第三的的多媒体播放软件，据说解析度很好。安装看视频教程。
```shell
sudo apt-get install vlc
```
## 二、浏览器
### chrome
谷歌的浏览器，喜欢它简洁的页面。但是特别吃内存。

## 三、文本编辑器
### vim
vim 被誉为上古神器，可以看得出 vim 在编辑器的地位是很高的。几乎没有vim不能查看的文件，看各种代码都方便。
```shell
sudo apt-get install vim
```
### Atom
`Atom` 是`github`专门为程序员推出的一个跨平台文本编辑器。软件简洁，代码显示效果漂亮，但是和`chrome`一样超级吃内存。可以编写md文件。

[Atom官方](https://atom.io/)上下载，直接双击安装。
### Typora
`Typora`是一款轻便简洁的`Markdown`编辑器，支持即时渲染技术，这也是与其它`Markdown`编辑器最显著的区别。即时渲染使得你写Markdown就想是写Word文档一样流畅自如，不像其他编辑器的有编辑栏和显示栏。
不过更喜欢用Atom写md。
``` shell
wget -qO - https://typora.io/linux/public-key.asc | sudo apt-key add -
sudo add-apt-repository 'deb https://typora.io/linux ./'
sudo apt-get update
sudo apt-get install typora
```
## 四、代码管理
### git
图形软件需用了gitg，主要是免费
``` shell
sudo apt-get install git
```
