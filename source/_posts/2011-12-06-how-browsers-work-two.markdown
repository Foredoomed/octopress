---
layout: post
title: "浏览器是如何工作的(二)"
date: 2011-12-06 22:18
comments: true
categories: browsers
---
2 渲染引擎

渲染引擎要做的就是渲染，也就是在浏览器的窗体里显示请求的内容。

一般情况下，渲染引擎可以显示HTML,XML和图片。它也可以通过插件的方式显示其他类型类型的文件，比如PDF。但是，在这章中我们将把焦点放在主要的用例上：显示由CSS格式化过的HTML和图片。

<!-- more -->

2.1 多种渲染引擎

我们作为参考的浏览器：Firefox, Chrome和Safari是构建在不同的两个渲染引擎之上的。Firefox使用的是Gecko，而Chrome和Safari使用的都是Webkit。

Webkit是开源的渲染引擎，它开始时只是针对Linux平台，后来由苹果公司修改后以兼容Mac和Windows平台。[webkit.org](http://webkit.org "webkit.org")上有更详细的介绍。

2.2 渲染的主要流程

渲染引擎是从网络层中获取请求的内容。然后是渲染的基本流程：

![flow](http://farm8.staticflickr.com/7012/6465898245_abf04a9566.jpg" width="500" height="55")

渲染引擎会开始解析HTML文档，并且把HTML标签转换成为被称作“内容树(content tree)”的DOM节点。还会解析不管是外部引用的还是内联的CSS。样式和可视指令(visual instructions)一起会被创建成为另一棵树-渲染树(render tree)。

在渲染树构造完毕后，浏览器就会开始处理布局。也就是说会分配给每个节点一个屏幕上准确位置的坐标。下一步就是绘制了，即使用UI后端(UI backend)来绘制渲染树每个节点。

理解这是个渐进的过程是非常重要的。为了更好的用户体验，渲染引擎会尽可能快的把内容显示在屏幕上。直到所有的HTML已经解析完毕，并且开始构造和布局渲染树之前，渲染引擎是不会停下等待的。部分的内容会被解析和显示，而在这个时候渲染引擎会继续处理从网络上返回的内容。

2.3 主要流程实例

![webkitflow](http://farm8.staticflickr.com/7151/6465898595_b573cc3861.jpg" width="500" height="232")

![geckoflow](http://farm8.staticflickr.com/7018/6465898835_b5e5658f50.jpg" width="500" height="232")

从上两张图山可以看到，尽管Webkit和Gecko使用不同的术语，但是整个流程是基本一样的。

在Gecko里，把已经经过视觉格式化过的元素树称为框架树(frame tree)。每个元素是一个框架。Webkit则用了另外一个术语：由渲染对象(render object)组成的渲染树(render tree)。

Webkit把元素的放置称为布局(layout)，而Gecko则称之为回流(reflow)。附件(attachment)是Webkit用来表示连接DOM节点和视觉信息来创建渲染树的过程。一个细小的，非语意上的区别是，Gecko在HTML和DOM之间又另外多出来一层。它被称为内容下沉层(content sink)，并且它是制造DOM元素的工厂。下面将会说明整个流程的各个部分。

