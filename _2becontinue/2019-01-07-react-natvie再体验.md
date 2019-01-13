---
layout:     post
title:   React-Native再体验（未完  mark)
subtitle:  面试的时候，有一家公司用rn开发了很多页面，准备再拿起rn再写一些。
date:     2019-01-07
author:     Pony
header-img: img/post-bg-cook.jpg
catalog: true
tags:
    - RN
---
## 搭建环境
RN开发环境依赖的有：
1. Node
2. Watchman
3. React Native Command Line
4. JDK
5. Android Studio

### Node 
在安装Node之前，先检查是否之前有下载过。通过
>brew list

进行检查，如果列表里面有就说明下载了。
推荐一个可视化查看Homebrew安装信息的工具。
![](https://ws3.sinaimg.cn/large/006tNc79ly1fyyfpxhyq3j30vo0mudra.jpg)
`Cakebrew`的安装指令：
>brew cask install cakebrew

这样就可以简单明了的了解到homebrew安装的工具信息了。

如果没有安装`node`，可以通过：
>brew install node

进行安装。如果已经安装`node`，请确保版本号在`v8.3`以上。

安装完`node`,建议设置`npm`镜像以加速后面的过程：
>npm config set registry https://registry.npm.taobao.org --global

>npm config set disturl https://npm.taobao.org/dist --global

###Watchman
Watchman是由`facebook`提供，用来**件事文件系统变化的工具**，安装它可以`提高开发时候的性能(packager可以快速捕获文件的变化，从而实时刷新)。`

### Yarn
Yarn是`facebook`提供用于替代`npm`的工具。**可以加速`node`模块的下载**

### rn的命令工具 (react-native-cli)
rn命令行工具用于执行`创建`,`初始化`,`更新项目`,`运行打包服务`,等任务。

安装指令：
>npm install -g yarn react-native-cli

安装完后设置镜像：
>yarn config set registry https://registry.npm.taobao.org --global
yarn config set disturl https://npm.taobao.org/dist --global

安装完`yarn`后，就可以用`yarn`替代`npm`了，例如用`yarn`替代`npm install`指令，用`yarn add 某三方库名` 替代`npm install xxx`。

### JDK
安装jdk，就不说了。
### android studio
不说了。
#### 安装SDK
#### 配置环境变量
```java
>echo $HOME
>~/.bash_profile
>export ANDROID_HOME=$HOME/Library/Android/sdk
export PATH=$PATH:$ANDROID_HOME/tools
export PATH=$PATH:$ANDROID_HOME/tools/bin
export PATH=$PATH:$ANDROID_HOME/platform-tools
export PATH=$PATH:$ANDROID_HOME/emulator
```
编辑完记得
```java
source $HOME/.bash_profile
```
是环境变量立即生效。

好了整体的环境搭建已经搞定。next,创建项目。

##新建项目
### 创建项目
通过rn命令行创建新项目
```java
react-native init XXXProjectName
```
可以在后面添加版本号，且版本号必须精确2个小数点:
```java
react-native init MyApp --version 0.44.3
```

### 运行
1. 切换到项目目录
```java
cd XXXXProjectName
```
2. 运行项目
 ```java
 react-native run-android
 ```
 如果配置的都可以
 就可以直接运行，如果报错，请参照报错原因。
 成功效果图：
 ![](https://reactnative.cn/docs/assets/GettingStartediOSSuccess.png)