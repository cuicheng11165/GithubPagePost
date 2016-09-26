---
layout: post
title:  "Python模拟HttpRequest的方法总结"
date:   2016-02-05
author: 独上高楼
category: 编程
tags: [Python]
---

Python可以说是爬网的利器，本文主要介绍了一些python来模拟http请求的一些方法和技巧。

Python处理请求的类库有两个,urllib,urllib2。 这两个类库并不是一个类库的两个不同版本，urllib主要用来处理一些url相关的内容，发送请求的时候，请求对象只能是一个url。urllib2可以用request对象来实现请求，这样就可以实现如伪造头部，设置代理，http get，http post等方法。

阅读本文需要了解http请求的一些基本知识，如：

* 什么是httpwebrequest，httpwebresponse
* 什么是get，post
* 什么是cookie


本文主要介绍模拟请求用到的这些方式：

* 设置代理
* 伪造头部或者Header信息
* 启用cookie
* url参数的处理


### 使用urllib2.urlopen直接发送

    import urllib2
    
    url = 'http://www.baidu.com/'
    response = urllib2.urlopen(url) ##urlopen接受传入参数是string或者是request
    response_text = response.read()
    
### 使用urllib.build_opener

#### 直接发送请求

    import urllib2
    
    url = 'http://www.baidu.com/'
    
    opener = urllib2.build_opener()
    response = opener.open(url)
    response_text = response.read()


#### 通过代理访问站点

    proxy_handler = urllib2.ProxyHandler({"http" : 'http://localhost:8888'})
    opener = urllib2.build_opener(proxy_handler)
    response = opener.open(url)
    response_text = response.read()

#### 请求中附带request body(http post)

    opener = urllib2.build_opener()
    response = opener.open(url,'request body')
    response_text = response.read()

body中如果是key-value形式的，可以参照下面的url处理部分来处理

#### 启用Cookie

    cookie = cookielib.CookieJar()
    opener = urllib2.build_opener(urllib2.HTTPCookieProcessor(cookie))
    response = opener.open(url)
    response_text = response.read()


### 使用urllib2.Request

#### 请求中添加自定义的Header信息
    
    request = urllib2.Request(url)
    request.add_data('1234567')
    request.add_header('User-Agent', 'fake-client')
    response = urllib2.urlopen(request)

### 处理url中的参数信息
无论是使用get方式还是post方式，经常会遇到需要使用参数的形式，处理参数可以使用下面的类库

#### 参数集合转string


    para = {'111':'222','aaa':'bbb'}
    encodeurl = urllib.urlencode(para)
    输出aaa=bbb&111=222


#### url参数转dictionary

    url = 'https://www.baidu.com/s?wd=python%20url%20querystring&pn=10&oq=python%20url%20querystring&tn=baiduhome_pg&ie=utf-8&usm=1&rsv_idx=2&rsv_pq=d09af93600035cb8&rsv_t=d151qRmNNdybGINHcKbyO360E2%2Fg%2FUs2t0MiKqRQXwhHZuNF3IlKyyStzYuofVZczQA3'
    
    splitresult_instance = urlparse.urlsplit(url)

输出对象：  
SplitResult(scheme='https', netloc='www.baidu.com', path='/s', query='wd=python%20url%20querystring&pn=10&oq=python%20url%20querystring&tn=baiduhome_pg&ie=utf-8&usm=1&rsv_idx=2&rsv_pq=d09af93600035cb8&rsv_t=d151qRmNNdybGINHcKbyO360E2%2Fg%2FUs2t0MiKqRQXwhHZuNF3IlKyyStzYuofVZczQA3', fragment='')

想转成集合只要

    result_dic=urlparse.parse_qs(splitresult.query)

通过这种处理方式，把data信息放在url上来实现http get，放在body中实现http post。

