---
layout:     post
title:     Android TextView 展示Html 并且根据超链接进行任意跳转
subtitle:   Spanned 富文本
date:       2016-11-15
author:     Pony
header-img: img/post-bg-cook.jpg
catalog: true
tags:
    - android
    - TextView
    - webview
---

## 需求来源：
早上开会，要扩充App站内信的功能，不仅仅是文本的展示，而且可以进行相关信息的扩展，包括跳转网页或者跳转本地的App，最不用动脑筋的方法就是后台通过键值对的形式<String,StringVoid>返回数据，获取字符串对应的功能。但是，这种方式太简单，之前看过一些富文本的Demo，就在会上主动提出可以通过解析Html标签的方式进行扩展。

开完会后，就去调研资料，搜了一些。最终实现了这个功能，晚上就将自己搜集到的资料总结一下。

## 效果展示：
![iamge](http://almostlover.com.cn/img/richtext01.gif)

解析所有的超链接文本

实现过程:
①：实现富文本样式：
TextView本身是支持Html标签的。

```java
<a href="...">
<b>
<big>
<blockquote>
<br>
<cite>
<dfn>
<div align="...">
<em>
<font size="..." color="..." face="...">
<h1>
<h2>
<h3>
<h4>
<h5>
<h6>
<i>
<img src="...">
<p>
<small>
<strike>
<strong>
<sub>
<sup>
<tt>
<u>
```
以上这些都支持，通过

```java
Spanned spannedHtml = Html.fromHtml(html);
```
方法将字符串类型的html转换成Spanned 富文本，然后通过

```java
TextView.setText(spannedHtml);
```
的方法即可实现富文本的样式。

 

样式是实现了，但是点击事件怎么弄？

和想的一样，想办法获取到文本中所有的超链接，当点击超链接的时候获取对应的事件。

可喜的是Android提供有这样的方法，顺水推舟。

```java
//把spanned 包装成 SpannableStringBuilder
SpannableStringBuilder clickableHtmlBuilder = new SpannableStringBuilder(spannedHtml);
//获取到容器中所有的url超链接的文本：
URLSpan[] urls = clickableHtmlBuilder.getSpans(0, spannedHtml.length(), URLSpan.class);
//通过循环去重新设置所有的超链接点击事件：
 int start = clickableHtmlBuilder.getSpanStart(urlSpan);
 int end = clickableHtmlBuilder.getSpanEnd(urlSpan);
 int flags = clickableHtmlBuilder.getSpanFlags(urlSpan);
 ClickableSpan clickableSpan = new ClickableSpan() {
 public void onClick(View view) {
 //Do something with URL here.
 Log.i(TAG, "onClick url=" + urlSpan.getURL() );
 }
 };
 clickableHtmlBuilder.setSpan(clickableSpan, start, end, flags);
```
已经将demo分享在[GitHub](https://github.com/liaopen123/RichTextView)。


