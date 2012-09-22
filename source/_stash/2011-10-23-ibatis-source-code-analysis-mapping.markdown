---
layout: post
title: "iBatis源码分析之映射"
date: 2011-10-23 19:14
comments: true
categories: opensource
---
**1.包结构分析**

<table border="1" width="100%" cellpadding="3" cellspacing="0" summary="">
<tbody><tr>
<th align="left" colspan="2" style="background-color:#CCCCFF">
<b>Packages</b></th></tr>
<tr bgcolor="white">
<td width="20%"><b>com.ibatis.sqlmap.engine.builder.xml</b></td>
<td>&nbsp;</td>
</tr>
<tr bgcolor="white">
<td width="20%"><b>com.ibatis.common.xml</b></td>
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
</tbody>
</table>

iBatis处理映射的包结构应该说还是比较清晰的，基本上看到包名就可以知道包下面的类是做什么用的。但是我们看到关于映射的包一共有10个，但是像dynamic，raw，simple，stat这些包下都只有一个类，显然这种分法是有问题的，我们应当尽可能地减少包，包越少结构就越清晰，但并不是说把类都放一个包下，要在这两者之间寻求平衡。 <!--more-->

**2.源码分析**

{% codeblock SqlMapClientBuilder.java lang:java %}
public static SqlMapClient buildSqlMapClient(Reader reader) {
    return new SqlMapConfigParser().parse(reader);
}
{% endcodeblock %}

我们要对数据库操作，首先要创建SqlMapClient接口，而SqlMapClientBuilder类就是用来创建SqlMapClient接口的。在buildSqlMapClient方法里创建了SqlMapConfigParser类，然后再调用该类的parse方法来解析XML配置文件。其实代码读到这里就可以看出SqlMapClient接口是线程安全的。

{% codeblock SqlMapConfigParser.java lang:java %}
public SqlMapConfigParser() {
    parser.setValidation(true);
    parser.setEntityResolver(new SqlMapClasspathEntityResolver());

    addSqlMapConfigNodelets();
    addGlobalPropNodelets();
    addSettingsNodelets();
    addTypeAliasNodelets();
    addTypeHandlerNodelets();
    addTransactionManagerNodelets();
    addSqlMapNodelets();
    addResultObjectFactoryNodelets();

}
{% endcodeblock %}

首先在创建SqlMapConfigParser类的时候就会为NodeletParser类预先定义好XML文件里各种DOM类型的解析方法，这个是通过实现Nodelet接口的process方法来实现的(Nodelet实际上就是回调接口)。我们可以看到iBatis对各种DOM类型自定义了一套表示方法，他们的定义在NodeletParser类里。在SqlMapConfigParser类的parse方法里再调用NodeletParser的parse方法解析XML文件。

{% codeblock NodeletParser.java lang:java %} 
  /**
   * Registers a nodelet for the specified XPath.  Current XPaths supported
   * are:
   * <ul>
   * <li> Text Path - /rootElement/childElement/text()
   * <li> Attribute Path  - /rootElement/childElement/@theAttribute
   * <li> Element Path - /rootElement/childElement/theElement
   * <li> All Elements Named - //theElement
   * </ul>
   */
  public void addNodelet(String xpath, Nodelet nodelet) {
    letMap.put(xpath, nodelet);
  }

  /**
   * A recursive method that walkes the DOM tree, registers XPaths and
   * calls Nodelets registered under those XPaths.
   */
  private void process(Node node, Path path) {
    if (node instanceof Element) {
      // Element
      String elementName = node.getNodeName();
      path.add(elementName);
      processNodelet(node, path.toString());
      processNodelet(node, new StringBuffer("//").append(elementName).toString());

      // Attribute
      NamedNodeMap attributes = node.getAttributes();
      int n = attributes.getLength();
      for (int i = 0; i < n; i++) {
        Node att = attributes.item(i);
        String attrName = att.getNodeName();
        path.add("@" + attrName);
        processNodelet(att, path.toString());
        processNodelet(node, new StringBuffer("//@").append(attrName).toString());
        path.remove();
      }

      // Children
      NodeList children = node.getChildNodes();
      for (int i = 0; i < children.getLength(); i++) {
        process(children.item(i), path);
      }
      path.add("end()");
      processNodelet(node, path.toString());
      path.remove();
      path.remove();
    } else if (node instanceof Text) {
      // Text
      path.add("text()");
      processNodelet(node, path.toString());
      processNodelet(node, "//text()");
      path.remove();
    }
  }

  ...
{% endcodeblock %}

