---
layout: post
title: "Java疑惑和陷阱(一)"
date: 2011-03-16 17:12
comments: true
categories: java
---
**1.找零时刻**

一定要用`BigDecimal(String)`构造器,而千万不要`BigDecimal(double)`。后一个构造器将用它的参数的“精确”值来创建一个实例:`new BigDecimal(.1)`将返回一个表示`0.100000000000000055511151231257827021181583404541015625`的`BigDecimal`。

在需要精确答案的地方,要避免使用`float`和`double`;对于货币计算,要使用`int`、`long`或`BigDecimal`。

**2.长整除**
{% codeblock LongDivision.java lang:java %}
public class LongDivision{
  public static void main(String args[]){
    final long MICROS_PER_DAY = 24 * 60 * 60 * 1000 * 1000;
    final long MILLIS_PER_DAY = 24 * 60 * 60 * 1000;
    System.out.println(MICROS_PER_DAY/MILLIS_PER_DAY);
  }
}
{% endcodeblock %}

这段程序打印出的值是5而不是1000。问题在于常数MICROS_PER_DAY的计算**确实**溢出了。这个计算完全是以int运算来执行的,并且只有在运算完成之后,其结果才被提升到long,而此时已经溢出了。

这个教训很简单:当你在操作很大的数字时,千万要提防溢出——它可是一个缄默杀手。即使用来保存结果的变量已显得足够大,也并不意味着要产生结果的计算具有正确的类型。当你拿不准时,就使用long运算来执行整个计算。 <!--more-->

**3.初级问题**
{% codeblock Elementary.java lang:java %}
public class Elementary{
  public static void main(String[] args){
    System.out.println(12345 + 5432l);
  }
}
{% endcodeblock %}

从表面上看,程序必定打印66666,但实际上它打印出的是17777。请注意左操作数开头的数字“1“和右操作数结尾的小写字母“l“之间的细微差异。恶心啊！

小写字母“l“和数字“1“在大多数打字机字体中都是几乎一样的。为避免你的程序的读者对二者产生混淆,千万不要使用小写的“l“来作为long型字面常量的结尾或是作为变量名。

**4.多重转型**
{% codeblock Multicast.java lang:java %}
public class Multicast{
  public static void main (String[] args){
    System.out.println((int)(char)(byte) -1);
  }
}
{% endcodeblock %}

Java使用了基于"2"的补码的二进制运算,因此int类型的数值-1的所有32位都是置位的。从int到byte的转型是很简单的,它执行了一个窄化原始类型转化,直接将除低8位之外的所有位全部砍掉,这样做留下的是一个8位都被置位了的byte,它仍旧表示-1。从byte到char的转型稍微麻烦一点,因为byte是一个有符号类型,而char是一个无符号类型。在将一个整数类型转换成另一个宽度更宽的整数类型时,通常是可以保持其数值的,但是却不可能将一个负的byte数值表示成一个char。因此,从byte到char的转换被认为不是一个拓宽原始类型的转换,而是一个拓宽并窄化原始类型的转换:byte被转换成了int,而这个int又被转换成了char。

所有这些听起来有点复杂,幸运的是,有一条很简单的规则能够描述从较窄的整型转换成较宽的整型时的符号扩展行为:如果最初的数值类型是有符号的,那么就执行符号扩展;如果它是char,那么不管它将要被转换成什么类型,都执行零扩展。因为byte是一个有符号的类型,所以在将byte数值-1转换成char时,会发生符号扩展。作为结果的char数值的16个位就都被置位了,因此它等于`2^16-1`,即65535。从char到int的转型也是一个拓宽原始类型转换,所以这条规则告诉我们,它将执行零扩展而不是符号扩展。作为结果的int数值也就成了65535,这正是程序打印出的结果。

如果你在将一个char数值c转型为一个宽度更宽的类型,并且你不希望有符号扩展,那么为清晰表达意图,可以考虑使用一个位掩码,即使它并不是必需的:
{% codeblock java lang:java %}
int i = c & 0xffff;
{% endcodeblock %}

