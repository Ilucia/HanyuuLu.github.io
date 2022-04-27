## Mybatis

### 步骤

##### 创建表

##### 创建实体

##### 创建配置文件

- **创建mybatis的配置⽂件，配置数据库的信息**
- **数据库我们可以配置多个，但是默认的只能⽤⼀个**

##### 创建映射文件

- SQL语句在映射文件中

##### CRUD

- create read update delete

- insert/update/delete标签只是⼀个模板，在做操作时，其实是以SQL语句为核⼼的
  - 即在做增删改时，insert/update/delete标签可通⽤，
- 但做查询时只能⽤select标签
- 我们提倡什么操作就⽤什么标签

#### 创建工具类测试

- Mybatis中的事务是默认开启的，因此我们在完成操作以后，需要我们⼿动去提交事务

### 扩展功能

#### 创建DAO

#### 分页

#### 动态SQL

##### 动态查询

```xml
<!--多条件查询【动态SQL】-->
<!--会⾃动组合成⼀个正常的WHERE字句-->
<!--name值会从map中寻找-->
<select id="findByCondition" resultMap="studentMap" parameterType="map">
  select * from users
  <where>
    <if test="uname!=null">
    	and uname=#{uname}
  	</if>
  	<if test="uage!=null">
  		and uage < #{uage}
  	</if>
  </where>
</select>
```

- 查询名字为name，年龄小于age的人
  - 为null则代表该字段无条件

```java
public static List<User> findByCondition(String name,Integer age) throws Exception {
    //得到连接对象
    SqlSession sqlSession = MybatisUtil.getSqlSession();
    try{
        //映射⽂件的命名空间.SQL⽚段的ID，就可以调⽤对应的映射⽂件中的SQL
        /**
         * 由于我们的参数超过了两个，⽽⽅法中只有⼀个Object参数收集
         * 因此我们可以使⽤Map集合来装载我们的参数
         */
        Map<String, Object> map = new HashMap();
        map.put("uname", name);
        map.put("uage", age);
        return sqlSession.selectList("findByCondition", map);
    }catch(Exception e){
        e.printStackTrace();
        sqlSession.rollback();
        throw e;
    }finally{
        MybatisUtil.closeSqlSession();
    }
}
```

## choose, when, otherwise

有些时候，我们不想用到所有的条件语句，而只想从中择其一二。针对这种情况，MyBatis 提供了 choose 元素，它有点像 Java 中的 switch 语句。

还是上面的例子，但是这次变为提供了"title"就按"title"查找，提供了"author"就按"author"查找，若两者都没有提供，就返回所有符合条件的BLOG（实际情况可能是由管理员按一定策略选出BLOG列表，而不是返回大量无意义的随机结果）。

```xml
<select id="findActiveBlogLike"
     resultType="Blog">
  SELECT * FROM BLOG WHERE state = ‘ACTIVE’
  <choose>
    <when test="title != null">
      AND title like #{title}
    </when>
    <when test="author != null and author.name != null">
      AND author_name like #{author.name}
    </when>
    <otherwise>
      AND featured = 1
    </otherwise>
  </choose>
</select>
```

## trim, where, set

前面几个例子已经合宜地解决了一个臭名昭著的动态 SQL 问题。现在考虑回到"if"示例，这次我们将"ACTIVE = 1"也设置成动态的条件，看看会发生什么。

```xml
<select id="findActiveBlogLike"
     resultType="Blog">
  SELECT * FROM BLOG 
  WHERE 
  <if test="state != null">
    state = #{state}
  </if> 
  <if test="title != null">
    AND title like #{title}
  </if>
  <if test="author != null and author.name != null">
    AND author_name like #{author.name}
  </if>
</select>
```

如果这些条件没有一个能匹配上将会怎样？最终这条 SQL 会变成这样：

```sql
SELECT * FROM BLOG
WHERE
```

这会导致查询失败。如果仅仅第二个条件匹配又会怎样？这条 SQL 最终会是这样:

```sql
SELECT * FROM BLOG
WHERE
AND title like 'someTitle'
```

这个查询也会失败。这个问题不能简单的用条件句式来解决，如果你也曾经被迫这样写过，那么你很可能从此以后都不想再这样去写了。

MyBatis 有一个简单的处理，这在90%的情况下都会有用。而在不能使用的地方，你可以自定义处理方式来令其正常工作。一处简单的修改就能得到想要的效果：

```
<select id="findActiveBlogLike"
     resultType="Blog">
  SELECT * FROM BLOG 
  <where> 
    <if test="state != null">
         state = #{state}
    </if> 
    <if test="title != null">
        AND title like #{title}
    </if>
    <if test="author != null and author.name != null">
        AND author_name like #{author.name}
    </if>
  </where>
</select>
```

where 元素知道只有在一个以上的if条件有值的情况下才去插入"WHERE"子句。而且，若最后的内容是"AND"或"OR"开头的，where 元素也知道如何将他们去除。

如果 where 元素没有按正常套路出牌，我们还是可以通过自定义 trim 元素来定制我们想要的功能。比如，和 where 元素等价的自定义 trim 元素为：

```
<trim prefix="WHERE" prefixOverrides="AND |OR ">
  ... 
</trim>
```

prefixOverrides 属性会忽略通过管道分隔的文本序列（注意此例中的空格也是必要的）。它带来的结果就是所有在 prefixOverrides 属性中指定的内容将被移除，并且插入 prefix 属性中指定的内容。

类似的用于动态更新语句的解决方案叫做 set。set 元素可以被用于动态包含需要更新的列，而舍去其他的。比如：

