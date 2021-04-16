---
layout:     post
title:      "iOS编译原理"
date:       2021-4-16 12:13:00
author:     "wanglilong"
header-img: "img/post-bg-2015.jpg"
catalog: true
tags:

- iOS原理

---

## 前言

iOS开发使用Object-C和Swift编译语言，两者都需要通过编译器（Clang + LLVM）把代码编译器生成机器码，机器码可以直接在CPU上执行。接下来详细介绍一下这两种语言的优缺点，可以更深入的理解为什么移动端开发会采用编译语言。

[编译语言](https://zh.wikipedia.org/wiki/編譯語言)和[直译式语言](https://en.wikipedia.org/wiki/Interpreted_language)两种编程语言的优缺点：

**编译语言/直译式语言优缺点比较：**

| 编译/直译式 | 优点                                                         | 缺点                                                         |
| ----------- | ------------------------------------------------------------ | ------------------------------------------------------------ |
| 编译语言    | 运行速度快（被预先编译成机器码，可以直接运行）               | 开发、调试比较长（程序开发速度，以及除错时间时间较长）       |
| 直译式语言  | 开发、调试比较短（程序开发速度，以及除错时间时间较长。应用程序不能脱离其解释器，但这种方式比较灵活，可以动态地调整、修改应用程序） | 运行速度慢（源代码一边由相应语言的解释器翻译”成目标代码（机器语言），一边执行） |

**编译语言**： 像C++、Objective-C（Swift）、C、Java等都是编译语言，必须通过编译器生成机器码，机器码可以直接在CPU上执行，所以你执行效率较高。而每次运行都需要编译器生成机器码再运行，故编写、调试、排错比较繁琐。（Swift是支持直译的）

**直译式语言**：像JavaScript、Python、PHP等都是直译式语言，不需要经过编译的过程，而是在执行的时候通过一个中间的解释器将代码解释为CPU可以执行的代码。所以，较编译语言来说，直译式语言效率低一些，但是编写比较灵活。

> **总之，由于移动端（iOS、Android）设备性能限制的情况下，采用编译语言进行开发是一种比较好的方式**

## 一、iOS编译器

> 在讲解编译过程之前，需要了解苹果公司采用的编译器，编译器采用哪种方式进行编译，才能更深入的理解编译底层的整个过程。

### 1.为什么选择LLVM编译器

#### 1.1编译器

##### 1.1.1经典编译器设计简介

传统静态编译器（如大多数C编译器）最流行的设计是三阶段设计，其主要组件是前端，优化器和后端。 前端解析源代码，检查它是否有错误，并构建一个特定于语言的抽象语法树（AST）来表示输入代码。 AST可选地转换为新的表示以进行优化，优化器和后端在代码上运行。

![img](/img/post-ios-compile/2664540-511eb8e90b8da635.png)



##### 1.1.2 LLVM编译器设计简介（实现三相设计）

在基于LLVM的编译器中，前端负责解析，验证和诊断输入代码中的错误，然后将解析的代码转换为LLVM IR（通常情况。但是也有例外，通过构建AST然后将AST转换为LLVM IR）。该IR可选地通过一系列改进代码的分析和优化过程提供，然后被发送到代码生成器以生成本机机器代码，如图下图所示。

![img](/img/post-ios-compile/2664540-427466dab6dd63d8.png)

为什么要使用三相设计？优势在哪？

> 首先解决了一个很大的问题：假如有N种语言（C、OC、C++、Swift...）的前端，同时也有M个架构（模拟器、arm64、x86...）的Target，是否就需要 N × M 个编译器？
> 三相架构的价值就体现出来了，通过共享优化器的中转，很好的解决了这个问题。
> 假如你需要增加一种语言，只需要增加一种前端；假如你需要增加一种处理器架构，也只需要增加一种后端，而其他的地方都不需要改动。这复用思想很牛逼吧。

#### 1.2 LLVM编译器的组成

LLVM项目是模块化和可重用的编译器和工具链技术的集合。LLVM主要的子项目有一下几个：

> **1.LLVM核心库:**
> LLVM提供一个独立的链接代码优化器为许多流行CPU（以及一些不太常见的CPU）的代码生成支持。这些库是围绕一个指定良好的代码表示构建的，称为LLVM中间表示（“LLVM IR”）。LLVM还可以充当JIT编译器 - 它支持x86 / x86_64和PPC / PPC64程序集生成，并具有针对编译速度的快速代码优化。。
> **2.Clang:**
> Clang是一个“LLVM原生”C / C ++ / Objective-C编译器，旨在提供惊人的快速编译（例如，在调试配置中编译Objective-C代码时比GCC快3倍），非常有用的错误和警告消息以及提供构建优秀源代码工具的平台。
> **3.LLDB项目:**
> LLDB项目以LLVM和Clang提供的库为基础，提供了一个出色的本机调试器。它使用Clang AST和表达式解析器，LLVM JIT，LLVM反汇编程序等，以便提供“正常工作”的体验。在加载符号时，它也比GDB快速且内存效率更高。
> **4.libc ++和libc++:**
> libc ++和libc++ ABI项目提供了C ++标准库的标准符合性和高性能实现，包括对C ++ 11的完全支持。
> **5.lld项目:**
> lld项目旨在成为clang / llvm的内置链接器。目前，clang必须调用系统链接器来生成可执行文件。
>
> ***其他的就不再详细介绍了,详情可以参考([LLVM](https://llvm.org/)和[Clang](https://clang.llvm.org/))\***

**总之，LLVM是Apple主导的开源框架，并提供一套使用于Apple平台的LLVM编译器，同时提供优秀的性能，所以Apple采用LLVM的方式进行编译**

### 2. Clang + LLVM 的编译简单过程

##### 3.1.Clang + LLVM 编译过程

LLVM采用三相设计，前端Clang负责解析，验证和诊断输入代码中的错误，然后将解析的代码转换为LLVM IR，后端LLVM编译把IR通过一系列改进代码的分析和优化过程提供，然后被发送到代码生成器以生成本机机器代码。

##### 3.2 Clang 编译前端

> 编译器前端的任务是进行：语法分析，语义分析，生成中间代码(intermediate representation )。在这个过程中，会进行类型检查，如果发现错误或者警告会标注出来在哪一行。

##### 3.3 LLVM 编译后端

编译器后端会进行机器无关的代码优化，生成机器语言，并且进行机器相关的代码优化。iOS的编译过程，后端的处理如下：

> LVVM优化器会进行BitCode的生成，链接期优化等等。
>
> LLVM机器码生成器会针对不同的架构，比如arm64等生成不同的机器码。



## 二、LLVM 编译详细过程原理

### 1.简述LLVM的使用场景

Clang是LLVM的一个前端，在Xcode编译iOS项目的时候，都是使用的LLVM，其实在编写代码以及调试的时候都在接触LLVM提供的功能，例如：代码的亮度（Clang）、实时代码检查（Clang）、代码提示（Clang）、debug断点调试（LLDB）。

### 2.项目编译过程简介

**下面来简单的讲讲整个 iOS 项目的编译过程,其中可能会有一些疑问，先保留着，后面会详细解释**

> 我们的项目是一个 target，一个编译目标，它拥有自己的文件和编译规则，在我们的项目中可以存在多个子项目，这在编译的时候就导致了使用了 Cocoapods 或者拥有多个 target 的项目会先编译依赖库。这些库都和我们的项目编译流程一致。Cocoapods 的原理解释将在文章后面一部分进行解释。

**iOS 项目的编译过程**

```css
1.写入辅助文件：将项目的文件结构对应表、将要执行的脚本、项目依赖库的文件结构对应表写成文件，方便后面使用；并且创建一个 .app 包，后面编译后的文件都会被放入包中；
2.运行预设脚本：Cocoapods 会预设一些脚本，当然你也可以自己预设一些脚本来运行。这些脚本都在 Build Phases 中可以看到；
3.编译文件：针对每一个文件进行编译，生成可执行文件 Mach-O，这过程 LLVM 的完整流程，前端、优化器、后端；
4.链接文件：将项目中的多个可执行文件合并成一个文件；
5.拷贝资源文件：将项目中的资源文件拷贝到目标包；
6.编译 storyboard 文件：storyboard 文件也是会被编译的；
7.链接 storyboard 文件：将编译后的 storyboard 文件链接成一个文件；
8.编译 Asset 文件：我们的图片如果使用 Assets.xcassets 来管理图片，那么这些图片将会被编译成机器码，除了 icon 和 launchImage；
9.运行 Cocoapods 脚本：将在编译项目之前已经编译好的依赖库和相关资源拷贝到包中。
10.生成 .app 包
11.将 Swift 标准库拷贝到包中
12.对包进行签名
13.完成打包
```

在上述流程中：2 - 9 步骤的数量和顺序并不固定，这个过程可以在 Build Phases 中指定。`Phases：阶段、步骤`。这个 Tab 的意思就是编译步骤。其实不仅我们的整个编译步骤和顺序可以被设定，包括编译过程中的编译规则（Build Rules）和具体步骤的参数（Build Settings），在对应的 Tab 都可以看到。

### 3.文件编译过程

#### 3.1. 预处理

- 预处理顾名思义是预先处理,那预处理都做了哪些事情呢？内容如下。
  - (1) import 头文件替换
    代码中会有很多 #import 宏，预处理的第一步就是将 import 引入的文件代码放入对应文件。
  - (2) macro 宏展开
    带参数宏和不带参数宏
  - (3)处理其他的预编译指令（其实预编译过程也是出了预编译指令的过程）
  - (4)总之简单来说，“#”这个符号是编译器预处理的标志

> 条件编译语句也是在预处理阶段完成，并且条件编译只允许编译源程序中满足条件的程序段，使生成的目标程序较短，从而减少了内存的开销并提高了程序的效率,如以下代码就只会保留一个return语句：

```cpp
#if DEBUG        
     return YES;
#else
     return NO;
#endif
```

#### 3.2.词法分析

使用 clang 命令 `clang -Xclang -dump-tokens main.m` 转化后的代码如下（去掉了#import <Foundation/Foundation.h>的内容）：

- 词法分析，只需要将源代码以字符文本的形式转化成Token流的形式，不涉及交验语义，不需要递归，是线性的。

> 什么是token流呢？可以这么理解：就是有"类型"，有"值"的一些小单元。

词法分析其实是编译器开始工作真正意义上的第一个步骤，其所做的工作主要为将输入的代码转换为一系列符合特定语言的词法单元，这些词法单元类型包括了关键字，操作符，变量等等。

可以通过下发被编译过的代码对应main.m文件，把所有的内容都一一对应起来。

```objectivec
// 编译前

int main(){
    NSObject *obj = [[NSObject alloc] init];
    id __attribute__((objc_ownership(weak))) obj1 = obj;
    NSLog(@"------%@--%d--",[obj1 class],10);
}
```

```csharp
//编译后
int 'int'    [StartOfLine]  Loc=<main3.m:11:1>
identifier 'main'    [LeadingSpace] Loc=<main3.m:11:5>
l_paren '('     Loc=<main3.m:11:9>
r_paren ')'     Loc=<main3.m:11:10>
l_brace '{'     Loc=<main3.m:11:11>
identifier 'NSObject'    [StartOfLine] [LeadingSpace]   Loc=<main3.m:13:5>
star '*'     [LeadingSpace] Loc=<main3.m:13:14>
identifier 'obj'        Loc=<main3.m:13:15>
equal '='    [LeadingSpace] Loc=<main3.m:13:19>
l_square '['     [LeadingSpace] Loc=<main3.m:13:21>
l_square '['        Loc=<main3.m:13:22>
identifier 'NSObject'       Loc=<main3.m:13:23>
identifier 'alloc'   [LeadingSpace] Loc=<main3.m:13:32>
r_square ']'        Loc=<main3.m:13:37>
identifier 'init'    [LeadingSpace] Loc=<main3.m:13:39>
r_square ']'        Loc=<main3.m:13:43>
semi ';'        Loc=<main3.m:13:44>
identifier 'id'  [StartOfLine] [LeadingSpace]   Loc=<main3.m:14:5>
__attribute '__attribute__'  [LeadingSpace] Loc=<main3.m:14:8 <Spelling=<built-in>:309:16>>
l_paren '('     Loc=<main3.m:14:8 <Spelling=<built-in>:309:29>>
l_paren '('     Loc=<main3.m:14:8 <Spelling=<built-in>:309:30>>
identifier 'objc_ownership'     Loc=<main3.m:14:8 <Spelling=<built-in>:309:31>>
l_paren '('     Loc=<main3.m:14:8 <Spelling=<built-in>:309:45>>
identifier 'weak'       Loc=<main3.m:14:8 <Spelling=<built-in>:309:46>>
r_paren ')'     Loc=<main3.m:14:8 <Spelling=<built-in>:309:50>>
r_paren ')'     Loc=<main3.m:14:8 <Spelling=<built-in>:309:51>>
r_paren ')'     Loc=<main3.m:14:8 <Spelling=<built-in>:309:52>>
identifier 'obj1'    [LeadingSpace] Loc=<main3.m:14:15>
equal '='    [LeadingSpace] Loc=<main3.m:14:20>
identifier 'obj'     [LeadingSpace] Loc=<main3.m:14:22>
semi ';'        Loc=<main3.m:14:25>
identifier 'NSLog'   [StartOfLine] [LeadingSpace]   Loc=<main3.m:15:5>
l_paren '('     Loc=<main3.m:15:10>
at '@'      Loc=<main3.m:15:11>
string_literal '"------%@--%d--"'       Loc=<main3.m:15:12>
comma ','       Loc=<main3.m:15:28>
l_square '['        Loc=<main3.m:15:29>
identifier 'obj1'       Loc=<main3.m:15:30>
identifier 'class'   [LeadingSpace] Loc=<main3.m:15:35>
r_square ']'        Loc=<main3.m:15:40>
comma ','       Loc=<main3.m:15:41>
numeric_constant '10'       Loc=<main3.m:15:42 <Spelling=main3.m:10:12>>
r_paren ')'     Loc=<main3.m:15:44>
semi ';'        Loc=<main3.m:15:45>
r_brace '}'  [StartOfLine]  Loc=<main3.m:17:1>
eof ''      Loc=<main3.m:17:2>
```

> **这里，每一个符号都会标记出来其位置，这个位置是宏展开之前的位置，这样后面如果发现报错，就可以正确的提示错误位置了。**

#### 3.3.语法分析

对代码进行标记并不是Clang最终的目的，而是一个Clang的一个过程，其实标记代码为了让代码更便于转化成机器语言，标记代码转化成抽象语法树（abstract syntax tree – AST）是一个必经之路。

##### 3.3.1 AST

使用 clang 命令 `clang -Xclang -ast-dump -fsyntax-only main.m`，转化后的树如下

```kotlin
-FunctionDecl 0x7fbbea20f538 <main3.m:11:1, line:17:1> line:11:5 main 'int ()'
  `-CompoundStmt 0x7fbbea20fa40 <col:11, line:17:1>
    |-DeclStmt 0x7fbbea20f6b8 <line:13:5, col:44>
    | `-VarDecl 0x7fbbea20f5e8 <col:5, col:43> col:15 used obj 'NSObject *' cinit
    |   `-ObjCMessageExpr 0x7fbbea20f688 <col:21, col:43> 'NSObject *' selector=init
    |     `-ObjCMessageExpr 0x7fbbea20f658 <col:22, col:37> 'NSObject *' selector=alloc class='NSObject'
    |-DeclStmt 0x7fbbea20f830 <line:14:5, col:25>
    | `-VarDecl 0x7fbbea20f778 <col:5, col:22> col:15 used obj1 '__weak id':'__weak id' cinit
    |   `-ImplicitCastExpr 0x7fbbea20f818 <col:22> 'id':'id' <BitCast>
    |     `-ImplicitCastExpr 0x7fbbea20f800 <col:22> 'NSObject *' <LValueToRValue>
    |       `-DeclRefExpr 0x7fbbea20f7d8 <col:22> 'NSObject *' lvalue Var 0x7fbbea20f5e8 'obj' 'NSObject *'
    `-ExprWithCleanups 0x7fbbea20fa28 <line:15:5, col:44> 'void'
      `-CallExpr 0x7fbbea20f9d0 <col:5, col:44> 'void'
        |-ImplicitCastExpr 0x7fbbea20f9b8 <col:5> 'void (*)(id, ...)' <FunctionToPointerDecay>
        | `-DeclRefExpr 0x7fbbea20f848 <col:5> 'void (id, ...)' Function 0x7fbbe94469e0 'NSLog' 'void (id, ...)'
        |-ImplicitCastExpr 0x7fbbea20fa10 <col:11, col:12> 'id':'id' <BitCast>
        | `-ObjCStringLiteral 0x7fbbea20f8a8 <col:11, col:12> 'NSString *'
        |   `-StringLiteral 0x7fbbea20f870 <col:12> 'char [15]' lvalue "------%@--%d--"
        |-ObjCMessageExpr 0x7fbbea20f908 <col:29, col:40> 'Class':'Class' selector=class
        | `-ImplicitCastExpr 0x7fbbea20f8f0 <col:30> 'id':'id' <LValueToRValue>
        |   `-DeclRefExpr 0x7fbbea20f8c8 <col:30> '__weak id':'__weak id' lvalue Var 0x7fbbea20f778 'obj1' '__weak id':'__weak id'
        `-IntegerLiteral 0x7fbbea20f938 <line:10:12> 'int' 10
```

这个main方法的抽象树可以看出来树顶是`FunctionDecl`:方法声明（Function Declaration）。

> 这里因为截取了部分代码，其实并不是整个树的树顶。真正的树顶描述应该是：TranslationUnitDecl。

详细的AST语法不多介绍，关于 AST 的详细解释可以查看：[Introduction to the Clang AST](http://clang.llvm.org/docs/IntroductionToTheClangAST.html)。

##### 3.3.2 静态分析

- 通过语法树进行代码静态分析，找出非语法性错误
- 模拟代码执行路径，分析出control-flow graph(CFG) 【MRC时代会分析出引用计数的错误】
- 预置了常用Checker（检查器）

#### 3.4. IR中间代码生成

当通过Clang语法解析，代码没有出现报错，Clang前端就将进入最后一步：生成LLVM IR中间代码，并将生成的LLVM IR代码递交给优化器。

使用命令 `clang -S -emit-llvm main3.m -o main.ll`生成LLVM 中间代码LLVM IR

```objectivec
// mian3.m代码
#import <Foundation/Foundation.h>
#define aa 10
int main(){
    NSObject *obj = [[NSObject alloc] init];
    NSLog(@"------%@--%d--",[obj class],aa);
}
```



```tsx
; ModuleID = 'main3.m'
source_filename = "main3.m"
target datalayout = "e-m:o-i64:64-f80:128-n8:16:32:64-S128"
target triple = "x86_64-apple-macosx10.13.0"

%0 = type opaque
%struct._class_t = type { %struct._class_t*, %struct._class_t*, %struct._objc_cache*, i8* (i8*, i8*)**, %struct._class_ro_t* }
%struct._objc_cache = type opaque
%struct._class_ro_t = type { i32, i32, i32, i8*, i8*, %struct.__method_list_t*, %struct._objc_protocol_list*, %struct._ivar_list_t*, i8*, %struct._prop_list_t* }
%struct.__method_list_t = type { i32, i32, [0 x %struct._objc_method] }
%struct._objc_method = type { i8*, i8*, i8* }
%struct._objc_protocol_list = type { i64, [0 x %struct._protocol_t*] }
%struct._protocol_t = type { i8*, i8*, %struct._objc_protocol_list*, %struct.__method_list_t*, %struct.__method_list_t*, %struct.__method_list_t*, %struct.__method_list_t*, %struct._prop_list_t*, i32, i32, i8**, i8*, %struct._prop_list_t* }
%struct._ivar_list_t = type { i32, i32, [0 x %struct._ivar_t] }
%struct._ivar_t = type { i64*, i8*, i8*, i32, i32 }
%struct._prop_list_t = type { i32, i32, [0 x %struct._prop_t] }
%struct._prop_t = type { i8*, i8* }
%struct.__NSConstantString_tag = type { i32*, i32, i8*, i64 }

@"OBJC_CLASS_$_NSObject" = external global %struct._class_t
@"OBJC_CLASSLIST_REFERENCES_$_" = private global %struct._class_t* @"OBJC_CLASS_$_NSObject", section "__DATA,__objc_classrefs,regular,no_dead_strip", align 8
@OBJC_METH_VAR_NAME_ = private unnamed_addr constant [6 x i8] c"alloc\00", section "__TEXT,__objc_methname,cstring_literals", align 1
@OBJC_SELECTOR_REFERENCES_ = private externally_initialized global i8* getelementptr inbounds ([6 x i8], [6 x i8]* @OBJC_METH_VAR_NAME_, i32 0, i32 0), section "__DATA,__objc_selrefs,literal_pointers,no_dead_strip", align 8
@OBJC_METH_VAR_NAME_.1 = private unnamed_addr constant [5 x i8] c"init\00", section "__TEXT,__objc_methname,cstring_literals", align 1
@OBJC_SELECTOR_REFERENCES_.2 = private externally_initialized global i8* getelementptr inbounds ([5 x i8], [5 x i8]* @OBJC_METH_VAR_NAME_.1, i32 0, i32 0), section "__DATA,__objc_selrefs,literal_pointers,no_dead_strip", align 8
@__CFConstantStringClassReference = external global [0 x i32]
@.str = private unnamed_addr constant [15 x i8] c"------%@--%d--\00", section "__TEXT,__cstring,cstring_literals", align 1
@_unnamed_cfstring_ = private global %struct.__NSConstantString_tag { i32* getelementptr inbounds ([0 x i32], [0 x i32]* @__CFConstantStringClassReference, i32 0, i32 0), i32 1992, i8* getelementptr inbounds ([15 x i8], [15 x i8]* @.str, i32 0, i32 0), i64 14 }, section "__DATA,__cfstring", align 8
@OBJC_METH_VAR_NAME_.3 = private unnamed_addr constant [6 x i8] c"class\00", section "__TEXT,__objc_methname,cstring_literals", align 1
@OBJC_SELECTOR_REFERENCES_.4 = private externally_initialized global i8* getelementptr inbounds ([6 x i8], [6 x i8]* @OBJC_METH_VAR_NAME_.3, i32 0, i32 0), section "__DATA,__objc_selrefs,literal_pointers,no_dead_strip", align 8
@llvm.compiler.used = appending global [7 x i8*] [i8* bitcast (%struct._class_t** @"OBJC_CLASSLIST_REFERENCES_$_" to i8*), i8* getelementptr inbounds ([6 x i8], [6 x i8]* @OBJC_METH_VAR_NAME_, i32 0, i32 0), i8* bitcast (i8** @OBJC_SELECTOR_REFERENCES_ to i8*), i8* getelementptr inbounds ([5 x i8], [5 x i8]* @OBJC_METH_VAR_NAME_.1, i32 0, i32 0), i8* bitcast (i8** @OBJC_SELECTOR_REFERENCES_.2 to i8*), i8* getelementptr inbounds ([6 x i8], [6 x i8]* @OBJC_METH_VAR_NAME_.3, i32 0, i32 0), i8* bitcast (i8** @OBJC_SELECTOR_REFERENCES_.4 to i8*)], section "llvm.metadata"

; Function Attrs: noinline optnone ssp uwtable
define i32 @main() #0 {
  %1 = alloca %0*, align 8
  %2 = load %struct._class_t*, %struct._class_t** @"OBJC_CLASSLIST_REFERENCES_$_", align 8
  %3 = load i8*, i8** @OBJC_SELECTOR_REFERENCES_, align 8, !invariant.load !8
  %4 = bitcast %struct._class_t* %2 to i8*
  %5 = call i8* bitcast (i8* (i8*, i8*, ...)* @objc_msgSend to i8* (i8*, i8*)*)(i8* %4, i8* %3)
  %6 = bitcast i8* %5 to %0*
  %7 = load i8*, i8** @OBJC_SELECTOR_REFERENCES_.2, align 8, !invariant.load !8
  %8 = bitcast %0* %6 to i8*
  %9 = call i8* bitcast (i8* (i8*, i8*, ...)* @objc_msgSend to i8* (i8*, i8*)*)(i8* %8, i8* %7)
  %10 = bitcast i8* %9 to %0*
  store %0* %10, %0** %1, align 8
  %11 = load %0*, %0** %1, align 8
  %12 = load i8*, i8** @OBJC_SELECTOR_REFERENCES_.4, align 8, !invariant.load !8
  %13 = bitcast %0* %11 to i8*
  %14 = call i8* bitcast (i8* (i8*, i8*, ...)* @objc_msgSend to i8* (i8*, i8*)*)(i8* %13, i8* %12)
  notail call void (i8*, ...) @NSLog(i8* bitcast (%struct.__NSConstantString_tag* @_unnamed_cfstring_ to i8*), i8* %14, i32 10)
  ret i32 0
}

; Function Attrs: nonlazybind
declare i8* @objc_msgSend(i8*, i8*, ...) #1

declare void @NSLog(i8*, ...) #2

attributes #0 = { noinline optnone ssp uwtable "correctly-rounded-divide-sqrt-fp-math"="false" "disable-tail-calls"="false" "less-precise-fpmad"="false" "no-frame-pointer-elim"="true" "no-frame-pointer-elim-non-leaf" "no-infs-fp-math"="false" "no-jump-tables"="false" "no-nans-fp-math"="false" "no-signed-zeros-fp-math"="false" "no-trapping-math"="false" "stack-protector-buffer-size"="8" "target-cpu"="penryn" "target-features"="+cx16,+fxsr,+mmx,+sse,+sse2,+sse3,+sse4.1,+ssse3,+x87" "unsafe-fp-math"="false" "use-soft-float"="false" }
attributes #1 = { nonlazybind }
attributes #2 = { "correctly-rounded-divide-sqrt-fp-math"="false" "disable-tail-calls"="false" "less-precise-fpmad"="false" "no-frame-pointer-elim"="true" "no-frame-pointer-elim-non-leaf" "no-infs-fp-math"="false" "no-nans-fp-math"="false" "no-signed-zeros-fp-math"="false" "no-trapping-math"="false" "stack-protector-buffer-size"="8" "target-cpu"="penryn" "target-features"="+cx16,+fxsr,+mmx,+sse,+sse2,+sse3,+sse4.1,+ssse3,+x87" "unsafe-fp-math"="false" "use-soft-float"="false" }

!llvm.module.flags = !{!0, !1, !2, !3, !4, !5, !6}
!llvm.ident = !{!7}

!0 = !{i32 1, !"Objective-C Version", i32 2}
!1 = !{i32 1, !"Objective-C Image Info Version", i32 0}
!2 = !{i32 1, !"Objective-C Image Info Section", !"__DATA,__objc_imageinfo,regular,no_dead_strip"}
!3 = !{i32 4, !"Objective-C Garbage Collection", i32 0}
!4 = !{i32 1, !"Objective-C Class Properties", i32 64}
!5 = !{i32 1, !"wchar_size", i32 4}
!6 = !{i32 7, !"PIC Level", i32 2}
!7 = !{!"Apple LLVM version 9.1.0 (clang-902.0.39.2)"}
!8 = !{}
```

在这里简单介绍一些 LLVM IR 的指令：

> %：局部变量
> @：全局变量
> alloca：分配内存堆栈
> i32：32 位的整数
> i32**：一个指向 32 位 int 值的指针的指针
> align 4：向 4 个字节对齐，即便数据没有占用 4 个字节，也要为其分配四个字节
> call：调用

LLVM IR 是Frontend的输出，也是LLVM Backend的输入，前后端的桥接语言, 更具生成的文件解析，其实生成的LLVM IR对Runtime进行桥接的一个文件

```objectivec
1.Class/Meta Class/Protocol/Category内存结构生成，并存放在指定section中（如Class：_DATA,_objc_classrefs）
2.Method/lvar/Property内存结构生成
3.组成method_list/ivar_list/property_list并填入Class
4.Non-Fragile ABI:为每个Ivar合成OBJC_IVAR_$_ 偏移值常量
5.存取Ivar的语句（ivar = 123; int a = ivar;）转写成base + OBJC_IVAR$_的形式
6.将语法树中的ObjcMessageExpr翻译成相应版本的objc_msgSend，7.对super关键字的调用翻译成objc_msgSendSuper
8.根据修饰符strong/weak/copy/atomic合成@property 自动实现的 setter/getter
9.处理@synthesize
10.生成block_layout的数据结构
11.变量的capture(__block/__weak)
12.生成_block_invoke函数
13.ARC：分析对象引用关系，将objc_storeStrong/objc_storeWeak等ARC代码插入
14.将ObjCAutoreleasePoolStmt转译成objc_autoreleasePoolPush/Pop
15.实现自动调用[super dealloc]
16为每个拥有ivar的Class合成.cxx_destructor方法来自动释放类的成员变量，代替MRC时代的“self.xxx = nil”
```

##### 3.5. 优化IR

使用命令 `clang -O3 -S -emit-llvm main3.m -o main3.ll`

这一步骤的优化是非常重要的，很多直接转换来的代码是不合适且消耗内存的，因为是直接转换，所以必然会有这样的问题，而优化放在这一步的好处在于前端不需要考虑任何优化过程，减少了前端的开发工作。

##### 3.6. 生成Target相关汇编

使用命令 `clang -S -o - main3.m | open -f`可以查看生成的汇编代码：

注意代码中的 .section 指令，它指定了接下来会执行的代码段。

在这篇文章中，详细解释了这些汇编指令或代码到底是如何工作的：[Mach-O 可执行文件](https://www.objccn.io/issue-6-3/)。

##### 3.7. Link生成Executable

在最后，LLVM 将会把这些汇编代码输出成二进制的可执行文件，使用命令 clang main3.m -o main.out 即可查看

```undefined
Segment __PAGEZERO: 0x100000000 (vmaddr 0x0 fileoff 0)
Segment __TEXT: 0x1000 (vmaddr 0x100000000 fileoff 0)
    Section __text: 0x34 (addr 0x100000f50 offset 3920)
    Section __stubs: 0x6 (addr 0x100000f84 offset 3972)
    Section __stub_helper: 0x1a (addr 0x100000f8c offset 3980)
    Section __cstring: 0xe (addr 0x100000fa6 offset 4006)
    Section __unwind_info: 0x48 (addr 0x100000fb4 offset 4020)
    total 0xaa
Segment __DATA: 0x1000 (vmaddr 0x100001000 fileoff 4096)
    Section __nl_symbol_ptr: 0x10 (addr 0x100001000 offset 4096)
    Section __la_symbol_ptr: 0x8 (addr 0x100001010 offset 4112)
    total 0x18
Segment __LINKEDIT: 0x1000 (vmaddr 0x100002000 fileoff 8192)
total 0x100003000
```

上面的代码中，每个 segment 的意义也不一样：

- `__PAGEZERO` segment 它的大小为 4GB。这 4GB 并不是文件的真实大小，但是规定了进程地址空间的前 4GB 被映射为 不可执行、不可写和不可读。
- `__TEXT` segment 包含了被执行的代码。它被以只读和可执行的方式映射。进程被允许执行这些代码，但是不能修改。
- `__DATA` segment 以可读写和不可执行的方式映射。它包含了将会被更改的数据。
- `__LINKEDIT` segment 指出了 link edit 表（包含符号和字符串的动态链接器表）的地址，里面包含了加载程序的元数据，例如函数的名称和地址。

#### 4. Swift的编译过程

在 [Swift 编译器结构](https://swift.org/compiler-stdlib/#compiler-architecture) 的官方文档中描述了 Swift 编译器是如何工作的，分为如下步骤：

- **解析：**解析器是一个简单的递归下降解析器（在 [lib / Parse](https://github.com/apple/swift/tree/master/lib/Parse) 中实现），带有集成的手动编码词法分析器。解析器负责生成没有任何语义或类型信息的抽象语法树（AST），并针对输入源的语法问题发出警告或错误。
- **语意分析：**语义分析（在 [lib / Sema](https://github.com/apple/swift/tree/master/lib/Sema) 中实现）负责解析 AST 并将其转换为格式良好的完全检查形式的 AST，并在源代码中发出语义问题的警告或错误。语义分析包括类型推断，如果成功，则所得到的代码是类型检查安全的 AST 。
- **Clang导入器：**Clang导入器（在 [lib / ClangImporter](https://github.com/apple/swift/tree/master/lib/ClangImporter) 中实现）导入Clang模块，并将它们导出的 C 或 Objective-C API 映射到相应的 Swift API中。结果导入的 AST 可以通过语义分析来引用。（swiftc？？？）
- **SIL生成：**Swift中间语言（Swift Intermediate Language，简称SIL）是一种高级的，Swift特有的中间语言，适用于 Swift 代码的进一步分析和优化。SIL 生成阶段（在 [lib / SILGen](https://github.com/apple/swift/tree/master/lib/SILGen) 中实现）将类型检查的 AST 降低到所谓的 “原始” SIL。SIL的设计描述在 [docs/ SIL.rst](https://github.com/apple/swift/blob/master/docs/SIL.rst) 中可以看到。
- **SIL优化：**在SIL优化（在 [lib/Analysis](https://github.com/apple/swift/tree/master/lib/SILOptimizer/Analysis)，[lib/ ARC](https://github.com/apple/swift/tree/master/lib/SILOptimizer/ARC)，[lib/LoopTransforms](https://github.com/apple/swift/tree/master/lib/SILOptimizer/LoopTransforms)，和 [lib/Transforms](https://github.com/apple/swift/tree/master/lib/SILOptimizer/Transforms) 中实现）执行额外的高级别，Swift 特有的优化的程序，包括（例如）自动引用计数优化，虚拟化和通用专业化。
- **LLVM IR生成：**IR生成（在 [lib/IRGen](https://github.com/apple/swift/tree/master/lib/IRGen) 中实现）将 SIL 降到 LLVM IR，此时LLVM可以继续对其进行优化并生成机器码。

相关内容不详细讲解，可参考：https://blog.csdn.net/aas319/article/details/78606342

## 三、 Cocoapods 原理

使用了 Cocoapods 后，我们的编译流程会多出来一些，虽然每个 target 的编译流程都是一致的，但是 Cocoapods 是如何将这些库导入我们的项目、原项目和其他库之间的依赖又是如何实现的仍然是一个需要了解的知识点。

- [深入理解 CocoaPods](https://www.objccn.io/issue-6-4/)
- [Cocoapods原理总结](https://juejin.im/entry/59dd94b06fb9a0451463030b)
- [CocoaPods 都做了什么？](https://zhuanlan.zhihu.com/p/22652365)



##参考资料：

[iOS编译原理](http://hchong.net/2019/07/30/iOS%E7%BC%96%E8%AF%91%E5%8E%9F%E7%90%86/)讲的更清晰一些

[深入剖析 iOS 编译 Clang / LLVM](https://ming1016.github.io/2017/03/01/deeply-analyse-llvm/#Clang-%E7%BC%96%E8%AF%91-m-%E6%96%87%E4%BB%B6)讲的详细

[iOS编译过程的原理和应用](https://blog.csdn.net/Hello_Hwc/article/details/53557308)

[浅谈iOS编译过程](https://www.jianshu.com/p/5b2cce762106)

