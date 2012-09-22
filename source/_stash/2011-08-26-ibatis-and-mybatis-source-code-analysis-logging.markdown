---
layout: post
title: "iBatis和myBatis源码分析之日志"
date: 2011-08-26 18:20
comments: true
categories: opensource
---
**一.包结构分析**

1.iBatis的logging包结构：

<table border="1" width="100%" cellpadding="3" cellspacing="0" summary=""><tbody><tr>
<th align="left" colspan="2" style="background-color:#CCCCFF">
<b>Packages</b></th></tr>
<tr bgcolor="white">
<td width="20%"><b>com.ibatis.common.logging</b></td>
<td>&nbsp;</td>
</tr>
<tr bgcolor="white">
<td width="20%"><b>com.ibatis.common.logging.jakarta</b></td>
<td>&nbsp;</td>
</tr>
<tr bgcolor="white">
<td width="20%"><b>com.ibatis.common.logging.jdk14</b></td>
<td>&nbsp;</td>
</tr>
<tr bgcolor="white">
<td width="20%"><b>com.ibatis.common.logging.log4j</b></td>
<td>&nbsp;</td>
</tr>
<tr bgcolor="white">
<td width="20%"><b>com.ibatis.common.logging.nologging</b></td>
<td>&nbsp;</td>
</tr>
</tbody></table>


iBatis把对logging支持的类全部放在了common包的子包下，并且又根据不同的logging实现又分成了commons-logging，jdk1.4自带的logging，log4j和nologging四个子包。个人感觉整个包的划分还是非常清晰的，但是nologging包的存在感觉有点鸡肋，因为日志是一个系统必备的功能之一，很难想象一个系统缺少了日志怎么做数据分析，性能调优等。难道作者认为有些时候或有一天就不需要日志了？ <!--more-->

2.myBatis的logging包结构：

<table border="1" width="100%" cellpadding="3" cellspacing="0" summary=""><tbody><tr>
<th align="left" colspan="2" style="background-color:#CCCCFF">
<b>Packages</b></th></tr>
<tr bgcolor="white">
<td width="20%"><b>org.apache.ibatis.logging</b></td>
<td>&nbsp;</td>
</tr>
<tr bgcolor="white">
<td width="20%"><b>org.apache.ibatis.logging.commons</b></td>
<td>&nbsp;</td>
</tr>
<tr bgcolor="white">
<td width="20%"><b>org.apache.ibatis.logging.jdbc</b></td>
<td>&nbsp;</td>
</tr>
<tr bgcolor="white">
<td width="20%"><b>org.apache.ibatis.logging.jdk14</b></td>
<td>&nbsp;</td>
</tr>
<tr bgcolor="white">
<td width="20%"><b>org.apache.ibatis.logging.log4j</b></td>
<td>&nbsp;</td>
</tr>
<tr bgcolor="white">
<td width="20%"><b>org.apache.ibatis.logging.nologging</b></td>
<td>&nbsp;</td>
</tr>
<tr bgcolor="white">
<td width="20%"><b>org.apache.ibatis.logging.slf4j</b></td>
<td>&nbsp;</td>
</tr>
<tr bgcolor="white">
<td width="20%"><b>org.apache.ibatis.logging.stdout</b></td>
<td>&nbsp;</td>
</tr>
</tbody></table>


可以看到myBatis的logging包结构较之iBatis基本没变，但是增加了jdbc，slf4j以及stdout的支持。对jdbc执行sql语句的日志支持看上去很美好，但是给性能带来的影响比较大，这个等下面的源码分析再详细说明；增加对slf4j的支持应该是大势所趋没有问题；而增加对stdout的支持是一个很好的功能，在测试阶段应该会是一个很好的日志功能实现。

**二.源码分析**

1.iBatis

(1)Log接口：

{% codeblock lang:java %}
public interface Log {

  boolean isDebugEnabled();

  void error(String s, Throwable e);

  void error(String s);
  
  public void debug(String s);

  public void warn(String s);

}
{% endcodeblock %}

Log接口定义了三个log级别，所有的log实现都会去实现这个接口，所以它是所有log的代表。

(2)LogFactory工厂类：

{% codeblock lang:java %}
public class LogFactory {

  private static Constructor logConstructor;

  static {
    tryImplementation("org.apache.commons.logging.LogFactory", "com.ibatis.common.logging.jakarta.JakartaCommonsLoggingImpl");
    tryImplementation("org.apache.log4j.Logger", "com.ibatis.common.logging.log4j.Log4jImpl");
    tryImplementation("java.util.logging.Logger", "com.ibatis.common.logging.jdk14.Jdk14LoggingImpl");
    tryImplementation("java.lang.Object", "com.ibatis.common.logging.nologging.NoLoggingImpl");
  }