```xml
<update id="updateAuthorIfNecessary">
  update Author
    <set>
      <if test="username != null">username=#{username},</if>
      <if test="password != null">password=#{password},</if>
      <if test="email != null">email=#{email},</if>
      <if test="bio != null">bio=#{bio}</if>
    </set>
  where id=#{id}
</update>
```

这里，set 元素会动态前置 SET 关键字，同时也会消除无关的逗号，因为用了条件语句之后很可能就会在生成的赋值语句的后面留下这些逗号。

若你对等价的自定义 trim 元素的样子感兴趣，那这就应该是它的真面目：

```
<trim prefix="SET" suffixOverrides=",">
  ...
</trim>
```

注意这里我们忽略的是后缀中的值，而又一次附加了前缀中的值。

## foreach

动态 SQL 的另外一个常用的必要操作是需要对一个集合进行遍历，通常是在构建 IN 条件语句的时候。比如：

```
<select id="selectPostIn" resultType="domain.blog.Post">
  SELECT *
  FROM POST P
  WHERE ID in
  <foreach item="item" index="index" collection="list"
      open="(" separator="," close=")">
        #{item}
  </foreach>
</select>
```

foreach 元素的功能是非常强大的，它允许你指定一个集合，声明可以用在元素体内的集合项和索引变量。它也允许你指定开闭匹配的字符串以及在迭代中间放置分隔符。这个元素是很智能的，因此它不会偶然地附加多余的分隔符。

注意 你可以将一个 List 实例或者数组作为参数对象传给 MyBatis，当你这么做的时候，MyBatis 会自动将它包装在一个 Map 中并以名称为键。List 实例将会以"list"作为键，而数组实例的键将是"array"。

到此我们已经完成了涉及 XML 配置文件和 XML 映射文件的讨论。下一部分将详细探讨 Java API，这样才能从已创建的映射中获取最大利益。

## bind

`bind` 元素可以从 OGNL 表达式中创建一个变量并将其绑定到上下文。比如：

```
<select id="selectBlogsLike" resultType="Blog">
  <bind name="pattern" value="'%' + _parameter.getTitle() + '%'" />
  SELECT * FROM BLOG
  WHERE title LIKE #{pattern}
</select>
```

## Multi-db vendor support

一个配置了"_databaseId"变量的 databaseIdProvider 对于动态代码来说是可用的，这样就可以根据不同的数据库厂商构建特定的语句。比如下面的例子：

```xml
<insert id="insert">
  <selectKey keyProperty="id" resultType="int" order="BEFORE">
    <if test="_databaseId == 'oracle'">
      select seq_users.nextval from dual
    </if>
    <if test="_databaseId == 'db2'">
      select nextval for seq_users from sysibm.sysdummy1"
    </if>
  </selectKey>
  insert into users values (#{id}, #{name})
</insert>
```

## 动态 SQL 中可插拔的脚本语言

MyBatis 从 3.2 开始支持可插拔的脚本语言，因此你可以在插入一种语言的驱动（language driver）之后来写基于这种语言的动态 SQL 查询。

可以通过实现下面接口的方式来插入一种语言：

```java
    public interface LanguageDriver {
      ParameterHandler createParameterHandler(MappedStatement mappedStatement, Object parameterObject, BoundSql boundSql);
      SqlSource createSqlSource(Configuration configuration, XNode script, Class parameterType);
      SqlSource createSqlSource(Configuration configuration, String script, Class parameterType);
    }
```

一旦有了自定义的语言驱动，你就可以在 mybatis-config.xml 文件中将它设置为默认语言：

```xml
<typeAliases>
  <typeAlias type="org.sample.MyLanguageDriver" alias="myLanguage"/>
</typeAliases>
<settings>
  <setting name="defaultScriptingLanguage" value="myLanguage"/>
</settings>
```

除了设置默认语言，你也可以针对特殊的语句指定特定语言，这可以通过如下的 `lang` 属性来完成：

```xml
<select id="selectBlog" lang="myLanguage">
  SELECT * FROM BLOG
</select>
```

或者在你正在使用的映射中加上注解 `@Lang` 来完成：

```java
    public interface Mapper {
      @Lang(MyLanguageDriver.class)
      @Select("SELECT * FROM BLOG")
      List selectBlog();
    }
```

注意 可以将 Apache Velocity 作为动态语言来使用，更多细节请参考 MyBatis-Velocity 项目。

你前面看到的所有 xml 标签都是默认 MyBatis 语言提供的，它是由别名为 `xml` 语言驱动器 `org.apache.ibatis.scripting.xmltags.XmlLanguageDriver` 驱动的。

### 注解

