[toc]

---

# 预编译的作用

1. 可以优化SQL执行

   预编译之后的 sql 多数情况下可以直接执行，DBMS 不需要再次编译，越复杂的sql，编译的复杂度将越大，预编译阶段可以合并多次操作为一个操作。可以提升性能。 

2. 防止sql注入

    使用预编译，而其后注入的参数将不会再进行SQL编译。也就是说其后注入进来的参数系统将不会认为它会是一条SQL语句，而默认其是一个参数，参数中的or或者and 等就不是SQL语法保留字了。 

# 数据库预编译

1,  数据库SQL编译特性

数据库接受到sql语句之后，需要词法和语义解析，优化sql语句，制定执行计划。这需要花费一些时间。但是很多情况，我们的一条sql语句可能会反复执行，或者每次执行的时候只有个别的值不同（比如query的where子句值不同，update的set子句值不同,insert的values值不同）。

2. 减少编译的方法

如果每次都需要经过上面的词法语义解析、语句优化、制定执行计划等，则效率就明显不行了。为了解决上面的问题，于是就有了预编译，预编译语句就是将这类语句中的值用占位符替代，可以视为将sql语句模板化或者说参数化。一次编译、多次运行，省去了解析优化等过程。

例如：

```sql
select * from user where id = ? -- 可以是1，2，3...
```

3. 缓存预编译

预编译语句被DB的编译器编译后的执行代码被缓存下来,那么下次调用时只要是相同的预编译语句就不需要编译,只要将参数直接传入编译过的语句执行代码中(相当于一个涵数)就会得到执行。
并不是所有预编译语句都一定会被缓存,数据库本身会用一种策略（内部机制）。 

4. 预编译实现方法

预编译是通过`PreparedStatement`和占位符来实现的。 

## 数据库开启预编译

1. 数据库是否默认开启预编译和 `Jdbc` 版本有关

   - 配置 `jdbc` 链接时强制开启预编译和缓存:`useServerPrepStmts`和`cachePrepStmts`参数。
   - 预编译和预编译缓存一定要同时开启或同时关闭。否则会影响执行效率 

   ```java
   Connection conn = DriverManager.getConnection("jdbc:mysql://localhost:3306/prepare_stmt_test?user=root&password=root&useServerPrepStmts=true&cachePrepStmts=true");  
   ```

2. `mysql`的预编译

   - 开启了预编译缓存后，connection之间，预编译的结果是独立的，是无法共享的，一个connection无法得到另外一个connection的预编译缓存结果。
   - 经过试验，`mysql`的预编译功能对性能影响不大，但在`jdbc`中使用`PreparedStatement`是必要的，可以有效地防止`sql注入`。
   - 相同`PreparedStatement`的对象 ，可以不用开启预编译缓存。

   ```java
   Connection conn = DriverManager.getConnection("jdbc:mysql://localhost:3306/prepare_stmt_test?user=root&password=root&useServerPrepStmts=true");  
   PreparedStatement stmt = conn.prepareStatement(sql);  
   stmt.setString(1, "aaa");  
   ResultSet rs1 = stmt.executeQuery();//第一次执行  
   s1.close();  
   stmt.setString(1, "ddd");  
   ResultSet rs2 = stmt.executeQuery();//第二次执行  
   rs2.close();  
   stmt.close(); 
   //查看mysql日志
   /*1 Prepare          select * from users where name = ?
       1 Execute          select * from users where name = 'aaa'
       1 Execute          select * from users where name = 'ddd'*/
   
   ```




# Mybatis 预编译

## 预编译的解析

预编译分为客户端和服务端预编译，而 `Mybatis` 就是一种客户端预编译。

那么我们看一下 `XMLStatementBuilder` 看看他是怎么实现的预编译

