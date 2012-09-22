---
layout: post
title: "Java常见疑惑和陷阱(二)"
date: 2011-08-07 16:20
comments: true
categories: java
---
**16.循环者的鬼魂**  
请提供一个对i的声明,将下面的循环转变为一个无限循环:
 
{% codeblock lang:java %}
while (i != 0) {
  i >>>= 1;
}
{% endcodeblock %}

对于无符号右移操作，0被从左移入到由移位操作而空出来的位上,即使被移位的负数也是如此。这个循环比前面三个循环要稍微复杂一点,因为其循环体非空。在其循环题中,i的值由它右移一位之后的值所替代。为了使移位合法,i必须是一个整数类型(byte、char、short、int或long)。无符号右移操作符把0从左边移入,因此看起来这个循环执行迭代的次数与最大的整数类型所占据的位数相同,即64次。如果你在循环的前面放置如下的声明,那么这确实就是将要发生的事情:

{% codeblock lang:java %}
long i = -1; // -1L has all 64 bits set
{% endcodeblock %}

你怎样才能将它转变为一个无限循环呢?解决本谜题的关键在于无符号右移是一个复合赋值操作符。有关混合操作符的一个不幸的事实是,它们可能会自动地执行窄化原始类型转换,这种转换把一种数字类型转换成了另一种更缺乏表示能力的类型。窄化原始类型转换可能会丢失级数的信息,或者是数值的精度。 <!--more-->

让我们更具体一些,假设你在循环的前面放置了下面的声明:

{% codeblock lang:java %}
short i = -1;
{% endcodeblock %}

因为i的初始值((short)0xffff)是非0的,所以循环体会被执行。在执行移位操作时,第一步是将i提升为int类型。所有算数操作都会对short、byte和char类型的操作数执行这样的提升。这种提升是一个拓宽原始类型转换,因此没有任何信息会丢失。这种提升执行的是符号扩展,因此所产生的int数值是0xffffffff。然后,这个数值右移1位,但不使用符号扩展,因此产生了int数值0x7fffffff。最后,这个数值被存回到i 中。为了将int数值存入short类型的变量,Java执行的是可怕的窄化原始类型转换,它直接将高16位截掉。这样就只剩下(short)oxffff了,我们又回到了开始处。循环的第二次以及后续的迭代行为都是一样的,因此循环将永远不会终止。

如果你将i声明为一个short或byte变量,并且初始化为任何负数,那么这种行为也会发生。如果你声明i为一个char,那么你将无法得到无限循环,因为char是无符号的,所以发生在移位之前的拓宽原始类型转换不会执行符号扩展。

总之,不要在short、byte或char类型的变量之上使用复合赋值操作符。因为这样的表达式执行的是混合类型算术运算,它容易造成混乱。更糟的是,它们执行将隐式地执行会丢失信息的窄化转型,其结果是灾难性的。

**17.循环者的诅咒**  
请提供一个对i的声明,将下面的循环转变为一个无限循环:

{% codeblock lang:java %}
while (i < = j && j <= i && i != j) {
}
{% endcodeblock %}

嘿,不要再给我看起来不可能的循环了！如果满足前2个条件，i不是肯定等于j吗?这一属性对实数肯定有效。事实上,它是如此地重要,以至于它有这样的定义:实数上的小于等于关系是反对称的，且在5.0版之前是反对称的,但是这从5.0版之后就不再是了。

直到5.0版之前,Java的数字比较操作符要求它们的两个操作数都是原始数字类型的(byte、char、short、int、long、float 和 double)。但是在5.0版中,规范作出了修改,新规范描述道:每一个操作数的类型必须可以转换成原始数字类型。问题难就难在这里了，在5.0版中,自动包装和自动反包装被添加到了Java语言中。小于等于操作符在原始数字类型集上仍然是反对称的,但是现在它还被应用到了被包装的数字类型上。(被包装的数字类型有:Byte、Character、Short、Integer、Long、Float和Double)。

让我们更具体一些,下面的声明会使表达式的值为true,从而将这个循环变成了一个无限循环：

{% codeblock lang:java %}
Integer i = new Integer(0);
Integer j = new Integer(0);
{% endcodeblock %}

前两个子表达式在i和j上执行解包转换,并且在数字上比较所产生的int数值。i和j都表示0,所以这两个子表达式都被计算为true。第三个子表达式(i!=j)在对象引用i和j上执行标识比较,因为它们都初始化为一个新的Integer实例,因此,第三个子表达式同样也被计算为true,循环也就永远地环绕下去了。

**18.循环者遇到了狼人**  
请提供一个对i的声明,将下面的循环转变为一个无限循环。这个循环不需要使用任何 5.0版的特性:

{% codeblock lang:java %}
while (i != 0 && i == -i) {
}
{% endcodeblock %}

这仍然是一个循环。在上面的条件表达式中,一元减号操作符作用于i,这意味着它的类型必须是数字型的:一元减号操作符作用于一个非数字型操作数是非法的。因此,我们要寻找一个非0的数字型数值,它等于它自己的负值。NaN不能满足这个属性,因为它不等于任何数值,因此,i必须表示一个实际的数字。肯定没有任何数字满足这样的属性吗?

除了0之外,没有任何浮点数等于其符号位反转之后的值,因此i的类型必然是整数。有符号的整数类型使用的是2的补码算术运算:为了对一个数值取其负值,你要反转其每一位,然后加1,从而得到结果。2的补码算术运算的一个很大的优势是,0具有唯一的示形式。如果你要对int数值0取负值,你将得到0xffffffff+1,它仍然是0。

但是,这也有一个相应的不利之处,总共存在偶数个int数值（准确地说有232个）其中一个用来表示0,这样就剩下奇数个int数值来表示正整数和负整数,这意味着正的和负的int数值的数量必然不相等。这暗示着至少有一个int数值,其负值不能正确地表示成为一个int数值。

事实上,恰恰就有一个这样的int数值,它就是Integer.MIN_VALUE,即-231。它的十六进制表示是0x80000000，其符号位为1,其余都是0。如果取个值的负数0x7fffffff+1=0x80000000=Integer.MIN_VALUE。换句话说,Integer.MIN_VALUE是它自己的负值,Long.MIN_VALUE也是一样。对这两个值取负值将会产生溢出,但是Java在整数计算中忽略了溢出。

总之,Java使用2的补码的算术运算,它是非对称的。对于每一种有符号的整数类型 (int、long、byte 和 short),负的数值总是比正的数值多一个,这个对多出来的值总是这种类型所能表示的最小数值。Integer.MIN_VALUE取负值得到的还是它没有改变过的值,Long.MIN_VALUE也是如此。对Short.MIN_VALUE取负值并将所产生的int数值转型回short,返回的同样是最初的值(Short.MIN_VALUE)。对于Byte.MIN_VALUE来说,也会产生相似的结果。更一般地讲,千万要当心溢出:就像狼人一样,它是个杀手。

**19.被计数击倒了**  
下面的程序有一个单重的循环,它记录迭代的次数,并在循环终止时打印这个数。那么,这个程序会打印出什么呢?

{% codeblock lang:java %}
public class Count {
  public static void main(String[] args) {
    final int START = 2000000000;
    int count = 0;
    for (float f = START; f < START + 50; f++){
      count++;
    }
    System.out.println(count);
  }
}
{% endcodeblock %}

表面的分析也许会认为这个程序将打印50,然而,这种分析遗漏了关键的一点:循环变量是float类型的,而非int类型的。F的初始值接近于Integer.MAX_VALUE,因此它需要用31位来精确表示,而float类型只能提供24位的精度（整数部分24位，小数部分8位）。对如此巨大的一个float数值进行增量操作将不会改变其值。因此,这个程序看起来应该无限地循环下去,因为f永远也不可能解决其终止值。但是,如果你运行该程序,就会发现它并没有无限循环下去,事实上,它立即就终止了,并打印出0。怎么回事呢?

问题在于终止条件测试失败了,其方式与增量操作失败的方式非常相似。这个循环只有在循环索引f比(float)(START+50)小的情况下才运行。在将一个int与一个float进行比较时,会自动执行从int到float的转换。遗憾的是,这种转换是会导致精度丢失的三种类型转换的一种(另外两个是从long到float和从long到double)。

f的初始值太大了,以至于在对其加上50,然后将结果转型为float时,所产生的值等于直接将f转换成float的值，即(float)2000000000==(float)2000000050,因此在循环体第一次执行之前就是false,所以,循环体也就永远的不到机会去运行。注意到2000000000有10个因子都是2:它是一个2乘以9个10,而每个10都是5×2，这意味着2000000000的二进制表示是以10个0结尾的。50的二进制表示只需要6位,所以将50加到2000000000上不会对右边6位之外的其他为产生影响。特别是,从右边数过来的第7位和第8位仍旧是0。提升这个31位的int到具有24位精度的float会在第 7位和第8位之间四舍五入,从而直接丢弃最右边的7位，因此它们的float表示是相同的。

这个谜题的教训是：不要使用浮点数作为循环索引,因为它会导致无法预测的行为。如果你在循环体内需要一个浮点数,那么请使用int或long循环索引,并将其转换为float或double。在将一个int或long转换成一个float或double时,你可能会丢失精度,但是至少它不会影响到循环本身。当你使用浮点数时,要使用double而不是 float,除非你肯定float提供了足够的精度,并且存在强制性的性能需求迫使你使用 float。适合使用float而不是double的时刻是非常非常少的。