| 注解                                                         |    目标     | 相对应的 XML                                                 | 描述                                                         |
| ------------------------------------------------------------ | :---------: | ------------------------------------------------------------ | ------------------------------------------------------------ |
| `@CacheNamespace`                                            |    `类`     | ``                                                           | 为给定的命名空间 (比如类) 配置缓存。 属性:implemetation,eviction, flushInterval,size 和 readWrite。 |
| `@CacheNamespaceRef`                                         |    `类`     | ``                                                           | 参照另外一个命名空间的缓存来使用。 属性:value,应该是一个名空间的字 符串值(也就是类的完全限定名) 。 |
| `@ConstructorArgs`                                           |  `Method`   | ``                                                           | 收集一组结果传递给一个劫夺对象的 构造方法。属性:value,是形式参数 的数组。 |
| `@Arg`                                                       |   `方法`    |                                                              | 单 独 的 构 造 方 法 参 数 , 是 ConstructorArgs 集合的一部分。属性: id,column,javaType,typeHandler。 id 属性是布尔值, 来标识用于比较的属 性,和XML 元素相似。 |
| `@TypeDiscriminator`                                         |   `方法`    | ``                                                           | 一组实例值被用来决定结果映射的表 现。 属性: column, javaType, jdbcType, typeHandler,cases。cases 属性就是实 例的数组。 |
| `@Case`                                                      |   `方法`    | ``                                                           | 单独实例的值和它对应的映射。属性: value,type,results。Results 属性是结 果数组,因此这个注解和实际的 ResultMap 很相似,由下面的 Results 注解指定。 |
| `@Results`                                                   |   `方法`    | ``                                                           | 结果映射的列表, 包含了一个特别结果 列如何被映射到属性或字段的详情。 属 性:value,是 Result 注解的数组。 |
| `@Result`                                                    |   `方法`    |                                                              | 在列和属性或字段之间的单独结果映 射。属 性:id,column, property, javaType ,jdbcType ,type Handler, one,many。id 属性是一个布尔值,表 示了应该被用于比较(和在 XML 映射 中的相似)的属性。one 属性是单 独 的 联 系, 和 相 似 , 而 many 属 性 是 对 集 合 而 言 的 , 和 相似。 它们这样命名是为了 避免名称冲突。 |
| `@One`                                                       |   `方法`    | `` | 复杂类型的单独属性值映射。属性: select,已映射语句(也就是映射器方 法)的完全限定名,它可以加载合适类 型的实例。注意:联合映射在注解 API 中是不支持的。这是因为 Java 注解的 限制,不允许循环引用。`fetchType`, which supersedes the global configuration parameter`lazyLoadingEnabled` for this mapping. |                                                              |
| `@Many`                                                      |   `方法`    | `` | A mapping to a collection property of a complex type. Attributes:`select`, which is the fully qualified name of a mapped statement (i.e. mapper method) that can load a collection of instances of the appropriate types,`fetchType`, which supersedes the global configuration parameter`lazyLoadingEnabled` for this mapping. NOTE You will notice that join mapping is not supported via the Annotations API. This is due to the limitation in Java Annotations that does not allow for circular references. |                                                              |
| `@MapKey`                                                    |   `方法`    |                                                              | 复 杂 类 型 的 集合 属 性 映射 。 属 性 : select,是映射语句(也就是映射器方 法)的完全限定名,它可以加载合适类 型的一组实例。注意:联合映射在 Java 注解中是不支持的。这是因为 Java 注 解的限制,不允许循环引用。 |
| `@Options`                                                   |   `方法`    | 映射语句的属性                                               | 这个注解提供访问交换和配置选项的 宽广范围, 它们通常在映射语句上作为 属性出现。 而不是将每条语句注解变复 杂,Options 注解提供连贯清晰的方式 来访问它们。属性:useCache=true , flushCache=false , resultSetType=FORWARD_ONLY , statementType=PREPARED , fetchSize=-1 , , timeout=-1 useGeneratedKeys=false , keyProperty="id"。 理解 Java 注解是很 重要的,因为没有办法来指定"null" 作为值。因此,一旦你使用了 Options 注解,语句就受所有默认值的支配。要 注意什么样的默认值来避免不期望的 行为。 |
| * `@Insert`,`@Update`, `@Delete`, `@Select`                  |   `方法`    | `<insert>`,`<update>`,`<delete>`,`<select>`                  | 这些注解中的每一个代表了执行的真 实 SQL。 它们每一个都使用字符串数组 (或单独的字符串)。如果传递的是字 符串数组, 它们由每个分隔它们的单独 空间串联起来。这就当用 Java 代码构 建 SQL 时避免了“丢失空间”的问题。 然而,如果你喜欢,也欢迎你串联单独 的字符串。属性:value,这是字符串 数组用来组成单独的 SQL 语句。 |
| @InsertProvider,@UpdateProvider,@DeleteProvider,@SelectProvider |    方法     | `<insert>`,`<update>`,`<delete>`,`<select>`                  | 这些可选的 SQL 注解允许你指定一个 类名和一个方法在执行时来返回运行 允许创建动态 的 SQL。 基于执行的映射语句, MyBatis 会实例化这个类,然后执行由 provider 指定的方法. 这个方法可以选择性的接 受参数对象作为它的唯一参数, 但是必 须只指定该参数或者没有参数。属性: type,method。type 属性是类的完全限 定名。method 是该类中的那个方法名。 注意: 这节之后是对 SelectBuilder 类的 讨论,它可以帮助你以干净,容于阅读 的方式来构建动态 SQL。 |
| `@Param`                                                     | `Parameter` | N/A                                                          | 如果你的映射器的方法需要多个参数, 这个注解可以被应用于映射器的方法 参数来给每个参数一个名字。否则,多 参数将会以它们的顺序位置来被命名 (不包括任何 RowBounds 参数) 比如。 #{param1} , #{param2} 等 , 这 是 默 认 的 。 使 用 @Param("person"),参数应该被命名为 #{person}。 |
| `@SelectKey`                                                 |  `Method`   | `<selectKey>`                                                | This annotation duplicates the `` functionality for methods annotated with`@Insert`,`@InsertProvider`,`@Update`or`@UpdateProvider`. It is ignored for other methods. If you specify a`@SelectKey`annotation, then MyBatis will ignore any generated key properties set via the`@Options`annotation, or configuration properties. Attributes: statement an array of strings which is the SQL statement to execute,`keyProperty`which is the property of the parameter object that will be updated with the new value, before which must be either`true`or`false`to denote if the SQL statement should be executed before or after the insert,`resultType`which is the Java type of the`keyProperty`, and`statementType=PREPARED`. |
| `@ResultMap`                                                 |  `Method`   | N/A                                                          | This annotation is used to provide the id of a `` element in an XML mapper to a`@Select`or`@SelectProvider`annotation. This allows annotated selects to reuse resultmaps that are defined in XML. This annotation will override any`@Results`or`@ConstructorArgs` annotation if both are specified on an annotated select. |
| `@ResultType`                                                |  `Method`   | N/A                                                          | This annotation is used when using a result handler. In that case, the return type is void so MyBatis must have a way to determine the type of object to construct for each row. If there is an XML result map, use the @ResultMap annotation. If the result type is specified in XML on the ` Night Mode |

#### 映射申明样例

这个例子展示了如何使用 @SelectKey 注解来在插入前读取数据库序列的值：

```
@Insert("insert into table3 (id, name) values(#{nameId}, #{name})")
@SelectKey(statement="call next value for TestSequence", keyProperty="nameId", before=true, resultType=int.class)
int insertTable3(Name name);
```

这个例子展示了如何使用 @SelectKey 注解来在插入后读取数据库识别列的值：

```
@Insert("insert into table2 (name) values(#{name})")
@SelectKey(statement="call identity()", keyProperty="nameId", before=false, resultType=int.class)
int insertTable2(Name name);
```

## MyBatis Plus

### 一、mybatis-plus简介：

Mybatis-Plus（简称MP）是一个 Mybatis 的增强工具，在 Mybatis 的基础上只做增强不做改变，为简化开发、提高效率而生。这是官方给的定义，关于mybatis-plus的更多介绍及特性，可以参考[mybatis-plus官网](https://links.jianshu.com/go?to=http%3A%2F%2Fmp.baomidou.com%2F%23%2F)。那么它是怎么增强的呢？其实就是它已经封装好了一些crud方法，我们不需要再写xml了，直接调用这些方法就行，就类似于JPA。

### 二、spring整合mybatis-plus:

正如官方所说，mybatis-plus在mybatis的基础上只做增强不做改变，因此其与spring的整合亦非常简单。只需把mybatis的依赖换成mybatis-plus的依赖，再把sqlSessionFactory换成mybatis-plus的即可。接下来看具体操作：

####  **1、pom.xml:**

 核心依赖如下：

```xml
        <!-- spring -->
        <dependency>
            <groupId>org.springframework</groupId>
            <artifactId>spring-context</artifactId>
            <version>4.3.14.RELEASE</version>
        </dependency>
        <dependency>
            <groupId>org.springframework</groupId>
            <artifactId>spring-orm</artifactId>
            <version>4.3.14.RELEASE</version>
        </dependency>
        <dependency>
            <groupId>org.springframework</groupId>
            <artifactId>spring-test</artifactId>
            <version>4.3.14.RELEASE</version>
            <scope>test</scope>
        </dependency>
        <!-- mp 依赖 -->
        <dependency>
            <groupId>com.baomidou</groupId>
            <artifactId>mybatis-plus</artifactId>
            <version>2.3</version>
        </dependency>