或者,书写一句注释来描述转换的行为:
{% codeblock java lang:java %}
int i = c; //不会执行符号扩展
{% endcodeblock %}

如果你在将一个char数值c转型为一个宽度更宽的整型,并且你希望有符号扩展,那么就先将char转型为一个short,它与char具有同样的宽度,但是它是有符号的。在给出了这种细微的代码之后,你应该也为它书写一句注释:
{% codeblock java lang:java %}
int i = (short) c; //转型将引起符号扩展
{% endcodeblock %}

如果你在将一个byte数值b转型为一个char,并且你不希望有符号扩展,那么你必须使用一个位掩码来限制它。这是一种通用做法,所以不需要任何注释:
{% codeblock java lang:java %}
char c = (char) (b & 0xff);
{% endcodeblock %}

**5.Dos Equis**
{% codeblock DosEquis.java lang:java %}
public class DosEquis{
  public static void main(String[] args){
    char x = 'X';
    int i = 0;
    System.out.println(true ? x : 0);
    System.out.println(false ? i : x);
  }
}
{% endcodeblock %}

这个程序应该打印:XX，然而,如果你运行该程序,你就会发现它打印出来的是**X88**。为什么呢？请注意在这两个表达式中,每一个表达式的第二个和第三个操作数的类型都不相同:x是char类型的,而0和i都是int类型的。混合类型的计算会引起混乱,而这一点比在条件表达式中比在其它任何地方都表现得更明显。

确定条件表达式结果类型的规则过于冗长和复杂,很难完全记住它们,但是其核心就是以下三点:

* 如果第二个和第三个操作数具有相同的类型,那么它就是条件表达式的类型。换句话说,你可以通过绕过混合类型的计算来避免大麻烦。
* 如果一个操作数的类型是T,T表示byte、short或char,而另一个操作数是一个int类型的常量表达式,它的值是可以用类型T表示的,那么条件表达式的类型就是T。
* 否则,将对操作数类型运用二进制数字提升,而条件表达式的类型就是第二个和第三个操作数被提升之后的类型。

在程序的两个条件表达式中,一个操作数的类型是char,另一个的类型是int。在两个表达式中,int操作数都是0,它可以被表示成一个char。然而,只有第一个表达式中的int操作数是常量(0),而第二个表达式中的int操作数是变量(i)。因此,第2点被应用到了第一个表达式上,它返回的类型是char,而第3点被应用到了第二个表达式上,其返回的类型是对int和char运用了二进制数字提升之后的类型,即int。

条件表达式的类型将确定哪一个重载的print方法将被调用。对第一个表达式来说, `PrintStream.print(char)`将被调用,而对第二个表达式来说, `PrintStream.print(int)`将被调用。前一个重载方法将变量x的值作为Unicode字符(X)来打印,而后一个重载方法将其作为一个十进制整数(88)来打印。

**6.复合赋值**

我们给出一个对变量x和i的声明即可,它肯定是一个合法的语句:
{% codeblock java lang:java %}
x += i;
{% endcodeblock %}
但是,它并不是:
{% codeblock java lang:java %}
x = x + i;
{% endcodeblock %}
许多程序员都会认为该迷题中的第一个表达式`x += i`只是第二个表达式`x =x + i`的简写方式。但是这并不十分准确。这两个表达式都被称为赋值表达式。
第二条语句使用的是简单赋值操作符(=),而第一条语句使用的是复合赋值操作符。(复合赋值操作符包括 +=、-=、*=、/=、%=、< <=、>>=、>>>=、&=、^=
和|=)Java语言规范中讲到,复合赋值`E1 op= E2`等价于简单赋值`E1 =(T)((E1)op(E2))`,其中"T"是"E1"的类型,除非"E1"只被计算一次。

换句话说,复合赋值表达式自动地将它们所执行的计算的结果转型为其左侧变量的类型。如果结果的类型与该变量的类型相同,那么这个转型不会造成任何影响。然而,如果结果的类型比该变量的类型要宽,那么复合赋值操作符将悄悄地执行一个窄化原始类型转换。
{% codeblock java lang:java %}
short x = 0;
int i = 123456;
{% endcodeblock %}

