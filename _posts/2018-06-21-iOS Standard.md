---
layout:     post
title:      "iOS开发规范"
subtitle:   " \"良好的开发习惯，让代码容易阅读\""
date:       2018-06-21 14:00:00
author:     "wanglilong"
header-img: "img/post-bg-2015.jpg"
catalog: true
tags:
    - iOS 开发
---
# iOS开发规范
## 目的
为了利于项目维护以及规范开发，促进成员之间Code Review的效率，故提出以下开发规范，如有更好的建议，欢迎提出。

本文档的预期读者包括：iOS开发人员。
## 命名规范
>代码中的命名严禁使用拼音与英文混合的方式,更不允许直接使用中文的方式。正确的英文拼写和语法可以让阅读者易于理解,避免歧义。
>
>注意：即使纯拼音命名方式也要避免采用。但alibab、taobao、youku、hangzhou等国际通用的名称，可视同英文.

	大驼峰规则：每个单词的首字母大写。例：NameTextField。
	小驼峰原则：第一个单词首字母小写，其余都大写。例：nameTextField。
	
#### 项目命名
项目名都遵循大驼峰命名。例如：AoRiseProject。

#### 类名
类的命名都遵循大驼峰命名。一般是：前缀 + 功能 + 类型。例如：`MW + Login + ViewController`。

| 控件名 | 类型 | 示例 |
| :-: | :-: | :-: |
| UIViewController | ViewController | MWBaseViewController |
| UView | View | MWBaseView |
| UITableView | TableView | MWOrderTableView |
| UITableViewCell | Cell | MWOrderListCell |
| UIButton | Button | MWSuccessButton |
| UILabel | Label| MWSuccessLabel |
|UIImageView|ImgView|MWGoodsImgView|
|UITextField |TextField| MWNameTextField |
|UITextView |TextView |MWSuggestTextView |

其它类相关对照表

| 功能 | 类型| 示例 |
| :-: | :---: | :-: | 
|工具类 |Tool |MWOrderTool |
|代理类 |Delegate |MWOrderListDelegate |
|管理类 |Manager | MWOrderListModel |
|模型类 |Model| MWOrderListModel |
|Service类 |Service |MWOrderService |
|布局类 |Layout| MWHomeLayout |
|数据库类 |DataBase、表名+DBHelper| MWFriendDataBase、MWUserTableDBHelper|
|类目|XXX+（范围，例如Extension， Additions 或者功能，例如Frame，Nib，Block）|MWUIButton+Additions、MWUIButton+Block|

#### UIViewController书写规范

```objc
#pragma mark - life cycle
#pragma mark - event response
#pragma mark - UITableViewDelegate && UITableViewDataSource
（代理顺序往下排列）
#pragma mark - getters and setters
#pragma mark - private

```

>注意：所有视图或者对象的创建请尽量使用懒加载，调用的时候全部使用self.textBtn这样的方式。如果是确定的不可变数组、字典，可直接给定数组中的元素。(getters and setters分类中，懒加载可出现_调用对象，其它情况请遵循self.调用原则)


```objc
#import "ViewController.h"

@interface ViewController ()
@property (nonatomic, strong)UIButton * textBtn;
@end

@implementation ViewController

#pragma mark - life cycle
- (void)viewDidLoad {
    [super viewDidLoad];
    [self.view addSubview:self.textBtn];
}

- (void)didReceiveMemoryWarning {
    [super didReceiveMemoryWarning];
    // Dispose of any resources that can be recreated.
    
}
#pragma mark - private 

#pragma mark - event response

#pragma mark - UITableViewDelegate && UITableViewDataSource
//（代理顺序往下排列）

#pragma mark - CTAPIManagerCallBackDelegate

#pragma mark - getters and setters

- (UIButton *)textBtn
{
    if (_textBtn == nil) {
        _textBtn = [UIButton buttonWithType:UIButtonTypeCustom];
        _textBtn.frame = CGRectMake(300, 250, 100, 100);
        _textBtn.backgroundColor = [UIColor yellowColor];
        _textBtn.titleLabel.text = @"text";
        [_textBtn addTarget:self action:@selector(text) forControlEvents:UIControlEventTouchUpInside];
    }
    return _textBtn;
}
@end

```

#### 变量和方法

变量和方法的命名都遵循小驼峰命名。例如：textVariableStr, - (void)textAction响应事件。

变量命名对照表（如果用到下表中没有列举出来，请去掉UI、NS遵循实际规则即可。或者一看就知道的通用简写）

方法命名对照表（方法多为动词或动名词）

| 功能 | 示例| 
| :-: | :---: |
| - (id)initXX  | 初始化相关方法,使用init为前缀标识，如初始化布局-(id)initView | 
| - (BOOL)isXX |  方法返回值为boolean型的请使用is前缀标识 | 
| - (UIView *)getXX | 返回某个值的方法，使用get为前缀标识 | 
| - (void)setXX | 设置某个属性值或者相关数据 | 
| - (void)updateXX|  更新数据 | 
| - (void)saveXX | 保存数据 | 
| - (void)drawXX | 绘制相关，使用draw前缀标识 | 
| - (void)clearXX | 清除数据 | 
| - (void)XXXAction | 响应事件，使用Action为后缀标识| 
| - (void)loadData | 加载数据（一般情况下VC中都会有这个方法）| 
| - (void)loadMoreData | 加载更多数据| 
| - (void)setupUI | 加载布局（一般情况下VC中都会有这个方法）| 