```

**注意：**这些是核心依赖，本项目还用到了mysql驱动、c3p0、日志（slf4j-api，slf4j-log4j2）、lombok。集成mybatis-plus要把mybatis、mybatis-spring去掉，避免冲突；lombok是一个工具，添加了这个依赖，开发工具再安装Lombok插件，就可以使用它了，最常用的用法就是在实体类中使用它的@Data注解，这样实体类就不用写set、get、toString等方法了。关于Lombok的更多用法，请自行百度。

#### **2、log4j.xml:**

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE log4j:configuration SYSTEM "log4j.dtd">
<log4j:configuration xmlns:log4j="http://jakarta.apache.org/log4j/">
    <appender name="STDOUT" class="org.apache.log4j.ConsoleAppender">
        <param name="Encoding" value="UTF-8" />
        <layout class="org.apache.log4j.PatternLayout">
            <param name="ConversionPattern" value="%-5p %d{MM-dd
HH:mm:ss,SSS} %m (%F:%L) \n" />
        </layout>
    </appender>
    <logger name="java.sql">
        <level value="debug" />
    </logger>
    <logger name="org.apache.ibatis">
        <level value="info" />
    </logger>
    <root>
        <level value="debug" />
        <appender-ref ref="STDOUT" />
    </root>
</log4j:configuration>
```

#### **3、jdbc.properties:**

```ruby
jdbc.driver=com.mysql.jdbc.Driver
jdbc.url=jdbc:mysql:///数据库名?useUnicode=true&characterEncoding=utf8
jdbc.username=#
jdbc.password=#
```

#### **4、mybatis-config.xml:**

```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE configuration
PUBLIC "-//mybatis.org//DTD Config 3.0//EN"
"http://mybatis.org/dtd/mybatis-3-config.dtd">
<configuration>
</configuration>
```

**注：**因为是与spring整合，所有mybatis-plus的大部分都写在spring的配置文件中，这里定义一个空的mybatis-config.xml即可。

#### **5、spring-dao.xml:**

