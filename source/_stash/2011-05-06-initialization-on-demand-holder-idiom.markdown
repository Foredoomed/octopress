---
layout: post
title: "initialization on demand holder idiom"
date: 2011-05-06 20:04
comments: true
categories: java 
---
This implementation is a well-performing and concurrent implementation valid in all versions of Java. The original implementation from Bill Pugh (see links below), based on the earlier work of Steve Quirk, has been modified to reduce the scope of LazyHolder.something  to private and to make the field final. 

{% codeblock Something.java lang:java %}
public class Something {
  private Something() {
  }
 
  private static class LazyHolder {
    private static final Something something = new Something();
  }
 
  public static Something getInstance() {
    return LazyHolder.something;
  }
}
{% endcodeblock %}

<!--more-->

**How it works**

The implementation relies on the well-specified initialization phase of execution within the Java Virtual Machine (JVM); see section 12.4 of Java Language Specification (JLS) for details.

When the class Something is loaded by the JVM, the class goes through initialization. Since the class does not have any static variables to initialize, the initialization completes trivially. The static class definition LazyHolder within it is not initialized until the JVM determines that LazyHolder must be executed. The static class LazyHolder is only executed when the static method getInstance is invoked on the class Something, and the first time this happens the JVM will load and initialize the LazyHolder class. The initialization of the LazyHolder class results in static variable something being initialized by executing the (private) constructor for the outer class Something. Since the class initialization phase is guaranteed by the JLS to be serial, i.e., non-concurrent, no further synchronization is required in the static getInstance method during loading and initialization. And since the initialization phase writes the static variable something in a serial operation, all subsequent concurrent invocations of the getInstance will return the same correctly initialized something without incurring any additional synchronization overhead.

**>When to use it**

Use this pattern if the initialization of the class is expensive and it cannot be done safely at class-loading time and the initialization is highly concurrent. The crux of the pattern is the safe removal of the synchronization overhead associated with accessing a singleton instance.

**When not to use it**

Using this pattern can result in unexpected behaviour if there is an error prone code inside the constructor of the singleton class Something. For instance, if the constructor is attempting to instantiate an external connection and the operation fails, the application will slip into a non-recoverable state. This is essentially because the nested static class LazyHolder has been initialized and loaded during the first call and the static variable INSTANCE has been assigned a null value (since the private constructor threw an exception). Any subsequent call to the static method getInstance() will fail with a NoClassDefFoundError while trying to access LazyHolder.something. 