复合赋值编译将不会产生任何错误:
{% codeblock java lang:java %}
x += i; //包含了一个隐藏的转型!
{% endcodeblock %}

你可能期望x的值在这条语句执行之后是123456,但是并非如此,它的值是-7616。因为int类型的数值123456对于short来说太大了，自动产生的转型悄悄地把int数值的高两位给截掉了。

相对应的简单赋值是非法的,因为它试图将int数值赋值给short变量,它需要一个显式的转型:
{% codeblock java lang:java %}
x = x + i; //不能编译，需要显示转换——“可能会丢失精度”
{% endcodeblock %}

复合赋值操作符要求两个操作数都是原始类型的,例如int,或包装了的原始类型,例如Integer,但是有一个例外:如果在+=操作符左侧的操作数是String类型的,那么它允许右侧的操作数是任意类型,在这种情况下,该操作符执行的是字符串连接操作。简单赋值操作符(=)允许其左侧的是对象引用类型,这就显得要宽松许多了:你可以使用它们来表示任何你想要表示的内容,只要表达式的右侧与左侧的变量是赋值兼容的即可。

总之,复合赋值操作符会悄悄地产生一个转型。如果计算结果的类型宽于变量的类型,那么所产生的转型就是一个危险的窄化转型。

**7.字符串奶酪**

下面的程序从一个字节序列创建了一个字符串,然后迭代遍历字符串中的字符,并将它们作为数字打印。请描述一下程序打印出来的数字序列:
{% codeblock StringCheese.java lang:java %}
public class StringCheese {
  public static void main(String[] args) {
    byte bytes[] = new byte[256];
    for (int i = 0; i < 256; i++)
    bytes[i] = (byte)i;
    String str = new String(bytes);
    for (int i = 0, n = str.length(); i < n; i++)
      System.out.println((int)str.charAt(i) + " ");
  }
}
{% endcodeblock %}
首先,byte数组用从0到255每一个可能的byte数值进行了初始化,然后这些byte数值通过String构造器被转换成了char数值。最后,char数值被转型为int数值并被打印。打印出来的数值肯定是非负整数,因为char数值是无符号的,因此,你可能期望该程序将按顺序打印出0到255的整数。

如果你运行该程序,可能会看到这样的序列。但是再运行一次,可能看到的就不是这个序列了。我们在四台机器上运行它,会看到四个不同的序列,包括前面描述的那个序列。这个程序甚至都不能保证会正常终止,比打印其他任何特定字符串都要缺乏这种保证，它的行为完全是不确定的。

这里的罪魁祸首就是`String(byte[])`构造器。有关它的规范描述道:“在通过解码使用平台缺省字符集的指定byte数组来构造一个新的String时,该新String的长度是字符集的一个函数,因此,它可能不等于byte数组的长度。当给定的所有字节在缺省字符集中并非全部有效时,这个构造器的行为是不确定的”。

J2SE运行期环境(JRE)的缺省字符集依赖于底层的操作系统和语言。如果你想知道你的JRE的缺省字符集,并且你使用的是5.0或更新的版本,那么你可以通过调用 `java.nio.charset.Charset.defaultCharset()`来了解。如果你使用的是较早的版本,那么你可以通过阅读系统属性`file.encoding`来了解。

当你在char序列和byte序列之间做转换时,你可以且通常是应该显式地指定字符集。还可以接受一个字符集名称的String构造器就是专为除了接受byte数字之外,此目的而设计的。如果你用下面的构造器去替换在最初的程序中的String构造器,那么不管缺省的字符集是什么,该程序都保证能够按照顺序打印从0到255的整数:
{% codeblock java lang:java %}
String str = new String(bytes, "ISO-8859-1");
{% endcodeblock %}

