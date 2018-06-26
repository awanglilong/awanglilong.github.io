---
layout:     post
title:      "iOS自动打包"
subtitle:   " \"就是懒呀\""
date:       2018-06-26 14:00:00
author:     "wanglilong"
header-img: "img/post-bg-2015.jpg"
catalog: true
tags:
    - iOS 开发
---
# iOS自动打包

### 概述
项目在测试阶段需要频繁打包给测试人员，对于这些固定化的操作我们可以使用自动化的手段去解决，将时间放在有意义的事情上。

脚本将分为三个步骤

1.`xcodebuild archive`生成ProjectName.xcarchive文件

2.`xcodebuild -exportArchive`将1步骤中的.xcarchive生成ipa安装包

3.上传ipa包到平台

### xcodebuild打包
#### 生成.xcarchive文件
`-workspace`指定想要编译的工程

`-scheme`指定对应的scheme

`-allowProvisioningUpdates`允许自动获取证书，需要在Xcode上勾选Automatically manage signing

`-configuration`指定配置文件

`-archivePath`指定生成ProjectName.xcarchive文件的路径

```shell
#xcodebuild  clean，archive出.xcarchive
xcodebuild  \
-workspace "$Workspace.xcworkspace" \
-scheme $Project_Name_TEST \
-configuration $Configuration \
-archivePath build/$Project_Name_TEST-adhoc.xcarchive \
clean \
archive \
build \
-allowProvisioningUpdates\

```

#### 生成ipa安装包

`-archivePath `指定ProjectName.xcarchive文件的路径

```objc
xcodebuild 
-exportArchive 
-archivePath build/$Project_Name_TEST-adhoc.xcarchive 
-exportOptionsPlist ${ADHOCExportOptionsPlist} 
-exportPath ~/Desktop/$Project_Name_TEST 
-allowProvisioningUpdates
```

### 分发
#### 上传到蒲公英
file 指定ipa所在位置
_api_key 是在蒲公英上apiKey

```objc
#执行上传至蒲公英的命令
curl -F  file=@${IPA_PATH}  -F  _api_key=${apiKey}  https://www.pgyer.com/apiv2/app/upload


```


#### 上传到AppStore
使用altool

```objc
#altool 简单示例
#validate
"$altoolPath" --validate-app -f "$ipaPath" -u "$appleid" -p "$applepassword" -t ios --output-format xml

#upload
"$altoolPath" --upload-app -f "$ipaPath" -u "$appleid" -p "$applepassword" -t ios --output-format xml

```


[自动上传脚本](https://pan.baidu.com/s/1qiPfOgtxLVW5Xij23z85TA)

参考
[iOS自动签名打包](https://www.cnblogs.com/CoderHong/p/8931562.html)

[Xcode一键发布到AppStore](https://blog.csdn.net/gukong/article/details/51578618)

[iOS开发系列-自动化分发测试打包](https://www.cnblogs.com/CoderHong/p/8931562.html)
