---
layout:     post
title:    ViewDragHelper使用说明
subtitle:   View 神器
date:       2016-11-28
author:     Pony
header-img: img/post-bg-cook.jpg
catalog: true
tags:
    - android
    - view
---
## 缘由
App的3.0.5版开发过程中，要做一个悬浮框的控件，时间紧，好在平时早有准备，先是自己写了一个控件，其实就是根据手指的左边移动view，但是会有一个小bug：不能分别点击和移动，虽然想办法解决了这个问题，但效果不尽人意，后来发现了ViewDragHelper，几乎不用开发者做太多事情，只要处理好它提供的几个回调就可以轻轻松松实现效果。让人很感觉很舒服。

然后看看效果：
![iamge](http://almostlover.com.cn/img/drag1.gif)

![iamge](http://almostlover.com.cn/img/draglistview1.gif)



## 使用
最简单使用(拖动任意view)：
1. 自定义ViewGroup继承任意布局，通过ViewDragHelper的静态方法

```java
ViewDragHelper.create(this, 1.0f, new ViewDragHelper.Callback(){方法});
```

去创建viewDragHelper对象，注意：需要在ViewDragHelper的回调中复写

①：tryCaptureView()(用来指定拖动的对象，如果返回值为true，则表示ViewGroup中的所有View都可拖动)。

②：clampViewPositionHorizontal()(用来限定拖动的宽度范围);

③：clampViewPositionVertical()(用来限定拖动的高度范围);

2.重写onInterceptTouchEvent和onTouchEvent方法传入viewDragHelper对象：

```java
@Override
public boolean onInterceptTouchEvent(MotionEvent event) {
    return viewDragHelper.shouldInterceptTouchEvent(event);
}

@Override
public boolean onTouchEvent(MotionEvent event){
    viewDragHelper.processTouchEvent(event);
    return true;
}
 
```
 

 

 

 

 


