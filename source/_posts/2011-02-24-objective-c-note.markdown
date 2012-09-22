---
layout: post
title: "Objective-C学习笔记"
date: 2011-02-24 14:26
comments: true
categories: notes
---
**一.基本数据类型**

<a href="http://www.flickr.com/photos/60110479@N08/5488573123/" title="Flickr 上 Foredoomed 的 objectivecprimitivetype"><img src="http://farm6.static.flickr.com/5214/5488573123_a7dfd0551c_z.jpg" width="640" height="563" alt="objectivecprimitivetype" /></a>
<a href="http://www.flickr.com/photos/60110479@N08/5488563233/" title="Flickr 上 Foredoomed 的 basicdatatypes"><img src="http://farm6.static.flickr.com/5176/5488563233_153dd27348_z.jpg" width="640" height="513" alt="basicdatatypes" /></a>

<!--more-->

id : 可以保存任何类型的对象，也就是对象的泛型类型。(id类型非常重要，因为他是Objective-C中的重要特性多态和动态绑定的基础)

**二.类,对象和函数**

(1)函数调用方式

`[ ClassOrInstance method ]`   也可以理解为：  `[ receiver message ]`

(2)类的声明与实现

`@interface`是类的声明关键字，一般的格式为：

{% codeblock Example lang:objc %}
@interface NewClassName: ParentClassName 
{ 
    memberDeclarations; 
}  
methodDeclarations; 
@end
{% endcodeblock %}

`@implementation`是类的实现关键字，一般的格式为：

{% codeblock Example lang:objc %}
@implementation NewClassName 
    methodDefinitions; 
@end
{% endcodeblock %}

(3)自动生成类属性的`setter`和`getter`函数的方法

先在类的声明里用`@property`关键字定义类的属性，然后在类实现文件中用`@synthesize`关键字。

(4)`self`：本类的实例

(5)`@class`：此关键字可以让编译器知道要引用的类的类型，但如果用到类中的属性或函数，那就要用`#import “XXX.h`来代替。

(6)如果某个对象调用继承自`NSObject`中的`release`函数，那么这个对象只会在没有任何引用的情况下才会释放内存，`release`函数通过调用实际释放内存的 `dealloc`函数来实现释放内存的操作(不要重写`release`函数，而是重写`dealloc`函数)。

(6)异常捕获
{% codeblock Example lang:objc %}
@try { 
  statement 
  statement 
  ... 
} 
@catch (NSException *exception) { 
  statement 
  statement 
  ... 
}
{% endcodeblock %}

(7)`volatile`：防止编译器优化看似多余的变量赋值

(8)`@protocol`：类似于Java中的接口，跟在`@interface`后尖括号（< ...>）中。声明在`@protocol`中的函数必须实现，而声明在`@optional`中的函数可以不实现。

(9)属性特性

<a href="http://www.flickr.com/photos/60110479@N08/5488588537/" title="Flickr 上 Foredoomed 的 propertyattributes"><img src="http://farm6.static.flickr.com/5180/5488588537_1f41be3c8e_z.jpg" width="564" height="504" alt="propertyattributes" /></a>
