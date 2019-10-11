---
layout:     post
title:      "MVC、MVVM模式"
subtitle:   " \"设计模式让开发更简单\""
date:       2019-10-11 11:27:00
author:     "wanglilong"
header-img: "img/post-bg-2015.jpg"
catalog: true
tags:
    - iOS 开发
---
# MVC、MVVM模式



经典的MVC模式如下图：

![](/img/post-mvc-mvvm/MVC架构.png)



### 在苹果的设想里MVC

View：是xib或者storyboard。负责整个页面上的view（button，lable）以及其布局。

Controller：负责处理点击事件，view页面动态管理，处理数据，网络请求。业务逻辑。

Model：数据模型

优点很明显：层级简单，分工明确，开发快捷，便于掌握。



### 纯代码实现MC

然而我们对xib和storyboard深恶痛绝，其git管理麻烦，不方便代码复用。最重要的不方便在view上使用各种设计模式，阻碍我们进步。所以我们当然选择纯代码了。

然后我们将页面布局写在controller中



分层如下：

Controller：负责处理点击事件，view页面动态管理。 负责整个页面上的view（button，lable）以及其布局。处理数据，网络请求。业务逻辑。

Model：数据模型



这样事件少传一层，写起来太方便了，「大笑」

<img src="/img/post-mvc-mvvm/MV架构.png" style="zoom:50%;" />



也样就形成了著名的Massive View Controller。

代码类似

```objective-c
@interface HBDownFileViewController ()
@property (nonatomic,strong)  UIView *headerView;
@property (nonatomic,strong)  UIButton *openFileButton;
@end

@implementation HBDownFileViewController

- (void)viewDidLoad {
    [super viewDidLoad];
    [self setUpUI];
}
-(void)setUpUI{
    [self.view addSubview:self.headerView];
    [self.view addSubview:self.fileView];
    [self.fileView addSubview:self.openFileButton];

    [self.headerView mas_makeConstraints:^(MASConstraintMaker *make) {
        make.left.right.top.mas_equalTo(self.view);
        make.height.mas_equalTo(60);
    }];
    
    [self.openFileButton mas_makeConstraints:^(MASConstraintMaker *make) {
        make.left.mas_equalTo(self.headerView).offset(16);
        make.right.mas_equalTo(self.headerView).offset(-16);
        make.bottom.mas_equalTo(self.headerView.mas_bottom).offset(-32);
        make.height.mas_equalTo(40);
    }];
}

#pragma mark - set&get
- (UIView *)headerView
{
    if (!_headerView) {
        _headerView = [[UIView alloc]init];
    }
    return _headerView;
}

- (UIButton *)openFileButton
{
    if (!_openFileButton) {
        _openFileButton = [[UIButton alloc]init];
        _openFileButton.titleLabel.font = [UIFont systemFontOfSize:14];
    }
    return _openFileButton;
}
@end

```



### 纯代码的MVC

既然使用xib和storyboard可以实现MVC，那么纯代码当然也可以。我们只需要一个view类来管理每个页面的布局以及简单的页面逻辑。



View：纯代码UIView。负责整个页面上的view（button，lable）的设置以及其布局。

Controller：负责处理点击事件，view页面动态管理。处理数据，网络请求

Model：数据模型



### MVC进阶

方法一、Model多做一些

Controller处理数据，网络请求。还要处理页面逻辑。Controller还是很庞大啊。

我们可以让Model来处理数据，网络请求。这样能减轻一下Controller的负担。



View：纯代码UIView。负责整个页面上的view（button，lable）的设置以及其布局。

Controller：负责处理点击事件，view页面动态管理。业务逻辑。

Model：数据模型。处理数据，网络请求。



这基本上能满足大多数开发情景了。

真的有少数Controller非常大，各种设计模式完全可以用起来。

然鹅，精通设计模式的人，实在太少了，我们还是需要简单粗暴的办法(更死的划分、限制)



方法二、MVP

Controller毕竟还负担着业务逻辑。还是抽出来好了。于是MVP横空出世。

本来名字应该叫MVPC，但是View和Controller都可以看做View。



View：纯代码UIView。负责整个页面上的view（button，lable）的设置以及其布局。

Controller：负责处理点击事件，view页面动态管理。

Presenter：业务逻辑。

Model：数据模型。处理数据，网络请求。



能不能直接合并View和Controller呢，感觉合并后好像MVC只是职测有点差别而已。



<img src="/img/post-mvc-mvvm/MVP架构.png" style="zoom:50%;" />

方法三、MVVM

毕竟Model还是要保持纯洁的。

Presenter应该是和ViewModel合并来减少层级，还是保持拆分保持单一职责。那就需要根据实际情况来权衡了。

如果再合并View和Controller呢，那还是回到了MVC。



View：纯代码UIView。负责整个页面上的view（button，lable）的设置以及其布局。

Controller：负责处理点击事件，view页面动态管理。

Presenter：业务逻辑。

ViewModel：处理数据（负责view的数据处理），网络请求。

Model：数据模型。

<img src="/img/post-mvc-mvvm/MVVM架构png.png" style="zoom:50%;" />

### 总结

对于MVVM和MVP来说，如果是为了解决View与Controller合并带来的Massive View Controller问题，那么他们也只是一个MVC模式。

相当于让View添加了一些职责（如果将处理点击事件，view页面动态管理）。改变一下名字「手动狗头」。



如果View和Controller 保持拆分状态，就需要权衡层级增加带来数据传递带来的成本增加、分层带来的好处。



组件化和MVC这些不在一个层级上。

### 参考

1、 [iOS 架构模式--解密 MVC，MVP，MVVM以及VIPER架构](https://www.cnblogs.com/oc-bowen/p/6255475.html)

2、[iOS架构入门 - MVC模式实例演示](https://www.jianshu.com/p/309f0477aac1)  不谋而合

3、[关于 MVC 的一个常见的误用](https://onevcat.com/2018/05/mvc-wrong-use/)

4、[iOS应用架构谈 view层的组织和调用方案](https://casatwy.com/iosying-yong-jia-gou-tan-viewceng-de-zu-zhi-he-diao-yong-fang-an.html)  

​     一位能把简单的东西讲的复杂的大神，VIPER的解读应该错了。

#### 

