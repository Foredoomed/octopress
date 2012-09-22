---
layout: post
title: "为什么Java匿名内部类中的方法参数必须定义为final"
date: 2011-01-28 13:01
comments: true
categories: java 
---
今天在做一个功能，就是用户在注册后给他发送帐号激活邮件。在做这个功能的时候，我用匿名内部类来创建一个线程发送激活邮件，代码如下：

{% codeblock UserController.java lang:java %}
@Controller()
@RequestMapping(value = "/users")
public class UserController {

  @Autowired
  private MailManager mailManager;

  public void setMailManager(MailManager  mailManager) {
    this.mailManager = mailManager;
  }

  @RequestMapping(method = RequestMethod.POST)
  public String doingSignUp(@RequestParam("user_email_address") String user_email_address, @RequestParam("user_password") String user_password, @RequestParam("user_name") String user_name) {

    try {
      //用户注册代码省略
      new AbstractExecutionThreadService() {
      @Override
      protected void run() throws Exception {
      mailManager.sendActivateMail(user);//发送邮件
      }}.start();
    } catch (Exception e) {
      e.printStackTrace();
    }
      return "index";
  }
}
{% endcodeblock %}

<!--more-->

MVC框架用的是Spring  MVC，这个框架以后还会写一些关于他的文章（从我目前使用过的MVC框架对比来看，Spring  MVC是我认为的使用起来最舒服的框架，至于和Struts比较的话，就仁者见仁，智者见智了）。创建多线程的时候使用了Google的 [guava](http://code.google.com/p/guava-libraries/ "guava") 工具包里的多线程类来创建多线程（以后还会写一些关于这个工具包的文章）。从代码中可以看到，`AbstractExecutionThreadService`是抽象类，在里面重写了`run`方法，`run`法里面就是干一件事就是发送邮件。那么到这里就有一个问题了，由于`sendActivateMail`这个方法需要一个`User`类作为参数，而因为这个发送邮件的方法是在匿名内部类的里面，所以这个`User`类必须：(1)作为`UserController`类的实例变量，或者(2)在`doingSignUp`方法中把`User`类定义为`final`的局部变量。

一般在写Web程序的时候碰到类的实例变量的时候我们就因该当心多线程问题了，这里也是一样。由于Spring  MVC里的`Controller`类不是线程安全的，所以第一种把`User`类定义为`UserController`类的实例变量的方法就不能用了，那么剩下就只有用第二种方法：把`User`类定义为`final`。到这里第二个问题来了，这个问题是我以前没有想过的，为什么匿名内部类里的方法参数只能是`final`的。

在《Core Java 8th Editon》里是这么说的： **A local variable  that is declared final cannot be modified after it has been initialized. Thus,  it is guaranteed that the local variable and the copy that is made inside the  local class always have the same value**.  这句话是在介绍局部内部类的时候说的，用到匿名内部类上是一样的。简单的两句话只说了局部变量要和和他的拷贝要保持一致，但为什么要保持一致没有说。在stackoverflow上搜索了一下，发现了这个帖子： [Why inner  classes require "final" outer instance variables](http://stackoverflow.com/questions/3910324?tab=active#tab-top "Why inner  classes require "final" outer instance variables") 在这个帖子里还提到了另一个词："Closures"（闭包），有兴趣的读者可以去看一下关于闭包的内容，这里只给出 [维基百科上关于闭包的解释](http://en.wikipedia.org/wiki/Closure_(computer_science) "维基百科上关于闭包的解释") 。在这个帖子里又一次提到了为了防止在匿名内部类中的方法执行之前改变外部类局部变量的值，避免一些不可预测（奇怪）的问题，必须把外部类的局部变量声明为`final`。

我们再来看一下`class`文件会发现，`UserController`类被编译成了两个`class`文件：`UserController.class`和`UserController$1.class`。其中`UserController.class`还是本来的类，而`UserController$1.class`则是匿名内部类。我们用`javap  -private classname`命令来看一下这两个`class`文件里面都有些什么。

{% codeblock UserController.class lang:java %}
D:\>javap -private UserController
Compiled from "UserController.java"
public class info.liuxuan.controller.UserController extends java.lang.Object{
private info.liuxuan.mail.MailManager mailManager;
public info.liuxuan.controller.UserController();
public void setMailManager(info.liuxuan.mail.MailManager);
public java.lang.String doingSignUp(java.lang.String, java.lang.String, java.lang.String);
static info.liuxuan.mail.MailManager access$0(info.liuxuan.controller.UserController);
}
{% endcodeblock %}

{% codeblock UserController$1.class lang:java %}
D:\>javap -private UserController$1
Compiled from "UserController.java"
class info.liuxuan.controller.UserController$1 extends com.google.common.util.concurrent.AbstractExecutionThreadService{
final info.liuxuan.controller.UserController this$0;
private final info.liuxuan.entity.User val$user;
info.liuxuan.controller.UserController$1(info.liuxuan.controller.UserController,  info.liuxuan.entity.User);
protected void run() throws java.lang.Exception;
}
{% endcodeblock %}

我们可以清楚的看到在编译后的`UserController$1.class`中，`User`类已经变成了`UserController$1`的私有成员变量，并且通过构造方法来初始化。现在来看的话就可以理解为什么要保持一致了：构造方法传入外部类方法的局部变量，如果在内部类方法执行之前改变的话将发生与预想不同的结果。

也许你已经注意到了，在`UserController.class`中的`access$0`方法被定义成了`static`，为什么被定义为`static`的呢？这是为了不被其他子类重写。

