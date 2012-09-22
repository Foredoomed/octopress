---
layout: post
title: "iBatis和myBatis源码分析之概览"
date: 2011-08-24 18:08
comments: true
categories: opensource
---
**一.写在最前**

记得第一次知道[iBatis](http://ibatis.apache.org/ "iBatis")这个框架还是在2、3年前，那段时间Hibernate风头正劲，Java EE开发已经被概括为SSH，各个软件公司把Hibernate作为Java程序员一种标配的技能而来考核。正是在这样一种情况下，iBatis作为与Hibernate不同的ORM思想的实现，仍然被一部分开发者喜爱和支持，当然我也是其中之一。而且在一个偶然的机会让我知道原来淘宝的ORM工具也是用的iBatis后，就更加坚定了我对iBatis的信心(可能是因为Rails的关系，现在我觉得[Spring Roo](http://www.springsource.org/roo "Spring Roo")会更有前途)。 <!--more-->

其实当初我开始学习Hibernate的时候就感觉非常的复杂，基本上大部分时间都是花在学习怎么编写hbm.xml映射文件上的。Hibernate文档中的配置说明又非常得多，对于数据库表的各种关系的映射配置都在文档中有着比较详细的描述，整个文档有好几百页长。所以，当时我就觉得这也有点太复杂了，如果一个表有很多字段，并且这个表与其他多个表之间有着复杂的关系，那么编写hbm.xml文件将会是一件苦不堪言的工作。更有甚者，编写完了映射关系还不算完，还要配置字段的属性，而字段属性数量繁多，足以让你在其中摸不着头脑，非常蛋疼。

我并不是说Hibernate不好，因为好与不好是相对的。个人觉得Hibernate在Java企业级开发中还是非常有市场的，毕竟Hibernate已经非常成熟，会使用Hibernate的开发人员也多。但是，在商业网站开发领域，随着对性能的要求不断增加或者说苛刻的地步，数据库端的查询已经基本上放弃了多表之间的join操作以求达到比较理想的性能要求。在这样一个趋势下，Hibernate的作用就不大了。而且hql不利于优化的缺点对于数据库优化来说将成为性能瓶颈所在。对于商业网站而言，Java不能满足快速迭代开发的要求，越来越多的Web2.0网站采用PHP、Ruby或者Python作为开发语言，这也是我为什么觉得Spring Roo更有前途的原因。

不管怎样，Hibernate和iBatis是Java EE平台上非常优秀的ORM框架。但是对我而言，iBatis更轻量，对SQL的映射更符合我的理念，所以就趁现在有时间就来分析一下iBatis的源码。而就在去年，iBatis已经改名为<a href="http://www.mybatis.org" title="myBatis">myBatis</a>，我决定在分析源码的同时对这两个版本做一个比较。但是并不只是分析源码，其中还会加入对JDK源码的分析以及依赖的第三方框架的分析，我希望这次的分析是一个非常细致的分析系列。

**二.概览**

这篇博文是iBatis/myBatis源码分析系列的第一篇，我想首先来对iBatis和myBatis的整个结构做一个概览。首先来看一下iBatis的包结构(版本为2.3.4.726)：

<table border="1" width="100%" cellpadding="3" cellspacing="0" summary=""><tbody><tr>
<th align="left" colspan="2" style="background-color:#CCCCFF">
<b>Packages</b></th></tr>
<tr bgcolor="white">
<td width="20%"><b>com.ibatis.common.beans</b></td>
<td>&nbsp;</td>
</tr>
<tr bgcolor="white">
<td width="20%"><b>com.ibatis.common.io</b></td>
<td>&nbsp;</td>
</tr>
<tr bgcolor="white">
<td width="20%"><b>com.ibatis.common.jdbc</b></td>
<td>&nbsp;</td>
</tr>
<tr bgcolor="white">
<td width="20%"><b>com.ibatis.common.jdbc.exception</b></td>
<td>&nbsp;</td>
</tr>
<tr bgcolor="white">
<td width="20%"><b>com.ibatis.common.jdbc.logging</b></td>
<td>&nbsp;</td>
</tr>
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
<tr bgcolor="white">
<td width="20%"><b>com.ibatis.common.resources</b></td>
<td>&nbsp;</td>
</tr>
<tr bgcolor="white">
<td width="20%"><b>com.ibatis.common.util</b></td>
<td>&nbsp;</td>
</tr>
<tr bgcolor="white">
<td width="20%"><b>com.ibatis.common.xml</b></td>
<td>&nbsp;</td>
</tr>
<tr bgcolor="white">
<td width="20%"><b>com.ibatis.sqlmap.client</b></td>
<td>This package contains the core library client interface.</td>
</tr>
<tr bgcolor="white">
<td width="20%"><b>com.ibatis.sqlmap.client.event</b></td>
<td>This package contains event handler interfaces.</td>
</tr>
<tr bgcolor="white">
<td width="20%"><b>com.ibatis.sqlmap.client.extensions</b></td>
<td>&nbsp;</td>
</tr>
<tr bgcolor="white">
<td width="20%"><b>com.ibatis.sqlmap.engine.accessplan</b></td>
<td>&nbsp;</td>
</tr>
<tr bgcolor="white">
<td width="20%"><b>com.ibatis.sqlmap.engine.builder.xml</b></td>
<td>&nbsp;</td>
</tr>
<tr bgcolor="white">
<td width="20%"><b>com.ibatis.sqlmap.engine.cache</b></td>
<td>&nbsp;</td>
</tr>
<tr bgcolor="white">
<td width="20%"><b>com.ibatis.sqlmap.engine.cache.fifo</b></td>
<td>&nbsp;</td>
</tr>
<tr bgcolor="white">
<td width="20%"><b>com.ibatis.sqlmap.engine.cache.lru</b></td>
<td>&nbsp;</td>
</tr>
<tr bgcolor="white">
<td width="20%"><b>com.ibatis.sqlmap.engine.cache.memory</b></td>
<td>&nbsp;</td>
</tr>
<tr bgcolor="white">
<td width="20%"><b>com.ibatis.sqlmap.engine.cache.oscache</b></td>
<td>&nbsp;</td>
</tr>
<tr bgcolor="white">
<td width="20%"><b>com.ibatis.sqlmap.engine.config</b></td>
<td>&nbsp;</td>
</tr>
<tr bgcolor="white">
<td width="20%"><b>com.ibatis.sqlmap.engine.datasource</b></td>
<td>&nbsp;</td>
</tr>
<tr bgcolor="white">
<td width="20%"><b>com.ibatis.sqlmap.engine.exchange</b></td>
<td>&nbsp;</td>
</tr>
<tr bgcolor="white">
<td width="20%"><b>com.ibatis.sqlmap.engine.execution</b></td>
<td>&nbsp;</td>
</tr>
<tr bgcolor="white">
<td width="20%"><b>com.ibatis.sqlmap.engine.impl</b></td>
<td>&nbsp;</td>
</tr>
<tr bgcolor="white">
<td width="20%"><b>com.ibatis.sqlmap.engine.mapping.parameter</b></td>
<td>&nbsp;</td>
</tr>
<tr bgcolor="white">
<td width="20%"><b>com.ibatis.sqlmap.engine.mapping.result</b></td>
<td>&nbsp;</td>
</tr>
<tr bgcolor="white">
<td width="20%"><b>com.ibatis.sqlmap.engine.mapping.result.loader</b></td>
<td>&nbsp;</td>
</tr>
<tr bgcolor="white">
<td width="20%"><b>com.ibatis.sqlmap.engine.mapping.sql</b></td>
<td>&nbsp;</td>
</tr>
<tr bgcolor="white">
<td width="20%"><b>com.ibatis.sqlmap.engine.mapping.sql.dynamic</b></td>
<td>&nbsp;</td>
</tr>
<tr bgcolor="white">
<td width="20%"><b>com.ibatis.sqlmap.engine.mapping.sql.dynamic.elements</b></td>
<td>&nbsp;</td>
</tr>
<tr bgcolor="white">
<td width="20%"><b>com.ibatis.sqlmap.engine.mapping.sql.raw</b></td>
<td>&nbsp;</td>
</tr>
<tr bgcolor="white">
<td width="20%"><b>com.ibatis.sqlmap.engine.mapping.sql.simple</b></td>
<td>&nbsp;</td>
</tr>
<tr bgcolor="white">
<td width="20%"><b>com.ibatis.sqlmap.engine.mapping.sql.stat</b></td>
<td>&nbsp;</td>
</tr>
<tr bgcolor="white">
<td width="20%"><b>com.ibatis.sqlmap.engine.mapping.statement</b></td>
<td>&nbsp;</td>
</tr>
<tr bgcolor="white">
<td width="20%"><b>com.ibatis.sqlmap.engine.scope</b></td>
<td>&nbsp;</td>
</tr>
<tr bgcolor="white">
<td width="20%"><b>com.ibatis.sqlmap.engine.transaction</b></td>
<td>&nbsp;</td>
</tr>
<tr bgcolor="white">
<td width="20%"><b>com.ibatis.sqlmap.engine.transaction.external</b></td>
<td>&nbsp;</td>
</tr>
<tr bgcolor="white">
<td width="20%"><b>com.ibatis.sqlmap.engine.transaction.jdbc</b></td>
<td>&nbsp;</td>
</tr>
<tr bgcolor="white">
<td width="20%"><b>com.ibatis.sqlmap.engine.transaction.jta</b></td>
<td>&nbsp;</td>
</tr>
<tr bgcolor="white">
<td width="20%"><b>com.ibatis.sqlmap.engine.transaction.user</b></td>
<td>&nbsp;</td>
</tr>
<tr bgcolor="white">
<td width="20%"><b>com.ibatis.sqlmap.engine.type</b></td>
<td>&nbsp;</td>
</tr>
</tbody></table>
我们可以看到，iBatis整个包结构非常的简单，一目了然。仔细观察后可以说就分为两大部分：**common**和**sqlmap**。common包中包括了除了iBatis核心包之外的所有工具类包；而sqlmap包则是整个iBatis的核心包的所在，它又是由两部分组成：**client**和**engine**。client包是操作CRUD的对象所在的包，engine包则是iBatis用来处理映射关系的核心包。

再来看myBatis的包结构(版本为3.05)：

<table border="1" width="100%" cellpadding="3" cellspacing="0" summary=""><tbody><tr>
<th align="left" colspan="2" style="background-color:#CCCCFF">
<b>Packages</b></th></tr>
<tr bgcolor="white">
<td width="20%"><b>org.apache.ibatis.annotations</b></td>
<td>&nbsp;</td>
</tr>
<tr bgcolor="white">
<td width="20%"><b>org.apache.ibatis.binding</b></td>
<td>&nbsp;</td>
</tr>
<tr bgcolor="white">
<td width="20%"><b>org.apache.ibatis.builder</b></td>
<td>&nbsp;</td>
</tr>
<tr bgcolor="white">
<td width="20%"><b>org.apache.ibatis.builder.annotation</b></td>
<td>&nbsp;</td>
</tr>
<tr bgcolor="white">
<td width="20%"><b>org.apache.ibatis.builder.xml</b></td>
<td>&nbsp;</td>
</tr>
<tr bgcolor="white">
<td width="20%"><b>org.apache.ibatis.builder.xml.dynamic</b></td>
<td>&nbsp;</td>
</tr>
<tr bgcolor="white">
<td width="20%"><b>org.apache.ibatis.cache</b></td>
<td>&nbsp;</td>
</tr>
<tr bgcolor="white">
<td width="20%"><b>org.apache.ibatis.cache.decorators</b></td>
<td>&nbsp;</td>
</tr>
<tr bgcolor="white">
<td width="20%"><b>org.apache.ibatis.cache.impl</b></td>
<td>&nbsp;</td>
</tr>
<tr bgcolor="white">
<td width="20%"><b>org.apache.ibatis.datasource</b></td>
<td>&nbsp;</td>
</tr>
<tr bgcolor="white">
<td width="20%"><b>org.apache.ibatis.datasource.jndi</b></td>
<td>&nbsp;</td>
</tr>
<tr bgcolor="white">
<td width="20%"><b>org.apache.ibatis.datasource.pooled</b></td>
<td>&nbsp;</td>
</tr>
<tr bgcolor="white">
<td width="20%"><b>org.apache.ibatis.datasource.unpooled</b></td>
<td>&nbsp;</td>
</tr>
<tr bgcolor="white">
<td width="20%"><b>org.apache.ibatis.exceptions</b></td>
<td>&nbsp;</td>
</tr>
<tr bgcolor="white">
<td width="20%"><b>org.apache.ibatis.executor</b></td>
<td>&nbsp;</td>
</tr>
<tr bgcolor="white">
<td width="20%"><b>org.apache.ibatis.executor.keygen</b></td>
<td>&nbsp;</td>
</tr>
<tr bgcolor="white">
<td width="20%"><b>org.apache.ibatis.executor.loader</b></td>
<td>&nbsp;</td>
</tr>
<tr bgcolor="white">
<td width="20%"><b>org.apache.ibatis.executor.parameter</b></td>
<td>&nbsp;</td>
</tr>
<tr bgcolor="white">
<td width="20%"><b>org.apache.ibatis.executor.result</b></td>
<td>&nbsp;</td>
</tr>
<tr bgcolor="white">
<td width="20%"><b>org.apache.ibatis.executor.resultset</b></td>
<td>&nbsp;</td>
</tr>
<tr bgcolor="white">
<td width="20%"><b>org.apache.ibatis.executor.statement</b></td>
<td>&nbsp;</td>
</tr>
<tr bgcolor="white">
<td width="20%"><b>org.apache.ibatis.io</b></td>
<td>&nbsp;</td>
</tr>
<tr bgcolor="white">
<td width="20%"><b>org.apache.ibatis.jdbc</b></td>
<td>&nbsp;</td>
</tr>
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
<tr bgcolor="white">
<td width="20%"><b>org.apache.ibatis.mapping</b></td>
<td>&nbsp;</td>
</tr>
<tr bgcolor="white">
<td width="20%"><b>org.apache.ibatis.metadata</b></td>
<td>&nbsp;</td>
</tr>
<tr bgcolor="white">
<td width="20%"><b>org.apache.ibatis.migration</b></td>
<td>&nbsp;</td>
</tr>
<tr bgcolor="white">
<td width="20%"><b>org.apache.ibatis.migration.commands</b></td>
<td>&nbsp;</td>
</tr>
<tr bgcolor="white">
<td width="20%"><b>org.apache.ibatis.parsing</b></td>
<td>&nbsp;</td>
</tr>
<tr bgcolor="white">
<td width="20%"><b>org.apache.ibatis.plugin</b></td>
<td>&nbsp;</td>
</tr>
<tr bgcolor="white">
<td width="20%"><b>org.apache.ibatis.reflection</b></td>
<td>&nbsp;</td>
</tr>
<tr bgcolor="white">
<td width="20%"><b>org.apache.ibatis.reflection.factory</b></td>
<td>&nbsp;</td>
</tr>
<tr bgcolor="white">
<td width="20%"><b>org.apache.ibatis.reflection.invoker</b></td>
<td>&nbsp;</td>
</tr>
<tr bgcolor="white">
<td width="20%"><b>org.apache.ibatis.reflection.property</b></td>
<td>&nbsp;</td>
</tr>
<tr bgcolor="white">
<td width="20%"><b>org.apache.ibatis.reflection.wrapper</b></td>
<td>&nbsp;</td>
</tr>
<tr bgcolor="white">
<td width="20%"><b>org.apache.ibatis.session</b></td>
<td>&nbsp;</td>
</tr>
<tr bgcolor="white">
<td width="20%"><b>org.apache.ibatis.session.defaults</b></td>
<td>&nbsp;</td>
</tr>
<tr bgcolor="white">
<td width="20%"><b>org.apache.ibatis.transaction</b></td>
<td>&nbsp;</td>
</tr>
<tr bgcolor="white">
<td width="20%"><b>org.apache.ibatis.transaction.jdbc</b></td>
<td>&nbsp;</td>
</tr>
<tr bgcolor="white">
<td width="20%"><b>org.apache.ibatis.transaction.managed</b></td>
<td>&nbsp;</td>
</tr>
<tr bgcolor="white">
<td width="20%"><b>org.apache.ibatis.type</b></td>
<td>&nbsp;</td>
</tr>
</tbody></table>

我们可以看到在myBatis的包结构中，各个独立的功能被抽取出来单独成为一了个包，可以说是细化了整个框架的功能分类，较之iBatis整个包结构也更加清晰。除了包结构的变化，我们还可以看到在myBatis中加入了一些新的特性,比如增加了对annotation的支持，日志增加了对slf4j的支持等。

整个概述就到这里，在接下来的一系列博文里将详细地分析各个包已经类和方法的作用。

