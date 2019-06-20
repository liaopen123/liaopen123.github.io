---
layout:     post
title:  AOP之APT
subtitle: APT主要作用是通过注解，在编译时期动态生成代码。
date:     2019-05-14
author:     Pony
header-img: img/post-bg-mayday-bubble.jpg
catalog: true
tags:
    - AOP  APT
---

![](https://ws4.sinaimg.cn/large/006tNc79ly1g2zgnxb7eoj30f00bu75e.jpg)

先看这张图，知道AOP三剑客的切入点时间。

APT:在**编译时期**动态生成.java文件。

AspectJ:在**编译时期**直接修改原代码生成的.class文件。

Javassist:在**运行时期**操作java字节码,通过手工编写字节码，再由jvm运行。



动态编程方法的比较:

性能：ASM>Javasisst>反射

实现成本:反射<Javassist<ASM



现实开发中，优先使用Javasisst实现原型，如果出现了性能瓶颈，在考虑使用ASM进行优化。



## 开发 步骤

### 自定义plugin

参照网上的一些资料 有点的太老了，填了2个坑。

1. 在android项目中创建一个`java library`的module，起名叫`BuildSrc`。

2. 删除里面没有用的文件，使最终的目录样子为：

   ```java
   └── src
       └── main
           ├── groovy
           │   └── cn
           │       └── com
           │           └── almostlover
           │               └── plugin
           │                   └── Register.groovy
           └── resources
               └── META-INF
                   └── gradle-plugins
                       └── cn.com.almostlover.plugin.properties
   ```

   ![image-20190523154711412](http://ww1.sinaimg.cn/large/006tNc79ly1g3barn1zmsj30ra0d6ab7.jpg)

3. `Register.groovy`的代码和`cn.com.almostlover.plugin.properties`下的代码:

   ```groovy
   package cn.com.almostlover.plugin
   
   import org.gradle.api.Plugin
   import org.gradle.api.Project
   //Register.groovy
   public class Register implements Plugin<Project>{
   
   
       @Override
       void apply(Project project) {
           project.logger.error "================自定义插件成功23333！=========="
       }
   }
   ```

   ```java
   implementation-class=cn.com.almostlover.plugin.Register  //cn.com.almostlover.plugin.properties
   ```

   需要注意的是META-INF下的目录是保证App能够正常识别插件，名称务必按照格式写去。 

4. 编写module下的`build.gradle`的代码

   ```java
   apply plugin: 'groovy'
   apply plugin: 'maven'
   
   dependencies {
       implementation gradleApi()  //老的项目implementation写的是compile 会导致报错:Don't know how to compute maven coordinate for artifact API,把compile替换成implementation即可。
       implementation localGroovy()
       implementation 'com.android.tools.build:gradle:3.1.1'  //老的项目是 2.2 太老了，我改的和project下的build.gradle的gradle版本一致就好了，至少不会出错
       implementation 'org.javassist:javassist:3.22.0-GA'//用于代码注入
       implementation 'com.android.tools.build:transform-api:1.5.0'
   }
   repositories{
       jcenter()
       google()  //这个仓库地址要添加  开发的过程中发现 高版本的gradle  jcenter和mavenCentral都没有  只能从google中去下载gradle高版本
       mavenCentral()
   }
   
   ```

   5.在app的`build.gradle`添加插件

   ​	不需要app去依赖`BuildSrc`这个module。

   ![image-20190523160332537](http://ww3.sinaimg.cn/large/006tNc79ly1g3bb8hlikxj30oo0dggni.jpg)

5. build项目

   如果发现我们定义的log，就说明插件安排成功了。

   ![image-20190523160441391](http://ww1.sinaimg.cn/large/006tNc79ly1g3bb9p9savj30o40eiac9.jpg)





### 自定义Transfrom

新增



注意如果报错:

```java
Error with transformClassesWithDesugarForDebug
Error:Execution failed for task ':app:transformClassesWithDesugarForDebug'.
> com.android.build.api.transform.TransformException: java.lang.RuntimeException: java.lang.RuntimeException: com.android.ide.common.process.ProcessException: 
Error while executing java process with main class com.google.devtools.build.android.desugar.Desugar with arguments 
{--input /Users/davidcorsalini/Documents/AndroidStudioProjects/controllori/app/build/intermediates/classes/debug --output 
/Users/davidcorsalini/Documents/AndroidStudioProjects/controllori/app/build/intermediates/transforms/desugar/debug/0 --classpath_entry 
/Users/davidcorsalini/Documents/AndroidStudioProjects/controllori/app/build/intermediates/classes/debug --classpath_entry 
/Users/davidcorsalini/Documents/AndroidStudioProjects/controllori/app/build/intermediates/transforms/stackFramesFixer/debug/0.jar --classpath_entry 
/Users/davidcorsalini/Documents/AndroidStudioProjects/controllori/app/build/intermediates/transforms/stackFramesFixer/debug/1.jar --classpath_entry 
/Users/davidcorsalini/Documents/AndroidStudioProjects/controllori/app/build/intermediates/transforms/stackFramesFixer/debug/2.jar --classpath_entry 
/Users/davidcorsalini/Documents/AndroidStudioProjects/controllori/app/build/intermediates/transforms/stackFramesFixer/debug/3.jar --classpath_entry 
 *all the same to 58.jar*
desugar_try_with_resources_omit_runtime_classes}
Information:BUILD FAILED in 2s
Information:2 errors
Information:0 warnings
Information:See complete output in console
```

则add `android.enableD8.desugaring = true` in gradle.properties file.即可。





如果一直报错:

```java
* What went wrong:
Execution failed for task ':app:transformClassesWithDexBuilderForDebug'.
> com.android.build.api.transform.TransformException: java.lang.IllegalArgumentException: java.lang.IllegalArgumentException
```

则在app的build.gradle中添加

```java
android {    
dexOptions{
      preDexLibraries = false
    }
}
```



### API

javassist中最重要的几个类:

 * **ClassPool**: 一个基于`HashMap`实现的`CtClass`容器，其中**key=类的名称**，**value=该类的CtClass对象**。默认的classpool使用的路径与底层jvm使用的路径相同，所以有时候，可能需要向classpool类添加类路径或类字节。
 * **CtClass**: 表示一个类，这些CtClass对象可以从**ClassPool**中获取。
 * **CtMethod**: 表示类中的方法。
 * **CtFields**: 表示类中的字段。

















参考日志:[秒懂Java动态编程（Javassist研究）](<https://blog.csdn.net/ShuSheng0007/article/details/81269295>)

[Android Gradle Plugin插件开发——基础](<https://blog.csdn.net/guy_guy/article/details/80914600>)

[Android动态编译技术 Plugin Transform Javassist操作Class文件](<https://www.jianshu.com/p/a6be7cdcfc65>)

