---
layout:     post
title:  A使用Travis-CI构建Android项目并自动打包部署到GitHub-Release
subtitle: 基本流程
date:     2019-09-24
author:     Pony
header-img: img/post-bg-mayday-bubble.jpg
catalog: true
tags:
    - Travis GitHub Android
---



# 使用Travis-CI构建Android项目并自动打包部署到GitHub-Release

## 前提准备工作

###  配置签名信息

因为`travis`是通过执行`./gradlew assembleRelease` 命令来构建apk文件，所以需要在`APP build.gradle`添加相关信息：

![image-20190924094047232](https://raw.githubusercontent.com/liaopen123/ImageRepo/master/YCgXFq.jpg)



**确认storeFile文件存放位置无误**

## 开始接入

在项目的更目录添加文件`.travis.yml`

文件格式内容固定，直接copy即可：

```yml
# .travis.yml 文件

# 声明构建语言环境
language: android
android:        # 配置信息
  components:
    # 你可能需要修改下面两项的版本
    - build-tools-29.0.2
    - android-28
    # Android Support Repository
    - extra-android-m2repository
    # Support Library
    - extra-android-support
script:
  # 生成 APK
  - ./gradlew assembleRelease

# 部署
deploy:
  # 部署到GitHub Release。
  # 除此之外，Travis CI还支持发布到fir.im、AWS、Google App Engine等
  provider: releases
  # Github oauth token
  api_key: "$GIT_TOKEN"
  # 部署文件路径；对于Android就部署生成的 apk 文件
  file:
    - app/build/outputs/apk/release/app-release.apk
  # 避免 Travis CI在部署之前清空生成的APK文件
  skip_cleanup: true
  # 发布时机
  on:
    # tags设置为true表示只有在有tag的情况下才部署
    tags: true
    all_branches: true
```



![image-20190924095221991](https://raw.githubusercontent.com/liaopen123/ImageRepo/master/2.png)



### 需要自行修改的地方：

##### 1.build-tools-xxx版本号，android-xx android平台号

这一块的值和`app build.gradle`的值相对应即可。

```groovy
android {
    compileSdkVersion 28
    buildToolsVersion "29.0.2"
    }
```

demo的版本号就改成为

```yml
- build-tools-29.0.2
- android-28
```

##### 2.申请github_key：

1.进入github首页，点击头像下方的`setting`:

![image-20190924100018444](https://raw.githubusercontent.com/liaopen123/ImageRepo/master/3.png)

2.点击`Personal settings`中的`Developer settings`:

![image-20190924100107298](https://raw.githubusercontent.com/liaopen123/ImageRepo/master/3.png)



3.选择`Personal access tokens`:![image-20190924100206534](https://raw.githubusercontent.com/liaopen123/ImageRepo/master/5.png)



4.点击`Generate new token`创建新的token![image-20190924100245812](/Users/pony/Desktop/travis/7.png)



5.输入`note名称`(随便写不影响)，**全部勾选下方的勾选框**(避免travis权限不足)。

![image-20190924100245812](https://raw.githubusercontent.com/liaopen123/ImageRepo/master/7.png)

6.进入travis官网，关联github，1.进入用户设置界面，2.同步账号，3.然后搜索到需要打包的github项目，4，打开开关，5，进入设置。

![image-20190924102235074](https://raw.githubusercontent.com/liaopen123/ImageRepo/master/8.png)

7.进入设置界面后，在环境变量一栏，填写GIT_TOKEN所对应的值，这样就可以，在build时候，动态替换掉`api_key`的值了。防止token暴露:

![image-20190924102649680](https://raw.githubusercontent.com/liaopen123/ImageRepo/master/9.png)





好了这样子就成功的把github项目与travis相关联了。





## 提交代码

现在我们就可以提交代码了，但是单纯的`git commit`和`git push`是不能在release界面生成安装包的。

它还需要tag指令,否则会报错:

```shell
Skipping a deployment with the releases provider because this is not a tagged commit
```

正确的完整命令:

```shell
git add .
git commit -m "注释"
git push origin


git tag -a v1.0.0 -m "1.第一个版本"  #每次版本号要递增
git push origin --tags
```



提交完后，travis会自动检测，并进行build，build成功后，即可在项目首页发现安装包，大功告成~~~

![image-20190924103326409](https://raw.githubusercontent.com/liaopen123/ImageRepo/master/10.png)







参考资料:

1. [使用Travis CI自动打包APK，并发布到fir](<http://www.voidcn.com/article/p-rirvihnb-bpu.html>)

2. [基于Travis CI搭建Android自动打包发布工作流](<https://www.jianshu.com/p/76de282093b9>)

3. [使用Travis-CI构建Android项目并自动打包部署到GitHub-Release](<https://www.jianshu.com/p/7517255bc056?utm_campaign=maleskine&utm_content=note&utm_medium=seo_notes&utm_source=recommendation>)

   