```xml
<?xml version="1.0" encoding="UTF-8"?>    
<beans xmlns="http://www.springframework.org/schema/beans"    
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"   
    xmlns:p="http://www.springframework.org/schema/p"  
    xmlns:aop="http://www.springframework.org/schema/aop"   
    xmlns:context="http://www.springframework.org/schema/context"  
    xmlns:jee="http://www.springframework.org/schema/jee"  
    xmlns:tx="http://www.springframework.org/schema/tx"  
    xsi:schemaLocation="    
        http://www.springframework.org/schema/aop http://www.springframework.org/schema/aop/spring-aop-4.0.xsd  
        http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans-4.0.xsd  
        http://www.springframework.org/schema/context http://www.springframework.org/schema/context/spring-context-4.0.xsd  
        http://www.springframework.org/schema/jee http://www.springframework.org/schema/jee/spring-jee-4.0.xsd  
        http://www.springframework.org/schema/tx http://www.springframework.org/schema/tx/spring-tx-4.0.xsd">    
        
    <!-- 配置整合mybatis-plus过程 -->
    <!-- 1、配置数据库相关参数properties的属性：${url} -->
    <context:property-placeholder location="classpath:jdbc.properties" />
    <!-- 2、配置数据库连接池 -->
    <bean id="dataSource" class="com.mchange.v2.c3p0.ComboPooledDataSource">
        <property name="driverClass" value="${jdbc.driver}"/>
        <property name="jdbcUrl" value="${jdbc.url}"/>
        <property name="user" value="${jdbc.username}"/>
        <property name="password" value="${jdbc.password}"/>
    </bean>
    <!-- mybatis的sqlsessionFactorybean：org.mybatis.spring.SqlSessionFactoryBean-->
    <!-- 3、配置mybatis-plus的sqlSessionFactory -->
    <bean id="sqlSessionFactory" class="com.baomidou.mybatisplus.spring.MybatisSqlSessionFactoryBean">
        <property name="dataSource" ref="dataSource" />
        <property name="configLocation" value="classpath:mybatis-config.xml"/>
        <property name="typeAliasesPackage" value="com.zhu.mybatisplus.entity"/>
    </bean>
    <!-- 4、DAO接口所在包名，Spring会自动查找其下的类 -->
    <bean class="org.mybatis.spring.mapper.MapperScannerConfigurer">
        <property name="basePackage" value="com.zhu.mybatisplus.dao" />
        <property name="sqlSessionFactoryBeanName" value="sqlSessionFactory"/>
    </bean> 
</beans>
```

#### **6、entity:**

```kotlin
@Data
@TableName(value = "tb_employee")//指定表名
public class Employee {
    //value与数据库主键列名一致，若实体类属性名与表主键列名一致可省略value
    @TableId(value = "id",type = IdType.AUTO)//指定自增策略
    private Integer id;
    //若没有开启驼峰命名，或者表中列名不符合驼峰规则，可通过该注解指定数据库表中的列名，exist标明数据表中有没有对应列
    @TableField(value = "last_name",exist = true)
    private String lastName;
    private String email;
    private Integer gender;
    private Integer age;
}
```

#### **7、mapper:**

```java
public interface EmplopyeeDao extends BaseMapper<Employee> {
}
```

这样就完成了mybatis-plus与spring的整合。首先是把mybatis和mybatis-spring依赖换成mybatis-plus的依赖，然后把sqlsessionfactory换成mybatis-plus的，然后实体类中添加`@TableName`、`@TableId`等注解，最后mapper继承`BaseMapper`即可。

#### **8、测试：**

```java
@RunWith(SpringJUnit4ClassRunner.class)
@ContextConfiguration({"classpath:spring/spring-dao.xml"})
public class test {
    @Autowired
    private DataSource dataSource;
    @Test
    public void testDataSource() throws SQLException {
        System.out.println(dataSource.getConnection());
    }
}
```

运行该junit，可输出获取到的连接，说明整合没问题：

### 三、mp的通用crud

- **需求：**
  - 存在一张 tb_employee 表，且已有对应的实体类 Employee，实现tb_employee 表的 CRUD 操作我们需要做什么呢？
- **基于 Mybatis：**
  - 需要编写 EmployeeMapper 接口，并在 EmployeeMapper.xml 映射文件中手动编写 CRUD 方法对应的sql语句。
- **基于 MP：**
  - 只需要创建 EmployeeMapper 接口, 并继承 BaseMapper 接口
  - 我们已经有了Employee、tb_employee了，并且EmployeeDao也继承了BaseMapper了，接下来就使用crud方法

#### **1、insert操作：**

```java
@RunWith(SpringJUnit4ClassRunner.class)
@ContextConfiguration({"classpath:spring/spring-dao.xml"})
public class test {
    @Autowired
    private EmplopyeeDao emplopyeeDao;
    @Test
    public void testInsert(){
        Employee employee = new Employee();
        employee.setLastName("东方不败");
        employee.setEmail("dfbb@163.com");
        employee.setGender(1);
        employee.setAge(20);
        emplopyeeDao.insert(employee);
        //mybatisplus会自动把当前插入对象在数据库中的id写回到该实体中
        System.out.println(employee.getId());
    }
}
```

执行添加操作，直接调用insert方法传入实体即可。

#### **2、update操作：**

```java
@Test
public void testUpdate(){
        Employee employee = new Employee();
        employee.setId(1);
        employee.setLastName("更新测试");
        //emplopyeeDao.updateById(employee);//根据id进行更新，没有传值的属性就不会更新
        emplopyeeDao.updateAllColumnById(employee);//根据id进行更新，没传值的属性就更新为null
}
```

**注：**注意这两个update操作的区别，`updateById`方法，没有传值的字段不会进行更新，比如只传入了lastName，那么age、gender等属性就会保留原来的值；`updateAllColumnById`方法，顾名思义，会更新所有的列，没有传值的列会更新为null。

