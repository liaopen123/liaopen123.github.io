---
layout:     post
title:   爬虫基础1 urllib
subtitle:  urllib
date:       2018-02-24
author:     Pony
header-img: img/post-bg-cook.jpg
catalog: true
tags:
    - Python
    - 爬虫
---

### 获取数据统计的几个网站

1. [百度指数](http://index.baidu.com/)
2. [微指数](http://data.weibo.com/)
3. [国家数据](http://data.stats.gov.cn/)
4. [世界银行公开数据](https://data.worldbank.org.cn/)

## urllib2

`urllib2`在pyhton3中被修改为`urllib.request`

### urllib与urllib2的区别

urllib：仅仅可以接受url，不能创建headers。但urllib提供的有urlencode()方法。但是urlib2是没有的。

### 安装路径

python自带的模块:/usr/lib/python2.7/urllib2.py

python第三方的模块:/usr/local/lib/python2.7/site-packages

#### 获取网页内容：

```python
import urllib2
response = urllib2.urlopen("http://www.baidu.com")
html = response.read()
print(html)    #写法缺点：不支持修改头内容  但是urllib2 头内容的clientversion="urllib2.2.7",容易被反爬虫。
```

#### 爬虫和反爬虫第一步（添加头部headers）

```python
import urllib2
headers = {"User-Agent":"Mozilla/5.0 (Macintosh; Intel Mac OS X 10_13_3) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/64.0.3282.140 Safari/537.36"}
request = urllib2.request("http://www.baidu.com",headers = headers)
response = urllib2.urlopen(request)
html = response.read()
code = response.getcode()
url = response.geturl()
info = response.info()
print(info)   
```

#### url编码

例如请求的url地址：`https://www.baidu.com/s?wd=传智播客`

```python
url  = "https://www.baidu.com/s"

params = {"wd":"传智播客"}

params = urllib.urlencode(params)

fullurl = url+"?"+"params"
```

### post请求写法

参数通过**字典**的形式写，并且通过urlencode转换，最总以data=data的形式带入进去

```python
formdata = {
"from" : "AUTO",
"to" : "AUTO",
}

data = urllib.urlencode(formdata)
request = urllib2.Request(url,headers=headers,data=data)
content = urllib2.urlopen(request).read()
```

#### file写入文件的另一种简单写法：

```python
     with open(str(page)+".html","w") as f:
        f.write(content)
```



### 利用cookie模拟登陆

一样 cookie为头信息  直接带入即可

### 访问不安全网站

访问https网站 但是证书不安全 会被拦截  例如12306

解决办法：忽略安全不认证

```python
import urllib2
import ssl
url = "http://www.12306.cn/mormhweb/"
context = ssl._create_unverified_context()
request = urllib2.Request(url)
content = urllib2.urlopen(request,context = context)
print(content)
```



### CA证书的概念

ca认证中心 相当于网站的身份证中心，一个网站认证后 会再浏览器输入框显示认证信息防止钓鱼网站的出现。ca认证中心相当于公安局，ca证书相当于每个网站的身份证。



## Handler处理器和自定义Opener

### 简单的opener

流程：

1. 创建httphandler或者httpshandler
2. 创建opener:opener = urllib2.build_opener(httphandler) 传入创建的handler
3. 创建request
4. 得到response = opener.open(request)

```python
import urllib2
# 构建一个HTTPHandler 处理器对象，支持处理HTTP请求
http_handler = urllib2.HTTPHandler()
# 构建一个HTTPHandler 处理器对象，支持处理HTTPS请求
# http_handler = urllib2.HTTPSHandler()
# 调用urllib2.build_opener()方法，创建支持处理HTTP请求的opener对象
opener = urllib2.build_opener(http_handler)
# 构建 Request请求
request = urllib2.Request("http://www.baidu.com/")
# 调用自定义opener对象的open()方法，发送request请求
response = opener.open(request)
# 获取服务器响应内容
print response.read()
```



### ProxyHandler处理器(代理设置)

反爬虫的第二大招(第一个大招：模拟header)

原因:有的网站会检测某一时间段ip的访问次数(通过流量统计，系统日志)，如果频繁会被禁用掉。

```python
import urllib2
proxyHandler = urllib2.ProxyHandler({"http":"114.113.126.86"})
opener = urllib2.build_opener(proxyHandler)
request = urllib2.Request("http://www.baidu.com")
response = opener.open(request)
print(response.read())
```

### HTTPPasswordMgrWithDefaultRealm()

这个方法将创建一个密码管理对象，用来保存http请求相关的用户名和密码，主要2个应用场景：

1. 私密代理验证授权的时候需要使用到(ProxyBasicAuthHandler())
2. 验证web客户端时候 需要 使用(HttpBasicAuthHandler())

#### ProxyBasicAuthHandler（代理授权验证）

使用私密代理需要验证账号密码，负责会报http407的错误。

```python
#urllib2_proxy2.py
import urllib2
import urllib
# 私密代理授权的账户
user = "mr_mao_hacker"
# 私密代理授权的密码
passwd = "sffqry9r"
# 私密代理 IP
proxyserver = "61.158.163.130:16816"
# 1. 构建一个密码管理对象，用来保存需要处理的用户名和密码
passwdmgr = urllib2.HTTPPasswordMgrWithDefaultRealm()
# 2. 添加账户信息，第一个参数realm是与远程服务器相关的域信息，一般没人管它都是写None，后面三个参数分别是 代理服务器、用户名、密码
passwdmgr.add_password(None, proxyserver, user, passwd)
# 3. 构建一个代理基础用户名/密码验证的ProxyBasicAuthHandler处理器对象，参数是创建的密码管理对象
#   注意，这里不再使用普通ProxyHandler类了
proxyauth_handler = urllib2.ProxyBasicAuthHandler(passwdmgr)
# 4. 通过 build_opener()方法使用这些代理Handler对象，创建自定义opener对象，参数包括构建的 proxy_handler 和 proxyauth_handler
opener = urllib2.build_opener(proxyauth_handler)
# 5. 构造Request 请求
request = urllib2.Request("http://www.baidu.com/")
# 6. 使用自定义opener发送请求
response = opener.open(request)
# 7. 打印响应内容
print response.read()
```

#### HTTPBasicAuthHandler(web客户端授权验证)

```python
import urllib
import urllib2
# 用户名
user = "test"
# 密码
passwd = "123456"
# Web服务器 IP
webserver = "http://192.168.199.107"
# 1. 构建一个密码管理对象，用来保存需要处理的用户名和密码
passwdmgr = urllib2.HTTPPasswordMgrWithDefaultRealm()
# 2. 添加账户信息，第一个参数realm是与远程服务器相关的域信息，一般没人管它都是写None，后面三个参数分别是 Web服务器、用户名、密码
passwdmgr.add_password(None, webserver, user, passwd)
# 3. 构建一个HTTP基础用户名/密码验证的HTTPBasicAuthHandler处理器对象，参数是创建的密码管理对象
httpauth_handler = urllib2.HTTPBasicAuthHandler(passwdmgr)
# 4. 通过 build_opener()方法使用这些代理Handler对象，创建自定义opener对象，参数包括构建的 proxy_handler
opener = urllib2.build_opener(httpauth_handler)
# 5. 可以选择通过install_opener()方法定义opener为全局opener
urllib2.install_opener(opener)
# 6. 构建 Request对象
request = urllib2.Request("http://192.168.199.107")
# 7. 定义opener为全局opener后，可直接使用urlopen()发送请求
response = urllib2.urlopen(request)
# 8. 打印响应内容
print response.read()
```

## Cookie

通过**cookielib**和**HTTPCookieProcessor**结合使用简化步骤。

> `cookielib`:主要提供用于存储cookie的对象
>
> `HTTPCookieProcessor`：主要用于处理这些cookie对象，并构建handler对象。

### cookielib

主要包含CookieJar，FileCookieJar，MozillaCookieJar，LWPCookieJar

**一般只用到CookieJar()，如果需要和本地文件交互，就要用到Mozilla或者LWP**

```python
import cookielib
import urllib2
cookiejar = cookielib.CookieJar()
handler = urllib2.HTTPCookieProcessor(cookiejar)

opener  = urllib2.build_opener(handler)
request =urllib2.Request("http://www.baidu.com")
response = opener.open(request)
for item in cookiejar:
    print(item.name + "=" + item.value + ";")
    
```

#### cookie写入本地 在读取本地cookie

写入：

```python
import cookielib
import urllib2
fileName = "cookie.txt"
cookie = cookielib.MozillaCookieJar(fileName)
handler = urllib2.HTTPCookieProcessor(cookie)
opener = urllib2.build_opener(handler)
response = opener.open("http://www.baidu.com")
cookie.save()
```

读取：

```python
import cookielib
import urllib2
cookiejar = cookielib.MozillaCookieJar()
cookiejar.load('cookie.txt')
handler = urllib2.HTTPCookieProcessor(cookiejar)
opener = urllib2.build_opener(handler)
response = opener.open("http://www.baidu.com")
```

## urllib2的异常处理方法

网络请求的异常一般发生在urlopen或者opener.open的过程中。

涉及2个类

1. URLError


1. HTTPError

出现URLError的情况

> 1. 没有网络连接
> 2. 服务器连接失败
> 3. 找不到指定的服务器

HTTPError是URLError的子类

但是HTTPError包含code码

所以我们处理异常可以先处理HTTPError的异常，处理不了的 交给父类处理

示例代码：

```python
# urllib2_botherror.py
import urllib2
requset = urllib2.Request('http://blog.baidu.com/itcast')
try:
    urllib2.urlopen(requset)
except urllib2.HTTPError, err:
    print err.code
except urllib2.URLError, err:
    print err
else:
    print "Good Job"
```


