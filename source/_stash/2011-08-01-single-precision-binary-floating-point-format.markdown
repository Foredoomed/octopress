---
layout: post
title: "单精度浮点类型的二进制表示格式"
date: 2011-08-01 16:12
comments: true
categories: programming
---
这两天在[Stack Overflow](http://stackoverflow.com "Stack Overflow")上闲逛的时候，发现有个关于Java中单精度浮点数（float）的问题:[what-is-the-maximum-number-in-the-mantissa-part-of-a-java-float](http://stackoverflow.com/questions/5849741/what-is-the-maximum-number-in-the-mantissa-part-of-a-java-float "what-is-the-maximum-number-in-the-mantissa-part-of-a-java-float")。我不得不承认，我被震精了。因为以前只知道float在Java中占4个字节（即32位），并不知道它内部用来表示1个小数的具体格式是什么。我记得我以前刚学Java的时候，最开始要学的就是基本数据类型（当然其他语言也一样），那时候我就对1个小数用二进制来表示产生过疑问，而且，书上对这个也没有详细的解释。难道这个底层的float表示格式不重要吗？我觉得这个很重要，人家不是都有问题出来了么？那么就乘这次把float的底层的二进制表示格式搞清楚吧。 
<!--more-->

在Java中，float采用的是[IEEE Standard for Floating-Point Arithmetic (IEEE 754)](http://en.wikipedia.org/wiki/IEEE_754-2008 "IEEE Standard for Floating-Point Arithmetic (IEEE 754)")标准,即float占4个字节，也就是32位，具体分配如下：
	
* 符号位：**1** 位
* 幂底数位：**8** 位
* 指数有效位：**23** 位


<img src="http://upload.wikimedia.org/wikipedia/commons/e/e8/IEEE_754_Single_Floating_Point_Format.svg" alt="single precision float point format" />

1.符号位是从左边数起的第一位数，0代表正数或零，1代表负数，这个没有什么疑问。

2.幂底数位是从右边数起的第23位到第30位。我们需要注意的是，这个数是用[偏移二进制（offset binary）](http://en.wikipedia.org/wiki/Offset_binary "offset binary")表示的，并不是用二进制的补码来表示，并且这个8位二进制数是没有符号位的。

这个偏移二进制与二进制补码的对应关系可以查看下面的表格：
<table style="text-align:center；margin: 1em 1em 1em 0;background: #F9F9F9;border: 1px #AAA solid;border-collapse: collapse;color: black;">
<tbody><tr>
<th>Offset Binary code, K=8</th>
<th>Decimal code</th>
<th>Twos' complement Binary</th>
</tr>
<tr>
<td>1111</td>
<td>7</td>
<td>0111</td>
</tr>
<tr>
<td>1110</td>
<td>6</td>
<td>0110</td>
</tr>
<tr>
<td>1101</td>
<td>5</td>
<td>0101</td>
</tr>
<tr>
<td>1100</td>
<td>4</td>
<td>0100</td>
</tr>
<tr>
<td>1011</td>
<td>3</td>
<td>0011</td>
</tr>
<tr>
<td>1010</td>
<td>2</td>
<td>0010</td>
</tr>
<tr>
<td>1001</td>
<td>1</td>
<td>0001</td>
</tr>
<tr>
<td>1000</td>
<td>0</td>
<td>0000</td>
</tr>
<tr>
<td>0111</td>
<td>−1</td>
<td>1111</td>
</tr>
<tr>
<td>0110</td>
<td>−2</td>
<td>1110</td>
</tr>
<tr>
<td>0101</td>
<td>−3</td>
<td>1101</td>
</tr>
<tr>
<td>0100</td>
<td>−4</td>
<td>1100</td>
</tr>
<tr>
<td>0011</td>
<td>−5</td>
<td>1011</td>
</tr>
<tr>
<td>0010</td>
<td>−6</td>
<td>1010</td>
</tr>
<tr>
<td>0001</td>
<td>−7</td>
<td>1001</td>
</tr>
<tr>
<td>0000</td>
<td>−8</td>
<td>1000</td>
</tr>
</tbody></table>

我们可以从上面的表格看到，偏移和补码的区别只是最高位的区别（右边数起）。当补码最高为是1的（也就是这个数是负数）时候，偏移表示的最高位是0，反之亦然。好了，既然指数的有效位使用偏移来表示的话，那么它的值就不是实际真正的值，而是要用它来与偏移量相减，从而得到真正的指数有效位值。这个偏移量为2<sup>n−1</sup>−1 ，其中n为所有位数。例如float里是用8位来表示，那么偏移量就是127。

细心的朋友可能会发现应该减去2<sup>n-1</sup>才是真正的值，但是为什么不减去2<sup>n</sup>，而是减去2<sup>n−1</sup>−1呢？[维基](http://en.wikipedia.org/wiki/Offset_binary" "offset binary")上是这么说的：

> Unusually however, instead of using "excess 2^(n-1)" it uses "excess 2^(n-1)-1" which means that inverting the leading (high-order) bit of the exponent will not convert the exponent to correct twos' complement notation.

也就是说这是有意这么做的，为了不把指数转换成正确的二进制补码形式。我搞不懂为什么要这样做，所以就在StackOverflow上提了个[问题](http://stackoverflow.com/questions/6871501/offset-binary-format-for-float-in-java "offset-binary-format-for-float-in-java")，大家可以参考一下。

这样一来，8位无符号指数的范围是0-255，再减去偏移量127，结果则为-127-128。另外全0和全1作为特殊处理,所以表示范围就变成了-126到127。特殊处理说明参考下表：
<table style="text-align:center；margin: 1em 1em 1em 0;background: #F9F9F9;border: 1px #AAA solid;border-collapse: collapse;color: black;">
<tbody><tr>
<th>Exponent</th>
<th>Significand zero</th>
<th>Significand non-zero</th>
<th>Equation</th>
</tr>
<tr>
<td>00<sub>H</sub></td>
<td><a href="http://en.wikipedia.org/wiki/0_(number)" title="0 (number)">zero</a>, <a href="http://en.wikipedia.org/wiki/%E2%88%920" title="−0" class="mw-redirect">−0</a></td>
<td><a href="http://en.wikipedia.org/wiki/Subnormal_numbers" title="Subnormal numbers" class="mw-redirect">subnormal numbers</a></td>
<td>(−1)<sup>signbits</sup>×2<sup>−126</sup>× 0.significandbits</td>
</tr>
<tr>
<td>01<sub>H</sub>, ..., FE<sub>H</sub></td>
<td colspan="2">normalized value</td>
<td>(−1)<sup>signbits</sup>×2<sup>exponentbits−127</sup>× 1.significandbits</td>
</tr>
<tr>
<td>FF<sub>H</sub></td>
<td>±<a href="http://en.wikipedia.org/wiki/Infinity" title="Infinity">infinity</a></td>
<td><a href="http://en.wikipedia.org/wiki/NaN" title="NaN">NaN</a> (quiet, signalling)</td>
</tr>
</tbody></table>

3.剩下的23位就是所谓的小数位了。但是注意：如果这23位全部是0的话，最高位将多出1位来，且这位上的值是1。

好了，这样所有32位都有了，自然可以得到二进制转换成十进制的推导公式为：

<img src="http://upload.wikimedia.org/math/1/d/e/1de9cd706cf2343bf03714ed2d4e4e65.png" alt="formula" />

对应到上图的例子为：

<img src="http://upload.wikimedia.org/math/2/5/6/2567776ddd65d7deb6b0a8e63dd82e52.png" alt="example" />

最后，双精度浮点数（double）的64位分配和单精度数差不多，具体情况如下图所示：
<img src="http://upload.wikimedia.org/wikipedia/commons/thumb/a/a9/IEEE_754_Double_Floating_Point_Format.svg/618px-IEEE_754_Double_Floating_Point_Format.svg.png" alt="double precision floating-point format" />

参考文章

[1] [Single_precision_floating-point_format](http://en.wikipedia.org/wiki/Single_precision_floating-point_format "Single_precision_floating-point_format")  
[2] [IEEE_754-1985#Exponent_biasing](http://en.wikipedia.org/wiki/IEEE_754-1985#Exponent_biasing "IEEE_754-1985#Exponent_biasing")

