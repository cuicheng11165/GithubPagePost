---
layout: post
title:  "带你看懂Dictionary内部实现"
date:   2016-02-02
author: 独上高楼
category: 编程
tags: [CSharp]
---


<p>&nbsp;</p>
<p>了解Dictionary的开发人员都了解，和List相比，字典添加会慢，但是查找会比较快，那么Dictionary是如何实现的呢？</p>
<h1>Dictionary的构造</h1>
<p>下面的代码我看看Dictionary在构造时都做了什么：</p>
<div class="cnblogs_code">
<pre>        <span style="color: #0000ff;">private</span> <span style="color: #0000ff;">void</span> Initialize(<span style="color: #0000ff;">int</span><span style="color: #000000;"> capacity)
        {
            </span><span style="color: #0000ff;">int</span> prime =<span style="color: #000000;"> HashHelpers.GetPrime(capacity);
            </span><span style="color: #0000ff;">this</span>.buckets = <span style="color: #0000ff;">new</span> <span style="color: #0000ff;">int</span><span style="color: #000000;">[prime];
            </span><span style="color: #0000ff;">for</span> (<span style="color: #0000ff;">int</span> i = <span style="color: #800080;">0</span>; i &lt; <span style="color: #0000ff;">this</span>.buckets.Length; i++<span style="color: #000000;">)
            {
                </span><span style="color: #0000ff;">this</span>.buckets[i] = -<span style="color: #800080;">1</span><span style="color: #000000;">;
            }
            </span><span style="color: #0000ff;">this</span>.entries = <span style="color: #0000ff;">new</span> Entry&lt;TKey, TValue&gt;<span style="color: #000000;">[prime];
            </span><span style="color: #0000ff;">this</span>.freeList = -<span style="color: #800080;">1</span><span style="color: #000000;">;
        } </span></pre>
</div>
<p>&nbsp;</p>
<p>我们看到，Dictionary在构造的时候做了以下几件事：</p>
<ol style="margin-left: 54pt;">
<li>初始化一个this.buckets = new int[prime]</li>
<li>初始化一个this.entries = new Entry&lt;TKey, TValue&gt;[prime]</li>
<li>Bucket和entries的容量都为大于字典容量的一个最小的质数</li>
</ol>
<p>其中this.buckets主要用来进行Hash碰撞，this.entries用来存储字典的内容，并且标识下一个元素的位置。</p>
<p>我们以Dictionary&lt;int,string&gt; 为例，来展示一下Dictionary如何添加元素：</p>
<p>首先，我们构造一个:</p>
<p>Dictionary&lt;int, string&gt; test = new Dictionary&lt;int, string&gt;(6);</p>
<h1>初始化后<span style="font-size: 11pt;">： </span></h1>
<p><img src="http://images0.cnblogs.com/blog/147759/201507/220046159287455.png" alt="" /></p>
<h1>添加元素时，集合内部Bucket和entries的变化</h1>
<h2>Test.Add(4,"4")后：</h2>
<p>根据Hash算法： 4.GetHashCode()%7= 4,因此碰撞到buckets中下标为4的槽上，此时由于Count为0，因此元素放在Entries中第0个元素上，添加后Count变为1</p>
<p><img src="http://images0.cnblogs.com/blog/147759/201507/220046164126140.png" alt="" /></p>
<p>&nbsp;</p>
<p>&nbsp;</p>
<h2>Test.Add(11,"11")</h2>
<p>根据Hash算法 11.GetHashCode()%7=4,因此再次碰撞到Buckets中下标为4的槽上，由于此槽上的值已经不为-1，此时Count=1，因此把这个新加的元素放到entries中下标为1的数组中，并且让Buckets槽指向下标为1的entries中，下标为1的entry之下下标为0的entries。</p>
<p><img src="http://images0.cnblogs.com/blog/147759/201507/220046169594040.png" alt="" /></p>
<h2>Test.Add(18,"18")</h2>
<p>我们添加18，让HashCode再次碰撞到Buckets中下标为4的槽上，这个时候新元素添加到count+1的位置，并且Bucket槽指向新元素，新元素的Next指向Entries中下标为1的元素。此时你会发现所有hashcode相同的元素都形成了一个链表，如果元素碰撞次数越多，链表越长。所花费的时间也相对较多。</p>
<p>&nbsp;</p>
<p><img src="http://images0.cnblogs.com/blog/147759/201507/220046174907710.png" alt="" /></p>
<h2>Test.Add(19,"19")</h2>
<p>再次添加元素19，此时Hash碰撞到另外一个槽上，但是元素仍然添加到count+1的位置。</p>
<p><img src="http://images0.cnblogs.com/blog/147759/201507/220046181316582.png" alt="" /></p>
<p>&nbsp;</p>
<p>&nbsp;</p>
<h2>删除元素时集合内部的变化</h2>
<h2>Test.Remove(4)</h2>
<p>我们删除元素时，通过一次碰撞，并且沿着链表寻找3次，找到key为4的元素所在的位置，删除当前元素。并且把FreeList的位置指向当前删除元素的位置，FreeCount置为1</p>
<p><img src="http://images0.cnblogs.com/blog/147759/201507/220046192098152.png" alt="" /></p>
<p>&nbsp;</p>
<h2>Test.Remove(18)</h2>
<p>删除Key为18的元素，仍然通过一次碰撞，并且沿着链表寻找2次，找到当前元素，删除当前元素，并且让FreeList指向当前元素，当前元素的Next指向上一个FreeList元素。</p>
<p>此时你会发现FreeList指向了一个链表，链表里面不包含任何元素，FreeCount表示不包含元素的链表的长度。</p>
<p><img src="http://images0.cnblogs.com/blog/147759/201507/220046201463766.png" alt="" /></p>
<h2>Test.Add(20,"20")</h2>
<p>再添加一个元素，此时由于FreeList链表不为空，因此字典会优先添加到FreeList链表所指向的位置，添加后FreeCount减1，FreeList链表长度变为1</p>
<p><img src="http://images0.cnblogs.com/blog/147759/201507/220046208035866.png" alt="" /></p>
<h1>总结：</h1>
<p>通过以上试验，我们可以发现Dictionary在添加，删除元素按照如下方法进行：</p>
<ol>
<li>通过Hash算法来碰撞到指定的Bucket上，碰撞到同一个Bucket槽上所有数据形成一个单链表</li>
<li>默认情况Entries槽中的数据按照添加顺序排列</li>
<li>删除的数据会形成一个FreeList的链表，添加数据的时候，优先向FreeList链表中添加数据，FreeList为空则按照count依次排列</li>
<li>字典查询及其的效率取决于碰撞的次数，这也解释了为什么Dictionary的查找会很快。</li>
</ol>
<p>&nbsp;</p>
<p>好吧，熬了半宿，今天先写到这了，如果看了有所收获就帮忙顶一下，有问题欢迎拍砖。</p>
<p>&nbsp;</p>