#### **3、select操作：**

**(1)、**根据id查询：

```undefined
Employee employee = emplopyeeDao.selectById(1);
```

**(2)、**根据条件查询一条数据：

```cpp
Employee employeeCondition = new Employee();
employeeCondition.setId(1);
employeeCondition.setLastName("更新测试");
//若是数据库中符合传入的条件的记录有多条，那就不能用这个方法，会报错
Employee employee = emplopyeeDao.selectOne(employeeCondition);
```

**注：**这个方法的sql语句就是`where id = 1 and last_name = 更新测试`，若是符合这个条件的记录不止一条，那么就会报错。

**(3)、**根据查询条件返回多条数据：
 当符合指定条件的记录数有多条时，上面那个方法就会报错，就应该用这个方法。

```dart
Map<String,Object> columnMap = new HashMap<>();
columnMap.put("last_name","东方不败");//写表中的列名
columnMap.put("gender","1");
List<Employee> employees = emplopyeeDao.selectByMap(columnMap);
System.out.println(employees.size());
```

**注：**查询条件用map集合封装，columnMap，写的是数据表中的列名，而非实体类的属性名。比如属性名为lastName，数据表中字段为last_name，这里应该写的是last_name。selectByMap方法返回值用list集合接收。

**(4)、**通过id批量查询：

```csharp
List<Integer> idList = new ArrayList<>();
idList.add(1);
idList.add(2);
idList.add(3);
List<Employee> employees = emplopyeeDao.selectBatchIds(idList);
System.out.println(employees);
```

**注：**把需要查询的id都add到list集合中，然后调用selectBatchIds方法，传入该list集合即可，该方法返回的是对应id的所有记录，所有返回值也是用list接收。

**(5)、**分页查询：

```csharp
List<Employee> employees = emplopyeeDao.selectPage(new Page<>(1,2),null);
System.out.println(employees);
```

**注：**selectPage方法就是分页查询，在page中传入分页信息，后者为null的分页条件，这里先让其为null，讲了条件构造器再说其用法。这个分页其实并不是物理分页，而是内存分页。也就是说，查询的时候并没有limit语句。等配置了分页插件后才可以实现真正的分页。

**4、delete操作：**

**(1)、**根据id删除：

```css
emplopyeeDao.deleteById(1);
```

**(2)、**根据条件删除：

```dart
Map<String,Object> columnMap = new HashMap<>();
columnMap.put("gender",0);
columnMap.put("age",18);
emplopyeeDao.deleteByMap(columnMap);
```

**注：**该方法与selectByMap类似，将条件封装在columnMap中，然后调用deleteByMap方法，传入columnMap即可，返回值是Integer类型，表示影响的行数。

**(3)、**根据id批量删除：

```csharp
 List<Integer> idList = new ArrayList<>();
 idList.add(1);
 idList.add(2);
 emplopyeeDao.deleteBatchIds(idList);
```

**注：**该方法和selectBatchIds类似，把需要删除的记录的id装进idList，然后调用deleteBatchIds，传入idList即可。

### 四、全局策略配置：

通过上面的小案例我们可以发现，实体类需要加@TableName注解指定数据库表名，通过@TableId注解指定id的增长策略。实体类少倒也无所谓，实体类一多的话也麻烦。所以可以在spring-dao.xml的文件中进行全局策略配置。

```xml
<!-- 5、mybatisplus的全局策略配置 -->
<bean id="globalConfiguration" class="com.baomidou.mybatisplus.entity.GlobalConfiguration">
        <!-- 2.3版本后，驼峰命名默认值就是true，所以可不配置 -->
        <!--<property name="dbColumnUnderline" value="true"/>-->
        <!-- 全局主键自增策略，0表示auto -->
        <property name="idType" value="0"/>
        <!-- 全局表前缀配置 -->
        <property name="tablePrefix" value="tb_"/>
</bean>
```

这里配置了还没用，还需要在sqlSessionFactory中注入配置才会生效。如下：

```xml
<!-- 3、配置mybatisplus的sqlSessionFactory -->
<bean id="sqlSessionFactory" class="com.baomidou.mybatisplus.spring.MybatisSqlSessionFactoryBean">
        <property name="dataSource" ref="dataSource" />
        <property name="configLocation" value="classpath:mybatis-config.xml"/>
        <property name="typeAliasesPackage" value="com.zhu.mybatisplus.entity"/>
        <!-- 注入全局配置 -->
        <property name="globalConfig" ref="globalConfiguration"/>
</bean>
```

如此一来，实体类中的@TableName注解和@TableId注解就可以去掉了

### 五、条件构造器(EntityWrapper)：

以上基本的 CRUD 操作，我们仅仅需要继承一个 BaseMapper 即可实现大部分单表 CRUD 操作。BaseMapper 提供了多达 17 个方法供使用, 可以极其方便的实现单一、批量、分页等操作，极大的减少开发负担。但是mybatis-plus的强大不限于此，请看如下需求该如何处理：

- **需求：**
  - 我们需要分页查询 tb_employee 表中，年龄在 18~50 之间性别为男且姓名为 xx 的所有用户，这时候我们该如何实现上述需求呢？
- **使用MyBatis :** 需要在 SQL 映射文件中编写带条件查询的 SQL,并用PageHelper 插件完成分页. 实现以上一个简单的需求，往往需要我们做很多重复单调的工作。
- **使用MP:** 依旧不用编写 SQL 语句，MP 提供了功能强大的条件构造器 ------  EntityWrapper。

**接下来就直接看几个案例体会EntityWrapper的使用。**

