# Spring Boot

#### 什么是 Spring Boot

- 它使用 “习惯优于配置” （项目中存在大量的配置，此外还内置一个习惯性的配置）的理念让你的项目快速运行起来
- 它并不是什么新的框架，而是默认配置了很多框架的使用方式，就像 Maven 整合了所有的 jar 包一样，Spring Boot 整合了所有框架
- 使用 Spring Boot 来开发项目则只需要非常少的几个配置就可以搭建起来一个 Web 项目，并且利用 IDEA 可以自动生成生成

#### 项目结构

- SpringbootApplication： 一个带有 main() 方法的类，用于启动应用程序
- SpringbootApplicationTests：一个空的 Junit 测试了，它加载了一个使用 Spring Boot 字典配置功能的 Spring 应用程序上下文
- application.properties：一个空的 properties 文件，可以根据需要添加配置属性
- pom.xml： Maven 构建说明文件

## 项目解析

### pom.xml解析

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>

    <groupId>cn.wmyskxz</groupId>
    <artifactId>springboot</artifactId>
    <version>0.0.1-SNAPSHOT</version>
    <packaging>jar</packaging>

    <name>springboot</name>
    <description>Demo project for Spring Boot</description>

    <parent>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-parent</artifactId>
        <version>2.0.1.RELEASE</version>
        <relativePath/> <!-- lookup parent from repository -->
    </parent>

    <properties>
        <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
        <project.reporting.outputEncoding>UTF-8</project.reporting.outputEncoding>
        <java.version>1.8</java.version>
    </properties>

    <dependencies>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>

        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-test</artifactId>
            <scope>test</scope>
        </dependency>
    </dependencies>

    <build>
        <plugins>
            <plugin>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-maven-plugin</artifactId>
            </plugin>
        </plugins>
    </build>
</project>
```

##### \<parent>标签

- 配置 Spring Boot 的父级依赖

```xml
<parent>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-parent</artifactId>
    <version>2.0.1.RELEASE</version>
    <relativePath/> <!-- lookup parent from repository -->
</parent>
```

- 有了这个，当前的项目才是 Spring Boot 项目，spring-boot-starter-parent 是一个特殊的 starter ，它用来提供相关的 Maven 默认依赖，**使用它之后，常用的包依赖就可以省去 version 标签**

#### 应用入口类

- Spring Boot 项目通常有一个名为 *Application 的入口类，入口类里有一个 main 方法， **这个 main 方法其实就是一个标准的 Javay 应用的入口方法**
- **@SpringBootApplication** 是 Spring Boot 的核心注解，它是一个组合注解，该注解组合了：**@Configuration、@EnableAutoConfiguration、@ComponentScan；** 若不是用 @SpringBootApplication 注解也可以使用这三个注解代替。
  - 其中，**@EnableAutoConfiguration 让 Spring Boot 根据类路径中的 jar 包依赖为当前项目进行自动配置**，例如，添加了 spring-boot-starter-web 依赖，会自动添加 Tomcat 和 Spring MVC 的依赖，那么 Spring Boot 会对 Tomcat 和 Spring MVC 进行自动配置。
  - **Spring Boot 还会自动扫描 @SpringBootApplication 所在类的同级包以及下级包里的 Bean** ，所以入口类建议就配置在 grounpID + arctifactID 组合的包名下（这里为 cn.ng.springboot 包）

### Spring Boot 配置文件

- Spring Boot 使用一个全局的配置文件 application.properties 或 application.yml，放置在【src/main/resources】目录或者类路径的 /config 下
- Spring Boot 不仅支持常规的 properties 配置文件，还支持 yaml 语言的配置文件。yaml 是以数据为中心的语言，在配置数据的时候具有面向对象的特征
  - 可以在一个单个文件中通过使用`spring.profiles`属性来定义多个不同的环境配置
  - 例如下面的内容，在指定为test环境时，`server.port`将使用8882端口；而在prod环境，`server.port`将使用8883端口；如果没有指定环境，`server.port`将使用8881端口

```yaml
server:
    port: 8881
---
spring:
    profiles: test
server:
    port: 8882
---
spring:
    profiles: prod
server:
    port: 8883
```

- Spring Boot 的全局配置文件的作用是对一些默认配置的配置值进行修改

##### 例 修改配置

- 删除.properties文件，使用yml文件进行配置

```yaml
server:
  port: 8080
name: 牛耕
age: 20
content: "name:${name},age:${age}"
```

- 使用@Value来获取配置属性
  - 我们并没有在 yml 文件中注明属性的类型，而是在使用的时候定义的

```java
@RestController
public class HelloController {
    @Value("${name}")
    private String name;

    @Value("${age}")
    private Integer age;
  
    @Value("${content}")
    private String content;
    
    @RequestMapping("/hello")
    public String hello() {
        return name + age;
      	//return content;
    }
}
```

- **问题：** 这样写配置文件繁琐而且可能会造成类的臃肿，因为有许许多多的 @Value 注解

##### 封装配置信息

```java
@Component
@ConfigurationProperties(prefix = "student")
public class StudentProperties {
    private String name;

    private Integer age;

    public void setName(String name){ this.name = name; }

    public void setAge(Integer age){ this.age = age; }

    public String getName(){ return name; }