这个谜题的教训是:每当你要将一个byte序列转换成一个String时,你都在使用某一个字符集,不管你是否显式地指定了它。如果你想让你的程序的行为是可预知的,那么就请你在每次使用字符集时都明确地指定。

**8.漂亮的火花**

面的程序用一个方法对字符进行了分类。这个程序会打印出什么呢?
{% codeblock Classifier.java lang:java %}
public class Classifier {
  public static void main(String[] args) {
    System.out.println(
    classify('n') + classify('+') + classify('2'));
  }
  static String classify(char ch) {
    if ("0123456789".indexOf(ch) >= 0)
      return "NUMERAL ";
    if ("abcdefghijklmnopqrstuvwxyz".indexOf(ch) >= 0)
      return "LETTER ";
    /* (Operators not supported yet)
    if ("+-*/&|!=" >= 0)
      return "OPERATOR ";
    */
    return "UNKNOWN";
  }
}
{% endcodeblock %}

如果你猜想该程序将打印**LETTER UNKNOWN NUMERAL**,那么你就掉进陷阱里面了。这个程序连编译都通不过。让我们再看一看相关的部分,这一次我们用粗体字突出注释部分:
{% codeblock java lang:java %}
  if ("abcdefghijklmnopqrstuvwxyz".indexOf(ch) >= 0)
    return "LETTER ";
  /* (Operators not supported yet)
  if("+-*/&|!=" >= 0)
    return "OPERATOR ";
  */
  return "UNKNOWN";
{% endcodeblock %}

正如你之所见,注释在包含了字符*/的字符串内部就结束了,结果使得程序在语法上变成非法的了。我们将程序中的一部分注释出来的尝试之所以失败了,是因为字符串字面常量在注释中没有被特殊处理。

总之,块注释不能可靠地注释掉代码段,应该用单行的注释序列来代替。

**9.我的类是什么?**

下面这段程序会打印出什么呢：
{% codeblock Me.java lang:java %}
package com.javapuzzlers;
public class Me {
  public static void main(String[] args){
    System.out.println(
    Me.class.getName().
    replaceAll(".","/") + ".class");
  }
}
{% endcodeblock %}

该程序看起来会获得它的类名`com.javapuzzlers.Me`,然后用“/”替换掉所有出现的字符串“.”,并在末尾追加字符串".class"。你可能会认为该程序将打印`com/javapuzzlers/Me.class`,该程序正式从这个类文件中被加载的。如果你运行这个程序,就会发现它实际上打印的是**///////////////////.class**。

问题在于`String.replaceAll`接受了一个正则表达式作为它的第一个参数,而并非接受了一个字符序列字面常量。(正则表达式已经被添加到了Java平台的1.4版中。)正则表达式“.”可以匹配任何单个的字符,因此,类名中的每一个字符都被替换成了一个斜杠,进而产生了我们看到的输出。

事实证明,`String.replaceAll`的第二个参数不是一个普通的字符串,而是一个替代字符串,就像在`java.util.regex`规范中所定义的样。在替代字符串中出现的反斜杠会把紧随其后的字符进行转义,从而导致其被按字面含义而处理了。

把上面的程序修改一下就能得到正确的输出了：
{% codeblock Me.java lang:java %}
package com.javapuzzlers;
public class Me {
  public static void main(String[] args){
    System.out.println(
    Me.class.getName().replaceAll("\\.","/") + ".class");
  }
}
{% endcodeblock %}

**10.URL的愚弄**

本谜题利用了Java编程语言中一个很少被人了解的特性。请考虑下面的程序将会做些什么?
{% codeblock BrowserTest.java lang:java %}
public class BrowserTest {
  public static void main(String[] args) {
    System.out.print("iexplore:");
    http://www.google.com;
    System.out.println(":maximize");
  }
}
{% endcodeblock %}

这是一个有点诡异的问题。该程序将不会做任何特殊的事情,而是直接打印
{% codeblock lang:java %}
iexplore::maximize**。
{% endcodeblock %}

在本谜题中所引用的“Java编程语言中很少被人了解的特性”实际上就是你可以在任何语句前面放置标号。这个程序标注了一个表达式语句,它是合法的,但是却没什么用处。