#### **1、分页查询年龄在18 - 50且gender为0、姓名为tom的用户：**

```dart
List<Employee> employees = emplopyeeDao.selectPage(new Page<Employee>(1,3),
     new EntityWrapper<Employee>()
        .between("age",18,50)
        .eq("gender",0)
        .eq("last_name","tom")
);
```

**注：**由此案例可知，分页查询和之前一样，new 一个page对象传入分页信息即可。至于分页条件，new 一个EntityWrapper对象，调用该对象的相关方法即可。between方法三个参数，分别是column、value1、value2，该方法表示column的值要在value1和value2之间；eq是equals的简写，该方法两个参数，column和value，表示column的值和value要相等。注意column是数据表对应的字段，而非实体类属性字段。

#### **2、查询gender为0且名字中带有老师、或者邮箱中带有a的用户：**

```dart
List<Employee> employees = emplopyeeDao.selectList(
                new EntityWrapper<Employee>()
               .eq("gender",0)
               .like("last_name","老师")
                //.or()//和or new 区别不大
               .orNew()
               .like("email","a")
);
```

**注：**未说分页查询，所以用selectList即可，用EntityWrapper的like方法进行模糊查询，like方法就是指column的值包含value值，此处like方法就是查询last_name中包含“老师”字样的记录；“或者”用or或者orNew方法表示，这两个方法区别不大，用哪个都可以，可以通过控制台的sql语句自行感受其区别。

#### **3、查询gender为0，根据age排序，简单分页：**

```dart
List<Employee> employees = emplopyeeDao.selectList(
                new EntityWrapper<Employee>()
                .eq("gender",0)
                .orderBy("age")//直接orderby 是升序，asc
                .last("desc limit 1,3")//在sql语句后面追加last里面的内容(改为降序，同时分页)
);
```

**注：**简单分页是指不用page对象进行分页。orderBy方法就是根据传入的column进行升序排序，若要降序，可以使用orderByDesc方法，也可以如案例中所示用last方法；last方法就是将last方法里面的value值追加到sql语句的后面，在该案例中，最后的sql语句就变为`select ······ order by desc limit 1, 3`，追加了`desc limit 1,3`所以可以进行降序排序和分页。

#### **4、分页查询年龄在18 - 50且gender为0、姓名为tom的用户：**

 条件构造器除了EntityWrapper，还有Condition。用Condition来处理一下这个需求：

```dart
 List<Employee> employees = emplopyeeDao.selectPage(
                new Page<Employee>(1,2),
                Condition.create()
                        .between("age",18,50)
                        .eq("gender","0")
 );
```

**注：**Condition和EntityWrapper的区别就是，创建条件构造器时，EntityWrapper是new出来的，而Condition是调create方法创建出来。

#### **5、根据条件更新：**

```java
@Test
public void testEntityWrapperUpdate(){
        Employee employee = new Employee();
        employee.setLastName("苍老师");
        employee.setEmail("cjk@sina.com");
        employee.setGender(0);
        emplopyeeDao.update(employee,
                new EntityWrapper<Employee>()
                .eq("last_name","tom")
                .eq("age",25)
        );
}
```

**注：**该案例表示把last_name为tom，age为25的所有用户的信息更新为employee中设置的信息。

#### **6、根据条件删除：**

```cpp
emplopyeeDao.delete(
        new EntityWrapper<Employee>()
        .eq("last_name","tom")
        .eq("age",16)
);
```

**注：**该案例表示把last_name为tom、age为16的所有用户删除。

### 代码生成工具

