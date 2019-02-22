---
layout:     post
title:  React Native导入antd-mobile的坑
subtitle: 一个接一个的报错
date:     2019-02-22
author:     Pony
header-img: img/post-bg-mayday-bubble.jpg
catalog: true
tags:
    - ReactNative
    - antd-mobile
---
从2019年的2月11号下午3:00开始到2019年的2月22号早上11点终于解决这个问题。
一个接着一个的报错。多个解决方案的排列组合。终于走出来了一条光明大道出来。
从创建项目开始，一个一个问题是如何解决的。我准备重新演示一遍。
## 立项：
>react-native init TestPoject

## 导入`antd-mobile`
照着[react native 使用antd-mobile，echarts，以及其原生组件](https://www.jianshu.com/p/fb5453910263)
进行导入：
* yarn add antd-mobile
* yarn add babel-plugin-import
在项目的根目录创建`.babelrc`文件，并添加内容：

```json
{

  "plugins": [["import", { "libraryName": "antd-mobile" }]],

  "presets": ["react-native"]

}
```


## 开始报错
### no bundle url present.
然后运行项目，很快出了第一个错：
![](https://ws4.sinaimg.cn/large/006tKfTcly1g0f545ogumj309o0izwff.jpg)
这个问题好说，一般就是退出重启，或者执行`: rm -f /node_modules&&npm install`就可以了。实在不行，**重启试试**。

果然，重启之后就好了，迎接了第二个错：
### Cannot find module 'babel-preset-react-native' from '/Users/pony/Desktop/TestProject'
![](https://ws4.sinaimg.cn/large/006tKfTcly1g0f54wy2p1j309o0iz40i.jpg)


顾名思义，找不到`babel-preset-react-native`,那咱们就安装一个就好了：

1. 删除babel-preset-react-native最新版本，并将其版本退回至3.0.2
>npm uninstall babel-preset-react-native
>
>npm install --save-dev babel-preset-react-native@^3.0.2

2. 重新启用应用
>react-native run-ios

如果报错，就清除，或者退出app重新启动一次。

接着出现了第三个错误：
###  bundling failed: TypeError: Cannot read property 'bindings' of null at Scope.moveBindingTo (/Users/pony/Desktop/TestProject/node_modules/@babel/traverse/lib/scope/index.js:867:13)
![](https://ws2.sinaimg.cn/large/006tKfTcly1g0f55xz6ykj309o0izjsz.jpg)


参考：[Cannot read property 'bindings' of null at Scope.moveBindingTo](https://github.com/babel/babel/issues/8575)
please use version 5.0.2 of babel-preset-react-native, it will work. Its because of old version of it.I had faced it before,and delete `package-lock.json` and `yarn.lock`。

执行完毕后 退出，重启项目。

发现新的错误：
 syntax 'nullishCoalescingOperator' isn't currently enabled (47:23):
![](https://ws4.sinaimg.cn/large/006tKfTcly1g0f58lt1voj309o0iz76p.jpg)


### syntax 'nullishCoalescingOperator' isn't currently enabled (47:23):
 报了一个语法错误的错，解决办法：
 [syntax 'nullishCoalescingOperator' isn't currently enabled](https://stackoverflow.com/questions/53944204/syntax-nullishcoalescingoperator-isnt-currently-enabled)
 
 修改`.babelrc`中的：
 
```json
{
  "presets": ["module:metro-react-native-babel-preset"]
}
```


继续跑起来,**如果还是失败，退出`终端`,重新启动项目**。
然后又报错了...
### registerEvents is not a function
![](https://ws1.sinaimg.cn/large/006tKfTcly1g0f594o21cj309o0izmyg.jpg)
   
  解决办法：[undefined is not a function (evaluating '_this._registerEvents()')](https://stackoverflow.com/questions/52713101/undefined-is-not-a-function-evaluating-this-registerevents)
  * watchman watch-del-all
  * rm -rf node_modules/.cache
  * react-native start --resetCache
  
  
  ## 最后大功告成
![](https://ws4.sinaimg.cn/large/006tKfTcly1g0f52xqtvkj309o0izt92.jpg)

  