**20.一分钟又一分钟**  
下面这段程序将打印分钟计数器，那么它会打印出什么呢?

{% codeblock lang:java %}
public class Clock {
  public static void main(String[] args) {
    int minutes = 0;
    for (int ms = 0; ms < 60*60*1000; ms++){
      if (ms % 60*1000 == 0)
    }
    minutes++;
    System.out.println(minutes);
  }
}
{% endcodeblock %}

你可能期望程序打印出60,毕竟,这就是一小时所包含的分钟数。但是,它打印的却是 60000。为什么它会如此频繁地对minutes执行了增量操作呢?

问题就出在(ms % 60*1000 == 0)。你可能会认为这个表达式等价于(ms % 60000 == 0),但是其实它们并不等价。取余和乘法操作符具有相同的优先级,因此表达式ms % 60*1000等价于(ms % 60)*1000。

订正该程序的最简单的方式就是在布尔表达式中插入一对括号,以强制规定计算的正确顺序:

{% codeblock lang:java %}
if (ms % (60 * 1000) == 0){
  minutes++;
}
{% endcodeblock %}

然而,有一个更好的方法可以订正该程序。用被恰当命名的常量来替代所有的魔幻数字:

{% codeblock lang:java %}
public class Clock {
  private static final int MS_PER_HOUR = 60 * 60 * 1000;
  private static final int MS_PER_MINUTE = 60 * 1000;
  public static void main(String[] args) {
    int minutes = 0;
    for (int ms = 0; ms < MS_PER_HOUR; ms++){
      if (ms % MS_PER_MINUTE == 0){
        minutes++;
      }
    System.out.println(minutes);
  }
}
{% endcodeblock %}

**21.极端不可思议**  
下面的三个程序每一个都会打印些什么?不要假设它们都可以通过编译:

{% codeblock lang:java %}
import java.io.IOException;
public class Arcane1 {
  public static void main(String[] args) {
    try {
      System.out.println("Hello world");
    } catch(IOException e) {
      System.out.println("I've never seen println fail!");
    }
  }
}

public class Arcane2 {
  public static void main(String[] args) {
    try {
      // If you have nothing nice to say, say nothing
    } catch(Exception e) {
      System.out.println("This can't happen");
    }
  }
}

interface Type1 {
  void f() throws CloneNotSupportedException;
}

interface Type2 {
  void f() throws InterruptedException;
}

interface Type3 extends Type1, Type2 {
}

public class Arcane3 implements Type3 {
  public void f() {
    System.out.println("Hello world");
  }

  public static void main(String[] args) {
    Type3 t3 = new Arcane3();
    t3.f();
  }
}
{% endcodeblock %}

第一个程序Arcane1,展示了已检查异常的一个基本原则。它看起来应该是可以编译的:try子句执行 I/O,并且catch子句捕获IOException异常。但是这个程序不能编译。因为println方法没有声明会抛出任何被检查异常,而IOException却正是一个被检查异常。语言规范中描述道:如果一个catch子句要捕获一个类型为E的被检查异常,而其相对应的try子句不能抛出E的某种子类型的异常,那么这就是一个编译期错误。

基于同样的理由,第二个程序Arcane2看起来应该是不可以编译的，但是它却可以。它之所以可以编译,是因为它唯一的catch子句检查了Exception。尽管Java语言规范在这一点上十分含混不清,但是捕获Exception或Throwble的catch子句是合法的,不管与其相对应的try子句的内容为何。

第三个程序Arcane3,看起来它也不能编译，实际上却可以。为什么呢?上述分析的缺陷在于对“Type3.f可以抛出在Type1.f上声明的异常和在Type2.f 上声明的异常”所做的假设。但是这并不正确,因为每一个接口都限制了方法f可以抛出的被检查异常集合。一个方法可以抛出的被检查异常集合是它所适用的所有类型声明要抛出的被检查异常集合的交集,而不是合集。因此,静态类型为Type3的对象上的f方法根本就不能抛出任何被检查异常。因此,Arcane3可以毫无错误地通过编译。

**22.不受欢迎的宾客**  
将尝试着从其环境中读取一个用户ID,如果这种尝试失败了,则缺省地认为它是一个来宾用户。该程序的作者将面对有一个静态域的初始化表达式可能会抛出异常的情况。那么,下面的程序会打印出什么呢?

{% codeblock lang:java %}
public class UnwelcomeGuest {
  public static final long GUEST_USER_ID = -1;
  private static final long USER_ID;
  static {
    try {
      USER_ID = getUserIdFromEnvironment();
    } catch (IdUnavailableException e) {
      USER_ID = GUEST_USER_ID;
      System.out.println("Logging in as guest");
    }
  }