```java
/**
   * 解析mapper中的SQL语句
   */
  public void parseStatementNode() {
      // 获取sql ID
    String id = context.getStringAttribute("id");
      
      // 获取数据库ID，判断databaseId是否匹配
    String databaseId = context.getStringAttribute("databaseId");
    if (!databaseIdMatchesCurrent(id, databaseId, this.requiredDatabaseId)) {
      return;
    }

      // 获取标签属性
      // 获取节点名称<select> <insert>等
    String nodeName = context.getNode().getNodeName();
      // 转换指令类型
    SqlCommandType sqlCommandType = SqlCommandType.valueOf(nodeName.toUpperCase(Locale.ENGLISH));
      // 判断是否是查询
    boolean isSelect = sqlCommandType == SqlCommandType.SELECT;
    boolean flushCache = context.getBooleanAttribute("flushCache", !isSelect);
    boolean useCache = context.getBooleanAttribute("useCache", isSelect);
    boolean resultOrdered = context.getBooleanAttribute("resultOrdered", false);

    // Include Fragments before parsing
    XMLIncludeTransformer includeParser = new XMLIncludeTransformer(configuration, builderAssistant);
    includeParser.applyIncludes(context.getNode());

      // 获取参数类型，转换出class
    String parameterType = context.getStringAttribute("parameterType");
    Class<?> parameterTypeClass = resolveClass(parameterType);

      // 获取驱动
    String lang = context.getStringAttribute("lang");
    LanguageDriver langDriver = getLanguageDriver(lang);

    // Parse selectKey after includes and remove them.
    processSelectKeyNodes(id, parameterTypeClass, langDriver);

    // Parse the SQL (pre: <selectKey> and <include> were parsed and removed)
      // 转换SQl， 主键生成器
    KeyGenerator keyGenerator;
    String keyStatementId = id + SelectKeyGenerator.SELECT_KEY_SUFFIX;
    keyStatementId = builderAssistant.applyCurrentNamespace(keyStatementId, true);
    if (configuration.hasKeyGenerator(keyStatementId)) {
      keyGenerator = configuration.getKeyGenerator(keyStatementId);
    } else {
      keyGenerator = context.getBooleanAttribute("useGeneratedKeys",
          configuration.isUseGeneratedKeys() && SqlCommandType.INSERT.equals(sqlCommandType))
          ? Jdbc3KeyGenerator.INSTANCE : NoKeyGenerator.INSTANCE;
    }

      // 重要:解析SQL语句,封装成一个SqlSource
    SqlSource sqlSource = langDriver.createSqlSource(configuration, context, parameterTypeClass);
    StatementType statementType = StatementType.valueOf(context.getStringAttribute("statementType", StatementType.PREPARED.toString()));
    Integer fetchSize = context.getIntAttribute("fetchSize");
    Integer timeout = context.getIntAttribute("timeout");
    String parameterMap = context.getStringAttribute("parameterMap");
    String resultType = context.getStringAttribute("resultType");
    Class<?> resultTypeClass = resolveClass(resultType);
    String resultMap = context.getStringAttribute("resultMap");
    String resultSetType = context.getStringAttribute("resultSetType");
    ResultSetType resultSetTypeEnum = resolveResultSetType(resultSetType);
    if (resultSetTypeEnum == null) {
      resultSetTypeEnum = configuration.getDefaultResultSetType();
    }
    String keyProperty = context.getStringAttribute("keyProperty");
    String keyColumn = context.getStringAttribute("keyColumn");
    String resultSets = context.getStringAttribute("resultSets");

      // 解析完毕,最后通过MapperBuilderAssistant创建MappedStatement对象
      // 统一保存到Configuration的mappedStatements属性中
    builderAssistant.addMappedStatement(id, sqlSource, statementType, sqlCommandType,
        fetchSize, timeout, parameterMap, parameterTypeClass, resultMap, resultTypeClass,
        resultSetTypeEnum, flushCache, useCache, resultOrdered,
        keyGenerator, keyProperty, keyColumn, databaseId, langDriver, resultSets);
  }
```