    public Integer getAge(){ return age; }
}
```

我们可以把配置信息封装成一个类，首先在我们的 name 和 age 前加一个 student 前缀，然后新建一个 StudentProperties 的类用来封装这些信息，并用上两个注解：

- @Component：表明当前类是一个 Java Bean
- @ConfigurationProperties(prefix = "student")：表示获取前缀为 sutdent 的配置信息

```java
@RestController
public class HelloController {
    private StudentProperties studentProperties;

    @RequestMapping("/hello")
    public String hello() {
        return studentProperties.getName() + studentProperties.getAge();
    }
}
```

```yaml
server:
  port: 8080
student:
  name: 牛耕
  age: 20
```

#### 热部署

- pom.xml添加依赖

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-devtools</artifactId>
    <optional>true</optional> <!-- 这个需要为 true 热部署才有效 -->
</dependency>
```

#### 支持JSP

- Spring Boot 的默认视图支持是 Thymeleaf 模板引擎

##### 修改pom.xml增加对jsp的支持

```xml
<!-- servlet依赖. -->
<dependency>
    <groupId>javax.servlet</groupId>
    <artifactId>javax.servlet-api</artifactId>
    <scope>provided</scope>
</dependency>
<dependency>
    <groupId>javax.servlet</groupId>
    <artifactId>jstl</artifactId>
</dependency>

<!-- tomcat的支持.-->
<dependency>
    <groupId>org.apache.tomcat.embed</groupId>
    <artifactId>tomcat-embed-jasper</artifactId>
    <scope>provided</scope>
</dependency>
```

##### 配置重定向 JSP 文件的位置

- 修改application.yml文件

```yaml
spring:
  mvc:
    view:
      prefix: /WEB-INF/views/
      suffix: .jsp
```

##### 修改HelloController

```java
@Controller
public class HelloController {
    @Autowired
    private StudentProperties studentProperties;

    @RequestMapping("/hello")
    public String hello(Model model) {
        model.addAttribute("now", DateFormat.getDateTimeInstance().format(new Date()));
        return "hello";
    }
}
```

##### 新建 hello.jsp 文件

```jsp
<%@ page language="java" contentType="text/html; charset=UTF-8"
         pageEncoding="UTF-8" %>
Hi JSP. 现在时间是 ${now}
```

### 集成mybatis

#### 修改pom.xml

```xml
 <!-- mybatis -->
<dependency>
    <groupId>org.mybatis.spring.boot</groupId>
    <artifactId>mybatis-spring-boot-starter</artifactId>
    <version>1.1.1</version>
</dependency>
<!-- mysql -->
<dependency>
    <groupId>mysql</groupId>
    <artifactId>mysql-connector-java</artifactId>
    <version>5.1.21</version>
</dependency>
```

#### 新增数据库链接参数

```yaml
spring:
  mvc:
    view:
      prefix: /WEB-INF/views/
      suffix: .jsp
    datasource:
      url: jdbc:mysql://127.0.0.1:3306/hive?characterEncoding=UTF-8
      username: root
      password: root
      driver-class-name: com.mysql.jdbc.Driver
    jpa:
      hibernate:
        ddl-auto: update #新增数据库链接必备的参数
```

#### 创建实体类

#### 创建映射接口

```java
package cn.ng.springboot.mapper;

import cn.ng.springboot.pojo.User;
import org.apache.ibatis.annotations.Mapper;
import org.apache.ibatis.annotations.Select;

import java.util.List;

@Mapper
public interface UserMapper {

    @Select("SELECT * FROM user")
    List<Student> findAll();
}
```

#### 创建UserController

```java
package cn.ng.springboot.controller;

import cn.ng.springboot.mapper.UserMapper;
import cn.ng.springboot.pojo.User;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Controller;
import org.springframework.ui.Model;
import org.springframework.web.bind.annotation.RequestMapping;

import java.util.List;

@Controller
public class UserController {

    @Autowired
    UserMapper userMapper;

    @RequestMapping("/listUser")
    public String listUser(Model model) {
        List<User> users = UserMapper.findAll();
        model.addAttribute("users", users);
        return "listUser";
    }
}
```

## YAML配置文件详解

- **YAML目前还有一些不足，它无法通过`@PropertySource`注解来加载配置。但是，YAML加载属性到内存中保存的时候是有序的，所以当配置文件中的信息需要具备顺序含义时，YAML的配置方式比起properties配置文件更有优势**

### 自定义参数

我们除了可以在Spring Boot的配置文件中设置各个Starter模块中预定义的配置属性，也可以在配置文件中定义一些我们需要的自定义属性。比如在`application.properties`中添加：

```properties
book.name=SpringCloudInAction
book.author=ZhaiYongchao
```

然后，在应用中我们可以通过`@Value`注解来加载这些自定义的参数，比如：

```java
@Component
public class Book {

    @Value("${book.name}")
    private String name;
    @Value("${book.author}")
    private String author;

    // 省略getter和setter
}
```

`@Value`注解加载属性值的时候可以支持两种表达式来进行配置：

- 一种是我们上面介绍的PlaceHolder方式，格式为 `${...}`，大括号内为PlaceHolder
- 另外还可以使用SpEL表达式（Spring Expression Language）， 格式为 `#{...}`，大括号内为SpEL表达式

### 参数引用

在`application.properties`中的各个参数之间，我们也可以直接通过使用PlaceHolder的方式来进行引用，就像下面的设置：

```properties
book.name=SpringCloud
book.author=ZhaiYongchao
book.desc=${book.author}  is writing《${book.name}》
```

`book.desc`参数引用了上文中定义的`book.name`和`book.author`属性，最后该属性的值就是`ZhaiYongchao is writing《SpringCloud》`。

