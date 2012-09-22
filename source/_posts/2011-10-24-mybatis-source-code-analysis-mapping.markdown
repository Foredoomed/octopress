---
layout: post
title: "myBatis源码分析之映射"
date: 2011-10-24 20:35
comments: true
categories: opensource
---
**1.包结构分析**

<table border="1" width="100%" cellpadding="3" cellspacing="0" summary="">
<tbody><tr>
<th align="left" colspan="2" style="background-color:#CCCCFF">
<b>Packages</b></th></tr>
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
<td width="20%"><b>org.apache.ibatis.mapping</b></td>
<td>&nbsp;</td>
</tr>
<tr bgcolor="white">
<td width="20%"><b>org.apache.ibatis.parsing</b></td>
<td>&nbsp;</td>
</tr>
</tbody>
</table>

myBatis在XML解析方面的包结构相对于iBatis精简了不少，而且还加入了对注解的支持。在myatis中除了在XML文件内映射SQ外，我们还可以用注解来映射SQL。当然，这样做的好处和坏处都有，网上讨论也很多，在选择的时候要做好充分考虑。

**2.源码分析**

myBatis相对于iBatis变化还是很大的，iBatis的客户端接口SqlMapClient在myBatis中已经不复存在，取而代之的是SqlSession接口，接下来我们就来看看SqlSession接口是怎么创建的。

{% codeblock SqlSessionFactoryBuilder.java lang:java %}
public SqlSessionFactory build(Reader reader, String environment, Properties properties) {
    try {
      XMLConfigBuilder parser = new XMLConfigBuilder(reader, environment, properties);
      return build(parser.parse());
    } catch (Exception e) {
      throw ExceptionFactory.wrapException("Error building SqlSession.", e);
    } finally {
      ErrorContext.instance().reset();
      try {
        reader.close();
      } catch (IOException e) {
        // Intentionally ignore. Prefer previous error.
      }
    }
}

public SqlSessionFactory build(Configuration config) {
    return new DefaultSqlSessionFactory(config);
}
{% endcodeblock %}

在SqlSessionFactoryBuilder中定义了许多重载的build方法，其他build方法再调用上面的有三个参数的build方法，在其中创建XMLConfigBuilder对象解析XML文件，然后再调用以Configuration为参数的build方法返回DefaultSqlSessionFactory对象。

{% codeblock XMLConfigBuilder.java lang:java %}
  private void parseConfiguration(XNode root) {
    try {
      typeAliasesElement(root.evalNode("typeAliases"));
      pluginElement(root.evalNode("plugins"));
      objectFactoryElement(root.evalNode("objectFactory"));
      objectWrapperFactoryElement(root.evalNode("objectWrapperFactory"));
      propertiesElement(root.evalNode("properties"));
      settingsElement(root.evalNode("settings"));
      environmentsElement(root.evalNode("environments"));
      typeHandlerElement(root.evalNode("typeHandlers"));
      mapperElement(root.evalNode("mappers"));
    } catch (Exception e) {
      throw new BuilderException("Error parsing SQL Mapper Configuration. Cause: " + e, e);
    }
  }
  
  ...
{% endcodeblock %}

XMLConfigBuilder类也重载了许多的构造方法，但是最终调用的还是有三个参数的构造方法。在parse方法中调用parseConfiguration方法解析在configuration.xml中所有的配置项。myBatis定义了一个XNode类用来辅助解析：

{% codeblock XNode.java lang:java %}
  private Properties parseAttributes(Node n) {
    Properties attributes = new Properties();
    NamedNodeMap attributeNodes = n.getAttributes();
    if (attributeNodes != null) {
      for (int i = 0; i < attributeNodes.getLength(); i++) {
        Node attribute = attributeNodes.item(i);
        String value = PropertyParser.parse(attribute.getNodeValue(), variables);
        attributes.put(attribute.getNodeName(), value);
      }
    }
    return attributes;
  }

  private Object evaluate(String expression, Object root, QName returnType) {
    try {
      return xpath.evaluate(expression, root, returnType);
    } catch (Exception e) {
      throw new BuilderException("Error evaluating XPath.  Cause: " + e, e);
    }
  }
{% endcodeblock %}

在XPathParser类中读取XML文件流创建Document类，而且其中定义了许多evalXXX方法，而后最终调用的是XPath类的evaluate方法。而且在XNode中一个SQL的映射片段XML会被解析后存放到一个Properties类中。那么现在就能明白为什么与解析相关的包精简了很多：就是因为iBatis是用SAX方式解析，而myBatis直接调用JDK自带的XPath的API解析，所以省去了许多工作，不用去自定义元素类型，这样做的结果就是整个包结构得到了精简，而且解析性能更好。

下面来看一下SQL的映射过程：首先在XMLConfigBuilder中的mapperElement方法中会创建XMLMapperBuilder对象，然后再调用它的parse方法。

