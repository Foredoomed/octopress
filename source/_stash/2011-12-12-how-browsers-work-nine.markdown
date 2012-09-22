---
layout: post
title: "浏览器是如何工作的(九)"
date: 2011-12-12 22:19
comments: true
categories: browsers
---
**5 布局**

当renderer被创建和加入到树中时，它是没有位置和大小的，计算这两个值称为布局或回流(reflow)。

HTML使用的是基于布局模型的流，这意味着大多数时间内，单一路径下计算几何值是可能的。后进入流的元素不会影响比它先进入流的元素的几何属性值，所以布局文档可以被从左到右，从上到下处理。但是也有例外：比如，HTML的table元素可能需要多条路径。

坐标系统是和根框架相关的，而且使用的是上坐标和左坐标。

布局是一个迭代的过程，它从根renderer(对应于HTML文档的`<html>`元素)开始。布局迭代部分或整个框架，计算每个renderer的几何信息。

根renderer的位置是`0,0`，它的范围是viewport(浏览器窗口的可视区域)。所有的renderer都有`layout`或`reflow`方法，每个renderer调用需要生成布局的孩子的`layout`方法。

<!-- more -->

**5.1 脏位系统**

为了在细小的改动是不重新生成整个布局，浏览器使用一个叫脏位(dirty bit)的系统。一个被修改或添加的renderer，它和它的孩子都会被标记为”dirty”，意思是说需要重新布局。

浏览器有2个标志位：“dirty”和”children are dirty”。后者的意思是也许这个renderer本身不需要重新布局，但是它的孩子中至少有一个需要重新布局。

**5.2 全局和增量布局**

布局可以在整个渲染树上被触发，这就叫做全局(global)布局。全局布局的触发条件为：

1. 全局样式的改变影响了所有的renderer，例如修改字体大小。
2. 窗口大小的改变。

布局可以是增量式的，只有脏renderer会被设置布局(这样会造成需要做额外布局的危害)。

当renderer被标记为脏时，增量布局被异步触发。例如在从网络上获取内容后，新renderer被附加到渲染树，并且被加入到DOM树中。

![reflow](http://farm8.staticflickr.com/7167/6498973627_ca37c1cac5.jpg" width="326" height="341")

**5.3 异步和同步布局**

增量布局的过程是异步的，Firefox把reflow命令放入队列，然后用一个调度器批量的执行这些命令。Webkit也有一个计时器来实现增量布局，遍历渲染树把脏renderer重新布局。

脚本请求样式信息，比如`offsetHeight`，会触发同步的增量布局。全部布局一般都是同步触发的，有些时候因为一些属性的原因，布局在初始后被作为回调函数触发，比如下拉位置的改变。

**5.4 优化**

当布局是因为大小改变或者renderer的位置改变而触发，那么renderer的大小将从cache从读取，不会重新计算。

在一些情况下，比如只有子树被修改和布局不是从根开始，这是由于修改是在本地而且没有影响到周围元素，比如文本插入到文本区域。

**5.5 布局过程**

布局总是有下面的模式：

1. 双亲renderer决定自己的宽度。
2. 双亲转为孩子并且：
    1. 设置孩子renderer的`x`和`y`
    2. 如果被标记为脏或者在全局布局中等原因，则调用孩子的`layout`方法，这会导致计算孩子的高度。
3. 双亲使用孩子累积的`height`，`margin`和`padding`来设置自己的对应值。
4. 设置脏位为false。

Firefox使用一个状态对象(nsHTMLReflowState)作为布局的一个参数，在其他状态中包括了双亲的宽度。Firefox布局的输出是一个`metrics`对象(nsHTMLReflowMetrics)，它包含了renderer的高度。

**5.6 宽度计算**

renderer的宽度是用容器宽度来计算的，renderer的`width`样式属性，`margin`和`border`属性。

例如计算下面`div`的宽度：

{% codeblock lang:html %}
<div style="width:30%"/>
{% endcodeblock %}

Webkit是这样计算的(RenderBox类的calcWidth方法)

* 容器的宽度取容器可以达到的最大宽度与0两者之间较大的那个，可以达到的最大宽度就是内容宽度，它的计算方式是：`clientWidth() - paddingLeft() - paddingRight()`，clientWidth和clientHeight表示的是除了border和scrollbar之外的内部对象。
* 元素的宽度是有样式属性`width`决定的，它是计算容器宽度的百分比得出的一个绝对数值。
* 然后加入水平边框和缩进

到目前为止是偏好宽度(preferred width)的计算，然后最小和最大宽度值会被计算。如果偏好宽度比最大宽度大，那么就会使用最大宽度，同样如果小于最小宽度，那么就会使用最小宽度。

以防万一宽度没有改变的情况下需要重新布局，所以宽度的值都会被缓存。

**5.7 换行**

当一个renderer在布局的中间时就需要被分开，这时它会停止布局并告诉双亲它需要被分开，然后双亲就会创建额外的renderer和在他们身上调用`layout`方法。

