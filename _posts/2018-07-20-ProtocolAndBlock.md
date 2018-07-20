---
layout:     post
title:      "协议与Block"
subtitle:   " \"Block是个好东西\""
date:       2018-07-20 14:00:00
author:     "wanglilong"
header-img: "img/post-bg-2015.jpg"
catalog: true
tags:
    - OC基础
---
# 协议与Block
## 简单的委托实现

   假如我们实现一个方法`getAllStudent`，但是在方法的网络请求部分要类`NetWorkTool`类里的方法`getAllStudentFromNetWork`实现.
   
   那么`getAllStudent`需要方法`getAllStudentFromNetWork`的返回数据。如果是同步网络请求，直接使用方法返回数据就好，如果是异步线程，就只能使用方法调用来返回数据来。
   类图结构如下图

![](/img/Post-Protocol-Block/协议与block1.png)

## 出现的问题
   如果是只有一个`Student`类中使用网络请求那没有什么问题。
   
   但是如果现在有`Teacher`类，`Animal`类...一堆的类中使用，如果使用一次，在接口里定义一次那也太烦人了。
   
## 协议
   第一种解决方法是用协议。
   
   既然要在多接口中定义方法，那么就抽出来好了。抽出一个接口文件进行单独定义。
   
   但是OC的接口文件是与实现文件一同创建的同名文件，并不支持多接口的。
   
   所以就定义了另外一个名字叫协议。
   
   但是我们需要知道哪些类里添加了协议，就有了实现协议。
   
   ![](/img/Post-Protocol-Block/协议与block3.png)
  
## Block
   虽然有了协议但是
   
   第一、协议有点难以理解，和接口容易混淆。
   
   第二、多定义一个协议文件，还要在需要的接口文件中实现协议。
   
   有没有更好的办法呢，于是`Block`就横空出世了。
   
   `Block`其实就是一个函数，只是一个没有名字的函数而已，就叫匿名函数。
   
   OC里函数里是不能定义函数的，但是可以有`Block`。
   
   于是我们在`getAllStudent`方法里定义一个函数（匿名函数`Block`），在`getAllStudentFromNetWork`函数中直接调用这个函数就可以了，因为匿名所以不需要在接口中定义了。
   
   问题完美解决了。内存什么的先不管了。
   
   ![](/img/Post-Protocol-Block/协议与block2.png)
   
   
   当然这是根据实际使用我自己的理解，没有在网上找到类似的说法。

   
     

