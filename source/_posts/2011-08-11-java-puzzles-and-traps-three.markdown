---
layout: post
title: "Java常见疑惑和陷阱(三)"
date: 2011-08-11 16:56
comments: true
categories: java
---
**29.令人混淆的构造器案例**  
下面的程序会打印出什么呢?

{% codeblock lang:java %}
public class Confusing {
  private Confusing(Object o) {
    System.out.println("Object");
  }

  private Confusing(double[] dArray) {
    System.out.println("double array");
  }

  public static void main(String[] args) {
    new Confusing(null);
  }
}
{% endcodeblock %}

如果运行该程序，你会发现打印的是“double array“。这是因为：Java的重载解析过程是以两阶段运行的。第一阶段选取所有可获得并且可应用的方法或构造器。第二阶段在第一阶段选取的方法或构造器中选取最精确的一个。**如果一个方法或构造器可以接受传递给另一个方法或构造器的任何参数,那么我们就说第一个方法比第二个方法缺乏精确性。**

如果想要调用`Confusing(Object)`构造方法,你需要这样改写代码:`new Confusing((Object)null)`。这可以确保只有`Confusing(Object)`是可应用的。更一般地讲,要想强制要求编译器选择一个精确的重载版本,需要将实际的参数转型为形式参数所声明的类型。 <!--more-->

**30.我所得到的都是静态的**  
下面的程序将打印出什么呢?

{% codeblock lang:java %}
class Dog {
  public static void bark() {
    System.out.print("woof");
  }
}

class Dog1 extends Dog {
  public static void bark() { }
}

public class Bark {
  public static void main(String args[]) {
    Dog woofer = new Dog();
    Dog nipper = new Basenji();
    woofer.bark();
    nipper.bark();
  }
}
{% endcodeblock %}

好像该程序应该只打印一个woof，毕竟Dog1继承自Dog,并且它的bark方法定义了什么也不做。main方法调用了bark方法,第一次是在Dog类型的woofer上调用,第二次是在Dog1类型的nipper上调用。但是如果你运行该程序,就会发现它打印的是 “woof woof“。

问题在于bark是一个静态方法,而对静态方法的调用不存在任何动态的分派机制。当一个程序调用了一个静态方法时,要被调用的方法都是在编译时刻被选定的,而这种选定是基于修饰符的编译期类型而做出的,修饰符的编译期类型就是我们给出的方法调用表达式中圆点左边部分的名字。在本例中,两个方法调用的修饰符分别是变量woofer 和nipper,它们都被声明为Dog类型。因为它们具有相同的编译期类型,所以编译器使得它们调用的是相同的方法:Dog.bark。

**31.比年龄小**  
如果运行下面的程序会打印出什么呢?

{% codeblock lang:java %}
public class Age {
  public static final Age INSTANCE = new Age();
  private final int age;
  private static final int CURRENT_YEAR=Calendar.getInstance().get(Calendar.YEAR);

  private Age() {
    age = CURRENT_YEAR - 1985;
  }

  public int getAge() {
    return age;
  }

  public static void main(String[] args) {
    System.out.println("My age is " + INSTANCE.getAge() + " years old.");
  }
}
{% endcodeblock %}

这个程序是在计算当前的年份减去1985的值。如果它是正确的,那么在2011年,该程序将打印出"My age is 26 years old"。但是如果你尝试着去运行该程序,该程序将打印出"My age is -1985 years old"。

该程序所遇到的问题是由类初始化顺序中的循环而引起的。让我们来看看其细节。首先,其静态域被设置为缺省值,其中INSTANCE被设置为null,CURRENT_YEAR被设置为0。接下来,静态域初始器按照其出现的顺序执行。第一个静态域是INSTANCE,它的值是通过调用 Age()构造器而计算出来的。这个构造器会用一个涉及静态域CURRENT_YEAR的表达式来初始化age。通常,读取一个静态域是会引起一个类被初始化的事件之一,但是我们已经在初始化Age类了，所以递归的初始化尝试会直接被忽略掉。因此,CURRENT_YEAR 的值仍旧是其缺省值0。这就是为什么我的年龄变成了-1985的原因。最后,从构造器返回以完成Age类的初始化,假设我们是在2011年运行该程序,那么我们就将静态域 CURRENT_YEAR初始化成了2011。但是已经太晚了，age的值已经是-1985了。这正是后续所有对Age.INSTANCE.getAge()的调用将返回的值。