### 使用随机数

在一些特殊情况下，有些参数我们希望它每次加载的时候不是一个固定的值，比如：密钥、服务端口等。在Spring Boot的属性配置文件中，我们可以通过使用`${random}`配置来产生随机的int值、long值或者string字符串，这样我们就可以容易的通过配置来属性的随机生成，而不是在程序中通过编码来实现这些逻辑。

`${random}`的配置方式主要有一下几种，读者可作为参考使用。

```properties
# 随机字符串
com.didispace.blog.value=${random.value}
# 随机int
com.didispace.blog.number=${random.int}
# 随机long
com.didispace.blog.bignumber=${random.long}
# 10以内的随机数
com.didispace.blog.test1=${random.int(10)}
# 10-20的随机数
com.didispace.blog.test2=${random.int[10,20]}
```

**该配置方式可以用于设置应用端口等场景，避免在本地调试时出现端口冲突的麻烦**

### 命令行参数

回顾一下在本章的快速入门中，我们还介绍了如何启动Spring Boot应用，其中提到了使用命令`java -jar`命令来启动的方式。该命令除了启动应用之外，还可以在命令行中来指定应用的参数，比如：`java -jar xxx.jar --server.port=8888`，直接以命令行的方式，来设置server.port属性，另启动应用的端口设为8888。

在命令行方式启动Spring Boot应用时，连续的两个减号`--`就是对`application.properties`中的属性值进行赋值的标识。所以，`java -jar xxx.jar --server.port=8888`命令，等价于我们在`application.properties`中添加属性`server.port=8888`。

通过命令行来修改属性值是Spring Boot非常重要的一个特性，通过此特性，理论上已经使得我们应用的属性在启动前是可变的，所以其中端口号也好、数据库连接也好，都是可以在应用启动时发生改变，而不同于以往的Spring应用通过Maven的Profile在编译器进行不同环境的构建。其最大的区别就是，Spring Boot的这种方式，可以让应用程序的打包内容，贯穿开发、测试以及线上部署，而Maven不同Profile的方案每个环境所构建的包，其内容本质上是不同的。但是，如果每个参数都需要通过命令行来指定，这显然也不是一个好的方案，所以下面我们看看如果在Spring Boot中实现多环境的配置。

### 多环境配置

我们在开发任何应用的时候，通常同一套程序会被应用和安装到几个不同的环境，比如：开发、测试、生产等。其中每个环境的数据库地址、服务器端口等等配置都会不同，如果在为不同环境打包时都要频繁修改配置文件的话，那必将是个非常繁琐且容易发生错误的事。

对于多环境的配置，各种项目构建工具或是框架的基本思路是一致的，通过配置多份不同环境的配置文件，再通过打包命令指定需要打包的内容之后进行区分打包，Spring Boot也不例外，或者说更加简单。

在Spring Boot中多环境配置文件名需要满足`application-{profile}.properties`的格式，其中`{profile}`对应你的环境标识，比如：

- `application-dev.properties`：开发环境
- `application-test.properties`：测试环境
- `application-prod.properties`：生产环境

至于哪个具体的配置文件会被加载，需要在`application.properties`文件中通过`spring.profiles.active`属性来设置，其值对应配置文件中的`{profile}`值。如：`spring.profiles.active=test`就会加载`application-test.properties`配置文件内容。

下面，以不同环境配置不同的服务端口为例，进行样例实验。

- 针对各环境新建不同的配置文件`application-dev.properties`、`application-test.properties`、`application-prod.properties`
- 在这三个文件均都设置不同的`server.port`属性，如：dev环境设置为1111，test环境设置为2222，prod环境设置为3333
- application.properties中设置`spring.profiles.active=dev`，就是说默认以dev环境设置
- 测试不同配置的加载
- 执行`java -jar xxx.jar`，可以观察到服务端口被设置为`1111`，也就是默认的开发环境（dev）
- 执行`java -jar xxx.jar --spring.profiles.active=test`，可以观察到服务端口被设置为`2222`，也就是测试环境的配置（test）
- 执行`java -jar xxx.jar --spring.profiles.active=prod`，可以观察到服务端口被设置为`3333`，也就是生产环境的配置（prod）

按照上面的实验，可以如下总结多环境的配置思路：

- `application.properties`中配置通用内容，并设置`spring.profiles.active=dev`，以开发环境为默认配置
- `application-{profile}.properties`中配置各个环境不同的内容
- 通过命令行方式去激活不同环境的配置

### 加载顺序

在上面的例子中，我们将Spring Boot应用需要的配置内容都放在了项目工程中，虽然我们已经能够通过`spring.profiles.active`或是通过Maven来实现多环境的支持。但是，当我们的团队逐渐壮大，分工越来越细致之后，往往我们不需要让开发人员知道测试或是生成环境的细节，而是希望由每个环境各自的负责人（QA或是运维）来集中维护这些信息。那么如果还是以这样的方式存储配置内容，对于不同环境配置的修改就不得不去获取工程内容来修改这些配置内容，当应用非常多的时候就变得非常不方便。同时，配置内容都对开发人员可见，本身这也是一种安全隐患。对此，现在出现了很多将配置内容外部化的框架和工具，后续将要介绍的Spring Cloud Config就是其中之一，为了后续能更好的理解Spring Cloud Config的加载机制，我们需要对Spring Boot对数据文件的加载机制有一定的了解。

Spring Boot为了能够更合理的重写各属性的值，使用了下面这种较为特别的属性加载顺序：