1. `Mybaits` 会对 `Sql标签` 做解析，然后转换成一个`SqlSource` 在生成一个`MappedStatement` 保存。
2. `langDriver.createSqlSource`这个方法。这该方法会通过`LanguageDriver`对`SQL`语句进行解析，生成一个`SqlSource`。 
3.  **`SqlSource`封装了映射文件或者注解中定义的`SQL`语句，它不能直接交给数据库执行，因为里面可能包含动态`SQL`或者占位符等元素**。 
4.  而`MyBatis`在实际执行`SQL`语句时，会调用`SqlSource`的`getBoundSql`()方法获取一个`BoundSql`对象，**`BoundSql`是将`SqlSource`中的动态内容经过处理后，返回的实际可执行的`SQL`语句，其中包含?占位符List封装的有序的参数映射关系，此外还有一些额外信息标识每个参数的属性名称等。** 

- `resultSetType` 类型
  - FORWARD_ONLY：结果集的游标只能向下滚动。
  - SCROLL_INSENSITIVE：结果集的游标可以上下移动，当数据库变化时，当前结果集不变。
  - SCROLL_SENSITIVE：返回可滚动的结果集，当数据库变化时，当前结果集同步改变。

- `statementType` 类型
  - STATEMENT：普通语句。
  - PREPARED：预处理。
  - CALLABLE：存储过程。

看一下 `XMLLanguageDriver.createSqlSource` 这个方法

```java
// 创建SqlSource
@Override
public SqlSource createSqlSource(Configuration configuration, XNode script, Class<?> parameterType) {
     //创建XMLScriptBuilder对象
    XMLScriptBuilder builder = new XMLScriptBuilder(configuration, script, parameterType);
    
  //通过XMLScriptBuilder解析SQL脚本
    return builder.parseScriptNode();
}

```

 `XMLScriptBuilder.parseScriptNodes` 进行解析 

```java
/**
   * 解析SQL脚本
   */
public SqlSource parseScriptNode() {
  //解析动态标签,包括动态SQL和${}。执行后动态SQL和${}已经被解析完毕。
  //此时SQL语句中的#{}还没有处理,#{}会在SQL执行时动态解析
  MixedSqlNode rootSqlNode = parseDynamicTags(context);

  //如果是dynamic的,则创建DynamicSqlSource,否则创建RawSqlSource
  SqlSource sqlSource = null;
  if (isDynamic) {
    sqlSource = new DynamicSqlSource(configuration, rootSqlNode);
  } else {
    sqlSource = new RawSqlSource(configuration, rootSqlNode, parameterType);
  }
  return sqlSource;
}
```

 是否为`动态SQL`的判断在`parseDynamicTags`方法中

```java
 protected MixedSqlNode parseDynamicTags(XNode node) {
    List<SqlNode> contents = new ArrayList<>();
    NodeList children = node.getNode().getChildNodes();
    for (int i = 0; i < children.getLength(); i++) {
      XNode child = node.newXNode(children.item(i));
        // 文本节点
      if (child.getNode().getNodeType() == Node.CDATA_SECTION_NODE || child.getNode().getNodeType() == Node.TEXT_NODE) {
          // 封装到 TextSqlNode 
        String data = child.getStringBody("");
        TextSqlNode textSqlNode = new TextSqlNode(data);
          // 如果包含${},则是动态Sql
        if (textSqlNode.isDynamic()) {
          contents.add(textSqlNode);
          isDynamic = true;
        } else {
            // 除了${}外，其它的都是静态
          contents.add(new StaticTextSqlNode(data));
        }
      } else if (child.getNode().getNodeType() == Node.ELEMENT_NODE) { // issue #628
        String nodeName = child.getNode().getNodeName();
        NodeHandler handler = nodeHandlerMap.get(nodeName);
        if (handler == null) {
          throw new BuilderException("Unknown element <" + nodeName + "> in SQL statement.");
        }
        handler.handleNode(child, contents);
        isDynamic = true;
      }
    }
    return new MixedSqlNode(contents);
  }

```

 如果是动态标签，创建的就是`DynamicSqlSource`，其获取的`BoundSql`就是直接进行字符串的替换。对于非动态标签，则创建`RawSqlSource`，对应`?占位符`的`SQL`语句

