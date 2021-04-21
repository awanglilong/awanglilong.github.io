---
layout:     post
title:      "AFNetworking原理"
date:       2021-4-21 12:13:00
author:     "wanglilong"
header-img: "img/post-bg-2015.jpg"
catalog: true
tags:

- iOS原理

---

阅读源码千头万绪，即使是分析文章也很少有能讲解清楚。真正对读懂整个框架结构帮助最大的是设计模式。识别框架中使用的设计模式，既能学习别人使用设计模式和原理的方式，也能快速掌握整个框架脉络。并且能帮助记忆。

## 概述

整个AFNetworking框架的核心类是`AFURLSessionManager`,主要封装了系统类`NSURLSession`进行网络请求。
![](https://img2020.cnblogs.com/blog/720083/202104/720083-20210408150815303-1332270787.png)

### AFHTTPSessionManager

`AFHTTPSessionManager`继承自`AFURLSessionManager`。

是为了方便进行HTTP请求，封装的一层更友好的接口。同时设置了一下默认的`AFHTTPRequestSerializer`和`AFHTTPResponseSerializer`方便进行网络请求。`AFHTTPSessionManager`接口设计的非常简洁，遵循了设计模式里的最少知识原则。

在父类`AFURLSessionManager`中暴露了全部网络请求功能。

这个分层设计即能保证框架功能的强大，同时保证了框架的易用性。



### AFURLRequestSerialization

`AFURLRequestSerialization`主要功能是处理网络请求参数和请求头，做网络请求的前期处理。对应HTTP请求的Request，这种对应关系能方便了解HTTP协议的，快速理解使用框架。



![](https://img2020.cnblogs.com/blog/720083/202104/720083-20210408150902403-980774527.png)


同时为了能创建不同类型的请求类型，使用了工厂模式。



### AFURLResponseSerialization

`AFURLResponseSerialization`主要功能是处理网络请求响应，做网络请求返回数据处理。对应HTTP请求的Response，这种对应关系能方便了解HTTP协议的，快速理解使用框架。


![](https://img2020.cnblogs.com/blog/720083/202104/720083-20210408150917291-407615102.png)

为了处理JSON，XML，List等不同的返回数据，框架使用了策略模式，方便设置切换不同的解析策略。



###  AFSecurityPolicy

`AFSecurityPolicy` 是 `AFNetworking` 用来保证 HTTP 请求安全的类，它被 `AFURLSessionManager` 持有，如果你在 `AFURLSessionManager` 的实现文件中搜索 *self.securityPolicy*，你只会得到两条结果：

1. 初始化 `self.securityPolicy = [AFSecurityPolicy defaultPolicy]`
2. 收到连接层的验证请求时

在 API 调用上，后两者都调用了 `- [AFSecurityPolicy evaluateServerTrust:forDomain:]` 方法来判断**当前服务器是否被信任**。

```objc
- (void)URLSession:(NSURLSession *)session
              task:(NSURLSessionTask *)task
didReceiveChallenge:(NSURLAuthenticationChallenge *)challenge
 completionHandler:(void (^)(NSURLSessionAuthChallengeDisposition disposition, NSURLCredential *credential))completionHandler
{
    NSURLSessionAuthChallengeDisposition disposition = NSURLSessionAuthChallengePerformDefaultHandling;
    __block NSURLCredential *credential = nil;

    if (self.taskDidReceiveAuthenticationChallenge) {
        disposition = self.taskDidReceiveAuthenticationChallenge(session, task, challenge, &credential);
    } else {
        if ([challenge.protectionSpace.authenticationMethod isEqualToString:NSURLAuthenticationMethodServerTrust]) {
            if ([self.securityPolicy evaluateServerTrust:challenge.protectionSpace.serverTrust forDomain:challenge.protectionSpace.host]) {
                disposition = NSURLSessionAuthChallengeUseCredential;
                credential = [NSURLCredential credentialForTrust:challenge.protectionSpace.serverTrust];
            } else {
                disposition = NSURLSessionAuthChallengeRejectProtectionSpace;
            }
        } else {
            disposition = NSURLSessionAuthChallengePerformDefaultHandling;
        }
    }

    if (completionHandler) {
        completionHandler(disposition, credential);
    }
}
```

如果没有传入 `taskDidReceiveAuthenticationChallenge` block，只有在上述方法返回 `YES` 时，才会获得认证凭证 `credential`。仅仅是一个工具类。

###  AFNetworkReachabilityManager

与 `AFSecurityPolicy` 相同，`AFURLSessionManager` 对网络状态的监控是由 `AFNetworkReachabilityManager` 来负责的，通过`AFURLSessionManager`持有一个 `AFNetworkReachabilityManager` 的对象。仅仅是一个工具类。

> 真正需要判断网络状态时，仍然**需要开发者调用对应的 API 获取网络状态**。



##AFURLSessionManager

除去其它分支，真正难理解的就是`AFURLSessionManager`类做了什么。`AFURLSessionManager`是基于`NSURLSession`的封装,所以想理解`AFURLSessionManager`，首先需要理解[NSURLSession](https://www.cnblogs.com/awanglilong/p/14622435.html)的使用。

### 创建 `NSURLSessionTask`

创建 `NSURLSessionDataTask` 的实例，同时处理`NSURLSessionTask`的回调。

```objc
///-------------------------
/// @name Running Data Tasks
///-------------------------

- (NSURLSessionDataTask *)dataTaskWithRequest:(NSURLRequest *)request
                               uploadProgress:(nullable void (^)(NSProgress *uploadProgress))uploadProgressBlock
                             downloadProgress:(nullable void (^)(NSProgress *downloadProgress))downloadProgressBlock
                            completionHandler:(nullable void (^)(NSURLResponse *response, id _Nullable responseObject,  NSError * _Nullable error))completionHandler;

```

通过 `request`为Key获取Task并返回，同时调用方法`addDelegateForDataTask`。

```objective-c
- (NSURLSessionDataTask *)dataTaskWithRequest:(NSURLRequest *)request
                               uploadProgress:(nullable void (^)(NSProgress *uploadProgress)) uploadProgressBlock
                             downloadProgress:(nullable void (^)(NSProgress *downloadProgress)) downloadProgressBlock
                            completionHandler:(nullable void (^)(NSURLResponse *response, id _Nullable responseObject,  NSError * _Nullable error))completionHandler {

    NSURLSessionDataTask *dataTask = [self.session dataTaskWithRequest:request];

    [self addDelegateForDataTask:dataTask uploadProgress:uploadProgressBlock downloadProgress:downloadProgressBlock completionHandler:completionHandler];

    return dataTask;
}
```

创建`AFURLSessionManagerTaskDelegate`,并将`Delegate`信息传入`setDelegate`方法。

```objective-c
- (void)addDelegateForDataTask:(NSURLSessionDataTask *)dataTask
                uploadProgress:(nullable void (^)(NSProgress *uploadProgress)) uploadProgressBlock
              downloadProgress:(nullable void (^)(NSProgress *downloadProgress)) downloadProgressBlock
             completionHandler:(void (^)(NSURLResponse *response, id responseObject, NSError *error))completionHandler
{
    AFURLSessionManagerTaskDelegate *delegate = [[AFURLSessionManagerTaskDelegate alloc] initWithTask:dataTask];
    delegate.manager = self;
    delegate.completionHandler = completionHandler;

    dataTask.taskDescription = self.taskDescriptionForSessionTasks;
    [self setDelegate:delegate forTask:dataTask];

    delegate.uploadProgressBlock = uploadProgressBlock;
    delegate.downloadProgressBlock = downloadProgressBlock;
}
```

`AFURLSessionManager` 通过字典 `mutableTaskDelegatesKeyedByTaskIdentifier` 来存储并管理每一个 `NSURLSessionTask`，它以 `taskIdentifier` 为键存储 task。

该方法使用 `NSLock` 来保证不同线程使用 `mutableTaskDelegatesKeyedByTaskIdentifier` 时，不会出现**线程竞争**的问题。同时调用 `- setupProgressForTask:`。

```objective-c
- (void)setDelegate:(AFURLSessionManagerTaskDelegate *)delegate
            forTask:(NSURLSessionTask *)task
{
    NSParameterAssert(task);
    NSParameterAssert(delegate);

    [self.lock lock];
    self.mutableTaskDelegatesKeyedByTaskIdentifier[@(task.taskIdentifier)] = delegate;
    [self addNotificationObserverForTask:task];
    [self.lock unlock];
}
```



### 使用 `AFURLSessionManagerTaskDelegate` 管理进度

在上面我们提到过 `AFURLSessionManagerTaskDelegate` 类，它主要为 task 提供**进度管理**功能，并在 task 结束时**回调**， 也就是调用在 `- [AFURLSessionManager dataTaskWithRequest:uploadProgress:downloadProgress:completionHandler:]` 等方法中传入的 `completionHandler`。



#### 代理方法 `URLSession:task:didCompleteWithError:`

在每一个 `NSURLSessionTask` 结束时，都会在代理方法 `URLSession:task:didCompleteWithError:` 中：

1. 调用传入的 `completionHander` block
2. 发出 `AFNetworkingTaskDidCompleteNotification` 通知

```objc
- (void)URLSession:(__unused NSURLSession *)session
              task:(NSURLSessionTask *)task
didCompleteWithError:(NSError *)error{
    #1：获取数据, 存储 `responseSerializer` 和 `downloadFileURL`
    if (error) {
    	#2：在存在错误时调用 `completionHandler`
    } else {
			#3：调用 `completionHandler`
    }
}
```

这是整个代理方法的骨架，先看一下最简单的第一部分代码：

```objc
__block NSMutableDictionary *userInfo = [NSMutableDictionary dictionary];
userInfo[AFNetworkingTaskDidCompleteResponseSerializerKey] = manager.responseSerializer;

//Performance Improvement from #2672
NSData *data = nil;
if (self.mutableData) {
   data = [self.mutableData copy];
   //We no longer need the reference, so nil it out to gain back some memory.
   self.mutableData = nil;
}

if (self.downloadFileURL) {
   userInfo[AFNetworkingTaskDidCompleteAssetPathKey] = self.downloadFileURL;
} else if (data) {
   userInfo[AFNetworkingTaskDidCompleteResponseDataKey] = data;
}
```

这部分代码从 `mutableData` 中取出了数据，设置了 `userInfo`。

```objc
userInfo[AFNetworkingTaskDidCompleteErrorKey] = error;

dispatch_group_async(manager.completionGroup ?: url_session_manager_completion_group(), manager.completionQueue ?: dispatch_get_main_queue(), ^{
    if (self.completionHandler) {
        self.completionHandler(task.response, responseObject, error);
    }

    dispatch_async(dispatch_get_main_queue(), ^{
        [[NSNotificationCenter defaultCenter] postNotificationName:AFNetworkingTaskDidCompleteNotification object:task userInfo:userInfo];
    });
});
```

如果当前 `manager` 持有 `completionGroup` 或者 `completionQueue` 就使用它们。否则会创建一个 `dispatch_group_t` 并在主线程中调用 `completionHandler` 并发送通知(在主线程中)。

如果在执行当前 task 时没有遇到错误，那么先**对数据进行序列化**，然后同样调用 block 并发送通知。

```objc
dispatch_async(url_session_manager_processing_queue(), ^{
    NSError *serializationError = nil;
    responseObject = [manager.responseSerializer responseObjectForResponse:task.response data:data error:&serializationError];

    if (self.downloadFileURL) {
        responseObject = self.downloadFileURL;
    }

    if (responseObject) {
        userInfo[AFNetworkingTaskDidCompleteSerializedResponseKey] = responseObject;
    }

    if (serializationError) {
        userInfo[AFNetworkingTaskDidCompleteErrorKey] = serializationError;
    }

    dispatch_group_async(manager.completionGroup ?: url_session_manager_completion_group(), manager.completionQueue ?: dispatch_get_main_queue(), ^{
        if (self.completionHandler) {
            self.completionHandler(task.response, responseObject, serializationError);
        }

        dispatch_async(dispatch_get_main_queue(), ^{
            [[NSNotificationCenter defaultCenter] postNotificationName:AFNetworkingTaskDidCompleteNotification object:task userInfo:userInfo];
        });
    });
});
```

#### 代理方法 `URLSession:dataTask:didReceiveData:` 和 `- URLSession:downloadTask:didFinishDownloadingToURL:`

这两个代理方法分别会在收到数据或者完成下载对应文件时调用，作用分别是为 `mutableData` 追加数据和处理下载的文件：

```objc
- (void)URLSession:(__unused NSURLSession *)session
          dataTask:(__unused NSURLSessionDataTask *)dataTask
    didReceiveData:(NSData *)data
{
    [self.mutableData appendData:data];
}

- (void)URLSession:(NSURLSession *)session
      downloadTask:(NSURLSessionDownloadTask *)downloadTask
didFinishDownloadingToURL:(NSURL *)location
{
    NSError *fileManagerError = nil;
    self.downloadFileURL = nil;

    if (self.downloadTaskDidFinishDownloading) {
        self.downloadFileURL = self.downloadTaskDidFinishDownloading(session, downloadTask, location);
        if (self.downloadFileURL) {
            [[NSFileManager defaultManager] moveItemAtURL:location toURL:self.downloadFileURL error:&fileManagerError];

            if (fileManagerError) {
                [[NSNotificationCenter defaultCenter] postNotificationName:AFURLSessionDownloadTaskDidFailToMoveFileNotification object:downloadTask userInfo:fileManagerError.userInfo];
            }
        }
    }
}
```

### 网络请求回调转换成`Block`

这类里相当一部分代码，是为了将网络请求delegate回调转换为block。这样可以将每个网络请求的区分开，不需要用户自己判断。但不知道为何不用系统方法的block返回，可能是系统的block返回设计的过于简单。

```objc
///---------------------------------
/// @name Getting Progress for Tasks
///---------------------------------

- (nullable NSProgress *)uploadProgressForTask:(NSURLSessionTask *)task;

- (nullable NSProgress *)downloadProgressForTask:(NSURLSessionTask *)task;

///-----------------------------------------
/// @name Setting Session Delegate Callbacks
///-----------------------------------------
- (void)setSessionDidBecomeInvalidBlock:(nullable void (^)(NSURLSession *session, NSError *error))block;

- (void)setSessionDidReceiveAuthenticationChallengeBlock:(nullable NSURLSessionAuthChallengeDisposition (^)(NSURLSession *session, NSURLAuthenticationChallenge *challenge, NSURLCredential * _Nullable __autoreleasing * _Nullable credential))block;

///--------------------------------------
/// @name Setting Task Delegate Callbacks
///--------------------------------------

- (void)setTaskNeedNewBodyStreamBlock:(nullable NSInputStream * (^)(NSURLSession *session, NSURLSessionTask *task))block;

- (void)setTaskWillPerformHTTPRedirectionBlock:(nullable NSURLRequest * _Nullable (^)(NSURLSession *session, NSURLSessionTask *task, NSURLResponse *response, NSURLRequest *request))block;

- (void)setAuthenticationChallengeHandler:(id (^)(NSURLSession *session, NSURLSessionTask *task, NSURLAuthenticationChallenge *challenge, void (^completionHandler)(NSURLSessionAuthChallengeDisposition , NSURLCredential * _Nullable)))authenticationChallengeHandler;

- (void)setTaskDidSendBodyDataBlock:(nullable void (^)(NSURLSession *session, NSURLSessionTask *task, int64_t bytesSent, int64_t totalBytesSent, int64_t totalBytesExpectedToSend))block;

- (void)setTaskDidCompleteBlock:(nullable void (^)(NSURLSession *session, NSURLSessionTask *task, NSError * _Nullable error))block;

#if AF_CAN_INCLUDE_SESSION_TASK_METRICS
- (void)setTaskDidFinishCollectingMetricsBlock:(nullable void (^)(NSURLSession *session, NSURLSessionTask *task, NSURLSessionTaskMetrics * _Nullable metrics))block AF_API_AVAILABLE(ios(10), macosx(10.12), watchos(3), tvos(10));
#endif
///-------------------------------------------
/// @name Setting Data Task Delegate Callbacks
///-------------------------------------------

- (void)setDataTaskDidReceiveResponseBlock:(nullable NSURLSessionResponseDisposition (^)(NSURLSession *session, NSURLSessionDataTask *dataTask, NSURLResponse *response))block;

- (void)setDataTaskDidBecomeDownloadTaskBlock:(nullable void (^)(NSURLSession *session, NSURLSessionDataTask *dataTask, NSURLSessionDownloadTask *downloadTask))block;

- (void)setDataTaskDidReceiveDataBlock:(nullable void (^)(NSURLSession *session, NSURLSessionDataTask *dataTask, NSData *data))block;

- (void)setDataTaskWillCacheResponseBlock:(nullable NSCachedURLResponse * (^)(NSURLSession *session, NSURLSessionDataTask *dataTask, NSCachedURLResponse *proposedResponse))block;

- (void)setDidFinishEventsForBackgroundURLSessionBlock:(nullable void (^)(NSURLSession *session))block AF_API_UNAVAILABLE(macos);

///-----------------------------------------------
/// @name Setting Download Task Delegate Callbacks
///-----------------------------------------------
- (void)setDownloadTaskDidFinishDownloadingBlock:(nullable NSURL * _Nullable  (^)(NSURLSession *session, NSURLSessionDownloadTask *downloadTask, NSURL *location))block;

- (void)setDownloadTaskDidWriteDataBlock:(nullable void (^)(NSURLSession *session, NSURLSessionDownloadTask *downloadTask, int64_t bytesWritten, int64_t totalBytesWritten, int64_t totalBytesExpectedToWrite))block;

- (void)setDownloadTaskDidResumeBlock:(nullable void (^)(NSURLSession *session, NSURLSessionDownloadTask *downloadTask, int64_t fileOffset, int64_t expectedTotalBytes))block;

@end
```



## ICNetworkingSDK

`ICNetworkingSDK`是自定义网络请求框架。

#### ICNHTTPManager

等同于`AFHTTPSessionManager`,主要对网络请求进行一次封装。方便用户进行网络请求。

同时因为业务需要，管理一份统一的请求头和请求body。方便对网络请求同一设置。

```objective-c
@property (nonatomic, strong) NSDictionary *dictCommonBodyParams;
@property (nonatomic, strong) NSDictionary *dictCommonHeaderParams;

```

#### ICNURLManager

封装`AFNetworking`的`AFURLSessionManager`,   加密和请求结果字典转模型，使用策略模式，可以动态指定其解密策略方式和字典转模型。

```objective-c
/**
 加解密策略
 */
@property (nonatomic, strong) Class<ICNURLSessionSecurityPolicy> sessionSecurityPolicy;

/**
 response的处理策略
 */
@property (nonatomic, strong) Class<ICNURLResponsePolicy> responsePolicy;

```

网络请求前组装`NSMutableURLRequest`

```objective-c
- (NSURLSessionUploadTask *)uploadTaskWithStreamedRequest:(ICNURLRequest *)icnRequest
                                                 progress:(void (^)(NSProgress *uploadProgress)) uploadProgressBlock
                                        completionHandler:(void (^)(NSURLResponse *response, id responseObject, NSError *error))completionHandler {
    //icnRequest完成组包逻辑
    NSMutableURLRequest *request = [icnRequest formedRequest];
    
    if (icnRequest.parameters) {
        [NSURLProtocol setProperty:icnRequest.parameters forKey:@"NSURLRequestParametersMetadataKey" inRequest:request];
    }
    
    __weak typeof(self) weakSelf = self;
    __block NSURLSessionUploadTask *dataTask = nil;
    dataTask = [self.sessionManager uploadTaskWithStreamedRequest:request
                                                         progress:uploadProgressBlock
                                                completionHandler:^(NSURLResponse * _Nonnull response, id  _Nullable responseObject, NSError * _Nullable error) {
        [weakSelf request:icnRequest didResponse:response responseObject:responseObject error:error completionHandler:completionHandler];
    }];
    
    return dataTask;
}
```

网络请求完成后，统一进行解密，并序列化。

```objective-c
- (void)request:(ICNURLRequest *)icnRequest
            didResponse:(NSURLResponse *)response
         responseObject:(id)responseObject
                  error:(NSError *)error
      completionHandler:(void (^)(NSURLResponse *response, id responseObject, NSError *error))completionHandler {
    if (completionHandler) {
        if (responseObject) {
            //解密response
            if (icnRequest.sessionSecurityPolicy && [icnRequest.sessionSecurityPolicy respondsToSelector:@selector(decryptResponseObject:)]) {
                responseObject = [icnRequest.sessionSecurityPolicy decryptResponseObject:responseObject];
            }
            
            //请求结果parse逻辑
            responseObject = [icnRequest parseResponseObject:responseObject error:&error];
        }
        if (error && [icnRequest respondsToSelector:@selector(getResponseFromCacheResponseObject:error:)]) {
            responseObject = [icnRequest getResponseFromCacheResponseObject:responseObject error:&error];
        }
        
        if ([NSThread isMainThread]) {
            if (completionHandler) {
                completionHandler(response, responseObject, error);
            }
        } else {
            dispatch_async(dispatch_get_main_queue(), ^{
                if (completionHandler) {
                    completionHandler(response, responseObject, error);
                }
            });
        }
    }
}
```



#### ICNURLRequest

1、请求Header组装。

2、负责请求的组装，body组装并进行加密。并可指定加密方式。

3、还是调用`AFHTTPRequestSerializer`的方法`requestWithMethod`进行组装。

```objective-c

- (NSDictionary *)formedRequestHeaderParams {
    NSMutableDictionary *dictBodyParams = [NSMutableDictionary dictionaryWithDictionary:self.commonBodyParams];
    if ([self.parameters isKindOfClass:[NSDictionary class]]) {
        [dictBodyParams addEntriesFromDictionary:self.parameters];
    }
    
    NSDictionary *dictSecurityHeaderParams = nil;
    if (self.sessionSecurityPolicy) {
        //预处理header params
        if ([self.sessionSecurityPolicy respondsToSelector:@selector(sessionSecurityHeaderParamsWithMethod:bodyParams:)]) {
            dictSecurityHeaderParams = [self.sessionSecurityPolicy sessionSecurityHeaderParamsWithMethod:self.method bodyParams:[dictBodyParams copy]];
        }
    }
    
    NSMutableDictionary *dictHeaderParams = [NSMutableDictionary dictionaryWithDictionary:self.commonHeaderParams];
    [dictHeaderParams addEntriesFromDictionary:dictSecurityHeaderParams];
    if (dictHeaderParams.count > 0) {
        [dictHeaderParams enumerateKeysAndObjectsUsingBlock:^(id  _Nonnull key, id  _Nonnull obj, BOOL * _Nonnull stop) {
            [self.requestSerializer setValue:obj forHTTPHeaderField:key];
        }];
    }
    
    return dictHeaderParams;
}

- (NSURLRequest *)formedRequest {
    [self formedRequestHeaderParams];
    NSDictionary *dictBodyParams = [self formedRequestBodyParams];
    
    NSString *path = [NSString stringWithFormat:@"%@%@", self.baseURL.path?self.baseURL:@"", self.path];
    NSError *serializationError = nil;
    NSMutableURLRequest *request = [self.requestSerializer requestWithMethod:self.method URLString:[[NSURL URLWithString:path relativeToURL:self.baseURL] absoluteString] parameters:dictBodyParams error:&serializationError];
    if (serializationError) {
        return nil;
    }
    
    return request;
}
```



#### ICNCodeDataResponse

与`AFURLResponseSerialization`不同。`ICNCodeDataResponse`是数据解析模型的Class，是返回数据字典转模型要转换到的模型。



参考 [AFNetworking 概述]([https://draveness.me/afnetworking1/)