1. 命令行中传入的参数。
2. `SPRING_APPLICATION_JSON`中的属性。`SPRING_APPLICATION_JSON`是以JSON格式配置在系统环境变量中的内容。
3. `java:comp/env`中的`JNDI`属性。
4. Java的系统属性，可以通过`System.getProperties()`获得的内容。
5. 操作系统的环境变量
6. 通过`random.*`配置的随机属性
7. 位于当前应用jar包之外，针对不同`{profile}`环境的配置文件内容，例如：`application-{profile}.properties`或是`YAML`定义的配置文件
8. 位于当前应用jar包之内，针对不同`{profile}`环境的配置文件内容，例如：`application-{profile}.properties`或是`YAML`定义的配置文件
9. 位于当前应用jar包之外的`application.properties`和`YAML`配置内容
10. 位于当前应用jar包之内的`application.properties`和`YAML`配置内容
11. 在`@Configuration`注解修改的类中，通过`@PropertySource`注解定义的属性
12. 应用默认属性，使用`SpringApplication.setDefaultProperties`定义的内容

**优先级按上面的顺序有高到低，数字越小优先级越高。**

可以看到，其中第7项和第9项都是从应用jar包之外读取配置文件，所以，实现外部化配置的原理就是从此切入，为其指定外部配置文件的加载位置来取代jar包之内的配置内容。通过这样的实现，我们的工程在配置中就变的非常干净，我们只需要在本地放置开发需要的配置即可，而其他环境的配置就可以不用关心，由其对应环境的负责人去维护即可。

## 2.x 新特性

在Spring Boot 2.0中推出了Relaxed Binding 2.0，对原有的属性绑定功能做了非常多的改进以帮助我们更容易的在Spring应用中加载和读取配置信息。下面本文就来说说Spring Boot 2.0中对配置的改进。

### 配置文件绑定

#### 简单类型

在Spring Boot 2.0中对配置属性加载的时候会除了像1.x版本时候那样**移除特殊字符**外，还会将配置均以**全小写**的方式进行匹配和加载。所以，下面的4种配置方式都是等价的：

- properties格式：

```properties
spring.jpa.databaseplatform=mysql
spring.jpa.database-platform=mysql
spring.jpa.databasePlatform=mysql
spring.JPA.database_platform=mysql
```

- yaml格式：

```yaml
spring:
  jpa:
    databaseplatform: mysql
    database-platform: mysql
    databasePlatform: mysql
    database_platform: mysql
```

**Tips：推荐使用全小写配合`-`分隔符的方式来配置，比如：`spring.jpa.database-platform=mysql`**

#### List类型

在properties文件中使用`[]`来定位列表类型，比如：

```properties
spring.my-example.url[0]=http://example.com
spring.my-example.url[1]=http://spring.io
```

也支持使用**逗号**分割的配置方式，上面与下面的配置是等价的：

```properties
spring.my-example.url=http://example.com,http://spring.io
```

而在yaml文件中使用可以使用如下配置：

```yaml
spring:
  my-example:
    url:
      - http://example.com
      - http://spring.io
```

也支持**逗号**分割的方式：

```yaml
spring:
  my-example:
    url: http://example.com, http://spring.io
```

**注意：在Spring Boot 2.0中对于List类型的配置必须是连续的，不然会抛出`UnboundConfigurationPropertiesException`异常，所以如下配置是不允许的：**

```properties
foo[0]=a
foo[2]=b
```

**在Spring Boot 1.x中上述配置是可以的，`foo[1]`由于没有配置，它的值会是`null`**

#### Map类型

Map类型在properties和yaml中的标准配置方式如下：

- properties格式：

```properties
spring.my-example.foo=bar
spring.my-example.hello=world
```

- yaml格式：

```yaml
spring:
  my-example:
    foo: bar
    hello: world
```

**注意：如果Map类型的key包含非字母数字和`-`的字符，需要用`[]`括起来，比如：**

```yaml
spring:
  my-example:
    '[foo.baz]': bar
```

### 环境属性绑定

**简单类型**

在环境变量中通过小写转换与`.`替换`_`来映射配置文件中的内容，比如：环境变量`SPRING_JPA_DATABASEPLATFORM=mysql`的配置会产生与在配置文件中设置`spring.jpa.databaseplatform=mysql`一样的效果。

**List类型**

由于环境变量中无法使用`[`和`]`符号，所以使用`_`来替代。任何由下划线包围的数字都会被认为是`[]`的数组形式。比如：

```properties
MY_FOO_1_ = my.foo[1]
MY_FOO_1_BAR = my.foo[1].bar
MY_FOO_1_2_ = my.foo[1][2]
```

另外，最后环境变量最后是以数字和下划线结尾的话，最后的下划线可以省略，比如上面例子中的第一条和第三条等价于下面的配置：

```properties
MY_FOO_1 = my.foo[1]
MY_FOO_1_2 = my.foo[1][2]
```

### 系统属性绑定

**简单类型**

系统属性与文件配置中的类似，都以移除特殊字符并转化小写后实现绑定，比如下面的命令行参数都会实现配置`spring.jpa.databaseplatform=mysql`的效果：

```properties
-Dspring.jpa.database-platform=mysql
-Dspring.jpa.databasePlatform=mysql
-Dspring.JPA.database_platform=mysql
```

**List类型**

系统属性的绑定也与文件属性的绑定类似，通过`[]`来标示，比如：

