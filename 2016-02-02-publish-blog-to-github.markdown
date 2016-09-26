---
layout: post
title:  "在Github上搭建自己的博客(Windows平台)"
date:   2016-02-02
author: 独上高楼
category: 建站
tags: [github]
---

折腾了好久，终于在Github上搭建了自己的博客。这里面总结一下过程希望对大家能有所帮助。

## Github建博优缺点
* 和 csdn，博客园，网易相比，在Github上可以自己实现功能，跳过运营商的限制  
* 和阿里云，VPS相比，github托管的代码是免费的  
* github上只能托管静态网页，后台有数据库的这种动态网站不能托管  

## 你有一个Github的账号
* 懂一些基本的Web开发相关的知识  
* Ruby运行时，可以在Windows或者linux环境下进行安装  

## 建站的过程

* 在github创建网站的代码库，名字必须为username.github.io格式  
* 将你的网站的代码上传到代码库中  
* 配置域名  

如果你每一次写博客的同时还要手写html css 等信息，那么工作量会很大，github支持jekyll来自动生成页面。

## Jekyll环境的搭建

* 如果你是在Windows环境下配置。首先需要安装Ruby运行时，Jekyll是一个用Ruby写的软件。
[Run Jekyll On Windows](http://jekyll-windows.juthilo.com/)  里面展示了Windows下使用Jekyll的方法，大体分以下几步完成：

1. 安装Ruby运行时，[下载地址](http://rubyinstaller.org/downloads/) ,windows 平台下的安装过程很简单，安装过程记得勾选Add Ruby excuteables to  your PATH
2. 安装RubyDevKi，和Ruby的版本相同。下载后解压,执行以下3步：
   cd c:\RubyDevki  
   ruby dk.rb init  
   ruby dk.rb install  
3. 安装jekyll，github上给出了配置文档 [install Jekyll](https://help.github.com/articles/using-jekyll-with-pages) ，Bundler是一个包管理器，让你更方便的使用Ruby里面的软件，如果你想要在本地编译Github页面，那么使用 gem install bundler 来安装bundler
4. 配置jekyll,在站点的根目录里创建一个文件 Gemfile,文件内容如下：
source 'https://rubygems.org'  
gem 'github-pages'  
然后运行bundle install即可安装Jekyll。  
5. 安装成功后，运行 jekyll newe sitename 即可创建一个站点
6. 命令行 bundle exec jekyll serve即可运行站点，成功后访问 [http://localhost:4000](http://localhost:4000) 即可。

## 配置域名
买一个域名，把域名的ip指向github服务器，在网站的目录下建一个文件名为CNAME，文件的内容是你的域名，提交。购买的域名就会指向你的博客。

## Jekyll网站的定制
网站的语言是基于liquid的，每一次你修改文件后，jekyll会自动重新生成网站。编辑成功后提交到github，访问http://username.github.io 访问你的博客。这里面有一些别人共享的网站模板可以供大家选择。[网站模板](http://jekyllthemes.org/)里面有很多别人共享的主题。

## Jekyll网站需要解决的一些问题
1. 网站是静态的，因此只需要发表博客重新生成即可，发表博客可以使用 [markdown](https://en.wikipedia.org/wiki/Markdown) 来实现，简化了写博客的过程
2. 网站的社交功能需要想办法实现，因为没有后台数据库，因此需要一些第三方的帮助来实现留言回复功能。常用的有多说，disqus等
3. 现有模板有很多的前台是基于[bootstrap](http://baike.baidu.com/link?url=_Ju_TPVJCSducO4xSr6TwL5Bx5ZDRhK9bvpTzC8_wgKVVfNA2VZtpWdMk04nL7cixReciDCO1C0o-w76AD7GqxqVLggXiQTW_46OF7HkNWC)来实现的，里面一些脚本的信息可能会被墙，需要自己找替代

## 编写博客
博客的编写一般是基于markdown来实现的，markdown本身是为了简化书写的设计的[语法](https://en.wikipedia.org/wiki/Markdown#Example)，大家不用写博客的时候还考虑html语法，关于markdown的编译，windows平台下推荐使用markdownpad，可以编辑markdown的同时进行preview，非常的方便。

参考文章：  
[如何在Windows下使用Jekyll](http://jekyll-windows.juthilo.com)  
[一步一步在Github上创建主页](http://www.pchou.info/web-build/2013/01/03/build-github-blog-page-01.html)  
[好用的Markdown编辑器一览](http://www.williamlong.info/archives/4319.html)
[Markdown Wiki](https://en.wikipedia.org/wiki/Markdown)
