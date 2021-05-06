---
layout:     post
title:      "组件化结构"
subtitle:   " \"组件化组件结构\""
date:       2020-08-06 12:13:00
author:     "wanglilong"
header-img: "img/post-bg-2015.jpg"
catalog: true
tags:
- iOS
---

# 组件化结构

项目总的架构图大体如下，但是组件分层并没有固定标准，图片是下面描述有不一致地方，但大体如此。

<img src="/img/post-component/iOS-component.png" style="zoom:70%;" />



## 一、基础层

主要负责丰富基础能力。

### 1、功能丰富的 Category 类型工具库

#### YYCategories

`YYCategories`中对`UIKit`, `Foundation`, `Quartz`中的常用类添加分类,里面还有好多实用的API来供我们项目开发使用。由于没有前缀，有的时候分不清是系统接口还是`YYCategories`的。

#### [JKCategories](https://github.com/shaojiankui/JKCategories)

`JKCategories`中对`UIKit`, `Foundation`, `QuartzCore`中的常用类添加分类。支持子包引入。比YYCategories的子类以及API更加丰富，但包也更大。其中包含了很多问题的通用的解决方案，有很高的参考价值。但框架不太稳定，部分方法存在崩溃问题。

#### ICCategoriesSDK（自定义）

由于在实际开发中仅仅习惯使用`YYCategories`和`JKCategories`中的部分功能，所以参照`YYCategories`设计自己的分类工具包。

### 2、键盘管理

#### IQKeyboardManager

在iOS开发中,经常会出现在`UITextField/UITextView`中输入东西的时候,弹起的键盘遮挡住了页面下面,很不方便,`IQKeyboardManager`就是解决这一棘手问题的.而且`IQKeyboardManager`使用简单,无需添加任何代码,也不需要特别的设置,上手很快.只需要pod一下,轻松解决问题.

### 3、路由跳转

#### ICRouterSDK（自定义）

路由框架。改造于`CTMediator`主要添加了协议。规范接口，并定义宏简化路由跳转。

#### ICProcessorSDK（自定义）

事件链框架，根据责任链模式实现。

#### ICURIParserSDK（自定义）

依赖于`ICProcessorSDK`,默认的事件梳理流程。

### 4、其它

#### FCUUID

`UUID`库作为旧的`UDID`和`identifierForVendor`的替代。这个库提供了获得具有不同持久性级别的统一惟一标识符的最简单的API。

可以检索为同一用户的所有设备创建的uuid，通过这种方式，只需一点服务器端帮助，就可以轻松地跨多个设备管理客户帐户。

#### FLAnimatedImage

`FLAnimatedImage`是一个高性能的iOS动画GIF引擎。同时播放多个gif，播放速度可比桌面浏览器。消除在第一次回放循环期间的延迟或阻塞。

#### SwipeBack

处理滑动返回问题。

#### TTTAttributedLabel

`TTTAttributedLabel` 是一个常用的富文本开源库，支持各种属性文本、数据探测器，链接等。

#### MBProgressHUD

旋转框架

## 二、数据层

主要负责网络请求以及图片等数据资源管理。

### 1、网络请求与解析

#### AFNetworking

整个`AFNetworking`框架的核心类是`AFURLSessionManager`,主要封装了系统类`NSURLSession`进行网络请求。

`AFURLRequestSerialization`主要功能是处理网络请求参数和请求头，做网络请求的前期处理。对应HTTP请求的Request，这种对应关系能方便了解HTTP协议的，快速理解使用框架。同时为了能创建不同类型的请求类型，使用了工厂模式。

`AFURLResponseSerialization`主要功能是处理网络请求响应，做网络请求返回数据处理。为了处理JSON，XML，List等不同的返回数据，框架使用了策略模式，方便设置切换不同的解析策略。

#### YYModel

`YYModel`是`YYKit`的高效组件之一，在实际场景中的非常实用，运用于项目中使用MVC或MVVM架构时，使用model做数据处理。

- 自动转换模型数据
- 自动检测数据安全性，避免carch
- 无需继承其他类，使用方便
- 适用model各种数据加载运用场景

#### ICNetworkingSDK（自定义）

`ICNetworkingSDK`是根据项目需求，在`AFNetworking`上封装的网络框架。

主要作用是网络请求的加密和签名，以及网络请求完成后的数据解密、数据解析以及转换为指定的数据模型。

