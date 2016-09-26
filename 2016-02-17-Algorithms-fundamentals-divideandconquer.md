---
layout: post
title:  "算法导论学习笔记 一 分治算法"
date:   2016-02-17
author: 独上高楼
category: 编程
tags: [算法]
---


分治策略是一种常见的算法。在分治策略中，我们递归的求解一个问题，在每层地柜中应用如下三个步骤：
1. 分解，将问题分解成规模更小但解决方案相同的子问题
2. 解决，递归的求解子问题，如果子问题足够小则停止递归，直接求解
3. 合并，将子问题的解组合成原问题的解

## 最大字数组问题

给你一段股市的波动图，找到在什么时候买入，什么时候卖出能获得最大的收益。

![](http://i.imgur.com/K4HuEf4.jpg)

### 暴力破解法
我们很容易的想到一个包里破解法，就是把所有买入，卖出的组合列举出来进行对比，n天中取任意两天作为买入日和卖出日，共有n*(n-1)/2种解法，因此这种解法的时间复杂度是O(n^2）。

### 问题变换
为了方便我们处理，我们把每个数组的值变为股票的净增量。这个时候，最佳解法就变成了求一个数组的最大字数组：

![](http://i.imgur.com/nT8zclC.jpg)

### 使用分治策略的求解法

我们可以认为，我们要寻找子数组array[start,end]的最大字数组分三种情况：

* 最大字数组在左半部分
* 最大字数组在右半部分
* 最大字数组起点在左半部分，终点在右半部分

对于前两种情况，我们的求解方式和分解前是相同的，我们需要单独处理第三种情况。

如图：

![](http://i.imgur.com/mAJQJHb.jpg)


那么我们给出解这个问题的思路：


    FindMaxCrossingSubArray(array,start,end)
		mid=(start+end)/2
		从mid开始向左找，找到以mid为终点的最大字数组为array[max-left,min]
		从mid开始向右找，找到以mid为起点的最大字数组array[min,max-right]
		跨越mid的最大字数组为array[max-left,max-right]
		max-leftchild =FindMaxCrossingSubArray(array,start,mid)
		max-rightchild =FindMaxCrossingSubArray(array,min,end)
		return max-leftchild,max-rightchild,array[max-left,max-right] 3个中的最大字数组

### 分治算法分析

这种方法我们需要遍历的次数会比n^2少一些，时间复杂度为O(nlgn)

## 代码实现



自己尝试用C#实现了下，托管在了[github](https://github.com/cuicheng11165/Algorithms)上



		




	









 