---
layout:     post
title:      "iOS路由技术"
subtitle:   " \"到底应该使用什么路由方案呢\""
date:       2019-09-30 11:00:00
author:     "wanglilong"
header-img: "img/post-bg-2015.jpg"
catalog: true
tags:
    - iOS 开发
---


# iOS路由技术

## 目的

就像一个公司规模大到一定程度，就会组建集团公司，将公司拆分成多个子公司。

系统的复杂性超过一定程度后，将大的系统拆分成小的子系统是一个自然而然的选择。

Java服务器选择的是微服务组建模块化。而iOS相应的采用的是组件化。

微服务需要满足高内聚，低耦合和单一职责设计原则。

尽量隔离各个组件间的联系，但还需要其能沟通自如.所以需要设计模块间通讯的功能我们一般叫它路由。

## 方案
主要有两种方案


## 蘑菇街方案
蘑菇街采用 注册表的方式，用URL表示接口，在模块启动时注册模块提供的接口

```
//Mediator.m 中间件
typedef void (^componentBlock) (id param);
@property (nonatomic, storng) NSMutableDictionary *cache

@implementation Mediator

- (void)registerURLPattern:(NSString *)urlPattern toHandler:(componentBlock)blk {
 [cache setObject:blk forKey:urlPattern];
}

- (void)openURL:(NSString *)url withParam:(id)param {
 componentBlock blk = [cache objectForKey:url];
 if (blk) blk(param);
}
@end
```


## 方案
1、URL作为跳转路径的

[蘑菇街 App 的组件化之路（MGJRouter）](https://limboy.me/tech/2016/03/10/mgj-components.html)

2、直接指定方法名（运行时target、action方案）

[iOS应用架构谈 组件化方案](https://casatwy.com/iOS-Modulization.html)

3、两种方案的对比(这一遍讲的最明白)

[iOS 组件化方案探索](http://blog.cnbang.net/tech/3080/)
