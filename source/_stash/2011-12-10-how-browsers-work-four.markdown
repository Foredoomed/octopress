---
layout: post
title: "浏览器是如何工作的(四)"
date: 2011-12-10 14:18
comments: true
categories: browsers
---
3.2 HTML解析器

HTML解析器的工作是把HTML标记语言解析成为解析树。

3.2.1 HTML语法定义

HTML的词汇和语法是由W3C组织制定的。当前的版本是HTML4，HTML5还在制定当中。

3.2.2 不是内容无关语法的HTML

我们在介绍解析的时候知道了，语法可以被定义为正式使用的格式，比如BNF。

不过不幸的是，所有约定的解析器都不适用于HTML。HTML不能简单的定义一个解析器需要的内容无关语法。

有一个定义HTML的正式格式 - DTD，但它也不是内容无关语法。

所以第一眼看上去就很奇怪，HTML更接近于XML。而XML的解析器却有很多，而且有一个HTML的XML版本 - XHTML，那么有什么不同呢？

不同之处在于，HTML的方式更加宽松，它允许你省略特定的标签，然后会隐式的为你加上去。总的来看，相对于XML来说，HTML是“软”语法。

显然，这个细小的差别让两者大不一样。一方面这是HTML流行的主要原因 - 允许你犯错，而且对于开发人员来说非常容易。另一方面，它有使的写一个正式语法变得非常困难。所以总结来看 - HTML不能被简单的解析，不能被约定的解析器解析，因为它不是内容无关语法，也不能被XML解析器解析。

<!-- more -->

3.2.3 HTML DTD

HTML的定义是一个DTD的格式。这个格式被用来定义[SGML](http://en.wikipedia.org/wiki/Standard_Generalized_Markup_Language "SGML")家族语言。它包含了对所以允许元素的定义，他们的属性和继承关系。我们已经知道，HTML DTD不能形成一个内容无关语法。

还有一些类型的DTD。像严格的类型会紧密遵守规范，但其他类型包含对以前浏览器使用的标签的支持，这个目的就是向后兼容旧内容。当前的严格版DTD在[这里](http://www.w3.org/TR/html4/strict.dtd "strict dtd")。

3.2.4 DOM

输出树(也就是解析树)，它包含了DOM元素和属性节点。像JavaScript一样，它是HTML文档和元素借口对外界的对象表示。

这棵树的根节点是[Document](http://www.w3.org/TR/1998/REC-DOM-Level-1-19981001/level-one-core.html#i-Document "Document")对象。

DOM是大多数情况下是一对一关系的标记。比如：

{% codeblock lang:html %}
<html>
  <body>
    <p>
      Hello World
    </p>
    <div> <img src="example.png"/></div>
  </body>
</html>
{% endcodeblock %}

会被解析成下面这棵树：

![dom tree](http://farm8.staticflickr.com/7155/6485284597_9b1d73e56d.jpg" width="400" height="219")

像HTML一样，DOM是由W3C组织定义的，参考[这里](http://www.w3.org/DOM/DOMTR "DOM")。这是一个处理文档的通用的规范。特殊的模块解释特殊的HTML元素。HTML的定义可以参考[这里](http://www.w3.org/TR/2003/REC-DOM-Level-2-HTML-20030109/idl-definitions.html "html")

我所说的解析树包含DOM节点的意思是，这棵树是由实现某个DOM接口的元素组成的，而浏览器在内部有包含其他属性的具体实现。

3.2.5 解析算法

在上一节里我们看到，HTML不能被自上而下或者自下而上的解析器解析。其中的原因是：

1. HTML的容错特性
2. 浏览器有传统的错误来支持为人熟知的非法HTML
3. 解析过程是重复进出的，一般情况下源代码不会改变，但是在HTML中，包含`document.write`的标签会另外加入标记，所以解析过程实际上会改变输入。

因为不能使用一般的解析技术，所以浏览器就为HTML开发了专用的解析器。

解析算法详细的定义在[HTML5规范](http://www.whatwg.org/specs/web-apps/current-work/multipage/parsing.html)里，算法是由二阶段标记和构建树组成。标记是在词法分析阶段处理，把输入解析成标记。HTML标记有开始标签，结束标签，属性名和属性值。

标记器可以识别标记，并且把他们传给树构造器，然后再处理下一个标记直到输入结束。

![parsing flow](http://farm8.staticflickr.com/7150/6485330329_01565f62ab.jpg" width="308" height="400")


