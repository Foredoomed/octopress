---
layout: post
title: "方法的标记参数"
date: 2011-06-28 15:16
comments: true
categories: programming
---
Martin Fowler 在他的博客里发表了一篇关于方法的标志参数（Flag Argument）的博文，原文参见[这里](http://martinfowler.com/bliki/FlagArgument.html "http://martinfowler.com/bliki/FlagArgument.html")。

我读完后立刻开始回忆起我以前的做法来，假设有这样一个场景，我们有两类人（即有钱人和穷人），现在房地产那边需要定义个买房子的方法，如果是有钱人就能买房；反之，如果是穷人则买不了房子。好了，如果是我的话，我想我会这么定义：

{% codeblock lang:java%}
public Customer buyHouse(Customer customer, boolean isRich){
    //do something
}
{% endcodeblock %}

在我看到这篇博文之前，我一定会这么干。看，这样做多简单，只要在这个方法里再根据这个布尔值判断一下就行了。但是在我看完这篇博文后，我的想法发生了比较大的改变。这样定义方法虽然简单，但对于阅读代码的人来说却很头大。如果结合真实情况来看，这个**isRich**参数在大多数情况下都要通过调用另外一个方法计算出来再传入这个方法。也就是说，在调用这个方法的时候，**isRich**是**True**还是**False**是不确定的。 <!--more-->

这样做有问题吗？按照Martin Fowler在博文上写的，他偏向于定义两个方法来处理这种情况，即：

{% codeblock lang:java%}
public Customer richesCanBuyHouse(Customer customer){...}
public Customer poorsCantBuyHouse(Customer customer){...}
{% endcodeblock %}

Martin Fowler的理由是调用这个方法的时候，就能清楚的知道程序员的意图，而不是要记住这个标记参数的意义。经过思考和实验，我发现将方法中的功能尽量单一化的做法将使整个系统收益，不仅可以提高测试覆盖率，易于定位Bug，而且还易于修改和重构。

Martin Fowler还举了另外一个例子，即布尔值的Set方法：

{% codeblock lang:java%}
//提倡这种做法
void setOn();
void setOff();

//而不是这种做法
void setSwitch(boolean on);
{% endcodeblock %}

这个例子其实和上面的例子差不多，虽然多了一个方法，但是程序将会变得清晰易读。