这就是说,我们没有任何可能的理由去使用与程序没有任何关系的标号和注释。本谜题的教训是:令人误解的注释和无关的代码会引起混乱。要仔细地写注释,并让它们跟上时代;要切除那些已遭废弃的代码。还有就是如果某些东西看起来过于奇怪,以至于不像对的,那么它极有可能就是错的。

**11.尽情享受每一个字节**

下面的程序循环遍历byte数值,以查找某个特定值。这个程序会打印出什么呢?
{% codeblock BigDelight.java lang:java %}
public class BigDelight {
  public static void main(String[] args) {
    for (byte b = Byte.MIN_VALUE; b < Byte.MAX_VALUE; b++) {
      if (b == 0x90)
      System.out.print("Joy!");
    }
  }
}
{% endcodeblock %}

这个循环在除了Byte.MAX_VALUE之外所有的byte数值中进行迭代,以查找0x90。这个数值适合用byte表示,并且不等于Byte.MAX_VALUE,因此你可能会想这个循环在该迭代会找到它一次,并将打印出"Joy!"。但是,所见为虚。如果你运行该程序,就会发现它没有打印任何东西。怎么回事?

简单地说,0x90是一个int常量,它超出了byte数值的范围。这与直觉是相悖的,因为0x90是一个两位的十六进制字面常量,每一个十六进制位都占据4个比特的位置,所以整个数值也只占据8个比特,即1个字节。问题在于字节（byte）是有符号类型。常量 0x90是一个正的最高位被置位的8位int数值。合法的byte数值是从-128到+127,但是int常量0x90等于**+144**。

请考虑表达式((byte)0x90 == 0x90),尽管外表看起来是成立的,但是它却等于 false。为了比较byte数值(byte)0x90和int数值0x90,Java通过拓宽原始类型转换将byte提升为一个int,然后比较这两个int数值。因为byte是一个有符号类型,所以这个转换执行的是符号扩展,将负的byte数值提升为了在数字上相等的int数值。

在本例中,该转换将(byte)0x90提升为int数值-112,它不等于int数值0x90，即+144。当你阅读这个程序时,请记住Java使用的是基于“2“的补码的二进制算术运算。(byte)0x90被转换成int类型的-112的步骤如下：
{% codeblock class lang:java %}
1.  10010000  //“0x90“是十进制表示，这是二进制表示
2.  11111111111111111111111110010000  //byte类型符号扩展为int类型
3.  //求上面这个二进制补码的十进制表式（先对各位取反，将其转换为十进制数
    //然后加上负号，再减去1）
4.  -112  //“(byte)0x90”被转换成int类型的十进制表示    
{% endcodeblock %}

**12.无情的增量操作**

下面的程序对一个变量重复地进行增量操作,然后打印它的值。那么这个值是什么呢?
{% codeblock Increment.java lang:java %}
public class Increment {
  public static void main(String[] args) {
    int j = 0;
    for (int i = 0; i < 100; i++){
      j = j++;
    }
    System.out.println(j);
  }
}
{% endcodeblock %}
乍一看,这个程序可能会打印100。毕竟,它对j做了100次增量操作。可能会令你感到有些震惊,它打印的不是100而是0。所有的增量操作都无影无踪了,为什么?

问题出在了执行增量操作的语句上:
{% codeblock java lang:java %}
j = j++;
{% endcodeblock %}

当++操作符被置于一个变量值之后时,其作用就是一个后缀增量操作符:表达式`j++`的值等于j在执行增量操作**之前**的初始值。因此,前面提到的赋值语句首先保存j的值,然后将j设置为其值加1,最后将j复位到它的初始值。换句话说,这个赋值操作等价于下面的语句序列:

{% codeblock java lang:java %}
int temp = j;
j = j + 1;
j = temp;
{% endcodeblock %}
程序重复该过程100次,之后j的值还是等于它在循环开始之前的值,即0。

