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

就像一个公司规模大到一定程度，就会组建集团公司，将公司拆分成多个小公司。

系统的复杂性超过一定程度后，就自然而然的的将系统拆分成小的子系统。

Java选择微服务组建模块化，而iOS相应的采用的是组件化。

微服务需要满足高内聚，低耦合和单一职责设计原则。

尽量隔离各个组件间的联系，还需要其能沟通自如.所以需要设计模块间通讯的功能我们一般叫它路由。

## 路由技术

### 蘑菇街方案

说到路由，很自然会想到java微服务中的路由。其结构如下

![](/img/post-ios-module/java微服务.png)

根据URL进行服务注册、服务发现。


这些思想都可以，应用到iOS的组件化中。

根据这个思想，实现的微服务，其核心代码可以简化如下

```
//Mediator.m 中间件
@implementation Mediator
typedef void (^componentBlock) (id param);
@property (nonatomic, storng) NSMutableDictionary *cache
- (void)registerURLPattern:(NSString *)urlPattern toHandler:(componentBlock)blk {
 [cache setObject:blk forKey:urlPattern];
}

- (void)openURL:(NSString *)url withParam:(id)param {
 componentBlock blk = [cache objectForKey:url];
 if (blk) blk(param);
}
@end

```

>其中核心思想：
>
>根据URL，将block存入一个多层的字典中实现服务注册。
>
>根据多层字典查找block实现服务发现。



这种方案最大的优点是，简单易于理解，对初学者最友好。而且对远程调用，和本地调用做了统一化处理。

缺点的话，[Casa已经做了全面化说明](https://casatwy.com/iOS-Modulization.html)

但是在大多数情况下，它的缺点可以忽略。但是如果路由是重度使用，就无法忍受了。


### casatwy组件化方案

casatwy的路由方案，核心代码量也很小，只有200多行代码。短小精悍，代码也很容易看懂。

其核心就一个函数

```
- (id)performTarget:(NSString *)targetName action:(NSString *)actionName params:(NSDictionary *)params
{
    
    // Class
    NSString *targetClassString =  [NSString stringWithFormat:@"Target_%@", targetName];
    Class targetClass = NSClassFromString(targetClassString);
    NSObject *target = [[targetClass alloc] init];

    // SEL
    NSString *actionString = [NSString stringWithFormat:@"Action_%@:", actionName];
    SEL action = NSSelectorFromString(actionString);
    
    return [target performSelector:action withObject:params];

}
```

```
@implementation Target_AClasss
- (UIViewController *)Action_nativeFetchViewController:(NSDictionary *)params
{
    DemoModuleADetailViewController *viewController = [[DemoModuleADetailViewController alloc] init];
    viewController.valueLabel.text = params[@"key"];
    return viewController;
}
@end
```
Target（class）+Action（function）实现服务注册

根据设置类名称和方法名称调用performTarget（runtime）实现服务发现

我们最后的路由实现为了方便管理添加了协议。


### 总结

从代码来看，这些代码其实很相似。都是实现了一个服务注册，和服务发现功能。

其差别其实非常细微。是一些设计思想的上差别，这个就很绕了。如果一开始就纠结于此，会很难搞懂路由的终极目的。

入门还是看bang神的总结[博客](http://blog.cnbang.net/tech/3080/)比较好

如果一开始就看casatwy的[iOS 组件化方案探索](http://blog.cnbang.net/tech/3080/)。很容易晕。


## 方案
1、URL作为跳转路径的

[蘑菇街 App 的组件化之路（MGJRouter）](https://limboy.me/tech/2016/03/10/mgj-components.html)

2、直接指定方法名（运行时target、action方案）

[iOS应用架构谈 组件化方案](https://casatwy.com/iOS-Modulization.html)

3、两种方案的对比(这一遍讲的最明白)

[iOS 组件化方案探索](http://blog.cnbang.net/tech/3080/)