- 如果`SQL`中的参数是用`${}`作为占位符的，那么该`SQL`属于动态`SQL`，封装为`DynamicSqlSource`。
  - 否则其他的都是非动态`SQL`，封装为`RawSqlSource`。



## 预编译的处理

我们来看看`RowSqlSource` 是怎么来做处理的，先看一下她的构造函数

```java
public RawSqlSource(Configuration configuration, String sql, Class<?> parameterType) {
    SqlSourceBuilder sqlSourceParser = new SqlSourceBuilder(configuration);
    Class<?> clazz = parameterType == null ? Object.class : parameterType;
    // 重点 sqlSourceParser.parse
    sqlSource = sqlSourceParser.parse(sql, clazz, new HashMap<>());
}

// SqlSourceBuilder.parse
public class SqlSourceBuilder extends BaseBuilder {
  public SqlSource parse(String originalSql, Class<?> parameterType, Map<String, Object> additionalParameters) {
      // 参数解析器
    ParameterMappingTokenHandler handler = new ParameterMappingTokenHandler(configuration, parameterType, additionalParameters);

      // 以#{}的解析器
    GenericTokenParser parser = new GenericTokenParser("#{", "}", handler);
    String sql;
      // 是否是收缩空白的SQL
    if (configuration.isShrinkWhitespacesInSql()) {
      sql = parser.parse(removeExtraWhitespaces(originalSql));
    } else {
        // 否则是原始sql
      sql = parser.parse(originalSql);
    }
    return new StaticSqlSource(configuration, sql, handler.getParameterMappings());
  }
}
```

 对于非动态`SQL`，会生成一个以 `#{` 为开头，`}` 为结尾的解析器。紧接着就会创建一个`StaticSqlSource` 类，在这里做个区分： 

- `RawSqlSource` : 存储的是只有 #{} 或者没有标签的`纯文本SQL信息`
- `DynamicSqlSource` : 存储的是写有 ${} 或者具有`动态SQL标签`的`SQL信息`
- `StaticSqlSource` : 是`DynamicSqlSource`和`RawSqlSource`解析为BoundSql的一个中间态对象类型。
- `BoundSql`：用于生成我们最终执行的`SQL`语句，属性包括参数值、映射关系、以及`SQL`（带问号的）

我们看一下关于这个解析器 `GenericTokenParser.parse()`这段代码中的使用

```java
// 转换sql字符
public String parse(String text) {
    // 空返回
    if (text == null || text.isEmpty()) {
        return "";
    }
    // search open token 查找 #{}
    int start = text.indexOf(openToken);
    // 查找不到#{}就返回
    if (start == -1) {
        return text;
    }
    char[] src = text.toCharArray();
    int offset = 0;
    final StringBuilder builder = new StringBuilder();
    StringBuilder expression = null;
    do {
        if (start > 0 && src[start - 1] == '\\') {
            // this open token is escaped. remove the backslash and continue.
            builder.append(src, offset, start - offset - 1).append(openToken);
            offset = start + openToken.length();
        } else {
            // found open token. let's search close token.
            if (expression == null) {
                expression = new StringBuilder();
            } else {
                expression.setLength(0);
            }
            builder.append(src, offset, start - offset);
            offset = start + openToken.length();
            int end = text.indexOf(closeToken, offset);
            while (end > -1) {
                if (end > offset && src[end - 1] == '\\') {
                    // this close token is escaped. remove the backslash and continue.
                    expression.append(src, offset, end - offset - 1).append(closeToken);
                    offset = end + closeToken.length();
                    end = text.indexOf(closeToken, offset);
                } else {
                    expression.append(src, offset, end - offset);
                    break;
                }
            }
            if (end == -1) {
                // close token was not found.
                builder.append(src, start, src.length - start);
                offset = src.length;
            } else {
	 // 就是找到了#{}的结束标识}，然后将中间的内容替换成？
             	builder.append(handler.handleToken(expression.toString()));
                offset = end + closeToken.length();
            }
        }
        start = text.indexOf(openToken, offset);
    } while (start > -1);
    if (offset < src.length) {
        builder.append(src, offset, src.length - offset);
    }
    return builder.toString();
}
```