订正该程序非常简单,只需从循环中移除无关的赋值操作,只留下:
{% codeblock java lang:java %}
for (int i = 0; i < 100; i++){
  j++;
}
{% endcodeblock %}
教训:不要在单个的表达式中对相同的变量赋值超过一次。

**13.在循环中**

下面的程序计算了一个循环的迭代次数,并且在该循环终止时将这个计数值打印了出来。那么,它打印的是什么呢?
{% codeblock InTheLoop.java lang:java %}
public class InTheLoop {
  public static final int END = Integer.MAX_VALUE;
  public static final int START = END - 100;
  public static void main(String[] args) {
    int count = 0;
    for (int i = START; i < = END; i++){
      count++;
    }
    System.out.println(count);
  }
}
{% endcodeblock %}
你可能会认为它将打印101,但如果你运行该程序,就会发现它压根就什么都没有印。更糟的是,它会持续运行直到你撤销它为止。它从来都没有机会去打印 count,因为在打印它的语句之前插入的是一个无限循环。

问题在于这个循环会在循环索引(i)小于或等于`Integer.MAX_VALUE`时持续运行,但是所有的int变量都是小于或等于`Integer.MAX_VALUE`的。因为它被定义为所有int数值中的最大值。当i达到`Integer.MAX_VALUE`,并且再次被执行增量操作时,它就有绕回到了`Integer.MIN_VALUE`。

如果你需要的循环会迭代到int数值的边界附近时,你最好是使用一个long变量作为循环索引。只需将循环索引的类型从int改变为long就可以解决该问题,从而使程序打印出我们所期望的101:
{% codeblock java lang:java %}
for (long i = START; i < = END; i++)
{% endcodeblock %}

不使用long类型的循环索引变量也可以解决该问题,但是它看起来并不那么漂亮:
{% codeblock java lang:java %}
int i = START;
do {
  count++;
}while (i++ != END);
{% endcodeblock %}
如果清晰性和简洁性占据了极其重要的地位,那么在这种情况下使用一个long类型的循环索引几乎总是最佳方案。但是有一个例外:如果你在所有的(或者几乎所有的)int数值上迭代,那么使用int类型的循环索引的速度大约可以提高一倍。

**14.变幻莫测的i值**

当你阅读这个程序时,请记住Java使用的是基于2的补码的二进制算术运算,因此-1 在任何有符号的整数类型中(byte、short、int或long)的表示都是所有的位被置位:
{% codeblock Shifty.java lang:java %}
public class Shifty {
  public static void main(String[] args) {
    int i = 0;
    while (-1 < < i != 0)
      i++;
    System.out.println(i);
  }
}
{% endcodeblock %}

常量-1是所有32位都被置位的int数值`0xffffffff`。左移操作符将0移入到由移位所空出的右边的最低位, 移位表达式将i最右边的位设置为0,并保持其余的32-i位为1。很明显,这个循环将完成32次迭代,因为(-1< &lt;i)对任何小于32的i来说都不等于0。你可能期望终止条件测试在i等于32时返回false,从而使程序打印32,但是它打印的并不是32。实际上,它不会打印任何东西,而是进入了一个无限循环。

问题在于`-1<<32`等于-1而不是0,因为移位操作符之使用其右操作数的低5位作为移位长度。如果其左操作数是一个long类型数值,用其低6位来移位。这条规则作用于全部的三个移位操作符:<<、>>和>>>。移位长度总是介于0到31之间,如果左操作数是long类型的,则介于0到63之间。这个长度是对32取余的,如果左操作数是long类型的,则对64取余。如果试图对一个int数值移位32位,或者是对一个long数值移位64位,都只能返回这个数值**自身的值**。没有任何移位长度可以让一个 int数值丢弃其所有的32位,或者是让一个long数值丢弃其所有的64位。