```properties
-D"spring.my-example.url[0]=http://example.com"
-D"spring.my-example.url[1]=http://spring.io"
```

同样的，他也支持逗号分割的方式，比如：

```properties
-Dspring.my-example.url=http://example.com,http://spring.io
```

### 属性的读取

上文介绍了Spring Boot 2.0中对属性绑定的内容，可以看到对于一个属性我们可以有多种不同的表达，但是如果我们要在Spring应用程序的environment中读取属性的时候，每个属性的唯一名称符合如下规则：

- 通过`.`分离各个元素
- 最后一个`.`将前缀与属性名称分开
- 必须是字母（a-z）和数字(0-9)
- 必须是小写字母
- 用连字符`-`来分隔单词
- 唯一允许的其他字符是`[`和`]`，用于List的索引
- 不能以数字开头

所以，如果我们要读取配置文件中`spring.jpa.database-platform`的配置，可以这样写：

```java
this.environment.containsProperty("spring.jpa.database-platform")
```

而下面的方式是无法获取到`spring.jpa.database-platform`配置内容的：

```java
this.environment.containsProperty("spring.jpa.databasePlatform")
```

**注意：使用`@Value`获取配置内容的时候也需要这样的特点**

### 全新的绑定API

在Spring Boot 2.0中增加了新的绑定API来帮助我们更容易的获取配置信息。下面举个例子来帮助大家更容易的理解：

**例子一：简单类型**

假设在properties配置中有这样一个配置：`com.didispace.foo=bar`

我们为它创建对应的配置类：

```java
@Data
@ConfigurationProperties(prefix = "com.ng")
public class FooProperties {

    private String foo;

}
```

接下来，通过最新的`Binder`就可以这样来拿配置信息了：

```java
@SpringBootApplication
public class Application {

    public static void main(String[] args) {
        ApplicationContext context = SpringApplication.run(Application.class, args);

        Binder binder = Binder.get(context.getEnvironment());

        FooProperties foo = binder.bind("com.ng", Bindable.of(FooProperties.class)).get();
        System.out.println(foo.getFoo());
    }
}
```

**例子二：List类型**

如果配置内容是List类型呢？比如：

```properties
com.didispace.post[0]=Why Spring Boot
com.didispace.post[1]=Why Spring Cloud

com.didispace.posts[0].title=Why Spring Boot
com.didispace.posts[0].content=It is perfect!
com.didispace.posts[1].title=Why Spring Cloud
com.didispace.posts[1].content=It is perfect too!
```

要获取这些配置依然很简单，可以这样实现：

```java
ApplicationContext context = SpringApplication.run(Application.class, args);

Binder binder = Binder.get(context.getEnvironment());

// 绑定List配置
List<String> post = binder.bind("com.didispace.post", Bindable.listOf(String.class)).get();
System.out.println(post);

List<PostInfo> posts = binder.bind("com.didispace.posts", Bindable.listOf(PostInfo.class)).get();
System.out.println(posts);
```

## 工程结构推荐

Spring Boot框架本身并没有对工程结构有特别的要求，但是按照最佳实践的工程结构可以帮助我们减少可能会遇见的坑，尤其是Spring包扫描机制的存在，如果您使用最佳实践的工程结构，可以免去不少特殊的配置工作。

## 典型示例

以下结构是比较推荐的package组织方式：

```
com
  +- example
    +- myproject
      +- Application.java
      |
      +- domain
      |  +- Customer.java
      |  +- CustomerRepository.java
      |
      +- service
      |  +- CustomerService.java
      |
      +- web
      |  +- CustomerController.java
      |
```

- `root package`：`com.example.myproject`，所有的类和其他package都在root package之下。
- 应用主类：`Application.java`，该类直接位于`root package`下。通常我们会在应用主类中做一些框架配置扫描等配置，我们放在root package下可以帮助程序减少手工配置来加载到我们希望被Spring加载的内容
- `com.example.myproject.domain`包：用于定义实体映射关系与数据访问相关的接口和实现
- `com.example.myproject.service`包：用于编写业务逻辑相关的接口与实现
- `com.example.myproject.web`：用于编写Web层相关的实现，比如：Spring MVC的Controller等

上面的结构中，`root package`与应用主类的位置是整个结构的关键。由于应用主类在`root package`中，所以按照上面的规则定义的所有其他类都处于`root package`下的其他子包之后。默认情况下，Spring Boot的应用主类会自动扫描`root package`以及所有子包下的所有类来进行初始化。

什么意思呢？举个例子，假设我们将`com.example.myproject.web`包与上面所述的`root package`：`com.example.myproject`放在同一级，像下面这样：

```
com
  +- example
    +- myproject
      +- Application.java
      |
      +- domain
      |  +- Customer.java
      |  +- CustomerRepository.java
      |
      +- service
      |  +- CustomerService.java
      |
    +- web
    |  +- CustomerController.java
    |
```

这个时候，应用主类`Application.java`在默认情况下就无法扫描到`com.example.myproject.web`中的Controller定义，就无法初始化Controller中定义的接口。

## 非典型结构下的初始化

那么如果，我们一定要加载非`root package`下的内容怎么办呢？

**方法一**：使用`@ComponentScan`注解指定具体的加载包，比如：

```java
@SpringBootApplication
@ComponentScan(basePackages="com.example")
public class Bootstrap {

    public static void main(String[] args) {
        SpringApplication.run(Bootstrap.class, args);
    }

}
```