{% codeblock XMLMapperBuilder.java lang:java %}
  private void parsePendingStatements() {
	  Collection<xmlstatementbuilder> incompleteStatements = configuration.getIncompleteStatements();
	  synchronized (incompleteStatements) {
		  Iterator<xmlstatementbuilder> iter = incompleteStatements.iterator();
		  while (iter.hasNext()) {
			  try {
				  iter.next().parseStatementNode();
				  iter.remove();
			  } catch (IncompleteStatementException e) {
				  // Statement is still missing a resource...
			  }
		  }
	  }
  }
{% endcodeblock %}

我们可以看到在parsePendingStatements方法中调用的是XMLStatementBuilder的parseStatementNode方法，所以我们继续跟踪。

{% codeblock XMLStatementBuilder.java lang:java %}
  public void parseStatementNode() {
    ...

    SqlSource sqlSource = new DynamicSqlSource(configuration, rootSqlNode);
    
    ...
  }
{% endcodeblock %}

SQL的配置被读取进来后保存到了SqlSource接口的实例DynamicSqlSource类中，其他是对参数，返回值和主键的解析。

{% codeblock DynamicSqlSource.java lang:java %}
  public BoundSql getBoundSql(Object parameterObject) {
    DynamicContext context = new DynamicContext(configuration, parameterObject);
    rootSqlNode.apply(context);
    SqlSourceBuilder sqlSourceParser = new SqlSourceBuilder(configuration);
    Class<?> parameterType = parameterObject == null ? Object.class : parameterObject.getClass();
    SqlSource sqlSource = sqlSourceParser.parse(context.getSql(), parameterType);
    BoundSql boundSql = sqlSource.getBoundSql(parameterObject);
    for (Map.Entry<string , Object> entry : context.getBindings().entrySet()) {
      boundSql.setAdditionalParameter(entry.getKey(), entry.getValue());
    }
    return boundSql;
  }
{% endcodeblock %}

没什么好说的，创建SqlSourceBuilder类再调用它的parse方法。

{% codeblock SqlSourceBuilder.java lang:java %}
  public SqlSource parse(String originalSql, Class<?> parameterType) {
    ParameterMappingTokenHandler handler = new ParameterMappingTokenHandler(configuration, parameterType);
    GenericTokenParser parser = new GenericTokenParser("#{", "}", handler);
    String sql = parser.parse(originalSql);
    return new StaticSqlSource(configuration, sql, handler.getParameterMappings());
  }
{% endcodeblock %}

在parse方法里又创建了GenericTokenParser类，然后调用它的parse方法去解析SQL。

{% codeblock GenericTokenParser.java lang:java %}
  public String parse(String text) {
    StringBuilder builder = new StringBuilder();
    if (text != null) {
      String after = text;
      int start = after.indexOf(openToken);
      int end = after.indexOf(closeToken);
      while (start > -1) {
        if (end > start) {
          String before = after.substring(0, start);
          String content = after.substring(start + openToken.length(), end);
          String substitution = handler.handleToken(content);
          builder.append(before);
          builder.append(substitution);
          after = after.substring(end + closeToken.length());
        } else if (end > -1) {
          String before = after.substring(0, end);
          builder.append(before);
          builder.append(closeToken);
          after = after.substring(end + closeToken.length());
        } else {
          break;
        }
        start = after.indexOf(openToken);
        end = after.indexOf(closeToken);
      }
      builder.append(after);
    }
    return builder.toString();
  }
{% endcodeblock %}

parse方法的目的和iBatis是一样的，是要把形如

{% codeblock lang:sql %}
SELECT * FROM TABLE WHERE ID = #{ID}
{% endcodeblock %}

转换成

{% codeblock lang:sql %}
SELECT * FROM TABLE WHERE ID = ？
{% endcodeblock %}

到这里为止，整个解析过程就结束了，所有解析得到的配置都会保存到MappedStatement类中。

**总结**

这篇博文是iBatis和myBatis源码分析系列的最后一篇，也是最长的一篇。我觉得这只能算是粗略的源码分析，因为只分析了主要的几个部分，其他非常细节的地方并没有去做研究。四篇博文写下来给我的感觉是myBatis在代码结构和质量上相较iBatis都有比较大的提高，但是有些地方还是感觉有点复杂和罗嗦，代码读起来不是那么的清晰。有些类中的重载方法非常多，如果可以适当减少一些的话可以提高代码的可读性；还有可以对参数绑定做一个归纳，减少BaseTypeHandler类的实现类等。

虽然iBatis或者myBatis很轻量也很好用，但是就像我在第一篇概览中说的那样，现在Java平台上ORM框架大多数都是基于POJO和XML之上的，这样做的后果就是对象贫血，开放速度变慢。POJO什么时候会被抛弃我不知道，但是我相信会有那一天的。

