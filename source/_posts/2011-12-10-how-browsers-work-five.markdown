---
layout: post
title: "浏览器是如何工作的(五)"
date: 2011-12-10 17:14
comments: true
categories: browsers 
---
3.2.6 标记算法

这个算法的输出是一个HTML标记，它可以用状态机来表示。每个状态消费输入流的一个或多个字符，然后根据这些字符更新下一个状态。状态的更新是由当前标记状态和树构造状态决定的。这意味着根据当前状态，同样的字符会对下一个正确的状态产生不同的结果。这个算法太复杂，所以不能完全描述清楚，所以让我来看一个简单的例子，这个例子会帮助我们理解这个算法。

标记下面的HTML：
{% codeblock %}
<html>
  <body>
    Hello world
  </body>
</html>
{% endcodeblock %}

<!-- more -->

初始状态是”**Data state**”。当遇到字符`<`后，状态被改变成”**Tag open state**”。消费一个`a`到`z`的字符会创建”**Start tag token**”，而状态会变成”**Tag name state**”。然后状态一直保持到遇到`>`字符。每个字符被加在了新标记名称之后，在这个例子中，被创建的标记是`html`。

当遇到`>`字符后，当前状态被改回成”**Data state**”。下面的`<body>`标签是同样的处理步骤。到目前为止，`<html>`和`<body>`标签处理完成了，回到了”**Data state**”状态。然后碰到的是`Hello world`中的`H`字符，这会导致标记的创建直到遇到`</body>`标签的`<`字符，然后为`Hello world`的每个字符释放字符标记。

我们现在回到了”**Tag open state**”状态。下面一个遇到的字符是`/`，这会导致创建`end tag token`并且状态改变为”**Tag name state**”。同样，这个状态持续到遇到`>`字符。然后新的标签会被释放，回到”**Data state**”状态。`</html>`标签是同样的处理流程。