```java
package com.zhx.plus_demo2.util;

import com.baomidou.mybatisplus.core.exceptions.MybatisPlusException;
import com.baomidou.mybatisplus.core.toolkit.StringPool;
import com.baomidou.mybatisplus.generator.AutoGenerator;
import com.baomidou.mybatisplus.generator.InjectionConfig;
import com.baomidou.mybatisplus.generator.config.*;
import com.baomidou.mybatisplus.generator.config.po.TableInfo;
import com.baomidou.mybatisplus.generator.config.rules.DateType;
import com.baomidou.mybatisplus.generator.config.rules.NamingStrategy;
import org.apache.commons.lang3.StringUtils;

import java.util.ArrayList;
import java.util.List;
import java.util.Scanner;

// 演示例子，执行 main 方法控制台输入模块表名回车自动生成对应项目目录中
public class MpGenerator {

    public static final String projectPath = System.getProperty("user.dir");

    /**
     * <p>
     * 读取控制台内容
     * </p>
     */
    public static String scanner(String tip) {
        Scanner scanner = new Scanner(System.in);
        System.out.println("请输入" + tip + "：");
        if (scanner.hasNext()) {
            String ipt = scanner.next();
            if (StringUtils.isNotEmpty(ipt)) {
                return ipt;
            }
        }
        throw new MybatisPlusException("请输入正确的" + tip + "！");
    }

    public static void main(String[] args) {
        // 全局配置
        GlobalConfig globalConfig = generateGlobalConfig();
        // 数据源配置
        DataSourceConfig dsc = generateDataSourceConfig();
        // 包设置
        final PackageConfig pc = generatePackageConfig();
        // 自定义设置
        InjectionConfig injectionConfig = generateInjectConfig(pc);
        // 策略配置
        StrategyConfig strategy = generateStrategy();

        // 配置模板
        TemplateConfig templateConfig = new TemplateConfig();

        // 配置自定义输出模板
        //指定自定义模板路径，注意不要带上.ftl/.vm, 会根据使用的模板引擎自动识别
        // templateConfig.setEntity("templates/entity2.java");
        // templateConfig.setService();
        // templateConfig.setController();

        templateConfig.setXml(null);


        // 代码生成器
        AutoGenerator mpg = new AutoGenerator();
        mpg.setGlobalConfig(globalConfig);
        mpg.setDataSource(dsc);
        mpg.setPackageInfo(pc);
        mpg.setStrategy(strategy);
        mpg.setCfg(injectionConfig);
        mpg.setTemplate(templateConfig);

        mpg.execute();
    }

    private static StrategyConfig generateStrategy() {
        // 策略配置
        StrategyConfig strategy = new StrategyConfig();
        // strategy.setCapitalMode(true);// 全局大写命名 ORACLE 注意
        //strategy.setTablePrefix(new String[] { "tlog_", "tsys_" });// 此处可以修改为您的表前缀
        strategy.setNaming(NamingStrategy.underline_to_camel);// 表名生成策略
        strategy.setInclude(scanner("表名").split(",")); // 需要生成的表
        strategy.setEntityLombokModel(true);
        strategy.setEntityBuilderModel(true);
        // strategy.setExclude(new String[]{"test"}); // 排除生成的表
        // 自定义实体父类
        // strategy.setSuperEntityClass("com.baomidou.demo.TestEntity");
        // 自定义实体，公共字段
        // strategy.setSuperEntityColumns(new String[] { "test_id", "age" });
        // 自定义 mapper 父类
        // strategy.setSuperMapperClass("com.baomidou.demo.TestMapper");
        // 自定义 service 父类
        // strategy.setSuperServiceClass("com.baomidou.demo.TestService");
        // 自定义 service 实现类父类
        // strategy.setSuperServiceImplClass("com.baomidou.demo.TestServiceImpl");
        // 自定义 controller 父类
        // strategy.setSuperControllerClass("com.baomidou.demo.TestController");
        // 【实体】是否生成字段常量（默认 false）
        // public static final String ID = "test_id";
        // strategy.setEntityColumnConstant(true);
        return strategy;
    }

    private static InjectionConfig generateInjectConfig(PackageConfig pc) {
        // 自定义配置
        InjectionConfig cfg = new InjectionConfig() {
            @Override
            public void initMap() {
                // to do nothing
            }
        };

        // 如果模板引擎是 freemarker
//        String templatePath = "/templates/mapper.xml.ftl";
        // 如果模板引擎是 velocity
        String templatePath = "/templates/mapper.xml.vm";
        // 自定义输出配置
        List<FileOutConfig> focList = new ArrayList<>();
        // 自定义配置会被优先输出
        focList.add(new FileOutConfig(templatePath) {
            @Override
            public String outputFile(TableInfo tableInfo) {
                // 自定义输出文件名 ， 如果你 Entity 设置了前后缀、此处注意 xml 的名称会跟着发生变化！！
                return projectPath + "/src/main/resources/mapper/" +
                        "/" + tableInfo.getEntityName() + "Mapper" + StringPool.DOT_XML;
            }
        });
        /*
        cfg.setFileCreate(new IFileCreate() {
            @Override
            public boolean isCreate(ConfigBuilder configBuilder, FileType fileType, String filePath) {
                // 判断自定义文件夹是否需要创建
                checkDir("调用默认方法创建的目录");
                return false;
            }
        });
        */
        cfg.setFileOutConfigList(focList);
        return cfg;
    }

    private static PackageConfig generatePackageConfig() {
        final PackageConfig pc = new PackageConfig();
//        pc.setModuleName(scanner("模块名"));
        // 包配置
//        final PackageConfig pc = new PackageConfig();
//        pc.setModuleName(scanner("模块名"));
//        pc.setParent("com.nfsq.es.demo.bean");

        pc.setParent("cn.ng.springboot");
        pc.setMapper("dao.mapper");//生成mapper接口
        pc.setEntity("dao.entity");
        pc.setServiceImpl("service.impl");
        pc.setService("service");
        pc.setXml(null);
        return pc;
    }

    private static DataSourceConfig generateDataSourceConfig() {
        DataSourceConfig dataSource = new DataSourceConfig();
//        dsc.setDbType(DbType.MYSQL);
        dataSource.setDriverName("com.mysql.jdbc.Driver");
        dataSource.setUsername("root");
        dataSource.setPassword("root");
        dataSource.setUrl("jdbc:mysql://localhost:3306/hive");

//        // 数据源配置
//        DataSourceConfig dsc = new DataSourceConfig();
//        dsc.setUrl("jdbc:mysql://localhost:3306/es_demo?useUnicode=true&useSSL=false&characterEncoding=utf8");
//        // dsc.setSchemaName("public");
//        dsc.setDriverName("com.mysql.jdbc.Driver");
//        dsc.setUsername("root");
//        dsc.setPassword("aptx1230");
        return dataSource;
    }

    private static GlobalConfig generateGlobalConfig() {
        // 全局配置
        GlobalConfig gc = new GlobalConfig();
        gc.setOutputDir(projectPath + "/src/main/java");
        gc.setAuthor("ng");
        gc.setOpen(false);
        gc.setFileOverride(true);
        gc.setActiveRecord(true);// 不需要ActiveRecord特性的请改为false
        gc.setEnableCache(false);// XML 二级缓存
        gc.setBaseResultMap(true);// XML ResultMap
        gc.setBaseColumnList(false);// XML columList
        gc.setKotlin(false); // 是否生成 kotlin 代码
        gc.setDateType(DateType.ONLY_DATE); //时间类型的映射规则

        return gc;
    }

}
```

