---
layout: post
title: "浏览器是如何工作的(八)"
date: 2011-12-11 21:32
comments: true
categories: browsers
---
4.3.3 简单规则匹配的例子

样式规则有下面几种：

* CSS规则，外部表单或style元素

    p {color:blue}

* 内联style属性

    <p style="color:blue" />

* HTML的视觉属性(会被映射到相关的样式规则)

    <p bgcolor="blue" />

<!-- more -->

后两种很容易和元素匹配，因为元素有style属性和HTML属性，他们可以用元素作为key来映射。

前面提到过的问题#2，CSS规则匹配可以是很复杂的。为了解决这个难题，所以对规则做一些处理，让获取规则更容易。

在解析样式表后，根据选择器规则会被加入到一个或多个hashmap中。这些map有以id为key的，有以class名key的，有以tag名为key的，还有一个通用map，它的key可以是任何类型。如果选择器是id选择器，那么规则会被加入到以id为key的map中，以此类推。

这个处理会使匹配规则变得更加容易。现在就没有必要查看每个定义，我们可以从map中得到相关的规则。这个优化消除了超过95%的规则，所以他们甚至在匹配过程中都不需要被考虑。

让我们看下面一个样式规则的例子：

{% codeblock %}
p.error {color:red}
#messageDiv {height:50px}
div {margin:5px}
{% endcodeblock %}

上面第一个规则会被插入到以class为key的map中，第二个规则会被插入到以id为key的map中，第三个规则会被插入到以tag名为key的map中。

对于下面的HTML片段：

{% codeblock lang:html %}
<p class="error">an error occurred </p>
<div id=" messageDiv">this is a message</div>
{% endcodeblock %}

首先尝试寻找对应`p`元素的规则，在以class为key的map里找到一个以`error`为key的规则`p.error`。div元素在id的map和tag名的map中都有相关规则，所以剩下要做的只是找到哪个规则是真正匹配的规则。

例如下面的div样式规则：

{% codeblock %}
table div {margin:5px}
{% endcodeblock %}

更好匹配的是tag名的map，因为tag名是最右端的选择器，但是如果div元素没有table锚点的话就不会匹配。

Webkit和Firefox都会做这个处理。

4.3.4 在正确的层级顺序中应用规则

style对象有对应于视觉属性的属性，如果属性没有对应的规则，那么一些属性可以继承双亲元素的style对象，其他的属性有默认值。

当有多个定义的时候问题就来了，解决这个问题靠的是层级顺序。

样式表的层级顺序

style属性的声明可以在多个样式表中被声明，有时在一个样式表中，这意味着应用规则的顺序变得非常重要。这被称为层级顺序，根据CSS2的规范说明，层级顺序是(从低到高)：

1. 浏览器定义
2. 用户普通定义
3. 开发者普通定义
4. 开发者重要定义
5. 用户重要定义

浏览器定义是最不重要的，用户定义的样式如果被标记为重要的就可以覆盖开发者定义的样式。如果是相同顺序的定义会被[特征(specificity)](http://www.html5rocks.com/en/tutorials/internals/howbrowserswork/#Specificity "specificity")排序，经过排序后的顺序是特征化的。HTML视觉属性被翻译成匹配的CSS声明，他们是以低优先级的开发者规则被对待的。

特征(Specificity)

选择器的特征在[CSS2规范](http://www.w3.org/TR/CSS2/cascade.html#specificity "CSS2")中被定义为：

* 如果样式声明是从style属性中来，而不是一个选择器的规则，那么特征就为1，否则就为0(= a)
* 计算选择器的ID属性的数量(= b)
* 计算选择器中其他属性和pseudo-class的数量(= c)
* 计算选择器中元素名和pseudo-element的数量(= d)

最后特征值就是把上面四个数字连起来：a-b-c-d

数的进制是上面四个数的最大值决定的。例如，如果`a = 14`，那么就可以使用16进制；而如果`a = 17`的话，就需要使用18或更高进制。后一种情况在选择器类似于`html body div div p ... `的情况下会发生。

一些例子：

{% codeblock %}
*             {}  /* a=0 b=0 c=0 d=0 -> specificity = 0,0,0,0 */
 li            {}  /* a=0 b=0 c=0 d=1 -> specificity = 0,0,0,1 */
 li:first-line {}  /* a=0 b=0 c=0 d=2 -> specificity = 0,0,0,2 */
 ul li         {}  /* a=0 b=0 c=0 d=2 -> specificity = 0,0,0,2 */
 ul ol+li      {}  /* a=0 b=0 c=0 d=3 -> specificity = 0,0,0,3 */
 h1 + *[rel=up]{}  /* a=0 b=0 c=1 d=1 -> specificity = 0,0,1,1 */
 ul ol li.red  {}  /* a=0 b=0 c=1 d=3 -> specificity = 0,0,1,3 */
 li.red.level  {}  /* a=0 b=0 c=2 d=1 -> specificity = 0,0,2,1 */
 #x34y         {}  /* a=0 b=1 c=0 d=0 -> specificity = 0,1,0,0 */
 style=""          /* a=1 b=0 c=0 d=0 -> specificity = 1,0,0,0 */
{% endcodeblock %}

规则排序

在所有的规则匹配之后，他们会根据层级规则被排序。Webkit使用冒泡排序法对小的集合排序，用归并排序法对大的集合排序。Webkit依靠重写`>`操作符来实现排序：

{% codeblock lang:c %}
static bool operator >(CSSRuleData& r1, CSSRuleData& r2)
{
    int spec1 = r1.selector()->specificity();
    int spec2 = r2.selector()->specificity();
    return (spec1 == spec2) : r1.position() > r2.position() : spec1 > spec2; 
}
{% endcodeblock %}

4.4 渐进的过程

Webkit使用一个标志来标记所有顶层样式表(包括`@imports`)被加载完成。如果在样式没有完全被加载时就去访问它，那么站位符就会被使并且在文档中被标记，在样式表被加载完成后重新计算。