1. 查找 `#{` 是否存在，如果存在则继续查找
2. 查找 `}` 是否存在，如果存在则将中间的内容替换掉 `?`
3. 返回 处理后了的 `sql`

 解析器`ParameterMappingTokenHandler` 的处理，它是`SqlSourceBuilder`的一个静态内部类： 就是将内容转换成 `?`，并将内容作为映射

```java
@Override
public String handleToken(String content) {
    parameterMappings.add(buildParameterMapping(content));
    return "?";
}

private ParameterMapping buildParameterMapping(String content) {
    Map<String, String> propertiesMap = parseParameterMapping(content);
    String property = propertiesMap.get("property");
    Class<?> propertyType;
    if (metaParameters.hasGetter(property)) { // issue #448 get type from additional params
        propertyType = metaParameters.getGetterType(property);
    } else if (typeHandlerRegistry.hasTypeHandler(parameterType)) {
        propertyType = parameterType;
    } else if (JdbcType.CURSOR.name().equals(propertiesMap.get("jdbcType"))) {
        propertyType = java.sql.ResultSet.class;
    } else if (property == null || Map.class.isAssignableFrom(parameterType)) {
        propertyType = Object.class;
    } else {
        MetaClass metaClass = MetaClass.forClass(parameterType, configuration.getReflectorFactory());
        if (metaClass.hasGetter(property)) {
            propertyType = metaClass.getGetterType(property);
        } else {
            propertyType = Object.class;
        }
    }
    ParameterMapping.Builder builder = new ParameterMapping.Builder(configuration, property, propertyType);
    Class<?> javaType = propertyType;
    String typeHandlerAlias = null;
    for (Map.Entry<String, String> entry : propertiesMap.entrySet()) {
        String name = entry.getKey();
        String value = entry.getValue();
        if ("javaType".equals(name)) {
            javaType = resolveClass(value);
            builder.javaType(javaType);
        } else if ("jdbcType".equals(name)) {
            builder.jdbcType(resolveJdbcType(value));
        } else if ("mode".equals(name)) {
            builder.mode(resolveParameterMode(value));
        } else if ("numericScale".equals(name)) {
            builder.numericScale(Integer.valueOf(value));
        } else if ("resultMap".equals(name)) {
            builder.resultMapId(value);
        } else if ("typeHandler".equals(name)) {
            typeHandlerAlias = value;
        } else if ("jdbcTypeName".equals(name)) {
            builder.jdbcTypeName(value);
        } else if ("property".equals(name)) {
            // Do Nothing
        } else if ("expression".equals(name)) {
            throw new BuilderException("Expression based parameters are not supported yet");
        } else {
            throw new BuilderException("An invalid property '" + name + "' was found in mapping #{" + content + "}.  Valid properties are " + PARAMETER_PROPERTIES);
        }
    }
    if (typeHandlerAlias != null) {
        builder.typeHandler(resolveTypeHandler(javaType, typeHandlerAlias));
    }
    return builder.build();
}
```

## 预编译的参数替换

 那么在执行`SQL`的时候，则会去根据`BoundSql`来完成参数的赋值等操作。我们来看下`RawSqlSource.getBoundSql`这个函数： 

```java

  @Override
  public BoundSql getBoundSql(Object parameterObject) {
    return sqlSource.getBoundSql(parameterObject);
  }

```

 因为只有`#{}`的`SQL`语句，在上文中可以看到最后会生成一个`StaticSqlSource`对象，而这个类中就重写了`getBoundSql`函数，里面主要构造了一个`BoundSql`对象。 

```java
public class StaticSqlSource implements SqlSource {
  @Override
  public BoundSql getBoundSql(Object parameterObject) {
    return new BoundSql(configuration, sql, parameterMappings, parameterObject);
  }
}
```