  private static long getUserIdFromEnvironment()
  throws IdUnavailableException {
    throw new IdUnavailableException();
  }

  public static void main(String[] args) {
    System.out.println("User ID: " + USER_ID);
  }
}

class IdUnavailableException extends Exception {
}
{% endcodeblock %}

该程序看起来很直观：getUserIdFromEnvironment方法将抛出一个异常,然后将GUEST_USER_ID(-1L)赋值给USER_ID,并打印“Loggin in as guest“,然后main方法执行,使程序打印“User ID: -1“。表象再次欺骗了我们,该程序并不能编译。如果你尝试着去编译它,你将看到和下面内容类似的一条错误信息:

{% codeblock lang:java %}
UnwelcomeGuest.java:10:
variable USER_ID might already have been assigned
USER_ID = GUEST_USER_ID;
{% endcodeblock %}

问题出在哪里了?USER_ID是一个空final(blank final),它是一个在声明中没有进行初始化操作的final值。很明显,只有在对USER_ID赋值失败时,才会在try语句块中抛出异常,因此,在catch语句块中赋值是相当安全的。不管怎样执行静态初始化操作语句块,只会对USER_ID赋值一次,这正是空final所要求的。为什么编译器不知道这些呢?

要确定一个程序是否可以不止一次地对一个空final进行赋值是一个很困难的问题。事实上,这是不可能的。这等价于经典的停机问题,它通常被认为是不可能解决的。为了能够编写出一个编译器,语言规范在这一点上采用了保守的方式。在程序中,一个空 final域只有在它是明确未赋过值的地方才可以被赋值。

解决这类问题的最好方式就是将这个烦人的域从空final类型改变为普通的final类型,用一个静态域的初始化操作替换掉静态的初始化语句块。实现这一点的最佳方式是重构静态语句块中的代码为一个助手方法:

{% codeblock lang:java %}
public class UnwelcomeGuest {
  public static final long GUEST_USER_ID = -1;
  private static final long USER_ID = getUserIdOrGuest;
  private static long getUserIdOrGuest {
    try {
      return getUserIdFromEnvironment();
    } catch (IdUnavailableException e) {
      System.out.println("Logging in as guest");
      return GUEST_USER_ID;
    }
  }
  ...// The rest of the program is unchanged
}
{% endcodeblock %}

**23.您好,再见!**  
下面的程序在寻常的Hello world程序中添加了一段不寻常的曲折操作。那么,它将会打印出什么呢?

{% codeblock lang:java %}
public class HelloGoodbye {
  public static void main(String[] args) {
    try {
      System.out.println("Hello world");
      System.exit(0);
    } finally {
      System.out.println("Goodbye world");
    }
  }
}
{% endcodeblock %}

执行程序你会发现它永远不会说再见:它只打印了Hello world。我们已经知道不论 try语句块的执行是正常地还是意外地结束,finally语句块确实都会执行。然而在这个程序中,try语句块根本就没有结束其执行过程。System.exit方法将停止当前线程和所有其他当场死亡的线程。finally子句的出现并不能给予线程继续去执行的特殊权限。

当System.exit方法被调用时,虚拟机在关闭前要执行两项清理工作。首先,它执行所有的关闭挂钩操作,这些挂钩已经注册到了Runtime.addShutdownHook上。这对于释放JVM以外的资源将很有帮助。务必要为那些必须在JVM退出之前发生的行为关闭挂钩。下面的程序版本示范了这种技术,它可以如我们所期望地打印出Hello world 和Goodbye world:

{% codeblock lang:java %}
public class HelloGoodbye1 {
  public static void main(String[] args) {
    System.out.println("Hello world");
    Runtime.getRuntime().addShutdownHook(new Thread() {
      public void run() {
        System.out.println("Goodbye world");
      }
    });
    System.exit(0);
  }
}
{% endcodeblock %}

JVM在System.exit方法被调用时执行的第二个清理任务与终结器有关。如果System.runFinalizerOnExit或Runtime.runFinalizersOnExit被调用了,那么JVM将在所有还未终结的对象上面调用终结器。这些方法很久以前就已经过时了,无论什么原因,永远不要调用System.runFinalizersOnExit和 Runtime.runFinalizersOnExit:它们属于Java类库中最危险的方法之一。调用这些方法导致的结果是,终结器会在那些其他线程正在并发操作的对象上面运行,从而导致不确定的行为或导致死锁。

总之,System.exit将立即停止所有的程序线程,它并不会使finally语句块得到调用,但是它在停止JVM之前会执行关闭挂钩操作。当JVM被关闭时,请使用关闭挂钩来终止外部资源。通过调用System.halt可以在不执行关闭挂钩的情况下停止JVM,但是这个方法很少使用。

**24.不情愿的构造器**  
尽管在一个方法声明中看到一个throws子句是很常见的,但是在构造器的声明中看到一个throws子句就很少见了。下面的程序就有这样的一个声明。那么,它将打印出什么呢?

{% codeblock lang:java %}
public class Reluctant {
  private Reluctant internalInstance = new Reluctant();
  public Reluctant() throws Exception {
    throw new Exception("I'm not coming out");
  }