这种方法通过注解直接指定要扫描的包，比较直观。如果有这样的需求也是可以用的，但是原则上还是推荐以上面的典型结构来定义，这样也可以少写一些注解，代码更加简洁。

**方法二**：使用`@Bean`注解来初始化，比如：

```java
@SpringBootApplication
public class Bootstrap {

    public static void main(String[] args) {
        SpringApplication.run(Bootstrap.class, args);
    }

    @Bean
    public CustomerController customerController() {
        return new CustomerController();
    }

}
```

这种方法在业务开发的时候并不是特别推荐，更适合用于框架封装等场景

## SpringBoot常用注解

#### 1 、 @RestController 和 @RequestMapping 注解

@RestController 注解，它继承自 @Controller 注解。 4.0 之前的版本， Spring MVC 的组件都使用 @Controller 来标识当前类是一个控制器 servlet 。使用这个特性，我们可以开发 REST 服务的时候不需要使用 @Controller 而专门的 @RestController 。

当你实现一个RESTful web services 的时候， response 将一直通过 response body 发送。为了简化开发， Spring 4.0 提供了一个专门版本的 controller 。下面我们来看看 @RestController 实现的定义：

```java
  @Target(value=TYPE)    
  @Retention(value=RUNTIME)    
  @Documented    
  @Controller    
  @ResponseBody    
 public @interface RestController
```

注： @RestController 和 @RequestMapping 注解是 Spring MVC 注解（它们不是 Spring Boot 的特定部分）

#### 2 、 @EnableAutoConfiguration 注解

第二个类级别的注解是 @EnableAutoConfiguration 。这个注解告诉 Spring Boot 根据添加的 jar 依赖猜测你想如何配置 Spring 。由于spring-boot-starter-web 添加了 Tomcat 和 Spring MVC ，所以 auto-configuration 将假定你正在开发一个 web 应用并相应地对 Spring 进行设置。Starter POMs 和 Auto-Configuration ：设计 auto-configuration 的目的是更好的使用 "Starter POMs" ，但这两个概念没有直接的联系。你可以自由地挑选 starter POMs 以外的 jar 依赖，并且 Spring Boot 将仍旧尽最大努力去自动配置你的应用。

你可以通过将 @EnableAutoConfiguration 或 @SpringBootApplication 注解添加到一个 @Configuration 类上来选择自动配置。

注：你只需要添加一个 @EnableAutoConfiguration 注解。我们建议你将它添加到主 @Configuration 类上。

如果发现应用了你不想要的特定自动配置类，你可以使用 @EnableAutoConfiguration 注解的排除属性来禁用它们。

```java
<pre name="code">import org.springframework.boot.autoconfigure.*;  
     import org.springframework.boot.autoconfigure.jdbc.*;  
     import org.springframework.context.annotation.*;  
     @Configuration  
     @EnableAutoConfiguration(exclude={DataSourceAutoConfiguration.class})  
     public class MyConfiguration {  
     }  
```

#### 3 、 @Configuration

