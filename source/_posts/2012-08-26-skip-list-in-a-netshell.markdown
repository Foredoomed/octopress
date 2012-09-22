---
layout: post
title: "Skip List原理简述"
date: 2012-08-26 14:05
comments: true
categories: algorithms
---
Skip List(以下简称SL)是由[William Pugh](http://en.wikipedia.org/wiki/William_Pugh "William Pugh")在1989年提出的，从字面上解释的话差不多就是“跳跃链表”，它是普通有序链表的一种改进，目的是为了提高搜索，插入和删除的速度。我们知道，普通List的搜索，插入和删除的时间复杂度都是`O(n)`，而SL则可以提高到`O(log n)`，这就使得SL变成了一个非常有用的数据结构，像现在大热的NoSQL之一[Redis](http://redis.io/ "Redis")就是使用了SL。JDK6之后也加入了[Doug Lea](http://g.oswego.edu/ "Doug Lea")的SL实现[ConcurrentSkipListSet](http://docs.oracle.com/javase/6/docs/api/java/util/concurrent/ConcurrentSkipListSet.html "ConcurrentSkipListSet")和[ConcurrentSkipListMap](http://docs.oracle.com/javase/6/docs/api/java/util/concurrent/ConcurrentSkipListMap.html "ConcurrentSkipListMap")。

<!-- more -->

{% img http://i1256.photobucket.com/albums/ii494/Foredoomed/Skip_list.png %}

上面这张图是维基上的SL结构示意图，我们可以清楚地看到与普通链表不同的是，SL是由多层链表组成，而且元素会重复出现在多个层次上，只不过越往上层元素出现的概率越低。通常情况下在`i`层出现的元素也出现在`i+1`层的概率`p`取`1/2`或`1/4`，然后根据[几何分布](http://en.wikipedia.org/wiki/Geometric_distribution "Geometric distribution")的公式逆运算来求出元素出现在第几层。

**搜索**

{% img http://i1256.photobucket.com/albums/ii494/Foredoomed/search42.gif %}

每次搜索都从最上层开始，知道遇到大于要搜索的元素才转入下一层搜索。首先从第3层(层数从0开始)开始搜索42，首先遇到的是9，再后面就到底了，而42>9，所以转入下一层往右搜索；下一层首先遇到的是19，而42>19，所以再转入下一层，如此反复最终找到了我们所需要的元素42。其实在这个搜索过程中已经能够看出这个搜索的过程有点像[二分查找](http://en.wikipedia.org/wiki/Binary_search_algorithm "Binary Search")，但不完全相同。但是可以控制元素出现的概率使SL和二分查找完全相同。而我们知道二分查找的时间复杂度为`O(log n)`，所以SL的搜索时间复杂度也为`O(log n)`。

**插入**

{% img http://i1256.photobucket.com/albums/ii494/Foredoomed/insert.gif %}

我们需要通过计算来得出元素在哪个层次出现，一般来说几何分布是比较好的元素分布情况，因为这样的话有50%的元素只出现在第0层，25%的元素出现在第0层和第1层，12.5%的元素出现在第0，1，2层，以此类推，这样一来就跟二叉排序树差不多了。根据几何分布公式`F(k) = 1 - (1 - p)^k`，我们可以得到层数`k = log (1 - F(k)) / log (1 - p)`。
 

**删除**

{% img http://i1256.photobucket.com/albums/ii494/Foredoomed/delete.gif %}

在SL中删除一个元素需要把在所有层次上的该元素都删除。如上图所示，要删除元素9，则需要把所以层次上的9全部都删除。

**实现**

网上有很多种的SL实现，但是我认为最简单直观的还是[这里](http://igoro.com/archive/skip-lists-are-fascinating)，虽然他用的是C#实现的，但改成Java不是一件难事。

**参考资料**

[1] [Skip List on Wikipedia](http://en.wikipedia.org/wiki/Skip_list)  
[2] [DATA STRUCTURES AND ALGORITHMS Project #25: SKIP LISTS](http://www.sable.mcgill.ca/~dbelan2/cs251/skip_lists.html)  
[3] [Skip lists are fascinating](http://igoro.com/archive/skip-lists-are-fascinating/)
