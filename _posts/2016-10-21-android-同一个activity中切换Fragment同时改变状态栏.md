---
layout:     post
title:      android-同一个activity中切换Fragment同时改变状态栏
subtitle:   
date:       2016-10-22
author:     Pony
header-img: img/post-bg-cook.jpg
catalog: true
tags:
    - android
---
##需求由来：

听说我们的app要大改了，UI的妹纸们正在设计效果图，我们也沉寂已久！终于早上产品的妹子终于主动找过来了,问我们能不能实现一个效果：**首页banner的状态栏透明，其他页面的状态栏不透明且颜色可调。**

想到了之前在gayhub上面看到了[StatusBarUtil](https://github.com/laobie/StatusBarUtil)状态栏工具栏，用来实现这个效果上面刚刚合适。于是今天也就搞定了这个小需求。

##效果展示：
android4.4系统（ZTE）：
![image](http://almostlover.com.cn/img/android_fragment_statusbar01.gif)
Android7.0(nexus 5x):
![image](http://almostlover.com.cn/img/android_fragment_statusbar02.gif)
通过效果图可以看到 Android4.4以上的系统得到了充分的兼用,这个是statusUtils给我们所带来的便利,接下来讲具体思路和使用方法

##思路及实践:
在切换fragment的同时通过statusUtils切换顶部的状态栏,但要注意一点:如果banner页面全屏展示的话,会造成其它页面的标题栏与状态栏重叠的现象:

![image](http://almostlover.com.cn/img/android_fragment_statusbar03.jpeg)
因此我们还需处理这个问题,通过如下方法获取**状态栏的高度**:

```java
private static int getStatusBarHeight(Context context) {
    // 获得状态栏高度
    int resourceId = context.getResources().getIdentifier("status_bar_height", "dimen", "android");
    int height = context.getResources().getDimensionPixelSize(resourceId);
    return context.getResources().getDimensionPixelSize(resourceId);
}
```
并且动态修改fragment所包含的view的paddingTop值,具体方法:

```java
@Override
public void onViewCreated(View view, Bundle savedInstanceState) {
    super.onViewCreated(view, savedInstanceState);
    initView(view);
    view.setPadding(0, getStatusBarHeight(getActivity()), 0, 0);
}
```
这样这个问题就解决了!效果如图:
![image](http://almostlover.com.cn/img/android_fragment_statusbar04.jpeg)
解决掉这些阻碍后,就可以顺利前行了:

在MainActivity(fragment所对应的activity)中点击Tab切换fragment的方法中进行判断是那个fragment和设置对于的状态栏颜色:

```java
@Override
public void onTabChange(String tag) {

    if (tag != null) {
        if (tag.equals("message")) {     StatusBarUtil.setTranslucentForImageViewInFragment(MainActivity.this, null);
            mPreviousTabIndex = mCurrentTabIndex;
            mCurrentTabIndex = 0;
            replaceFragment(HomeFragment.class);
        } else if ("service".equals(tag)) {
            mPreviousTabIndex = mCurrentTabIndex;
            mCurrentTabIndex = 1;
          StatusBarUtil.setColor(MainActivity.this,Color.BLACK,111);
            replaceFragment(FinancialSupermarketFragment.class);
        } else if (tag.equals("myassets")) {
            mPreviousTabIndex = mCurrentTabIndex;
            mCurrentTabIndex = 2;
            StatusBarUtil.setColor(MainActivity.this,Color.GREEN,111);
            replaceFragment(MyAssetsFragmentNew.class);
        } else if (tag.equals("settings")) {
            mPreviousTabIndex = mCurrentTabIndex;
            mCurrentTabIndex = 3;
            StatusBarUtil.setColor(MainActivity.this,Color.RED,111);
            replaceFragment(ListFragmentTwo.class);
        }
    }
```
这样大功告成，就可以实现这个效果了。


