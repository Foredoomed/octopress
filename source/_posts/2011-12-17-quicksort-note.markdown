---
layout: post
title: "快速排序复习笔记"
date: 2011-12-17 14:27
comments: true
categories: algorithms
---
最近晚上躺在床上没事就拿着手机看了几节MIT的公开课《Introduction to Algorithms》，看着看着就觉得自己关于排序算法的原理基本上都忘光了。顿感非常失落，所以特意又对算法复习了一遍，特别是快速排序，这篇博客就是复习笔记，毕竟发表是最好的记忆嘛。

这门课是由两位老师来上的的，一位是[Charles E.Leiserson](http://people.csail.mit.edu/cel/ "Charles E.Leiserson")教授，光听这个名字可能有点陌生，但是如果提到他参与编写的《算法导论》这本书，对于学计算机的人来说那是无人不知无人不晓的经典著作；另一位是80后的Erik Demaine[Erik Demaine](http://www.csail.mit.edu/user/666 "Erik Demaine")，据说此人在20岁时就当上了MIT的教授，真是牛人啊，而且大多数的课也都是由他来讲的。还有一点我比较在意的是一节课的时间大约是1个小时20分钟左右，这跟中国大学的45分钟一节课很不一样，个人觉得这样的时间安排可以使一节课更加连贯，学习效率也会更好。

<!-- more -->

好了，回到正题快速排序上。话说快速排序是被广泛实际应用的排序算法，究其原因就是优秀的时间复杂度，平均为O(n*log(n))和O(n)的空间负责度，但是快速排序不是稳定排序，因为不能保证相同的元素在排序后还是原来的顺序。

快速排序有三个步骤：

1. 从元素列表中选择一个元素，这个元素被称为pivot。
2. 把所有小于pivot的元素都放到pivot的左边，所有大于pivot的元素都放到pivot的右边，这个过程称为partition。
3. 在左右两个子列表中迭代步骤2。

下面举个例子来说明快速排序的排序过程，假如我们有这样一组数：`6 10 13 5 8 3 2 11`，并且选取左端点元素`6`作为初始pivot，那么上面步骤2的过程应该是下面这个样子：

    6  5  13  10  8   3   2  11  
    6  5   3  10  8  13   2  11  
    6  5   3   2  8  13  10  11  
    2  5   3   6  8  13  10  11

因为要把小于pivot的数都放到pivot的左边，而大于等于pivot的数都放到pivot的右边，所以在保证空间复杂度O(n)的前提下，采用一个变量`j`，初始为pivot的后一个位置，当循环这个数组发现有小于pivot的数时就和`j`这个位置上的数交换，然后`j + 1`，等到循环结束后再交换pivot和`j`位置上的数。当然如果空间够多的话，直接创建两个新数组往里塞更容易一点。到了上面最后一步后，pivot就变成了`6`，然后再在它的左右子数组中迭代步骤2。

好了，现在我们可以来写代码了，根据上面的思想不难得出下面的代码：

{% codeblock Quicksort1.java lang:java %}
public static void quickSort1(int[] array, int p, int q) {

  if (p > q) {
    return;
  }

  // 选取左端点为pivot
  int pivot = array[p];
  int j = p;

  for (int i = j + 1; i <= q; i++) {
    if (array[i] < pivot) {
      // 如果小于pivot则与j+1位置上数交换
      swap(array, ++j, i);

    }
}

  // 交换pivot与j位置上的数
  swap(array, p, j);

  // 迭代左右子数组
  quickSort1(array, p, j - 1);
  quickSort1(array, j + 1, q);
}

public static void swap(int[] array, int i, int j) {
  int temp = array[i];
  array[i] = array[j];
  array[j] = temp;
}
{% endcodeblock %}

但是这个算法有个缺陷，在已经排序好的数组上性能非常差。所以人们就对原始快速排序算法进行了改进，改进的地方就是pivot的选取，用随机数或者中间位置的数取代最左端的数；还有就是初始化两个指针，分别从数组的两端向pivot扫描，小于pivot的数和大于等于pivot的数直接交换等，具体可以参看这篇文章：[快速排序及优化](http://www.blogjava.net/killme2008/archive/2010/09/08/quicksort_optimized.html "快速排序及优化")。

我们来看一下经过上述两个方法优化过的快速排序算法：

{% codeblock Quicksort2.java lang:java %}
public static void quickSort2(int[] array, int p, int q) {

  int i = p, j = q;
		
  // 选中间数为pivot
  int pivot = array[i + (q - p) / 2];

  while (i <= j) {
			
    while (array[i] < pivot) {
      i++;
    }
			
    while (array[j] > pivot) {
      j--;
    }

    // 交换小于pivot的数和大于pivot的数
    if (i <= j) {
      swap(array, i, j);
      i++;
      j--;
	 }
  }
		
  // 迭代
  if (p < j)
    quickSort2(array, p, j);
  if (i < q)
    quickSort2(array, i, q);
}
{% endcodeblock %}

就是这样，快速排序其实蛮简单的，它和冒泡排序的区别就是：在冒泡排序中，数组中的数需要和其他多个数比较，越前面的数比较的次数越多；而快速排序在一次排序中比较的是一个固定的pivot，所以效率要比冒泡排序高很多。

下面是排序算法的比较：

| Sort | Average | Best | Worst | Space | Stability | Remarks |
|----
| Bubble sort   | O(n^2)   | O(n^2)   | O(n^2) | Constant | Stable | Always use a modified bubble sort |
| Modified Bubble sort   | O(n^2)   | O(n) | O(n^2) | Constant | Stable | Stops after reaching a sorted array |
| Selection Sort | O(n^2) | O(n^2) | O(n^2) | Constant | Stable | Even a perfectly sorted input requires scanning the entire array |
| Insertion Sort | O(n^2) | O(n) | O(n^2) | Constant | Stable | In the best case (already sorted), every insert requires constant time |
| Heap Sort | O(nlog(n)) | O(nlog(n)) | O(nlog(n)) | Constant | Instable | By using input array as storage for the heap, it is possible to achieve constant space |
| Merge Sort | O(nlog(n)) | O(nlog(n)) | O(nlog(n)) | Depends | Stable | On arrays, merge sort requires O(n) space; on linked lists, merge sort requires constant space |
| Quicksort | O(nlog(n)) | O(nlog(n)) | O(n^2) | Constant | Stable | Randomly picking a pivot value (or shuffling the array prior to sorting) can help avoid worst case scenarios such as a perfectly sorted array |
{: class="datalist" }  

参考文章

1. [快速排序及优化](http://www.blogjava.net/killme2008/archive/2010/09/08/quicksort_optimized.html "快速排序及优化")  
2. [Quicksort in Java](http://www.vogella.de/articles/JavaAlgorithmsQuicksort/article.html "Quicksort in Java")  
3. [Sorting Algorithm Comparison](http://www.cprogramming.com/tutorial/computersciencetheory/sortcomp.html "Sorting Algorithm Comparison")

