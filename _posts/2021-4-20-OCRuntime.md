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

[](/img/post-OCRuntime/objc-method-after-realize-class.jpeg)



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

[](/img/post-OCRuntime/oc_object_map.jpeg)

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



`addMethod`直接在二维方法数组末尾添加一个数组

```c
static IMP  addMethod(Class cls, SEL name, IMP imp, const char *types, bool replace)
{
    IMP result = nil;
    method_t *m;
    if ((m = getMethodNoSuper_nolock(cls, name))) {
        // already exists
        if (!replace) {
            result = m->imp(false);
        } else {
            result = _method_setImplementation(cls, m, imp);
        }
    } else {
        // fixme optimize
        method_list_t *newlist;
        newlist = (method_list_t *)calloc(method_list_t::byteSize(method_t::bigSize, 1), 1);
        newlist->entsizeAndFlags = 
            (uint32_t)sizeof(struct method_t::big) | fixed_up_method_list;
        newlist->count = 1;
        auto &first = newlist->begin()->big();
        first.name = name;
        first.types = strdupIfMutable(types);
        first.imp = imp;
        addMethods_finish(cls, newlist);
        result = nil;
    }
    return result;
}
```



###运行时

[Runtime（一）Runtime简介](http://wenghengcong.com/posts/a182534/)

[Runtime（二）isa指针](http://wenghengcong.com/posts/1574014f/)  [从 NSObject 的初始化了解 isa](https://draveness.me/isa/)   [isa 和 Class](https://halfrost.com/objc_runtime_isa_class/#toc-13)

[Runtime（三）方法缓存](http://wenghengcong.com/posts/497dcda2/)

[Runtime（四）objc_msgSend](http://wenghengcong.com/posts/de99a8a4/)

[Runtime（五）类的判定](http://wenghengcong.com/posts/bb109840/)



## 三、分类

分类的内存结构大体如下

```c
truct category_t {
    const char *name; //分类名
    classref_t cls; //类
    struct method_list_t *instanceMethods;//实例方法列表
    struct method_list_t *classMethods; //类方法列表
    struct protocol_list_t *protocols; //协议列表
    struct property_list_t *instanceProperties; //属性列表
    struct property_list_t *_classProperties;
};
```

1、在运行时加载阶段，会依次加载类相应的分类。

2、分类的方法和实例变量合并到`class_rw_t`相应的二维数组中。

3、后合并的分类数据，放到原来数据的前面。



```c
static void
attachCategories(Class cls, const locstamped_category_t *cats_list, uint32_t cats_count,
                 int flags)
{
    /*
     * Only a few classes have more than 64 categories during launch.
     * This uses a little stack, and avoids malloc.
     *
     * Categories must be added in the proper order, which is back
     * to front. To do that with the chunking, we iterate cats_list
     * from front to back, build up the local buffers backwards,
     * and call attachLists on the chunks. attachLists prepends the
     * lists, so the final result is in the expected order.
     */
    constexpr uint32_t ATTACH_BUFSIZ = 64;
    method_list_t   *mlists[ATTACH_BUFSIZ];
    property_list_t *proplists[ATTACH_BUFSIZ];
    protocol_list_t *protolists[ATTACH_BUFSIZ];

    uint32_t mcount = 0;
    uint32_t propcount = 0;
    uint32_t protocount = 0;
    bool fromBundle = NO;
    bool isMeta = (flags & ATTACH_METACLASS);
    auto rwe = cls->data()->extAllocIfNeeded();

    for (uint32_t i = 0; i < cats_count; i++) {
        auto& entry = cats_list[i];
        method_list_t *mlist = entry.cat->methodsForMeta(isMeta);
        if (mlist) {
            if (mcount == ATTACH_BUFSIZ) {
                prepareMethodLists(cls, mlists, mcount, NO, fromBundle, __func__);
                rwe->methods.attachLists(mlists, mcount);
                mcount = 0;
            }
            mlists[ATTACH_BUFSIZ - ++mcount] = mlist;
            fromBundle |= entry.hi->isBundle();
        }

        property_list_t *proplist =
            entry.cat->propertiesForMeta(isMeta, entry.hi);
        if (proplist) {
            if (propcount == ATTACH_BUFSIZ) {
                rwe->properties.attachLists(proplists, propcount);
                propcount = 0;
            }
            proplists[ATTACH_BUFSIZ - ++propcount] = proplist;
        }

        protocol_list_t *protolist = entry.cat->protocolsForMeta(isMeta);
        if (protolist) {
            if (protocount == ATTACH_BUFSIZ) {
                rwe->protocols.attachLists(protolists, protocount);
                protocount = 0;
            }
            protolists[ATTACH_BUFSIZ - ++protocount] = protolist;
        }
    }

    if (mcount > 0) {
        prepareMethodLists(cls, mlists + ATTACH_BUFSIZ - mcount, mcount,
                           NO, fromBundle, __func__);
        rwe->methods.attachLists(mlists + ATTACH_BUFSIZ - mcount, mcount);
        if (flags & ATTACH_EXISTING) {
            flushCaches(cls, __func__, [](Class c){
                // constant caches have been dealt with in prepareMethodLists
                // if the class still is constant here, it's fine to keep
                return !c->cache.isConstantOptimizedCache();
            });
        }
    }
    rwe->properties.attachLists(proplists + ATTACH_BUFSIZ - propcount, propcount);
    rwe->protocols.attachLists(protolists + ATTACH_BUFSIZ - protocount, protocount);
}

```



[OC语言（三）Category](http://wenghengcong.com/posts/b8e84edc/)



## 四、关联对象

通过单例存储一个两层的Map。

第一层Map，类（实例）为Key，map为value。

第二层Map，关联对象为Key，关联的对象为value。

[OC语言（四）关联对象](http://wenghengcong.com/posts/5fe15e03/)



## 五、KVC和KVO

KVO通过动态子类的方式实现观察者模式。

当为类添加KVO时，动态为类对象创建一个子类。

覆盖kvo对象的set方法。

```c
-(void)_setValueAndNotify{
  willChangeValueForKey();
  setAge();
  didChangeValueForKey();
}
```

[OC语言（五）KVC与KVO](http://wenghengcong.com/posts/f4c075c4/)



## 六、load和initialize

<img src="http://blog-1251606168.file.myqcloud.com/blog_2018/2018-12-11-120613.png" style="zoom: 50%;" />

[OC语言（六）load和initialize](http://wenghengcong.com/posts/a69d9d1f/)

