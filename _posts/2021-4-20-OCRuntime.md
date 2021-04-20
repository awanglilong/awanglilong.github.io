---
layout:     post
title:      "OC运行时机制"
date:       2021-4-20 12:13:00
author:     "wanglilong"
header-img: "img/post-bg-2015.jpg"
catalog: true
tags:

- iOS原理

---

## 一、内存结构

### objc_class

OC对象都会转为结构体。OC对象对应的结构体就是`objc_class`。

```c
struct objc_class {
    Class isa;
    Class superclass;
    cache_t cache;             // 方法缓存
    class_data_bits_t bits;    // 具体的类信息(指针)
};
```

### class_ro_t

`class_ro_t`存储了当前类在编译期就已经确定的`属性`、`方法`以及`遵循的协议`，里面是没有分类的方法的。那些运行时添加的方法将会存储在运行时生成的`class_rw_t`中。`ro`即表示`read only`，是无法进行修改的。

```c
struct class_ro_t {
    uint32_t flags;
    uint32_t instanceStart;
    uint32_t instanceSize; //对象占用内存空间
    const char * name; //类名
    method_list_t *baseMethodList; //方法
    protocol_list_t * baseProtocols; //接口
    property_list_t *baseProperties; //属性
    ivar_list_t * ivars; //成员变量
    const uint8_t * weakIvarLayout; // 弱指针
};
```

### class_rw_t

`ObjC` 类中的属性、方法还有遵循的协议等信息都保存在 `class_rw_t`中

```c
struct class_rw_t {
    uint32_t flags;
    uint32_t version;
    const class_ro_t *ro; //指向ro_t指针
    /*
     这三个都是二位数组，是可读可写的，包含了类的初始内容、分类的内容。
     methods中，存储 method_list_t ----> method_t
     二维数组，method_list_t --> method_t
     这三个二位数组中的数据有一部分是从class_ro_t中合并过来的。
    */
    method_array_t methods; //方法列表
    property_array_t properties; //属性列表
    protocol_array_t protocols; //协议列表
    Class firstSubclass;
    Class nextSiblingClass;
};
```

### method_t

与类和对象一样，方法在内存中也是一个结构体。

```c
struct method_t {
    SEL name;
    const char *types;
    IMP imp;
};
```

![objc-method-after-realize-class](https://img.draveness.me/2016-04-23-objc-method-after-realize-class.png)

### 其它参考

[深入解析 ObjC 中方法的结构](https://draveness.me/method-struct/)

[OC语言（一）对象内存分析](http://wenghengcong.com/posts/38431d60/)

[OC语言（二）对象的本质及分类](http://wenghengcong.com/posts/ec4474d1/)

## 二、继承的实现

> isa指针是为了查找，方法和类方法。superclass是为了实现继承。方法查找可看成是树上的查找（更准确的说是有向图）。

通过源码可以知道`isa`的`isa_t`类型的内部结构

```c
union isa_t {
    isa_t() { }
    isa_t(uintptr_t value) : bits(value) { }
    Class cls;
    uintptr_t bits;
};
```



下面这张图介绍了对象，类与元类之间的关系。

![img](https://img.draveness.me/2016-04-21-14611715787360.jpg)

### 消息发送机制

#### 一、消息发送

1、`objc_class`的cache中查找(二分查找)   

​      cache是个散列表，散列函数为`f(@selector()) = @selector() & _mask`，散列处理方式是--i；

2、`class_rw_t`的方法列表中查找，如果方法没排序遍历查找，如果排序则二分查找

3、从superclass的的cache中查找

4、从superclass的`rw_t`中查找

(查找结束后，将方法添加到缓存中)

#### 二、动态解析

1、是否已动态解析---->下个阶段

2、resolveInstanceMethod:

​      resolveClassMethod：

3、标注已动态解析

4、动态解析后，重走消息发送流程

#### 三、消息转发

1、forwardingTargetForSelector.  --------->消息转发另个对象

2、methodSignatureForSelector.  ----------->完备消息转发

3、doesNotRecognizeSelecter（抛出异常）



###运行时

[Runtime（一）Runtime简介](http://wenghengcong.com/posts/a182534/)

[Runtime（二）isa指针](http://wenghengcong.com/posts/1574014f/)  [从 NSObject 的初始化了解 isa](https://draveness.me/isa/)   [isa 和 Class](https://halfrost.com/objc_runtime_isa_class/#toc-13)

[Runtime（三）方法缓存](http://wenghengcong.com/posts/497dcda2/)

[Runtime（四）objc_msgSend](http://wenghengcong.com/posts/de99a8a4/)

[Runtime（五）类的判定](http://wenghengcong.com/posts/bb109840/)