该程序表明,在final类型的静态域被初始化之前,存在着读取它的值的可能,而此时该静态域包含的还只是其所属类型的缺省值。这是与直觉相违背的,因为我们通常会将final类型的域看作是常量。final类型的域只有在其初始化表达式是常量表达式时才是常量。

要想修正这个程序,需要重新对静态域的初始化进行排序,使得每一个初始化都出现在任何依赖于它的其他的初始化之前。

**32.不是你的类型**  
下面的三个程序每一个都会打印出什么呢?

{% codeblock lang:java %}
public class Type1 {
  public static void main(String[] args) {
    String s = null;
    System.out.println(s instanceof String);
  }
}

public class Type2 {
  public static void main(String[] args) {
    System.out.println(new Type2() instanceof String);
  }
}

public class Type3 {
  public static void main(String args[]) {
    Type3 t3 = (Type3) new Object();
  }
}
{% endcodeblock %}

第一个程序,Type1展示了instanceof操作符应用于一个空对象引用时的行为。尽管null对于每一个引用类型来说都是其子类型,但是instanceof操作符被定义为在其左操作数为null时返回false。因此,Type1将打印false。这被证明是实践中非常有用的行为。如果instanceof告诉你一个对象引用是某个特定类型的实例,那么你就可以将其转型为该类型,并调用该类型的方法,而不用担心会抛出ClassCastException或NullPointerException。

第二个程序,Type2展示了instanceof操作符在测试一个类的实例,以查看它是否是某个不相关的类的实例时所表现出来的行为。你可能会期望该程序打印出false。毕竟,Type2的实例不是String的实例,因此该测试应该失败。遗憾是,instanceof 测试在编译时刻就失败了,我们只能得到下面这样的出错消息:

{% codeblock lang:bash %}
Type2.java:3: inconvertible types
found: Type2, required: java.lang.String
       System.out.println(new Type2() instanceof String);
{% endcodeblock %}

该程序编译失败是因为instanceof操作符有这样的要求:**如果两个操作数的类型都是类,其中一个必须是另一个的子类型**。Type2和String彼此都不是对方的子类型,所以instanceof测试将导致编译期错误。

第三个程序,Type3展示了当要被转型的表达式的静态类型是转型类型的超类时转型操作符的行为:如果在一个转型操作中的两种类型都是类,那么其中一个必须是另一个的子类型。尽管对我们来说,这个转型很显然会失败,但是类型系统还没有强大到能够洞悉表达式new Object()的运行期类型不可能是Type3的一个子类型。因此,该程序将在运行期抛出ClassCastException。

**33.特创论**  
下面的程序会打印出什么呢?
{% codeblock lang:java %}
public class Creator {
  public static void main(String[] args) {
    for (int i = 0; i < 100; i++)
      Creature creature = new Creature();
    System.out.println(Creature.numCreated());
  }
}

class Creature {
  private static long numCreated = 0;
    public Creature() {
      numCreated++;
    }

  public static long numCreated() {
    return numCreated;
  }
}
{% endcodeblock %}

该程序看起来似乎应该打印100,但是它没有打印任何东西,因为它根本就不能编译。如果你尝试着去编译它,你就会发现编译器的诊断信息基本没什么用处。下面就是javac打印的东西:

{% codeblock lang:bash %}
Creator.java:4: not a statement
Creature creature = new Creature();
^
Creator.java:4: ';' expected
Creature creature = new Creature();
^
{% endcodeblock %}

一个本地变量声明看起来像是一条语句,但是从技术上说,它不是;它应该是一个本地变量声明语句(local variable declaration statement)。Java语言规范不允许一个本地变量声明语句作为一条语句在for、while或do循环中重复执行。一个本地变量声明作为一条语句只能直接出现在一个语句块中。(一个语句块是由一对花括号以及包含在这对花括展中的语句和声明构成的)。

有两种方式可以修正这个问题。最显而易见的方式是将这个声明至于一个语句块中:

{% codeblock lang:java %}
for (int i = 0; i < 100; i++) {
  Creature creature = new Creature();
}
{% endcodeblock %}

然而,请注意,该程序没有使用本地变量creature。因此,将该声明用一个无任何修饰的构造器调用来替代将更具实际意义,这样可以强调对新创建对象的引用正在被丢弃:

{% codeblock lang:java %}
for (int i = 0; i < 100; i++)
  new Creature();
{% endcodeblock %}