  private static void tryImplementation(String testClassName, String implClassName) {
    if (logConstructor == null) {
      try {
        Resources.classForName(testClassName);
        Class implClass = Resources.classForName(implClassName);
        logConstructor = implClass.getConstructor(new Class[]{Class.class});
      } catch (Throwable t) {
      }
    }
  }

  public static Log getLog(Class aClass) {
    try {
      return (Log)logConstructor.newInstance(new Object[]{aClass});
    } catch (Throwable t) {
      throw new RuntimeException("Error creating logger for class " + aClass + ".  Cause: " + t, t);
    }
  }

  public static synchronized void selectLog4JLogging() {
    try {
      Resources.classForName("org.apache.log4j.Logger");
      Class implClass = Resources.classForName("com.ibatis.common.logging.log4j.Log4jImpl");
      logConstructor = implClass.getConstructor(new Class[]{Class.class});
    } catch (Throwable t) {
    }
  }
  
  public static synchronized void selectJavaLogging() {
    try {
      Resources.classForName("java.util.logging.Logger");
      Class implClass = Resources.classForName("com.ibatis.common.logging.jdk14.Jdk14LoggingImpl");
      logConstructor = implClass.getConstructor(new Class[]{Class.class});
    } catch (Throwable t) {
    }
  }
}
{% endcodeblock %}

在这个工厂类里我们可以非常容易地感觉到“坏味道”，那就是在catch块中什么都不做，这是应该尽量避免出现的情况。在static块中会尝试依次加载所有的log实现类，这点也是值得商榷的。因为在一个系统中一般只使用一种日志实现，一次性加载所有的实现只会带来性能问题；还有一个问题就是是否需要动态切换日志实现的功能，至少我觉得这个功能也是个鸡肋。

2.myBatis

(1)Log接口与iBatis的没有变化

(3)LogFactory工厂类：

{% codeblock lang:java %}
public class LogFactory {

  private static Constructor< ? extends Log> logConstructor;

  static {
    tryImplementation(new Runnable() {
      public void run() {
        useSlf4jLogging();
      }
    });
    tryImplementation(new Runnable() {
      public void run() {
        useCommonsLogging();
      }
    });
    tryImplementation(new Runnable() {
      public void run() {
        useLog4JLogging();
      }
    });
    tryImplementation(new Runnable() {
      public void run() {
        useJdkLogging();
      }
    });
    tryImplementation(new Runnable() {
      public void run() {
        useNoLogging();
      }
    });
  }

  public static Log getLog(Class< ?> aClass) {
    try {
      return logConstructor.newInstance(new Object[]{aClass});
    } catch (Throwable t) {
      throw new LogException("Error creating logger for class " + aClass + ".  Cause: " + t, t);
    }
  }

  public static synchronized void useSlf4jLogging() {
    setImplementation("org.apache.ibatis.logging.slf4j.Slf4jImpl");
  }

  public static synchronized void useCommonsLogging() {
    setImplementation("org.apache.ibatis.logging.commons.JakartaCommonsLoggingImpl");
  }

  public static synchronized void useLog4JLogging() {
    setImplementation("org.apache.ibatis.logging.log4j.Log4jImpl");
  }

  public static synchronized void useJdkLogging() {
    setImplementation("org.apache.ibatis.logging.jdk14.Jdk14LoggingImpl");
  }

  public static synchronized void useStdOutLogging() {
    setImplementation("org.apache.ibatis.logging.stdout.StdOutImpl");
  }

  public static synchronized void useNoLogging() {
    setImplementation("org.apache.ibatis.logging.nologging.NoLoggingImpl");
  }

  private static void tryImplementation(Runnable runnable) {
    if (logConstructor == null) {
      try {
        runnable.run();
      } catch (Throwable t) {
        //ignore
      }
    }
  }

  private static void setImplementation(String implClassName) {
    try {
      @SuppressWarnings("unchecked")
      Class< ? extends Log> implClass = (Class< ? extends Log>) Resources.classForName(implClassName);
      Constructor< ? extends Log> candidate = implClass.getConstructor(new Class[]{Class.class});
      Log log = candidate.newInstance(new Object[]{LogFactory.class});
      log.debug("Logging initialized using '" + implClassName + "' adapter.");
      logConstructor = candidate;
    } catch (Throwable t) {
      throw new LogException("Error setting Log implementation.  Cause: " + t, t);
    }
  }
{% endcodeblock %}

在myBatis的实现里，除了加入了范型，多线程外没有什么太大的变化。还是要加载所有的log实现类，但是更大的问题是加载方法用synchronized来修饰了，且不论是否有必要全部加载，但就全部的同步方法就会给性能造成负面影响；而且在iBatis中的“坏味道”依旧存在。

三.总结

在这篇博文里比较了iBatis和myBatis的log实现，发现了存在“坏味道”，即在catch块中什么事都没做，这种情况应该避免发生；还有一次性加载所有的log实现类以及动态切换log实现都是不太常用的功能，如果要用iBatis或myBatis的log实现的话需要修改源代码来提高性能。