幸运的是,有一个非常容易的方式能够订正该问题。我们不是让-1重复地移位不同的移位长度,而是将前一次移位操作的结果保存起来,并且让它在每一次迭代时都向左再移 1位。下面这个版本的程序就可以打印出我们所期望的32:
{% codeblock Shifty.java lang:java %}
public class Shifty {
  public static void main(String[] args) {
    int distance = 0;
    for (int val = -1; val != 0; val < <= 1)
      distance++;
    System.out.println(distance);
  }
}
{% endcodeblock %}

**15.循环者**

考虑下面的for循环,使得该循环无限循环下去:
{% codeblock java lang:java %}
for (int i = start; i < = start + 1; i++) {}
{% endcodeblock %}

看起来它好像应该只迭代两次,但是通过利用溢出行为,可以使它无限循环下去。下面的的声明就采用了这项技巧:
{% codeblock java lang:java %}
int start = Integer.MAX_VALUE - 1;
{% endcodeblock %}

现在该轮到你了。什么样的声明能够让下面的循环变成一个无限循环?
{% codeblock java lang:java %}
while (i == i + 1) {}
{% endcodeblock %}

仔细查看这个while循环,它真的好像应该立即终止。一个数字永远不会等于它自己加1,对吗?嗯,如果这个数字是无穷大的,又会怎样呢?Java强制要求使用IEEE754浮点数算术运算,它可以让你用一个double或float来表示无穷大。正如我们在学校里面学到的,无穷大加1还是无穷大。如果i在循环开始之前被初始化为无穷大,那么终止条件测试`i == i + 1`就会被计算为true,从而使循环永远都不会终止。

你可以用任何被计算为无穷大的浮点算术表达式来初始化i,例如:
{% codeblock java lang:java %}
double i = 1.0 / 0.0;
{% endcodeblock %}

不过,你最好是能够利用标准类库为你提供的常量:
{% codeblock java lang:java %}
double i = Double.POSITIVE_INFINITY;
{% endcodeblock %}

事实上,你不必将i初始化为无穷大以确保循环永远执行。任何足够大的浮点数都可以实现这一目的,例如:
{% codeblock java lang:java %}
double i = 1.0e40;
{% endcodeblock %}

另一个例子:请提供一个对i的声明,将下面的循环转变为一个无限循环:
{% codeblock java lang:java %}
while (i != i) {
}
{% endcodeblock %}
这个循环可能比前一个还要使人感到困惑。不管在它前面作何种声明,它看起来确实应该立即终止。一个数字总是等于它自己,对吗?

对,但是IEEE754浮点算术保留了一个特殊的值用来表示一个不是数字的数量。这个值就是NaN(“不是一个数字(Not a Number)”的缩写),对于所有没有良好的数字定义的浮点计算,例如 0.0/0.0,其值都是它。规范中描述道,NaN不等于任何浮点数值,包括它自身在内。因此,如果i在循环开始之前被初始化为NaN,那么终止条件测试(i != i)的计算结果就是true,循环就永远不会终止。很奇怪但却是事实。

你可以用任何计算结果为NaN的浮点算术表达式来初始化i,例如:
{% codeblock java lang:java %}
double i = 0.0 / 0.0;
{% endcodeblock %}

同样,为了表达清晰,你可以使用标准类库提供的常量:
{% codeblock java lang:java %}
double i = Double.NaN;
{% endcodeblock %}

NaN还有其他的惊人之处。任何浮点操作,只要它的一个或多个操作数为NaN,那么其结果为NaN。这条规则是非常合理的,但是它却具有奇怪的结果。例如,下面的程序将打印false:
{% codeblock java lang:java %}
class Test {
  public static void main(String[] args) {
    double i = 0.0 / 0.0;
    System.out.println(i - i == 0);
  }
}
{% endcodeblock %}
这条计算NaN的规则所基于的原理是:一旦一个计算产生了NaN,它就被损坏了,没有任何更进一步的计算可以修复这样的损坏。NaN值意图使受损的计算继续执行下去,直到方便处理这种情况的地方为止。

总之,float和double类型都有一个特殊的NaN值,用来表示不是数字的数量。对于涉及NaN值的计算,其规则很简单也很明智,但是这些规则的结果可能是违背直觉的。