#### ICDataLoaderKit（自定义）

`ICDataLoaderKit`将所有的网络请求以策略模式的形式，甩到壳工程里。是网络请求的数据中转中心。

#### SDWebImage

它支持从网络中下载且缓存图片，并设置图片到对应的`UIImageView`控件或者`UIButton`控件。在项目中使用`SDWebImage`来管理图片加载相关操作可以极大地提高开发效率，让我们更加专注于业务逻辑实现。

#### ICEncryptCacheSDK (自定义)

自定义的加解密以及数据存储框架。

### 2、资源管理

#### ICResourceLoaderKit

资源加载框架，主要处理组件库里的组件资源。包括文字、颜色、图片等。

#### ICColorSDK

颜色主题框架管理App的颜色主题，目前主要根据QMUIKit框架颜色模块实现。具体方案未定。

### 3、其它

#### MMKV

MMKV 是基于 mmap 内存映射的移动端通用 key-value 组件，底层序列化/反序列化使用 protobuf 实现，性能高，稳定性强。主要用于快速存储。

## 三、核心层

### 1、基类

#### ICVCKit（自定义）

主要作为所有模块的基类，处理页面跳转，滑动。

通过模板模式实现导航栏以及状态栏的设置，并处理滑动时的异常。

#### ICUIKit（自定义）

封装旋转框、弹窗、Toast的UI模块。

#### ICFoundationSDK（自定义）

主要封装加密、钥匙串处理等方法。

### 2、页面布局

#### IGListKit

`IGListKit` 是 `Instagram` 推出的新的 `UICollectionView` 框架，使用数据驱动，旨在创造一个更快更灵活的列表控件。

`IGListKit`更好的实现了复杂页面拆分，降低了复杂页面的开发难度。并通过IGListDiffKit优化了页面的刷新。

#### ICGListKit（自定义）

在IGListKit上进一步开发的框架，将IGListKit改造为全数据驱动框架。

并支持样式协议，支持根据后台样式数据调整页面结构。

#### ICConfigurableListKit（自定义）

在`ICGListKit`上封装的页面模板

#### ICPyramidSDK（自定义）

在`ICGListKit`以及`ICConfigurableListKit`上开发的框架，负责与后台页面配置服务的交互。根据后台配置调整TabBar。

### 3、webView

#### ICWebViewSDK（自定义）

浏览器框架。白名单、导航栏、返回、沉浸式分享和桥的抛出。

#### WebViewJavascriptBridge

webview桥组件

#### ICJsBridge（自定义）

桥框架依赖于`WebViewJavascriptBridge`,使用装饰者模式可以动态添加桥的处理。

### 4、其它

#### XHLaunchAd

开屏广告

#### JKCountDownButton

发送验证码读秒框架。

#### JXCategoryView、JXPagingView

分页导航框架。使用协议封装指示器逻辑，可以为所欲为的自定义指示器效果。提供更加全面丰富、高度自定义的效果。使用子类化管理cell样式，逻辑更清晰，扩展更简单。高度封装列表容器，使用便捷，完美支持列表的生命周期调用。

#### CYLTabBarController

一行代码实现 Lottie 动画TabBar，支持中间带+号的TabBar样式，自带红点角标，支持动态刷新。

#### TZImagePickerController

图片选择器框架

#### KMNavigationBarTransition

主要处理页面切换导航栏错位问题。一个用来统一管理导航栏转场以及当 push 或者 pop 的时候使动画效果更加顺滑的通用库，并且同时支持竖屏和横屏。你不用为这个库写一行代码，所有的改变都悄然发生。

## 四、业务层

### 1、核心业务

#### ICAccountSDK

注册登录

#### ICRealNameAuthSDK

实名认证

#### ICPrivacyPermissionsSDK

app启动隐私和权限弹窗

#### JDCityAuthorizationSDK

京东授权

#### SQMuseumSDK

展览馆

#### ICJDPay

京东支付封装

### 2、社会化分享

#### GSSocialSDK

改造于`SocialSDK`，由于`SocialSDK`不再更新维护。将`SocialSDK`依赖微信、qq、新浪的包改为pod依赖。并进行分包管理。

#### ICSocialSDK

对`GSSocialSDK`的简单封装。也是社会化分享的隔离层。





最后参考服务端架构图

客户端的架构与服务端越来越接近。

![web-component](/img/post-component/web-component.jpg)