解析XML采用的是SAX方式，为了区分XML中的元素类型自定义了一套XPath，然后根据XPath进行相应的处理。processNodelet方法里调用Nodelet接口的process方法，而这个process方法已经在创建SqlMapConfigParser类的时候实现了。

{% codeblock SqlStatementParser.java lang:java %}
public class SqlStatementParser {

  public void parseGeneralStatement(Node node, MappedStatement statement) {

    // get attributes
    Properties attributes = NodeletUtils.parseAttributes(node, state.getGlobalProps());
    String id = attributes.getProperty("id");
    String parameterMapName = state.applyNamespace(attributes.getProperty("parameterMap"));
    String parameterClassName = attributes.getProperty("parameterClass");
    String resultMapName = attributes.getProperty("resultMap");
    String resultClassName = attributes.getProperty("resultClass");
    String cacheModelName = state.applyNamespace(attributes.getProperty("cacheModel"));
    String xmlResultName = attributes.getProperty("xmlResultName");
    String resultSetType = attributes.getProperty("resultSetType");
    String fetchSize = attributes.getProperty("fetchSize");
    String allowRemapping = attributes.getProperty("remapResults");
    String timeout = attributes.getProperty("timeout");

    if (state.isUseStatementNamespaces()) {
      id = state.applyNamespace(id);
    }
    String[] additionalResultMapNames = null;
    if (resultMapName != null) {
      additionalResultMapNames = state.getAllButFirstToken(resultMapName);
      resultMapName = state.getFirstToken(resultMapName);
      resultMapName = state.applyNamespace(resultMapName);
      for (int i = 0; i < additionalResultMapNames.length; i++) {
        additionalResultMapNames[i] = state.applyNamespace(additionalResultMapNames[i]);
      }
    }

    String[] additionalResultClassNames = null;
    if (resultClassName != null) {
      additionalResultClassNames = state.getAllButFirstToken(resultClassName);
      resultClassName = state.getFirstToken(resultClassName);
    }
    Class[] additionalResultClasses = null;
    if (additionalResultClassNames != null) {
      additionalResultClasses = new Class[additionalResultClassNames.length];
      for (int i = 0; i < additionalResultClassNames.length; i++) {
        additionalResultClasses[i] = resolveClass(additionalResultClassNames[i]);
      }
    }

    state.getConfig().getErrorContext().setMoreInfo("Check the parameter class.");
    Class parameterClass = resolveClass(parameterClassName);

    state.getConfig().getErrorContext().setMoreInfo("Check the result class.");
    Class resultClass = resolveClass(resultClassName);

    Integer timeoutInt = timeout == null ? null : new Integer(timeout);
    Integer fetchSizeInt = fetchSize == null ? null : new Integer(fetchSize);
    boolean allowRemappingBool = "true".equals(allowRemapping);

    MappedStatementConfig statementConf = state.getConfig().newMappedStatementConfig(id, statement,
        new XMLSqlSource(state, node), parameterMapName, parameterClass, resultMapName, additionalResultMapNames,
        resultClass, additionalResultClasses, resultSetType, fetchSizeInt, allowRemappingBool, timeoutInt, cacheModelName,
        xmlResultName);

    findAndParseSelectKey(node, statementConf);
  }
}
{% endcodeblock %}

在SqlStatementParser类的parseGeneralStatement方法中创建MappedStatementConfig对象。