主要看的是`SimpleExecutor.prepareStatement`

```java
private Statement prepareStatement(StatementHandler handler, Log statementLog) throws SQLException {
    Statement stmt;
    Connection connection = getConnection(statementLog);
    stmt = handler.prepare(connection, transaction.getTimeout());
    handler.parameterize(stmt);
    return stmt;
}
```

最终会走到 `DefaultParameterHandler.setParameters`执行参数替换

```java
@Override
public void setParameters(PreparedStatement ps) {
    ErrorContext.instance().activity("setting parameters").object(mappedStatement.getParameterMap().getId());
    // 获取传入参数
    List<ParameterMapping> parameterMappings = boundSql.getParameterMappings();
    if (parameterMappings != null) {
        for (int i = 0; i < parameterMappings.size(); i++) {
            ParameterMapping parameterMapping = parameterMappings.get(i);
            // mode属性有三种：IN, OUT, INOUT。如果参数为 OUT 或 INOUT，参数对象属性的真实值将会被改变
            if (parameterMapping.getMode() != ParameterMode.OUT) {
                Object value;
                String propertyName = parameterMapping.getProperty();
                if (boundSql.hasAdditionalParameter(propertyName)) { // issue #448 ask first for additional params
                    value = boundSql.getAdditionalParameter(propertyName);
                } else if (parameterObject == null) {
                    value = null;
                } 
                  // 如果类型处理器里面有这个类型，直接赋值即可。
                else if (typeHandlerRegistry.hasTypeHandler(parameterObject.getClass())) {
                    value = parameterObject;
                } 
                
		  // 否则转化为元数据处理，通过反射来完成get/set赋值
                else {
                    MetaObject metaObject = configuration.newMetaObject(parameterObject);
                    value = metaObject.getValue(propertyName);
                }
                TypeHandler typeHandler = parameterMapping.getTypeHandler();
                JdbcType jdbcType = parameterMapping.getJdbcType();
                if (value == null && jdbcType == null) {
                    jdbcType = configuration.getJdbcTypeForNull();
                }
                try {
                    // 使用不同的类型处理器向jdbc中的PreparedStatement设置参数
                    typeHandler.setParameter(ps, i + 1, value, jdbcType);
                } catch (TypeException | SQLException e) {
                    throw new TypeException("Could not set parameters for mapping: " + parameterMapping + ". Cause: " + e, e);
                }
            }
        }
    }
}
```

## 小结

 首先预编译对于`Mybatis`而言，相当于构建出了一条`SQL`的模板，将`#{}`对应的参数改为`?`而已。届时只需要更改参数的值即可，无需再对`SQL`进行语法解析等操作。 

 对于动态`SQL`的判断，就是在于是否包含`${}`占位符。如果包含了就通过`DynamicSqlSource`来解析。而这里则影响到`SQL`的解析： 

- `DynamicSqlSource`：解析包含`${}`的语句，其实也会解析`#{}`的语句。
- `RawSqlSource`：解析只包含`#{}`的语句。

 这两种类型到最后都会转化为`StaticSqlSource`，然后由他创建一个`BoundSql`对象。包括参数值、映射关系、以及转化好的`SQL`。 

 最后是关于`SQL`的执行，即如何将真实的参数赋值到我们上面生成的模板`SQL`中。这部分逻辑发生在`SQL`的执行过程中，其入口`SimpleExecutor.prepareStatement`主要做了这么几件事。 

1. 根据我们上面生成的`BoundSql`对象。拿到我们传入的参数。
2. 对每个参数进行解析，转化成对应的类型。
3. 如果转化出的参数值为`null`，则直接赋值，否则，还要通过类型处理器来完成赋值操作。`typeHandler.setParameter(ps, i + 1, value, jdbcType);`
4. 每种类型处理器，则会对对应的参数进行赋值。

 Mybatis支持的类型处理器可以看TypeHandlerRegistry这个类 