  public static void main(String[] args) {
    try {
      Reluctant b = new Reluctant();
      System.out.println("Surprise!");
    } catch (Exception ex) {
      System.out.println("I told you so");
    }
  }
}
{% endcodeblock %}

你可能期望catch子句能够捕获这个异常,并且打印“I told you so“。但是当你尝试着去运行它时,它却抛出StackOverflowError异常,为什么呢?

与大多数抛出StackOverflowError异常的程序一样,本程序也包含了一个无限递归。当你调用一个构造器时,实例变量的初始化操作将先于构造器的程序体而运行。在本谜题中,internalInstance变量的初始化操作递归调用了构造器,而该构造器通过再次调用Reluctant构造器而初始化该变量自己的internalInstance,如此无限递归下去就会抛出StackOverflowError异常。因为StackOverflowError是 Error的子类型而不是Exception的子类型,所以catch子句无法捕获它。

总之,实例初始化操作是先于构造器的程序体而运行的。实例初始化操作抛出的任何异常都会传播给构造器。如果初始化操作抛出的是被检查异常,那么构造器必须声明也会抛出这些异常,但是应该避免这样做,因为它会造成混乱。

**25.域和流**  
下面的方法将一个文件拷贝到另一个文件,并且被设计为要关闭它所创建的每一个流,即使它碰到I/O错误也要如此。遗憾的是,它并非总是能够做到这一点。为什么不能呢,你如何才能订正它呢?

{% codeblock lang:java %}
static void copy(String src, String dest) throws IOException {
  InputStream in = null;
  OutputStream out = null;
  try {
    in = new FileInputStream(src);
    out = new FileOutputStream(dest);
    byte[] buf = new byte[1024];
    int n;
    while ((n = in.read(buf)) > 0)
    out.write(buf, 0, n);
  } finally {
    if (in != null) in.close();
    if (out != null) out.close();
  }
}
{% endcodeblock %}

问题出在finally语句块自身中，close方法也可能会抛出IOException。如果这正好发生在in.close被调用之时,那么这个异常就会阻止out.close被调用,从而使输出流仍保持在开放状态。

解决方式是将每一个close都包装在一个嵌套的try语句块中。下面的finally语句块的版本可以保证在两个流上都会调用close:

{% codeblock lang:java %}
finally {
  if (in != null) {
  try {
    in.close();
  } catch (IOException ex) {
    // There is nothing we can do if close fails
  }
  if (out != null)
  try {
    out.close();
  } catch (IOException ex) {
    // There is nothing we can do if close fails
  }
}

//从5.0版本开始,你可以利用Closeable接口对代码进行重构:
finally {
  closeIgnoringException(in);
  closeIgnoringEcception(out);
}

private static void closeIgnoringException(Closeable c) {
  if (c != null) {
    try {
    c.close();
  } catch (IOException ex) {
    // There is nothing we can do if close fails
}
{% endcodeblock %}

总之,当你在finally语句块中调用close方法时,要用一个嵌套的try-catch语句来保护它,以防止IOException的传播。更一般地讲,对于任何在finally语句块中可能会抛出的被检查异常都要进行处理,而不是任其传播。

**26.异常地危险**  
在JDK1.2中,Thread.stop、Thread.suspend以及其他许多线程相关的方法都因为它们不安全而不推荐使用了。下面的方法展示了你用Thread.stop可以实现的可怕事情之一:

{% codeblock lang:java %}
// Don’t do this - circumvents exception checking!
public static void sneakyThrow(Throwable t) {
  Thread.currentThread().stop(t); // Deprecated
}
{% endcodeblock %}

这个讨厌的小方法所做的事情正是throw语句要做的事情,但是它绕过了编译器的所有异常检查操作。你可以(卑鄙地)在你的代码的任意一点上抛出任何受检查的或不受检查的异常,而编译器对此连眉头都不会皱一下。

不使用任何不推荐的方法,你也可以编写出在功能上等价于sneakyThrow的方法。事实上,至少有两种方式可以这么实现这一点,其中一种只能在5.0或更新的版本中运行。你能够编写出这样的方法吗?它必须是用Java 而不是用JVM字节码编写的,你不能在其客户对它编译完之后再去修改它。你的方法不必是完美无瑕的:如果它不能抛出一两个Exception的子类,也是可以接受的。

本谜题的一种解决之道是利用Class.newInstance方法中的设计缺陷,该方法通过反射来对一个类进行实例化。引用有关该方法的文档中的话:“请注意,该方法将传播从空的(就是无参数的)构造器所抛出的任何异常,包括受检查的异常。使用这个方法可以有效地绕开在其他情况下都会执行的编译期异常检查。”一旦你了解了这一点,编写一个sneakyThrow的等价方法就不是太难了。

{% codeblock lang:java %}
public class Thrower {
  private static Throwable t;
  private Thrower() throws Throwable {
    throw t;
  }
  public static synchronized void sneakyThrow(Throwable t) {
    Thrower.t = t;
    try {
      Thrower.class.newInstance();
    } catch (InstantiationException e) {
      throw new IllegalArgumentException();
    } catch (IllegalAccessException e) {
      throw new IllegalArgumentException();
    } finally {
      Thrower.t = null; // Avoid memory leak
    }
  }
}
{% endcodeblock %}

在这个解决方案中将会发生许多微妙的事情。我们想要在构造器执行期间所抛出的异常不能作为一个参数传递给该构造器,因为Class.newInstance调用的是一个类的无参数构造器。因此,sneakyThrow方法将这个异常藏匿于一个静态变量中。为了使该方法是线程安全的,它必须被同步,这使得对其的并发调用将顺序地使用静态域t。要注意的是,t这个域在从finally语句块中出来时是被赋为空的:这只是因为该方法虽然是卑鄙的,但这并不意味着它还应该是内存泄漏的。如果t不被赋为空,那么它阻止该异常被垃圾回收。注意,如果你让该方法抛出一个InstantiationException或IllegalAccessException异常,它将以抛出IllegalArgumentException,这是这项技术的一个内在限制。

Class.newInstance的文档继续描述道：“Constructor.newInstance方法通过将构造器抛出的任何异常都包装在(已检查的)InvocationTargetException异常中而避免了这个问题”很明显,Class.newInstance应该是做了相同的处理。但是纠正这个缺陷已经为时过晚,因为这么做将引入源代码级别的不兼容性，这将使许多依赖于Class.newInstance的程序崩溃。而弃用这个方法也不切实际,因为它太常用了。当你在使用它时,一定要意识到Class.newInstance可以抛出它没有声明过的受检查异常。

被添加到5.0版本中的“范型”可以为本谜题提供一个完全不同的解决方案。为了实现最大的兼容性,通用类型是通过类型擦除来实现的:通用类型信息是在编译期而非运行期检查的。下面的解决方案就利用了这项技术:

{% codeblock lang:java %}
// Don't do this either - circumvents exception checking!
class TigerThrower<t extends Throwable> {
  public static void sneakyThrow(Throwable t) {
    new TigerThrower<error>().sneakyThrow2(t);
  }

  private void sneakyThrow2(Throwable t) throws T {
    throw (T) t;
  }
}
{% endcodeblock %}

这个程序在编译时将产生一条警告信息:

{% codeblock lang:bash %}
TigerThrower.java:7:warning: [unchecked] unchecked cast found
: java.lang.Throwable, required: T
throw (T) t;
^
{% endcodeblock %}

警告信息是编译器所采用的一种手段,用来告诉你:你可能正在搬起石头砸自己的脚,而且事实也正是如此。“不受检查的转型”警告告诉你这个有问题的转型将不会在运行时刻受到检查。当你获得了一个不受检查的转型警告时,你应该修改你的程序以消除它,或者你可以确信这个转型不会失败。如果你不这么做,那么某个其他的转型可能会在未来不确定的某个时刻失败,而你也就很难跟踪此错误到其源头了。对于本谜题所示的情况,其情况更糟糕:在运行期抛出的异常可能与方法的签名不一致。sneakyThrow2方法正是利用了这一点。

总之,Java 的异常检查机制并不是虚拟机强制执行的。它只是一个编译期工具,被设计用来帮助我们更加容易地编写正确的程序,但是在运行期可以绕过它。要想减少你因为这类问题而被曝光的次数,就不要忽视编译器给出的警告信息。

**27.切掉类**  
请考虑下面的两个类:

{% codeblock lang:java %}
public class Strange1 {
  public static void main(String[] args) {
    try {
      Missing m = new Missing();
    } catch (java.lang.NoClassDefFoundError ex) {
      System.out.println("Got it!");
    }
  }
}

public class Strange2 {
  public static void main(String[] args) {
    Missing m;
    try {
      m = new Missing();
    } catch (java.lang.NoClassDefFoundError ex) {
      System.out.println("Got it!");
    }
  }
}

class Missing {
  Missing() { }
}
{% endcodeblock %}

如果你编译后并且在运行Strange1和Strange2之前删Missing.class文件,你就会发现这两个程序的行为有所不同。其中一个抛出了NoClassDefFoundError,而另一个却打印出了“Got it!“到底哪一个程序具有哪一种行为,你又如何去解释这种行为上的差异呢?

Strange1只在其try语句块中提及Missing类型,因此你可能会认为NoClassDefFoundError被捕获并打印“Got it!“。Strange2在try语句块之外声明了一个Missing类型的变量,因此你可能会认为NoClassDefFoundError不会被捕获。如果你试着运行这些程序,就会看到它们的行为正好相反，怎样才能解释这些奇怪的行为呢?

如果你去查看Java规范以找出应该抛出NoClassDefFoundError的地方,那么你不会得到很多的信息。该规范描述道：这个错误可以“在(直接或间接)使用某个类的程序中的任何地方”抛出。当JVM调用Strange1和Strange2的main方法时,这些程序都间接使用了Missing类,因此,它们都在其权利范围内于这一点上抛出了该错误。于是,本谜题的答案就是这两个程序可以依据其实现而展示出各自不同的行为。但是这并不能解释为什么这些程序在所有我们所知的Java实现上的实际行为,与你所认为的必然行为都正好相反。要查明为什么会是这样,我们需要研究一下由编译器生成的这些程序的字节码。

如果你去比较Strange1和Strange2的字节码,就会发现几乎是一样的。除了类名之外,唯一的差异就是catch语句块所捕获的参数ex与JVM本地变量之间的映射关系不同。尽管哪一个程序变量被指派给了哪一个JVM变量的具体细节会因编译器的不同而有所差异,但是对于和上述程序一样简单的程序来说,这些细节不太可能会差异很大。下面是通过执行`javap -c Strange1`命令而显示的Strange1.main的字节码:

{% codeblock lang:bash %}
0   new info.liuxuan.test.Missing [16]
3   dup
4   invokespecial info.liuxuan.test.Missing() [18]
7   astore_1 [m]
8   goto 20
11  astore_1 [ex]
12  getstatic java.lang.System.out : java.io.PrintStream [19]
15  ldc <string "Got it!"> [25]
17  invokevirtual java.io.PrintStream.println(java.lang.String) : void [27]
20  return
      Exception Table:
        [pc: 0, pc: 8] -> 11 when : java.lang.NoClassDefFoundError
      Line numbers:
        [pc: 0, line: 14]
        [pc: 11, line: 15]
        [pc: 12, line: 16]
        [pc: 20, line: 18]
      Local variable table:
        [pc: 0, pc: 21] local: args index: 0 type: java.lang.String[]
        [pc: 8, pc: 11] local: m index: 1 type: info.liuxuan.test.Missing
        [pc: 12, pc: 20] local: ex index: 1 type: java.lang.NoClassDefFoundError
</string>
{% endcodeblock %}

Strange2.main相对应的字节码与其只有一条指令不同:

{% codeblock lang:java %}
11: astore_2
{% endcodeblock %}

这是一条将catch语句块中的捕获异常存储到捕获参数ex中的指令。在Strange1中,这个参数是存储在JVM变量1中的,而在Strange2中,它是存储在JVM变量2中的。这就是两个类之间唯一的差异,但是它所造成的程序行为上的差异是多么地大呀!

为了运行一个程序,JVM要加载和初始化包含main方法的类。在加载和初始化间,JVM 必须链接类。链接的第一阶段是校验,校验要确保一个类是良构的,并且遵循语言的语法要求。校验非常关键,它维护着可以将像Java这样的安全语言与像C或C++这样的不安全语言区分开的各种承诺。在Strange1和Strange2这两个类中,本地变量m碰巧都被存储在JVM变量1中。两个版本的main都有一个连接点,从两个不同位置而来的控制流汇聚于此。该连接点就是指令20,即从main返回的指令。在正常结束try语句块的情况下,我们执行到指令8,即goto20,从而可以到达指令20;对于在catch语句块中结束的情况,我们将执行指令17,并按顺序执行下去,到达指令20。连接点的存在使得在校验Strange1类时产生异常,而在校验Strange2类时并不会产生异常。当校验执行对Strange1.main的流分析时,由于指令20可以通过两条不同的路径到达,因此校验器必须合并在变量1中的类型。两种类型是通过计算它们的首个公共超类而合并的(两个类的首个公共超类是它们所共有的最详细而精确的超类)。在 Strange1.main方法中,当从指令8到达指令20时,JVM变量1的状态包含了一个 Missing类的实例。当从指令17到达时,它包含了一个NoClassDefFoundError 类的实例。为了计算首个公共超类,校验器必须加载Missing类以确定其超类。因为 Missing.class文件已经被删除了,所以校验器不能加载它,因而抛出了一个NoClassDefFoundError。请注意,这个异常是在校验期间、在类被初始化之前,并且在main方法开始执行之前很早就抛出的。这就解释了为什么没有打印出任何关于这个未被捕获异常的跟踪栈信息。要想编写一个能够探测出某个类是否丢失的程序,请使用反射来引用类而不要使用通常的语言结构。下面展示了用这种技巧重写的程序:

{% codeblock lang:java %}
public class Strange {
  public static void main(String[] args) throws Exception{
    try {
      Object m = Class.forName("Missing").newInstance();
    } catch (ClassNotFoundException ex) {
      System.err.println("Got it!");
    }
  }
}
{% endcodeblock %}

总之,不要对捕获NoClassDefFoundError形成依赖。语言规范非常仔细地描述了类初始化是在何时发生的,但是类被加载的时机却显得更加不可预测。更一般地讲,捕获Error及其子类型几乎是完全不恰当的。这些异常是为那些不能被恢复的错误而保留的。

**28.令人疲惫不堪的测验**  
本谜题将测试你对递归的了解程度。下面的程序将做些什么呢?

{% codeblock lang:java %}
public class Workout {
  public static void main(String[] args) {
    workHard();
    System.out.println("It's nap time.");
  }

  private static void workHard() {
    try {
      workHard();
    } finally {
      workHard();
    }
  }
}
{% endcodeblock %}

要不是有try-finally语句,该程序的行为将非常明显:workHard方法递归调用它自身,直到StackOverflowError被抛出而终止。但是,try-finally语句把事情搞得复杂了。当它试图抛出StackOverflowError 时,程序将会在finally语句块的workHard方法中终止。这样,它就递归调用了自己。这看起来确实就像是一个无限循环的秘方,但是这个程序真的会无限循环下去吗?如果你运行它,它似乎确实是这么做的,但是要想确认的唯一方式就是分析它的行为。

Java虚拟机对栈的深度限制到了某个预设的水平。当超过这个水平时,JVM就抛出 StackOverflowError。为了让我们能够更方便地考虑程序的行为,我们假设栈的深度为3,这比它实际的深度要小得多。现在让我们来跟踪其执行过程。main方法调用 workHard,而它又从其try语句块中递归地调用了自己,然后它再一次从其try语句块中调用了自己。在此时,栈的深度是3。当workHard方法试图从其try语句块中再次调用自己时,该调用立即就会以抛出StackOverflowError而失败。这个错误是在最内部的finally语句块中被捕获的,在此处栈的深度已经达到了3。在那里,workHard 方法试图递归地调用它自己,但是该调用却以抛出StackOverflowError而失败。这个错误将在上一级的finally语句块中被捕获,在此处站的深度是2。该finally中的调用与相对应的try语句块具有相同的行为:都会产生一个StackOverflowError。

所以，一个深度为0的调用(即main中的调用),两个深度为1的调用,四个深度为2的调用,和八个深度为3的调用,总共是15个调用。那八个深度为3的调用每一个都会立即产生StackOverflowError。至少在把栈的深度限制为3的JVM上,该程序不会是一个无限循环:它在15个调用和8个异常之后就会终止。但是对于真实的JVM又会怎样呢?它仍然不会是一个无限循环。其调用图与前面的图相似,只不过要大得多得多而已。那么,究竟大到什么程度呢?许多JVM都将栈的深度限制为1024,因此,调用的数量就是1+2+4+8...+21,024=21,025-1,而抛出的异常的数量是 21,024。假设我们的机器可以在每秒钟内执行1010个调用,并产生1010个异常,按照当前的标准,这个假设的数量已经相当高了。在这样的假设条件下,程序将在大约 1.7×10291年后终止。为了让你对这个时间有直观的概念,我告诉你,我们的太阳的生命周期大约是1010年,所以我们可以很确定,我们中没有任何人能够看到这个程序终止的时刻。尽管它不是一个无限循环,但是它也就算是一个无限循环吧。

实际上,这个调用是一棵完全二叉树,它的深度就是JVM的栈深度的上限。WorkOut程序的执行过程等于是在先序遍历这棵树。在先序遍历中,程序先访问一个节点,然后递归地访问它的左子树和右子树。对于树中的每一条边,都会产生一个调用,而对于树中的每一个节点,都会抛出一个异常。