{% codeblock MappedStatementConfig.java lang:java %}
MappedStatementConfig(SqlMapConfiguration config, String id, MappedStatement statement, SqlSource processor,
                        String parameterMapName, Class parameterClass, String resultMapName,
                        String[] additionalResultMapNames, Class resultClass, Class[] additionalResultClasses,
                        String cacheModelName, String resultSetType, Integer fetchSize, boolean allowRemapping,
                        Integer timeout, Integer defaultStatementTimeout, String xmlResultName) {
    this.errorContext = config.getErrorContext();
    this.client = config.getClient();
    SqlMapExecutorDelegate delegate = client.getDelegate();
    this.typeHandlerFactory = config.getTypeHandlerFactory();
    errorContext.setActivity("parsing a mapped statement");
    errorContext.setObjectId(id + " statement");
    errorContext.setMoreInfo("Check the result map name.");
    if (resultMapName != null) {
      statement.setResultMap(client.getDelegate().getResultMap(resultMapName));
      if (additionalResultMapNames != null) {
        for (int i = 0; i < additionalResultMapNames.length; i++) {
          statement.addResultMap(client.getDelegate().getResultMap(additionalResultMapNames[i]));
        }
      }
    }
    errorContext.setMoreInfo("Check the parameter map name.");
    if (parameterMapName != null) {
      statement.setParameterMap(client.getDelegate().getParameterMap(parameterMapName));
    }
    statement.setId(id);
    statement.setResource(errorContext.getResource());
    if (resultSetType != null) {
      if ("FORWARD_ONLY".equals(resultSetType)) {
        statement.setResultSetType(new Integer(ResultSet.TYPE_FORWARD_ONLY));
      } else if ("SCROLL_INSENSITIVE".equals(resultSetType)) {
        statement.setResultSetType(new Integer(ResultSet.TYPE_SCROLL_INSENSITIVE));
      } else if ("SCROLL_SENSITIVE".equals(resultSetType)) {
        statement.setResultSetType(new Integer(ResultSet.TYPE_SCROLL_SENSITIVE));
      }
    }
    if (fetchSize != null) {
      statement.setFetchSize(fetchSize);
    }

    // set parameter class either from attribute or from map (make sure to match)
    ParameterMap parameterMap = statement.getParameterMap();
    if (parameterMap == null) {
      statement.setParameterClass(parameterClass);
    } else {
      statement.setParameterClass(parameterMap.getParameterClass());
    }

    // process SQL statement, including inline parameter maps
    errorContext.setMoreInfo("Check the SQL statement.");
    Sql sql = processor.getSql();
    setSqlForStatement(statement, sql);

    // set up either null result map or automatic result mapping
    ResultMap resultMap = (ResultMap) statement.getResultMap();
    if (resultMap == null && resultClass == null) {
      statement.setResultMap(null);
    } else if (resultMap == null) {
      resultMap = buildAutoResultMap(allowRemapping, statement, resultClass, xmlResultName);
      statement.setResultMap(resultMap);
      if (additionalResultClasses != null) {
        for (int i = 0; i < additionalResultClasses.length; i++) {
          statement.addResultMap(buildAutoResultMap(allowRemapping, statement, additionalResultClasses[i], xmlResultName));
        }
      }

    }
    
    ...
}
{% endcodeblock %}

在MappedStatementConfig类的构造方法里又调用了SqlSource接口的getSql方法，而SqlSource接口只有一个实现类XMLSqlSource，继续看XMLSqlSource类里的getSql方法是怎么实现的。

{% codeblock XMLSqlSource.java lang:java %}
public Sql getSql() {
    state.getConfig().getErrorContext().setActivity("processing an SQL statement");

    boolean isDynamic = false;
    StringBuffer sqlBuffer = new StringBuffer();
    DynamicSql dynamic = new DynamicSql(state.getConfig().getClient().getDelegate());
    isDynamic = parseDynamicTags(parentNode, dynamic, sqlBuffer, isDynamic, false);
    String sqlStatement = sqlBuffer.toString();
    if (isDynamic) {
      return dynamic;
    } else {
      return new RawSql(sqlStatement);
    }
}
{% endcodeblock %}

接下来我们就来看一下iBatis是怎么处理SQL映射的:

{% codeblock DynamicSql.java lang:java %}
private void processBodyChildren(StatementScope statementScope, SqlTagContext ctx, Object     parameterObject, Iterator localChildren, PrintWriter out) {
    while (localChildren.hasNext()) {
      SqlChild child = (SqlChild) localChildren.next();
      if (child instanceof SqlText) {
        SqlText sqlText = (SqlText) child;
        String sqlStatement = sqlText.getText();
        if (sqlText.isWhiteSpace()) {
          out.print(sqlStatement);
        } else if (!sqlText.isPostParseRequired()) {

          // BODY OUT
          out.print(sqlStatement);

          ParameterMapping[] mappings = sqlText.getParameterMappings();
          if (mappings != null) {
            for (int i = 0, n = mappings.length; i < n; i++) {
              ctx.addParameterMapping(mappings[i]);
            }
          }
        } else {

          IterateContext itCtx = ctx.peekIterateContext();

          if(null != itCtx && itCtx.isAllowNext()){
            itCtx.next();
            itCtx.setAllowNext(false);
            if(!itCtx.hasNext()) {
              itCtx.setFinal(true);
            }
          }

          if(itCtx!=null) {
            StringBuffer sqlStatementBuffer = new StringBuffer(sqlStatement);
            iteratePropertyReplace(sqlStatementBuffer, itCtx);
            sqlStatement = sqlStatementBuffer.toString();
          }

          sqlText = PARAM_PARSER.parseInlineParameterMap(delegate.getTypeHandlerFactory(), sqlStatement);

          ParameterMapping[] mappings = sqlText.getParameterMappings();
          out.print(sqlText.getText());
          if (mappings != null) {
             for (int i = 0, n = mappings.length; i < n; i++) {
               ctx.addParameterMapping(mappings[i]);
             }
          }
        }
      } else if (child instanceof SqlTag) {
        ...
      }
    }
}
{% endcodeblock %}

注意到在processBodyChildren方法里调用了InlineParameterMapParser类的parseInlineParameterMap方法，继续跟踪下去。

{% codeblock InlineParameterMapParser.java lang:java %}
public SqlText parseInlineParameterMap(TypeHandlerFactory typeHandlerFactory, String sqlStatement, Class parameterClass) {

    String newSql = sqlStatement;

    List mappingList = new ArrayList();

    StringTokenizer parser = new StringTokenizer(sqlStatement, PARAMETER_TOKEN, true);
    StringBuffer newSqlBuffer = new StringBuffer();

    String token = null;
    String lastToken = null;
    while (parser.hasMoreTokens()) {
      token = parser.nextToken();
      if (PARAMETER_TOKEN.equals(lastToken)) {
        if (PARAMETER_TOKEN.equals(token)) {
          newSqlBuffer.append(PARAMETER_TOKEN);
          token = null;
        } else {
          ParameterMapping mapping = null;
          if (token.indexOf(PARAM_DELIM) > -1) {
            mapping = oldParseMapping(token, parameterClass, typeHandlerFactory);
          } else {
            mapping = newParseMapping(token, parameterClass, typeHandlerFactory);
          }

          mappingList.add(mapping);
          newSqlBuffer.append("?");
          token = parser.nextToken();
          if (!PARAMETER_TOKEN.equals(token)) {
            throw new SqlMapException("Unterminated inline parameter in mapped statement (" + "statement.getId()" + ").");
          }
          token = null;
        }
      } else {
        if (!PARAMETER_TOKEN.equals(token)) {
          newSqlBuffer.append(token);
        }
      }

      lastToken = token;
    }

    newSql = newSqlBuffer.toString();

    ParameterMapping[] mappingArray = (ParameterMapping[]) mappingList.toArray(new ParameterMapping[mappingList.size()]);

    SqlText sqlText = new SqlText();
    sqlText.setText(newSql);
    sqlText.setParameterMappings(mappingArray);
    return sqlText;
}
{% endcodeblock %}

终于找到解析SQL的类了，在parseInlineParameterMap方法中会将下面的SQL

{% codeblock lang:sql %}
SELECT * FROM TABLE WHERE ID = #ID#
{% endcodeblock %}

解析成能被JDBC执行的SQL

{% codeblock lang:sql %}
SELECT * FROM TABLE WHERE ID = ?
{% endcodeblock %}

等到解析完成后会把SQL保存到SqlText类中，再把SqlText保存到MappedStatement类里，有了MappedStatement对象就可以执行相应的CRUD操作了。

