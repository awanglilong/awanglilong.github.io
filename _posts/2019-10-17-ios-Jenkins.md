---
layout:     post
title:      "iOS自动化部署Jenkins"
subtitle:   " \"自动化部署\""
date:       2019-10-17 10:13:00
author:     "wanglilong"
header-img: "img/post-bg-2015.jpg"
catalog: true
tags:
- iOS 开发
---
# iOS自动化部署Jenkins

## 一、前言

之前写过一个自动化打包的shell脚本，但是老大觉得自动化程度还是不高，建议用Jenkins来搞。所以就研究一下Jenkins的自动化部署。



## 二、Jenkins安装

安装brew

```shell
/usr/bin/ruby -e "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/master/install)"
```



由于jenkins依赖于java8（之上），所以需要先安装java

```shell
brew cask install homebrew/cask-versions/adoptopenjdk8
```



然后使用brew安装jenkins

```shell
brew install jenkins
```



## 三、启动Jenkins

命令

```shell
service jenkins start //启动
service jenkins stop  //停止
service jenkins restart //重启
```



登录网址管理Jenkins

```
 http://localhost:8080/ 
```



## 四、插件

一般会选择Jenkins推荐安装的插件

另外一般会另外安装插件

```shell
Keychains and Provisioning Profiles Management//方便管理打包证书
Xcode integration//由于需要使用Xcode编译环境，因此必须要安装插件
```



但是这两个插件我好像都木有用到



## 五、组件化项目自动化打包

由于项目用了RN，并且用了组件化。普通方式的自动化打包无法使用。所以只能使用流水线的方式来打包。

### (1)、配置环境变量

组件化是使用pod进行管理的，所以需要用pod命令

```shell
pod install
```

如果没有配置环境变量会提示 pod: command not found。所以需要配置环境变量

<img src="/img/post-ios-Jenkins/环境配置1.jpg" alt="环境配置1" style="zoom:50%;" />



然后选择Environment variables。其中，PATH是固定的，值是在终端输入:`$echo $PATH`命令获取，将输入命令后得到的值粘贴过来就可以了。

<img src="/img/post-ios-Jenkins/环境配置2.png" style="zoom:50%;" />

### (2)、配置Keychains and Provisioning Profiles Management

见（参考2）用流水方式打包好像并没有用

###(3)、流水线

#### 首先、创建流水线项目

<img src="/img/post-ios-Jenkins/流水线1.png" style="zoom:50%;" />



#### 设置执行脚本的时间

<img src="/img/post-ios-Jenkins/流水线2.png" alt="流水线2" style="zoom:50%;" />

#### 设置脚本

<img src="/img/post-ios-Jenkins/流水线3.png" style="zoom:50%;" />

#### 脚本语法