Spring Boot 提倡基于 Java 的配置。尽管你可以使用一个 XML 源来调用 SpringApplication.run() ，我们通常建议你使用 @Configuration 类作为主要源。一般定义外汇返佣[http://www.fx61.com/](https://link.zhihu.com/?target=http%3A//www.fx61.com/) 方法的类也是主要 @Configuration 的一个很好候选。你不需要将所有的 @Configuration 放进一个单独的类。 @Import 注解可以用来导入其他配置类。另外，你也可以使用 @ComponentScan 注解自动收集所有的 Spring 组件，包括@Configuration 类。

如果你绝对需要使用基于XML 的配置，我们建议你仍旧从一个 @Configuration 类开始。你可以使用附加的 @ImportResource 注解加载 XML配置文件。

@Configuration 注解该类，等价 与 XML 中配置 beans ；用 @Bean 标注方法等价于 XML 中配置 bean

```java
@ComponentScan(basePackages = "com.hyxt",includeFilters = {@ComponentScan.Filter(Aspect.class)})
```

#### 4 、 @SpringBootApplication

很多Spring Boot 开发者总是使用 @Configuration ， @EnableAutoConfiguration 和 @ComponentScan 注解他们的 main 类。由于这些注解被如此频繁地一块使用（特别是你遵循以上最佳实践时）， Spring Boot 提供一个方便的 @SpringBootApplication 选择。

该 @SpringBootApplication 注解等价于以默认属性使用 @Configuration ， @EnableAutoConfiguration 和 @ComponentScan 。

```java
package com.example.myproject;  
    import org.springframework.boot.SpringApplication;  
    import org.springframework.boot.autoconfigure.SpringBootApplication;  
    @SpringBootApplication // same as @Configuration @EnableAutoConfiguration @ComponentScan  
    public class Application {  
        public static void main(String[] args) {  
            SpringApplication.run(Application.class, args);  
        }  
    }
```

Spring Boot 将尝试校验外部的配置，默认使用 JSR-303 （如果在 classpath 路径中）。你可以轻松的为你的 @ConfigurationProperties 类添加 JSR-303 javax.validation 约束注解：

```java
@Component  
    @ConfigurationProperties(prefix="connection")  
    public class ConnectionSettings {  
    @NotNull  
    private InetAddress remoteAddress;  
    // ... getters and setters  
    }
```

#### 5 、 @ResponseBody

表示该方法的返回结果直接写入HTTP response body 中

一般在异步获取数据时使用，在使用@RequestMapping 后，返回值通常解析为跳转路径，加上

@responsebody 后返回结果不会被解析为跳转路径，而是直接写入 HTTP response body 中。比如

异步获取json 数据，加上 @responsebody 后，会直接返回 json 数据。

#### 6 、 @Component

泛指组件，当组件不好归类的时候，我们可以使用这个注解进行标注。一般公共的方法我会用上这个注解

#### 7 、 @AutoWired

byType 方式。把配置好的 Bean 拿来用，完成属性、方法的组装，它可以对类成员变量、方法及构

造函数进行标注，完成自动装配的工作。

当加上（required=false ）时，就算找不到 bean 也不报错。

#### 8 、 @RequestParam

用在方法的参数前面：

`@RequestParam String a =request.getParameter("abc")`

#### 9 、 @PathVariable

路径变量。

`RequestMapping("user/get/mac/{macAddress}")`

`public String getByMacAddress(@PathVariable String macAddress){`

`//can do something;`

`}`

参数与大括号里的名字一样要相同。

以上注解的示范

```java
/**
 * 用户进行评论及对评论进行管理的 Controller 类；
 */  
@Controller  
@RequestMapping("/msgCenter")  
public class MyCommentController extends BaseController {  
    @Autowired  
    CommentService commentService;  
    @Autowired  
    OperatorService operatorService;  
    /**
     * 添加活动评论；
     *
     * @param applyId 活动 ID ；
     * @param content 评论内容；
     * @return
     */  
    @ResponseBody  
    @RequestMapping("/addComment")  
    public Map<String, Object> addComment(@RequestParam("applyId") Integer applyId, @RequestParam("content") String content) {  
        ....  
        return result;  
    }  
}
```

#### 10 、 @ControllerAdvice

全局处理异常的：

@ControllerAdvice

包含@Component 。可以被扫描到。

统一处理异常。

@ExceptionHandler （ Exception.class ）：

用在方法上面表示遇到这个异常就执行以下方法。

```java
/**
 * 全局异常处理
 */  
@ControllerAdvice  
class GlobalDefaultExceptionHandler {  
    public static final String DEFAULT_ERROR_VIEW = "error";  
    @ExceptionHandler({TypeMismatchException.class,NumberFormatException.class})  
    public ModelAndView formatErrorHandler(HttpServletRequest req, Exception e) throws Exception {  
        ModelAndView mav = new ModelAndView();  
        mav.addObject("error"," 参数类型错误 ");  
        mav.addObject("exception", e);  
        mav.addObject("url", RequestUtils.getCompleteRequestUrl(req));  
        mav.addObject("timestamp", new Date());  
        mav.setViewName(DEFAULT_ERROR_VIEW);  
        return mav;  
}}
```

#### 11 、 @ComponentScan

做过web 开发的同学一定都有用过 @Controller ， @Service ， @Repository 注解，查看其源码你会发现，他们中有一个共同的注解@Component ，没错 @ComponentScan 注解默认就会装配标识了 @Controller ， @Service ， @Repository ， @Component 注解的类到 spring容器中。

```java
@ComponentScan(value = "com.abacus.check.api")
public class CheckApiApplication {
    public static void main(String[] args) {
        SpringApplication.run(CheckApiApplication.class, args);
    }
}
```

@SpringBootApplication 注解也包含了 @ComponentScan 注解，所以在使用中我们也可以通过 @SpringBootApplication 注解的scanBasePackages 属性进行配置。

```java
@SpringBootApplication(scanBasePackages = {"com.abacus.check.api", "com.abacus.check.service"})
public class CheckApiApplication {
    public static void main(String[] args) {
        SpringApplication.run(CheckApiApplication.class, args);
    }
```

}

#### 12 、 @Conditional

@Conditional 是 Spring4 新提供的注解，通过 @Conditional 注解可以根据代码中设置的条件装载不同的 bean ，在设置条件注解之前，先要把装载的 bean 类去实现 Condition 接口，然后对该实现接口的类设置是否装载的条件。 Spring Boot 注解中的 @ConditionalOnProperty 、@ConditionalOnBean 等以 @Conditional* 开头的注解，都是通过集成了 @Conditional 来实现相应功能的。

@Conditional 的定义：

// 此注解可以标注在类和方法上 `@Target({ElementType.TYPE, ElementType.METHOD}) @Retention(RetentionPolicy.RUNTIME) @Documented public @interface Conditional { Class<? extends Condition>[] value(); }`

从代码中可以看到，需要传入一个Class 数组，并且需要继承 Condition 接口：

`public interface Condition { boolean matches(ConditionContext var1, AnnotatedTypeMetadata var2); }`

Condition 是个接口，需要实现 matches 方法，返回 true 则注入 bean ， false 则不注入。

示例：

首先，创建Person 类：

```java
public class Person {
private String name;
private Integer age;
public String getName() { return name; }
 public void setName(String name) { this.name = name; }
public Integer getAge() { return age; }
public void setAge(Integer age) { this.age = age; }
public Person(String name, Integer age) { this.name = name; this.age = age; }
 @Override
public String toString() { return "Person{" + "name='" + name + '\'' + ", age=" + age + '}'; } }
```

创建BeanConfig 类，用于配置两个 Person 实例并注入，一个是比尔盖茨，一个是林纳斯。

```java
@Configuration
public class BeanConfig {
@Bean(name = "bill")
public Person person1(){ return new Person("Bill Gates",62); }
@Bean("linus")
public Person person2(){ return new Person("Linus",48); }
}
```

