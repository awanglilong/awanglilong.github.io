[我的博客](https://awanglilong.github.io/)

## 移动端开发

### 编译原理

>简单来说分编译前端和编译后端，中间产物为IR。clang将OC语言编译为IR（swift使用的是swiftc），LLVM将IR编译为对应汇编。
>
>clang不会将OC转换为c++.

[iOS编译原理](https://awanglilong.github.io/2021/04/16/iOScompile/)简单介绍OC的编译步骤和原理

### 动态库和静态库

>动态库优点是包小，可共用，能动态加载。缺点是加载速度慢。但可共用和动态加载Apple不允许使用。所以优点是包小，缺点是加载速度慢。

[如何打造一个让人愉快的框架](https://onevcat.com/2016/01/create-framework/#cocoa-touch-framework) 介绍如何开发开源框架，介绍动态库和静态库。

[iOS 静态库和动态库](https://www.cnblogs.com/dins/p/ios-jing-tai-ku-he-dong-tai-ku.html) 动态库包小，静态库冷启动速度快

### iOS基础

> OC语言底层原理研究方式主要通过读源码、clang转换为c++、看内存地址三种方式研究底层实现原理

#### 语言基础

[OC语言（一）Runtime](https://awanglilong.github.io/2021/04/20/OCRuntime/)  OC的消息派发主要是通过有向图，通过图上搜索的方式，复杂度最高，灵活性最好。

[OC语言（一）Hook](https://awanglilong.github.io/2021/04/20/OCRuntime/)  OC的消息派发主要是通过有向图，通过图上搜索的方式，复杂度最高，灵活性最好。

[OC语言（二）Block的实现](https://blog.devtang.com/2013/07/28/a-look-inside-blocks/) 

[Swift（一）对象内存模型](https://mp.weixin.qq.com/s/zIkB9KnAt1YPWGOOwyqY3Q)  Swift的对象的内存模型也是结构体，复杂度比OC低 [Swift 中的类](https://www.jianshu.com/p/07f7523f2d6d)

[Swift（二）消息派发](https://awanglilong.github.io/2021/04/19/swift-message/)  结构体消息派发是直接派发，类的消息派发主要是虚派表

[Swift（三）协议的使用](https://onevcat.com/2016/11/pop-cocoa-1/#%E9%9D%A2%E5%90%91%E5%AF%B9%E8%B1%A1%E7%BC%96%E7%A8%8B%E7%9A%84%E5%9B%B0%E5%A2%83)使用Struct和Protocol的面向协议编程，是Swift的高效方式。

#### 多线程

[多线程（一）方案](https://wenghengcong.com/posts/6d5d08d4/)

[多线程（二）锁](https://wenghengcong.com/posts/b3d3fe6/)

[多线程（三）NSOperation](https://www.cnblogs.com/dins/p/nsoperationqueue.html)

#### 内存管理

[内存管理（一）引入](http://wenghengcong.com/posts/4dedf510/)

[内存管理（二）TaggedPointer](http://wenghengcong.com/posts/b6becb26/)

[内存管理（三）引用计数与weak](http://wenghengcong.com/posts/7162dd05/)

[内存管理（四）copy](http://wenghengcong.com/posts/bf6902cc/)

[内存管理（五）autorelease](http://wenghengcong.com/posts/c458827d/)

[内存管理（六）Swift弱引用](https://zhuanlan.zhihu.com/p/58179258) 

Swift的弱引用是SideTable管理，是个散列表，但是存的是指出的weak对象。

OC的弱引用是SideTable管理，是个全局三层散列表，weak_entry_t里存的是指向当前对象的，weak对象。

#### RunLoop

[RunLoop（一 ）对象](http://wenghengcong.com/posts/5c027118/)

[RunLoop（二）运行](http://wenghengcong.com/posts/ec1e2951/)

[RunLoop（三）应用](http://wenghengcong.com/posts/25ecb79e/)

### 源码解读

[AFNetworking源码解读](https://awanglilong.github.io/2021/04/21/AFNetworking-code-read/)

[Aspect](https://halfrost.com/ios_aspect/#toc-17)源码分析  实现有点像KVO的实现原理

[fishhook](https://juejin.cn/post/6844904175625568270)源码分析
[willChangeValueForKey：中做了什么](https://www.jianshu.com/p/90af8277958a)


[Analyze](https://github.com/draveness/analyze)iOS开源框架源码分析

[Laucp'sBlog](https://chipengliu.github.io/)源码分析

[WebViewJavascriptBridge原理解析](https://www.jianshu.com/p/d45ce14278c7)

[YYModel阅读摘要（一）基础](http://wenghengcong.com/posts/ec42f57/)

[YYModel阅读摘要（二）特性](http://wenghengcong.com/posts/d41ed060/)

[YYModel阅读摘要（三）参考](http://wenghengcong.com/posts/b9644035/)

[构建iOSModel层（一）最简单的实现Model解析](http://wenghengcong.com/posts/814d3fa9/)

[构建iOS-Model层（二）类型解析](http://wenghengcong.com/posts/e4c737c7/)

[构建iOS-Model层（三）嵌套解析](http://wenghengcong.com/posts/e4b6b7db/)

[JJException](https://github.com/jezzmemo/JJException)

[iOS Crash防护](https://juejin.cn/post/6874435201632583694)

### 面试问题

[2020年iOS面试题总结](https://www.xuebaonline.com/2020%E5%B9%B4iOS%E9%9D%A2%E8%AF%95%E9%A2%98%E6%80%BB%E7%BB%93(%E4%B8%80)/)

[2020年iOS面试题总结(二)](https://www.xuebaonline.com/2020%E5%B9%B4iOS%E9%9D%A2%E8%AF%95%E9%A2%98%E6%80%BB%E7%BB%93(%E4%BA%8C)/)

[2020年iOS面试题总结(三)](https://www.xuebaonline.com/2020%E5%B9%B4iOS%E9%9D%A2%E8%AF%95%E9%A2%98%E6%80%BB%E7%BB%93(%E4%B8%89)/)

[iOSInterviewQuestions](https://github.com/ChenYilong/iOSInterviewQuestions)

[iOS知识思维导图](https://github.com/MisterBooo/ReadyForBAT)

[出一套 iOS 高级面试题](https://juejin.cn/post/6844903645243260941#heading-2)

[iOS卡顿问题处理](https://blog.ibireme.com/2015/11/12/smooth_user_interfaces_for_ios/)

[iOS拾遗——为什么必须在主线程操作UI](https://juejin.cn/post/6844903763011076110)

[APP性能检测方案汇总](https://www.jianshu.com/p/95df83780c8f)

[iOS应用架构谈](https://casatwy.com/iosying-yong-jia-gou-tan-kai-pian.html)iOS架构师写的

[跳出面向对象思想(一)继承](https://casatwy.com/tiao-chu-mian-xiang-dui-xiang-si-xiang-yi-ji-cheng.html)但是一些基本体系的继承都在使用

[iOS沙盒机制](https://zhuyunsun.github.io/2020/04/21/iOS%E6%B2%99%E7%9B%92%E6%9C%BA%E5%88%B6/)

[iOS App瘦身实践](https://mp.weixin.qq.com/s/xzlFQJ2b-rrw5QIszSLXXQ)

## 算法

[fucking-algorithm](https://github.com/labuladong/fucking-algorithm)提出算法结题框架

[小浩算法](https://www.geekxh.com/)常见200算法题目解法

[五分钟学算法](https://www.cxyxiaowu.com/)算法动画做的最好的

## 设计模式

[设计模式SOLID原则](https://segmentfault.com/a/1190000023114300)

[图说设计模式](https://design-patterns.readthedocs.io/zh_CN/latest/)时序图对理解设计模式非常有帮助

## 计算机基础

[键入网址后，到网页显示](https://www.cnblogs.com/xiaolincoding/p/12508499.html)

[TCP 三次握手和四次挥手](https://www.cnblogs.com/xiaolincoding/p/12638546.html)

[TCP 重传、滑动窗口、流量控制、拥塞控制](https://www.cnblogs.com/xiaolincoding/p/12732052.html)

[HTTP 常见的面试题](https://www.cnblogs.com/xiaolincoding/p/12442435.html)



[计算机笔记](https://github.com/wolverinn/Waking-Up)

## 前端开发

[千古壹号](https://github.com/qianguyihao/Web) 最好的前端入门教程，最主要是成体系

[Flex布局教程：语法篇](http://www.ruanyifeng.com/blog/2015/07/flex-grammar.html) 弹性布局讲的最好的

[阿里的中后台UI组件库](https://github.com/ant-design/ant-design)源码、[Taro-UI](https://github.com/NervJS/taro-ui)源码、[iview](https://github.com/iview/iview)UI组件学习

[冴羽的博客](https://github.com/mqyqingfeng/Blog)

[KieSun](https://github.com/KieSun/Dream)

[前端面试题](https://github.com/haizlin/fe-interview)


## 后台开发

[advanced-java](https://github.com/doocs/advanced-java)分布式，高并发，微服务讲解最清晰的

[JavaGuide](https://github.com/Snailclimb/JavaGuide)后端知识将的更全面

[CS-Notes](https://github.com/CyC2018/CS-Notes)

[后端框架源码解析](https://github.com/doocs/source-code-hunter)

[awesome-architecture](https://github.com/toutiaoio/awesome-architecture)


## 知名博客

[冰霜之地](https://halfrost.com/)

[draveness](https://draveness.me/) 早期iOS,现大多是后台相关

[xuebaonline](https://www.xuebaonline.com)  swift文章比较多

[美团技术团队](https://tech.meituan.com/)

[ruanyifeng](http://www.ruanyifeng.com/) 前端大神

[Dirtmelon](https://dirtmelon.github.io/)

[wenghengcong](https://wenghengcong.com/)

[刘望舒](http://liuwangshu.cn/) 安卓的大神

[纯洁的微笑](http://www.ityouknow.com/)精通微服务架构,现在呵呵

## 论坛与工具

[优设](https://www.uisdc.com/)		[zool](https://www.zcool.com.cn/)		[花瓣](https://huaban.com/)     [薪酬可视化](https://duibiao.info/visualization)

[产品](http://www.woshipm.com/)		[pmcaff](https://www.pmcaff.com/)

[MacTalk](http://macshuo.com)

[yapi](https://github.com/YMFE/yapi)接口管理测试平台