点击上图中的[Pipeline Syntax](http://localhost:8080/job/Hello/pipeline-syntax),可以通过生成器生成脚本



#### 组件化项目打包方式

>首先：拉下来iOS各个模块的代码
>
>然后：pod install
>
>最后：将项目打包并上传



1、从git服务器上拉下代码到指定的文件夹

```shell
node {
    checkout([$class: 'GitSCM', branches: [[name: '*/dev']], doGenerateSubmoduleConfigurations: false, extensions: [[$class: 'RelativeTargetDirectory', relativeTargetDir: 'HelloBeijing/ios/HelloBeijing_iOS/']], submoduleCfg: [], userRemoteConfigs: [[credentialsId: 'ccb75a88-4fe2-44e7-846d-0fea1965cef4', url: 'http://git.jd.com/helloBJ/HelloBeijing_iOS.git']]])
}

```

branches:为分支设置

relativeTargetDir：指定导出的路径，根目录为项目目录

url：为git代码网址

credentialsId：为用户名密码生成的（目前只知道代码生成器的方式生成）



2、进行pod

```shell
node {
    sh label: '', 
    script: '''
cd /Users/用户名/.jenkins/workspace/HelloBeijingAll/HelloBeijing/ios/HelloBeijing_iOS
pod install
		'''
}

```



3、最后shell脚本打包并上传

```shell
node {
    sh label: '', script: '''
cd /Users/wanglilong3/.jenkins/workspace/HelloBeijingAll/HelloBeijing/ios/HelloBeijing_iOS
#工程名字(Target名字)
Project_Name_TEST="******"
#workspace的名字
Workspace_Name="******"
#配置环境，Release或者Debug,默认release
Configuration="Release"

#加载各个版本的plist文件
ADHOCExportOptionsPlist=./ADHOCExportOptionsPlist.plist
ADHOCExportOptionsPlist=${ADHOCExportOptionsPlist}

#adhoc脚本
xcodebuild -workspace $Workspace_Name.xcworkspace -scheme $Project_Name_TEST -configuration $Configuration -archivePath build/$Project_Name_TEST-adhoc.xcarchive clean archive build -allowProvisioningUpdates
xcodebuild  -exportArchive -archivePath build/$Project_Name_TEST-adhoc.xcarchive -exportOptionsPlist ${ADHOCExportOptionsPlist} -exportPath ~/Desktop/$Project_Name_TEST -allowProvisioningUpdates

#执行上传至蒲公英的命令
apiKey="蒲公英的apiKey"
IPA_PATH=~/Desktop/$Project_Name_TEST/$Project_Name_TEST.ipa
curl -F  file=@${IPA_PATH}  -F  _api_key=${apiKey}  https://www.pgyer.com/apiv2/app/upload
'''
}
```

shell脚本写法请见[iOS自动打包](https://awanglilong.github.io/2018/06/26/iOSAutomaticPackaging/)



4、判断git上代码是否有更新

根据**全局变量参考**上说明，可根据currentBuild的changeSets，来判断git是否有更新。

```
currentBuild

changeSets
a list of changesets coming from distinct SCM checkouts; each has a kind and is a list of commits; each commit has a commitId, timestamp, msg, author, and affectedFiles each of which has an editType and path; the value will not generally be Serializable so you may only access it inside a method marked @NonCPS
```

代码如下

```
node {
     stage('build') {
          if (currentBuild.changeSets.isEmpty()) {
                echo 'no change'
          } else {
                echo 'build'
          }      
      }
}
```



## 六：参考

1、[占坑！利用 JenKins 持续集成 iOS 项目时遇到的问题](https://juejin.im/entry/5b5e7bdb6fb9a04fcc44af91)



2、[使用jenkins实现xcode自动打包](https://www.jianshu.com/p/3668979476ad)



3、最终脚本

```shell
pipeline {
    agent any

    stages {
        stage('checkout') {
            steps {
                checkout([$class: 'GitSCM', branches: [[name: '*/dev']], doGenerateSubmoduleConfigurations: false, extensions: [[$class: 'RelativeTargetDirectory', relativeTargetDir: 'HelloBeijing/ios/HelloBeijing_iOS/']], submoduleCfg: [], userRemoteConfigs: [[credentialsId: 'ccb75a88-4fe2-44e7-846d-0fea1965cef4', url: 'http://git.jd.com/helloBJ/HelloBeijing_iOS.git']]])
                echo 'ios'

                checkout([$class: 'GitSCM', branches: [[name: '*/dev']], doGenerateSubmoduleConfigurations: false, extensions: [[$class: 'RelativeTargetDirectory', relativeTargetDir: 'iCityAppSDK/ICFoundationSDK/']], submoduleCfg: [], userRemoteConfigs: [[credentialsId: 'ccb75a88-4fe2-44e7-846d-0fea1965cef4', url: 'http://git.jd.com/iCityAppSDK/ICFoundationSDK.git']]])
                echo 'ICFoundationSDK'
                
                checkout([$class: 'GitSCM', branches: [[name: '*/dev']], doGenerateSubmoduleConfigurations: false, extensions: [[$class: 'RelativeTargetDirectory', relativeTargetDir: 'iCityAppSDK/ICNetworkingSDK']], submoduleCfg: [], userRemoteConfigs: [[credentialsId: 'ccb75a88-4fe2-44e7-846d-0fea1965cef4', url: 'http://git.jd.com/iCityAppSDK/ICNetworkingSDK.git']]])
                echo 'ICNetworkingSDK'

                checkout([$class: 'GitSCM', branches: [[name: '*/dev']], doGenerateSubmoduleConfigurations: false, extensions: [[$class: 'RelativeTargetDirectory', relativeTargetDir: 'iCityAppSDK/ICUIKit']], submoduleCfg: [], userRemoteConfigs: [[credentialsId: 'ccb75a88-4fe2-44e7-846d-0fea1965cef4', url: 'http://git.jd.com/iCityAppSDK/ICUIKit.git']]])
                echo 'ICUIKit'
            }
        }
        
        stage('pod') {
            steps {
    		    sh label: '', 
   		        script: '''
			        cd /Users/wanglilong3/.jenkins/workspace/HelloBeijingAll/HelloBeijing/ios/HelloBeijing_iOS
		        	pod install
		        '''
		        echo 'pod'
            }
        }
    }
}
node {
     stage('build') {
          if (currentBuild.changeSets.isEmpty()) {
                echo 'no change'
          } else {
                sh label: '', 
   		        	script: '''
			        	cd /Users/wanglilong3/.jenkins/workspace/HelloBeijingAll/HelloBeijing/ios/HelloBeijing_iOS
		        		sh xcode_Jenkins.sh
		        		'''
                echo 'build'
          }      
      }
}

```



4、代码判断

判断Git是否有新的提交，如果有的话拉取代码构建，如果没有不构建。Jenkins需要安装插件Conditional BuildStep。但是在流水线中无法生效。

```shell
#!/bin/bash
if [ ! $GIT_PREVIOUS_SUCCESSFUL_COMMIT ];then
    echo "GIT_PREVIOUS_SUCCESSFUL_COMMIT is not exists."
    exit 0
else
    echo "GIT_COMMIT=[$GIT_COMMIT],GIT_PREVIOUS_SUCCESSFUL_COMMIT=[$GIT_PREVIOUS_SUCCESSFUL_COMMIT]"
    if [ $GIT_PREVIOUS_SUCCESSFUL_COMMIT == $GIT_COMMIT ];then
        echo "GIT_COMMIT is equals to GIT_PREVIOUS_SUCCESSFUL_COMMIT,skip build."
        exit -1
    else
        echo "GIT_COMMIT is not equals to GIT_PREVIOUS_SUCCESSFUL_COMMIT"
        exit 0
    fi
fi
```