接着写一个测试类进行验证这两个Bean 是否注入成功。

```java
public class ConditionalTest {
AnnotationConfigApplicationContext applicationContext = new AnnotationConfigApplicationContext(BeanConfig.class);
@Test
public void test1(){
Map<String, Person> map = applicationContext.getBeansOfType(Person.class); System.out.println(map);
    }
}
```

运行，输出结果是这样的，两个 Person 实例被注入进容器。

运行，输出结果是这样的，两个Person 实例被注入进容器。

这是一个简单的例子，现在问题来了，如果我想根据当前操作系统来注入Person 实例， windows 下注入 bill ， linux 下注入 linus ，怎么实现呢？

这就需要我们用到 @Conditional 注解了，前言中提到，需要实现 Condition 接口，并重写方法来自定义 match 规则。

首先，创建一个WindowsCondition 类：

```java
public class WindowsCondition implements Condition {
/**
 * @param conditionContext: 判断条件能使用的上下文环境
 * @param annotatedTypeMetadata: 注解所在位置的注释信息
 * */
@Override public boolean matches(ConditionContext conditionContext, AnnotatedTypeMetadata annotatedTypeMetadata) {
// 获取 ioc 使用的 beanFactory
ConfigurableListableBeanFactory beanFactory = conditionContext.getBeanFactory();
// 获取类加载器
ClassLoader classLoader = conditionContext.getClassLoader();
// 获取当前环境信息
Environment environment = conditionContext.getEnvironment();
// 获取 bean 定义的注册类
BeanDefinitionRegistry registry = conditionContext.getRegistry();
 // 获得当前系统名
String property = environment.getProperty("os.name");
// 包含 Windows 则说明是 windows 系统，返回 true
 if (property.contains("Windows")){
        return true;
    }
return false;
    }
}
```

matches 方法的两个参数的意思在注释中讲述了，值得一提的是， conditionContext 提供了多种方法，方便获取各种信息，也是 SpringBoot中 @ConditonalOnXX 注解多样扩展的基础。

接着，创建LinuxCondition 类：

```java
public class LinuxCondition implements Condition {
@Override
public boolean matches(ConditionContext conditionContext, AnnotatedTypeMetadata annotatedTypeMetadata) {
Environment environment = conditionContext.getEnvironment();
String property = environment.getProperty("os.name");
if (property.contains("Linux")){
         return true;
        }
        return false;
    }
}
```

接着就是使用这两个类了，因为此注解可以标注在方法上和类上，所以分开测试：

标注在方法上：

修改BeanConfig ：

```java
@Configuration
public class BeanConfig {
// 只有一个类时，大括号可以省略
// 如果 WindowsCondition 的实现方法返回 true ，则注入这个 bean @Conditional({WindowsCondition.class})
@Bean(name = "bill")
 public Person person1(){
        return new Person("Bill Gates",62);
    }
```

// 如果 LinuxCondition 的实现方法返回 true ，则注入这个 bean

```java
@Conditional({LinuxCondition.class})
 @Bean("linus")
public Person person2(){
        return new Person("Linus",48);
     }
}
```

修改测试方法，使其可以打印当前系统名：

运行结果如下：

我是运行在windows 上的所以只注入了 bill ，嗯，没毛病。

接着实验linux 下的情况，不能运行在 linux 下，但可以修改运行时参数：

修改后启动测试方法：

一个方法只能注入一个 bean 实例，所以 @Conditional 标注在方法上只能控制一个 bean 实例是否注入。

标注在类上：

一个类中可以注入很多实例，@Conditional 标注在类上就决定了一批 bean 是否注入。

我们试一下，将BeanConfig 改写，这时，如果 WindowsCondition 返回 true ，则两个 Person 实例将被注入 ( 注意：上一个测试将 os.name改为 linux ，这是我将把这个参数去掉 ) ：

```java
@Conditional({WindowsCondition.class})
@Configuration
public class BeanConfig {
@Bean(name = "bill")
public Person person1(){
        return new Person("Bill Gates",62);
    }
@Bean("linus")
public Person person2(){
        return new Person("Linus",48);
    }
}
```

结果两个实例都被注入：

如果将类上的WindowsCondition.class 改为 LinuxCondition.class ，结果应该可以猜到：

结果就是空的，类中所有bean 都没有注入。

多个条件类：

前言中说，@Conditional 注解传入的是一个 Class 数组，存在多种条件类的情况。

这种情况貌似判断难度加深了，测试一波，新增新的条件类，实现的matches 返回 false （这种写死返回 false 的方法纯属测试用，没有实际意义 O( ∩ _ ∩ )O ）

```java
public class ObstinateCondition implements Condition {
 @Override
public boolean matches(ConditionContext conditionContext, AnnotatedTypeMetadata annotatedTypeMetadata) {
        return false;
    }
}
```

BeanConfig 修改一下：

```java
@Conditional({WindowsCondition.class,ObstinateCondition.class})
@Configuration
public class BeanConfig {
@Bean(name = "bill")
public Person person1(){
        return new Person("Bill Gates",62);
     }
@Bean("linus")
public Person person2(){
        return new Person("Linus",48);
    }
}
```

结果：

现在如果将ObstinateCondition 的 matches 方法返回值改成 true ，两个 bean 就被注入进容器：

结论得：

第一个条件类实现的方法返回true ，第二个返回 false ，则结果 false ，不注入进容器。

第一个条件类实现的方法返回true ，第二个返回 true ，则结果 true ，注入进容器中。