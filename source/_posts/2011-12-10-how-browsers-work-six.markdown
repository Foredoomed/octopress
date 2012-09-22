---
layout: post
title: "浏览器是如何工作的(六)"
date: 2011-12-10 20:57
comments: true
categories: browsers
---
3.3 CSS解析

和HTML不同，CSS是内容无关的语法，所以可以被一般解析器解析。CSS规范定义了词法和语法。

让我们来看一些例子：

CSS的文法(词汇)是用正则表达式定义的

{% codeblock %}
comment   \/\*[^*]*\*+([^/*][^*]*\*+)*\/
num   [0-9]+|[0-9]*"."[0-9]+
nonascii  [\200-\377]
nmstart   [_a-z]|{nonascii}|{escape}
nmchar    [_a-z0-9-]|{nonascii}|{escape}
name    {nmchar}+
ident   {nmstart}{nmchar}*
{% endcodeblock %}

ident是identifier的缩写，比如class的名字；name是元素id(#号引用)。

<!-- more -->

而语法使用BNF描述的：

{% codeblock %}
ruleset
  : selector [ ',' S* selector ]*
    '{' S* declaration [ ';' S* declaration ]* '}' S*
  ;
selector
  : simple_selector [ combinator selector | S+ [ combinator? selector ]? ]? 
  ;
simple_selector
  : element_name [ HASH | class | attrib | pseudo ]*
  | [ HASH | class | attrib | pseudo ]+
  ;
class
  : '.' IDENT
  ;
element_name
  : IDENT | '*'
  ;
attrib
  : '[' S* IDENT S* [ [ '=' | INCLUDES | DASHMATCH ] S*
    [ IDENT | STRING ] S* ] ']'
  ;
pseudo
  : ':' [ IDENT | FUNCTION S* [IDENT S*] ')' ]
  ;
{% endcodeblock %}

比如下面的CSS：

{% codeblock %}
div.error , a.error {
  color:red;
  font-weight:bold;
}
{% endcodeblock %}

`div.error`和`a.error`是选择器，大括号里包含的就是ruleset里定义的规则，它适用于下面的定义：

{% codeblock %}
ruleset
  : selector [ ',' S* selector ]*
    '{' S* declaration [ ';' S* declaration ]* '}' S*
  ;
{% endcodeblock %}

也就是说，ruleset是一个或多个选择器，他们由逗号隔开，S代表空格。ruleset包括大括号和其中的一个或多个，由分号隔开的声明。”声明”和”选择器“会在下面的BNF定义中介绍。

3.3.1 Webkit的CSS解析器

Webkit使用Flex和Bison解析器生成器，从CSS语法文件自动创建解析器，而Bison创建的是自下而上的shift-reduce解析器。Firefox使用手动写的自上而下的解析器。两种情况下，每个CSS文件都会被解析成为StyleSheet对象，每个对象都包含CSS规则。CSS规则对象包含选择器和声明对象，还有其他CSS语法中对应的对象。

![parsing css](http://farm8.staticflickr.com/7145/6486240717_22094df7b7.jpg" width="500" height="393")

3.4 处理scripts和style sheets的顺序

3.4.1 Scripts

web的模型是异步的。开发者希望当遇到`<script>`标签后，脚本能够立即被解析和执行。文档的解析会被挂起，知道脚本执行完毕。如果脚本是从外部引用的，那么必须先从网络上把拿到这个脚本，这个过程是同步的，直到脚本被抓取到本地后才会开始解析。这个模型已经存在了很多年，而且在HTML4和5规范中也有定义。开发者可以把脚本标记为”differ”，这样的话就不会挂起文档解析，脚本解析完后就会执行。HTML5加入了一个可以标记脚本为异步的选项，所以它会被另一个线程解析和执行。

3.4.2 Speculative parsing

Webkit和Firefox都会做这种优化。当脚本在执行时，另一个线程解析剩下的文档和找出其他需要从网络上加载的资源，并且加载他们。在这种方式下，资源可以在并行连接的情况下加载，而且总的速度会更好。注意，speculative parser不会修改DOM树，然后把它交给主解析器，它只解析外部资源的引用，比如外部脚本，样式表和图片。

3.4.3 Style sheets

没有理由等待DOM树和停止文档解析。有个脚本的问题就是在文档解析阶段脚本请求样式的信息。如果样式还没有被加载和解析，脚本会获得错误的结果，这样明显会产生许多问题。这看上去是一个边界条件，但又是非常普遍。Firefox在CSS加载和解析的时候会阻塞所有的脚本。Webkit只在尝试获取某个特定的CSS属性，而这个属性会被未加载的CSS影响到时阻塞脚本。

4 渲染构造树

当DOM树被构造完成后，浏览器会构造另一棵树，即渲染树。这棵树是按顺序的视觉元素构成的，而且他们会被显示。它是文档的视觉展现。渲染树的目的是使内容在正确的顺序上绘制。

Firefox把渲染树里的元素称为"frames"，Webkt则是render或render object。
Webkit的RenderObject类是renderer的基本类，它的定义如下：

{% codeblock lang:c %}
class RenderObject{
  virtual void layout();
  virtual void paint(PaintInfo);
  virtual void rect repaintRect();
  Node* node;  //the DOM node
  RenderStyle* style;  // the computed style
  RenderLayer* containgLayer; //the containing z-index layer
}
{% endcodeblock %}

每个renderer代表了一个矩形区域，类似于节点的CSS盒子模型。它包括了几何信息，比如宽度，高度和位置。

盒子的类型会被”display”样式的属性影响(参看style computation部分)。下面的Webkit代码是决定的是，根据显示属性，什么类型的renderer应该被创建为DOM节点。

{% codeblock lang:c %}
RenderObject* RenderObject::createObject(Node* node, RenderStyle* style)
{
    Document* doc = node->document();
    RenderArena* arena = doc->renderArena();
    ...
    RenderObject* o = 0;

    switch (style->display()) {
        case NONE:
            break;
        case INLINE:
            o = new (arena) RenderInline(node);
            break;
        case BLOCK:
            o = new (arena) RenderBlock(node);
            break;
        case INLINE_BLOCK:
            o = new (arena) RenderBlock(node);
            break;
        case LIST_ITEM:
            o = new (arena) RenderListItem(node);
            break;
       ...
    }

    return o;
}
{% endcodeblock %}

元素类型也是会被考虑的，例如，form控制和table有特殊的框架。在Webkit中，如果元素想要创建特殊的renderer，它会覆盖`createRenderer`方法。renderer指向包含非几何信息的style object。

