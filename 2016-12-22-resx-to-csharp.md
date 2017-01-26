---
title: ResX文件生成C#类的几种方式
date: 2016-12-22 18:13:05
categories: 编程
tags: [CSharp,ResX]
---

通过ResX来生成C#代码

<!-- more -->

主要记录通过resource.resx文件来反向生成c#类的方式：

1. 通过第三方Tool
2. 通过微软提供的命令行，或者类库
3. 自定义代码


## Extended Strongly Typed Resource Generator ##

Extended Strongly Typed Resource Generator是一个第三方的VS插件，可以通过ResX文件来生成cs文件。
如图：

![](http://i.imgur.com/vcb11GC.png)
下载地址及其使用方法：[连接](https://www.codeproject.com/articles/13830/extended-strongly-typed-resource-generator)

## 命令行，或者类库 ##

**Resgen.exe** 也可以实现类似的功能：[文档地址](https://msdn.microsoft.com/en-us/library/ccec7sz1(v=vs.100).aspx)

例如想要通过resource.resx生成c#类只需要：
resgen resource.resx /str:C#,Namespace1,MyClass,MyFile.cs

**StronglyTypedResourceBuilder 类** 可以通过代码来实现上面的功能：[示例代码](https://msdn.microsoft.com/zh-cn/library/system.resources.tools.stronglytypedresourcebuilder.aspx)

## 自定义代码 ##

上面的两种方式生成类可能和现有的项目类名，方法名不同，那么我们只能通过编程的方式来生成代码。

主要的思路就是通过**CodeDOM**来生成代码：
[相关文档](https://msdn.microsoft.com/en-us/library/y2k85ax6(v=vs.110).aspx)



