---
layout:     post
title:      "登录打通SDK设计"
subtitle:   " \"如何设计SDK\""
date:       2019-10-25 10:13:00
author:     "wanglilong"
header-img: "img/post-bg-2015.jpg"
catalog: true
tags:
- iOS 开发
---
# 登录打通SDK设计

> JDCAuthorizationSDK包含了日志、弹窗、网络请求、web、接口、京东授权六大部分。以完成京东授权登录打通的工作



### 一、日志系统

日志系统使用的是裁剪后的[CocoaLumberjack](https://github.com/CocoaLumberjack/CocoaLumberjack)框架。

原CocoaLumberjack框架类图如下。

![](/img/post-one-sdk-design/CocoaLumberjackClassDiagram.png)

DDAbstractLogger的类簇，是管理日志输出到哪里。分别是 Xcode console，MAC的日志系统，文件和数据库。

DDLogFormatter的类簇，是管理输出格式的。有多线程，黑白名单，和多格式输出。



我们只使用了其部分功能。所以对其进行裁剪。

1、只用到 Xcode console 输入。所以DDFileLogger、DDAbstractDatabaseLogger、DDASLogger类全部删除，只保留输出到 Xcode console 的DDTTYLogger。

2、输出格式DDLogFormatter的子类，全部删除。改为使用自定义格式。

3、为防止发生冲突，所有的类，全部改为JDC开头。



自定义JDCNormalLogFormatter如下。

```objc
- (NSString *)formatLogMessage:(JDCLogMessage *)logMessage{
    NSString *functionName = [NSString stringWithCString:logMessage.function.UTF8String encoding:NSASCIIStringEncoding];
    
    NSString *flagName = @"";
    if (logMessage.flag&JDCLogFlagError) {
        flagName = @"Error";
    }else if (logMessage.flag&JDCLogFlagWarning){
        flagName = @"Warning";
    }else if (logMessage.flag&JDCLogFlagInfo){
        flagName = @"Info";
    }else if (logMessage.flag&JDCLogFlagDebug){
        flagName = @"Debug";
    }else if (logMessage.flag&JDCLogFlagVerbose){
        flagName = @"Verbose";
    }
    
    return [NSString stringWithFormat:@"%@ [JDCAuthorizationSDK] [%@]:%@\n end:{fun:%@ ,line:%lu}",logMessage.timestamp,flagName,logMessage.message,functionName, (unsigned long)logMessage.line];
}
```



### 二、弹窗系统

弹窗系统使用[JKCategories](https://github.com/shaojiankui/JKCategories)中的一个类（UIView+JKToast）。对其进行重命名为UIView+JDCToast。

并在JDCityToast类中进行简单封装。

1、对具体实现进行简单隔离，方便框架的隔离。

2、使其在对顶端view上弹出。

```objective-c
+ (UIView *)getView
{
    UIWindow *topView = [UIApplication sharedApplication].keyWindow;
    for (UIWindow *win in [[UIApplication sharedApplication].windows  reverseObjectEnumerator]) {
        if ([win isEqual: topView]) {
            continue;
        }
        if (win.windowLevel > topView.windowLevel && win.hidden != YES ) {
            topView =win;
        }
    }
    return topView;
}

```



### 三、网络请求

由于框架内仅包含一个网络请求。为了减少依赖，未选用第三方网络请求框架。而是使用系统NSURLRequest类发起网络请求。

仅仅对http头部信息进行简单封装。另外其包含一个MD5加密的简单工具类。

```objective-c
+(NSMutableURLRequest *)makeURLRequest:(NSString *)urlString secretKey:(NSString *)secretKey appCode:(NSString*) appCode modelCode:(NSString *)modelCode{
    NSString *packageName = [NSString stringWithFormat:@"%@%@",[JDCNetwork getBundleID],secretKey];
    NSString *packageNameMD5 = [packageName md5HashToLower32Bit];

    NSURL *url = [NSURL URLWithString:urlString];
    NSMutableURLRequest *request = [NSMutableURLRequest requestWithURL:url];
    [request setValue: @"application/json" forHTTPHeaderField:@"Content-Type"];
    [request setValue: @"ios" forHTTPHeaderField:@"clientType"];
    [request setValue: appCode forHTTPHeaderField:@"appCode"];
    [request setValue: packageNameMD5 forHTTPHeaderField:@"packageName"];
    
    request.HTTPBody = [NSJSONSerialization dataWithJSONObject:@{@"modelCode":modelCode} options:NSJSONWritingPrettyPrinted error:nil];
    request.HTTPMethod = @"POST";
    return request;
}

```



```objective-c
+(void)requestAuthWithSecretKey:(NSString *)secretKey
                        appCode:(NSString*) appCode
                      modelCode:(NSString *)modelCode
                        success:(nullable void (^)(NSDictionary *responseDic,NSString *urlString,NSString *jdAppId))success
                        failure:(nullable void (^)(NSError *error))failure

{
    NSURLSessionConfiguration * configuration = [NSURLSessionConfiguration defaultSessionConfiguration];
    NSURLSession *sharedSession = [NSURLSession sessionWithConfiguration:configuration];
    NSURLRequest *request = [self makeURLRequest:JDCAuthURL secretKey:secretKey appCode:appCode modelCode:modelCode];
    
    NSURLSessionDataTask *dataTask = [sharedSession dataTaskWithRequest:request completionHandler:^(NSData * _Nullable data, NSURLResponse * _Nullable response, NSError * _Nullable error) {
    }];
    [dataTask resume];
}

```



### 四、Web管理系统

在JDCURLJumpManager类中对其进行隔离，对外只暴露方法

```objective-c
/**
 普通跳转
 
 @param url 要跳转的url
 */
+ (void)jumpWithUrl:(NSString *)url;
```



Webview的类使用了，洪亮的BaseViewController和BaseWKWebViewController。其主要对进度条，导航栏进行了管理。

我主要对其进行重命名，并继承后使用。

```objective-c
#pragma mark - ICUBaseVCProtocol
- (BOOL)icu_baseVCNavigationBarHidden {
    return NO;
}

- (BOOL)icu_baseVCNavigationBarBottomLineHidden {
    return YES;
}

- (BOOL)icu_baseVCNavigationBarTranslucent {
    return NO;
}

- (UIStatusBarStyle)preferredStatusBarStyle {
    return UIStatusBarStyleDefault;
}

- (UIImage *)icu_imageForCloseButton {
    return [UIImage jdc_imageNamed:@"main_navi_webView_close_icon"];
}

- (UIImage *)icu_imageForBackButton {
    if ([self.wkWebView canGoBack]) {
        return [UIImage jdc_imageNamed:@"main_navi_webView_back_icon"];
    }else {
        return [UIImage jdc_imageNamed:@"main_navi_back_icon"];
    }
}

- (BOOL)icu_shouldResetNavigationBarState {
    return YES;
}
```

包内图片获取使用了一个小工具

```objective-c
+ (UIImage *)jdc_imageNamed:(NSString *)name {
    NSBundle *bundle = [self pickerBundle];
    JDCLogInfo(@"bundle %@",bundle);
    return [UIImage imageNamed:name
                      inBundle:bundle compatibleWithTraitCollection:nil];
}

+ (NSBundle *)pickerBundle{
    return [self icr_bundleForClass:NSClassFromString(@"JDCAuth") name:@"JDCAuthorizationSource"];
    
}
+ (NSBundle *)icr_bundleForClass:(Class)class name:(NSString *)bundleName{
    // Get the top level "bundle" which may actually be the framework
    NSBundle *mainBundle = [NSBundle bundleForClass:class];
    
    // Check to see if the resource bundle exists inside the top level bundle
    NSBundle *resourcesBundle = [NSBundle bundleWithPath:[mainBundle pathForResource:bundleName ofType:@"bundle"]];
    
    if (resourcesBundle == nil) {
        resourcesBundle = mainBundle;
    }
    return resourcesBundle;
}

```



### 五、接口暴露

为了不暴露实际实现，我们只暴露了协议JDCAuthProtocol。实际的方法调用，通过JDCAuth类代理给JDCAuthManager。代理通过消息发送机制的消息转发实现。

```objective-c
+ (id)forwardingTargetForSelector:(SEL)aSelector
{
    return [NSClassFromString(@"JDCAuthManager") class];
}

- (id)forwardingTargetForSelector:(SEL)aSelector
{
    return [NSClassFromString(@"JDCAuthManager") sharedManager];
}

```



#### 1、接口的设计

1、只需要一个管理类管理京 东登录，所以使用单例生成实例。

2、不希望用户关闭日志系统和Toast系统，分别通过单独接口来开关。

3、appScheme、secretKey 、appCode 只需要设置一次，故通过一个函数来控制。

4、每次启动App可能需要不同的modelCode，所以启动模块时同时设置modelCode。

```objective-c

@protocol JDCAuthProtocol <NSObject>

#pragma mark - 配置
/**
 返回实例方法
 
 @return 遵循JDCAuthHeader协议的实例
 */
+ (id<JDCAuthProtocol>)sharedManager;

/**
 是否开启日志系统（默认开启，只debug模式下生效）
 
 @param isLog 是否开启
 */
- (void)setIsLog:(BOOL)isLog;

/**
 是否开启Toast（默认开启）
 
 @param isToast 是否开启
 */
- (void)setIsToast:(BOOL)isToast;

/**
 配置信息
 
 @param appScheme app的Scheme
 @param secretKey 分配的秘钥
 @param appCode app的code
 */
- (void)setAppScheme:(nonnull NSString *)appScheme andSecretKey:(nonnull NSString *)secretKey andAppCode:(nonnull NSString *)appCode;

#pragma mark - 启动&退出

/***
 启动模块(请先配置信息)
 
 @param modelCode 模块的code
 */
- (void)jdUnionLoginWithModelCode:(NSString *)modelCode;

/**
 * 退出登录
 */
- (void)logout;

@end
```



#### 2、错误正确返回

错误返回通过block进行抛出。但是由于京东SDK的错误分为两类，一种是只有NSError，一种是errorMessage+replyCode。所以返回为NSError、errorMessage和replyCode。

```objective-c
/**
 成功block
 @param tokenString token字符串
 @param url url在webview中直接调用（如果使用自定义webview）
 */
typedef void (^JDCAuthSuccessBlock) (NSString * _Nullable tokenString,NSString * _Nonnull url);

/**
 错误block
 @param errorMessage 错误消息（一般为服务器返回。存在时，error为空）
 @param replyCode 错误码 （一般为服务器返回。0）
 @param error 错误消息（与errorMessage互斥）
 @param errorType 错误类型
 */
typedef void (^JDCAuthFailedBlock) (NSString  * __nullable errorMessage,NSUInteger replyCode,NSError * __nullable error,JDCAuthErrorType errorType);

```



另外还包含一些自己的错误，所以添加`JDCAuthErrorType errorType`进行区分。

```objective-c
typedef enum JDCAuthErrorType {
    JDCAuthErrorNullData = 100001,                         // 有空数据
    JDCAuthErrorGetURLFailed = 100002,                     // 获取连接失败

    JDCAuthErrorAuthorizeFailed = 200001,                  // 登录失败，服务器返回错误
    JDCAuthErrorAuthorizeError = 200002,                   // 登录失败，网络访问错误
    JDCAuthErrorAuthorizeCancle = 200003,                  // 放弃登录
    
    JDCAuthErrorValidateSigntureFailed = 300001,           // 验证失败，服务器返回错误
    JDCAuthErrorValidateSigntureError = 300002,            // 验证失败，网络访问错误

    JDCAuthErrorJumpTokenFailed = 400001,                  // APP 跳转 H5 失败
    JDCAuthErrorJumpTokenError = 400002,                   // APP 跳转 H5 出错
    JDCAuthErrorJumpTokenNoURL = 400003,                   // 没有要跳转的url

    JDCAuthErrorH5BackToAppFailed = 500001,                // 从h5页跳回 APP 失败
    JDCAuthErrorH5BackToAppError = 500002,                 // 从h5页跳回 APP 出错

    JDCAuthErrorAccountLocked = 600001,                    // 账号被锁定，需要去找回密码
    JDCAuthErrorRiskWithBindedPhone = 600002,              // 风险用户登录
    JDCAuthErrorLimitTime = 600003,                        // 账号因为安全策略被禁止登录
    JDCAuthErrorVerificationFailed = 600004,               // 登录失败(也包括刷新验证码失败、刷新票据失败等)
    JDCAuthErrorVerificationError = 600005,                // 登录出错


} JDCAuthErrorType;

```



同时通过👇方法来添加block，获取返回。

```objective-c
/**
 成功和失败的返回
 
 @param isClose 是否关闭SDK的webview（默认为NO）
 @param authSuccess 成功返回
 @param authFailed 失败返回
 */
- (void)addCompletionWebView:(BOOL) isClose sucess:(JDCAuthSuccessBlock) authSuccess failed:(JDCAuthFailedBlock) authFailed;


```





### 六、京东授权

#### 1、京东授权逻辑

如果京东已经安装则调起京东App获取a2字符串，获取成功后可以打开URL。

如果未安装则走降级策略，拼出新的URL在App浏览器中直接打开。

```objective-c
- (void)jdUnionLogin
{
    if (self.isJDAppInstall) {
        NSString *a2String = self.a2String;
        if (![a2String isEmpty]) {
            [self loginWithA2String];
            return;
        }
        [self JDStartAuthorizeScheme:_appScheme];
    } else {
        // 未安装的话 走降级策略
        JDCLogInfo(@"京东app未安装，走降级策略");
        NSString * jdH5LoginUrl = [self getJDH5LoginUrl:self.appID andURL:self.h5DirectUrl];
        if (self.authSuccessBlock) {
            self.authSuccessBlock(nil, jdH5LoginUrl);
        }
        if (!self.isCloseWebView) {
            [JDCURLJumpManager jumpWithUrl:jdH5LoginUrl];
        }
    }
}

```



#### 2、京东返回处理

为了拆分JD管理类，并保证类的功能的单一性。通过分类来处理京东授权SDK的返回。

（1）在分类JDCAuthManager+authorizeProtocol中处理登录验证回调。

```objective-c
/**
 * @brief  登录验证回调协议。
 */
@protocol WJLoginVerificationProtocol <NSObject>

```



（2）在分类JDCAuthManager+jumpTokenProtocol.h中处理H5跳转回调。

```objective-c
/**
 * @brief  APP 跳转 H5 回调协议。
 */
@protocol WJLoginJumpTokenProtocol <NSObject>
```



```objective-c
/*
* @brief h5跳回app。
 *
 */
@protocol WJLoginH5BackToAppProtocol <NSObject>
```



（3）在分类JDCAuthManager+verificationProtocol.h中处理token验证和签名接口回调。

```objective-c
/**************************
 jd 第三方授权登录业务、验证第三方token接口
 **************************/
@protocol WJLoginAuthorizeProtocol <NSObject>
```



```objective-c
/**************************
 jd 第三方授权登录业务、 验证第三方签名接口
 **************************/
@protocol WJLoginAuthorizeValidateSigntureProtocol <NSObject>
```



#### 3、AppDelegate处理

为了让用户尽量少写代码的原则。

在分类中，通过运行时的HOOK机制，HOOK了UIApplicationDelegate协议。来处理京东授权成功，成功返回的工作。

问题：用户还是需要实现`application:(UIApplication *)app openURL:(NSURL *)url options:(NSDictionary<UIApplicationOpenURLOptionsKey,id> *)options`方法才生效。

```objective-c
@implementation JDCAuthManager (AppDelegate)
+ (void)load
{
    static dispatch_once_t onceToken;
    dispatch_once(&onceToken, ^{
        KMSwizzleMethod([UIApplication class],
                        @selector(setDelegate:),
                        [JDCAuthManager class],
                        @selector(jdc_hook_setDelegate:));
    });
}

- (void)jdc_hook_setDelegate:(id<UIApplicationDelegate>)delegate{
    [self jdc_hook_setDelegate:delegate];
    
    KMSwizzleMethod([delegate class],
                    @selector(application:openURL:options:),
                    [JDCAuthManager class],
                    @selector(jdc_hook_application:openURL:options:));
}

- (BOOL)jdc_hook_application:(UIApplication *)app openURL:(NSURL *)url options:(NSDictionary<UIApplicationOpenURLOptionsKey,id> *)options
{
    NSString *openURLString = [url absoluteString];
    NSString *appSchemeFilter = [NSString stringWithFormat:@"%@://",[JDCAuthManager sharedManager].appScheme];
    if ([openURLString hasPrefix:appSchemeFilter]&&[openURLString containsString:@"typeJDAuthLogin?token"]) {
        [[JDCAuthManager sharedManager] handleOpenURL:url];
        JDCLogInfo(@"京东授权成功，成功返回");
        return YES;
    }
    return [self jdc_hook_application:app openURL:url options:options];
}

#pragma mark - Tool

void KMSwizzleMethod(Class originalCls, SEL originalSelector, Class swizzledCls, SEL swizzledSelector) {
    Method originalMethod = class_getInstanceMethod(originalCls, originalSelector);
    Method swizzledMethod = class_getInstanceMethod(swizzledCls, swizzledSelector);

    BOOL didAddMethod =
    class_addMethod(originalCls,
                    swizzledSelector,
                    method_getImplementation(swizzledMethod),
                    method_getTypeEncoding(swizzledMethod));

    if (didAddMethod) {
        Method newMethod = class_getInstanceMethod(originalCls, swizzledSelector);
        method_exchangeImplementations(originalMethod, newMethod);
    }else{
        method_exchangeImplementations(originalMethod, swizzledMethod);
    }
}

@end

```