#### 常量

宏：小写k+大驼峰 即为：`#define kUserAgeKey @“ageKey”`

全局常量：工程前+缀全大写，下划线隔开 即为：`extern const NSString MW_USER_AGE_KEY`

#### 参数名
参数名以小驼峰命名，尽量参考苹果原生方法风格编写。尽量可读性好，看到方法名就知道这个方法是用来干什么的。参数应该避免用单个字符命名。例：`- (void)setDataImageUrl:(NSString *)imageUrl name:(NSString *)nameStr content:(NSString *)contentStr`


## 注释规范

为了减少他人阅读你代码的痛苦值，请在关键地方做好注释。

#### 类注释
```objc
//
//  MyViewController.m
//  text
//
//  Created by mac on 2017/9/12.
//  Copyright © 2017年 mac. All rights reserved.
//

```

该注释是自动生成的，在xcode中设置即可。Created by 电脑用户名on 创建该文件的时间。Copyright 2017 后面的名字。具体可在xcode工程，Project Document中设置。这样便可在每次新建类的时候自动加上该头注释。

#### 方法注释

>方法注释，方法外部统一用option + command + /，方法内部统一用//注释。

```objc
/**
 测试
 */
- (void)text
{
    //测试按钮事件响应   
}
```

#### 模型注释
每个model中的，包含的每个属性，都必须要写上相对应的注释，用///注释。阅读者一看这个model，就清楚知道model中的每个字段代表的意思，用来做什么事情的。

```objc
@interface DeliveryModel : NSObject
///提货劵所在商圈id
@property (nonatomic, assign) long long mallId;
///商圈全称
@property (nonatomic, copy) NSString *mallFullName;
///商圈简称
@property (nonatomic, copy) NSString *mallShortName;
///提货劵号
@property (nonatomic, copy) NSString *credentialsCode;
@end
```


>如果不是model的属性，是其它类属性，需要注释，请按照model属性注释方式。

## 编码规范

* 所有的方法之间空一行。
* 所有的代码块之间空一行，删除多余的注释。
* 所有自定义的方法需要给出注释。
* 尽量使用懒加载，在控制器分类时有提及和要求，其它自定义类按照控制器格式分类，没有的分类不写即可。
* 代码后的’{‘不需要独占一行，包括方法之后，if，switch等。
* 必须要统一的要求，属性的定义请按照下图property之后，空一格，括号之后空一格，写上类名，空一格之后跟上*和属性名。

```
@property (nonatomic, strong) UITableView *tableView;
@property (nonatomic, strong) DeliveryModel *delivery;
@property (nonatomic, strong) DeliveryLookAdapter *lookAdapter;
@property (nonatomic, strong) DeliveryLookAPIManager *lookManager;
```
* 遵循一般代码规范，多模仿苹果API。
* 删除不用的代码。
* 如果有方法一直不会用到，请删除（除工具类）。
* 没有执行任何业务逻辑的方法，请删除或给予注释，删除多余的资。源或文件，添加必要的注释。
* 比较大的代码块需要给出注释。

## 资源文件规范
#### 资源文件命名
全部小写，采用下划线命名法，加前缀区分。所有的资源文件都需要加上工程前缀（小写形式）。
命名模式：可加后缀_small表示小图,_big表示大图，逻辑名称可由多个单词加下划线组成，采用以下规则：

`用途_模块名_逻辑名称`

`用途_模块名_颜色`

`用途_逻辑名称`

`用途_颜色`

|说明 |前缀（工程前缀示例MW）| 示例|
| :-: |:-: |:-:|
|按钮相关 |`mw_btn_ `|`mw_btn_home_normal`、`mw_btn_red`,`mw_btn_red_big` |
|背景相关 |`mw_bg_ ` |`mw_bg_home_header`、`mw_bg_main` |
|图标相关 |`mw_ic_` |`mw_ic_home_location`、`mw_bg_input` |
|分割线相关 |`mw_div_` |`mw_ic_home_location`、`mw_bg_input` |
|默认相关 |`mw_def_ ` |`mw_ic_home_location`、`mw_bg_input` |

#### 文件夹命名

创建文件夹最好创建实体文件夹，找到工程目录，创建相应文件夹并拖入工程。文件夹命名使用相应模块结构分层的英文，首字母要大写。例：Model，View，Controller，Tool，Other，Service等等。

## 版本规范

采用A.B.C 三位数字命名，比如：1.0.2，当有版本更新的时候，依据下面的情况来确定版本号规范。

|版本号 |说明 |示例|
| :-: |:-: |:-:|
|A.b.c |属于重大更新内容 |1.0.2 -> 2.00 |
|a.B.c |属于小部分更新内容 |1.0.2 -> 1.2.2 |
|a.b.C |属于补丁更新内容 |1.0.2 -> 1.0.4|

## 第三方库规范

开源库的选取，一般都需要选择比较稳定的版本，作者在维护的项目，要考虑作者对issue的解决，以及开发者的知名度等各方面。选取之后，一定的封装是必要的。

项目使用cocoapods统一管理开源第三库文件，不需要手动导入和手动添加依赖库。

[iOS开发规范](https://www.jianshu.com/p/1784cd67e8de)