![tokenizing state machine](http://farm8.staticflickr.com/7145/6485458369_9f574b3101.jpg" width="500" height="309")

3.2.7 树的构造算法

当解析器被创建的时候，文档对象就被创建了。在整个构造树的过程中，DOM树的根结点会被修改或者添加元素。每个节点的释放由树构造器处理。对于每个标记，规范定义了与之相对应的DOM元素，所以会创建相对应的DOM元素。除了向DOM树中添加元素外，还会向开放元素的栈中添加。这个栈是被用来纠正嵌套错误和未关闭的标签。这个算法也被描述为一个状态机，那些状态被称为**插入模式**(insertion modes)。

让我们来看一下树的构造过程，比如下面的输入：

{% codeblock lang:html %}
<html>
  <body>
    Hello world
  </body>
</html>
{% endcodeblock %}

输入对于构造树的阶段来说，是一个初始模式是”**initial mode**”的标记序列。获取HTML标记会引发向”**before html**”模式的移动，并且在那个模式下的再次处理。这会造成`HTMLHtmlElement`元素的创建，然后被附加到对象模型的根结点上去。

然后状态会变成”**before head**”。尽管没有`head`标记，当得到`body`标记，`HTMLHeadElement`元素会被隐式地创建，然后加入到树中。

现在模式移动，从”**in head**”模式再到”**after head**”模式。`body`标记被再次处理，`HTMLBodyElement`元素被创建，并且模式被转移到了”**in body**”。

现在碰到的是`Hello world`字符串的字符标记。首先会创建和插入`Text`节点，其他字符会附加到这个节点上去。

如果碰到`body`的关闭标记的话会使模式转入”**after body**”。然后碰到`html`关闭标签，模式转换成"**after after body**"。碰到文件结束标记后，解析结束。

![tree construction](http://farm8.staticflickr.com/7156/6485575299_706ab0d663.jpg" width="346" height="500")

3.2.8 解析结束后

解析结束后会标记文档为interactive，然后开始解析应该在文档解析完成后执行的脚本。这个文档状态会被设置成”complete”，而且”load”事件会被触发。

3.2.9 浏览器的容错性

你永远不会在HTML页面上得到一个"Invalid Syntax"错误。浏览器会修复任何一个非法的文档，使它能够正常运行。

拿下面这个例子来说：

{% codeblock lang:html %}
<html>
  <mytag>
  </mytag>
  <div>
  <p>
  </div>
    Really lousy HTML
  </p>
</html>
{% endcodeblock %}

在这个例子中我犯了许多错误：`mytag`不是标准标签，`p`和`div`元素都嵌套错误。但即使这样，浏览器也能正确执行这段HTML，所以这些错误都被浏览器给纠正了。

错误处理在各个浏览器中是非常一致的，但是令人惊奇的是这并不是HTML当前规范的一部分。就像书签和前进/后退按钮一样，这是在浏览器多年进化的结果。

HTML5规范定义了一些上述的需求。Webkit在它的HTML解析类的开头美妙地总结如下：
> The parser parses tokenized input into the document, building up the document tree. If the document is well-formed, parsing it is straightforward.  

> Unfortunately, we have to handle many HTML documents that are not well-formed, so the parser has to be tolerant about errors.  

> We have to take care of at least the following error conditions:  

> 1. The element being added is explicitly forbidden inside some outer tag. In this case we should close all tags up to the one, which forbids the element, and add it afterwards.
2. We are not allowed to add the element directly. It could be that the person writing the document forgot some tag in between (or that the tag in between is optional). This could be the case with the following tags: HTML HEAD BODY TBODY TR TD LI (did I forget any?).
3. We want to add a block element inside to an inline element. Close all inline elements up to the next higher block element.
4. If this doesn't help, close elements until we are allowed to add the element or ignore the tag.

让我们来看一些Webkit的容错例子:

**`</br>`代替`<br>`**

有些站点使用`</br>`代替`<br>`，为了跟IE和Firefox兼容，Webkit会把`</br>`处理成`<br>`。

Webkit代码：

{% codeblock lang:c %}
if (t->isCloseTag(brTag) && m_document->inCompatMode()) {
     reportError(MalformedBRError);
     t->beginTag = true;
}
{% endcodeblock %}

注意：错误处理只在内部执行，不会展现给用户。

**A stray table**

stras table指的是一个table元素里直接里包含了另一个table元素，就像下面的例子：

{% codeblock lang:html %}
<table>
    <table>
        <tr><td>inner table</td></tr>
    </table>
    <tr><td>outer table</td></tr>
</table>
{% endcodeblock %}

Webkit会改变它的继承结构为两个同级别的table：
{% codeblock lang:html %}
<table>
    <tr><td>outer table</td></tr>
</table>
<table>
    <tr><td>inner table</td></tr>
</table>
{% endcodeblock %}

处理这部分的代码为：

{% codeblock lang:c %}
if (m_inStrayTableContent && localName == tableTag)
        popBlock(tableTag);
{% endcodeblock %}

Webkit利用栈来保存当前内容，它会弹出内部的table，这样的话这两个table就是同级别的了。

**从元素嵌套**

为了防止用户把一个form放到另一个form内，内部的form将被丢弃，代码为：

{% codeblock lang:c %}
if (!m_currentFormElement) {
        m_currentFormElement = new HTMLFormElement(formTag,    m_document);
}
{% endcodeblock %}

**过深的标签嵌套**

注释说明的非常清楚：

{% codeblock lang:c %}
// www.liceo.edu.mx is an example of a site that achieves a level of nesting of about 
// 1500 tags, all from a bunch of <b>s. We will only allow at most 20 nested tags   
// of the same type before just ignoring them all together.
bool HTMLParser::allowNestedRedundantTag(const AtomicString& tagName)
{

unsigned i = 0;
for (HTMLStackElem* curr = m_blockStack;
         i < cMaxRedundantTagDepth && curr && curr->tagName == tagName;
     curr = curr->next, i++) { }
return i != cMaxRedundantTagDepth;
}
{% endcodeblock %}

**html或者body的关闭标签位置错误**

同样，注释说明的非常清楚：

{% codeblock lang:c %}
// Support for really broken html. We never close the body tag, since some stupid web pages 
// close it before the actual end of the doc. Let's rely on th//e end() call to close things.
if (t->tagName == htmlTag || t->tagName == bodyTag )
        return;
{% endcodeblock %}

所以web开发人员要注意，除非你想作为Webkit的容错代码片段的例子出现，否则要写格式良好的HTML。

