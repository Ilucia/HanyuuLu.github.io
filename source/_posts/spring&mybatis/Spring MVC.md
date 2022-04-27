## Spring MVC

### Spring ioc

![](/Users/ng/Documents/spring&mybatis/pictures/SpringIOC.jpg)

### 原理

![](/Users/ng/Documents/spring&mybatis/pictures/SpringMVC工作流程.png)

#### 1. DispatcherServlet

- DispatcherServlet 会拦截所有的请求，并且将这些请求发送给 Spring MVC 控制器

#### 2. HandlerMapping

- DispatcherServlet 会查询一个或多个处理器映射来确定请求的下一站在哪里，处理器映射会**根据请求所携带的 URL 信息来进行决策**
- 例如通过配置 simpleUrlHandlerMapping 来将 /hello 地址交给 helloController 处理

```xml
<bean id="simpleUrlHandlerMapping"
      class="org.springframework.web.servlet.handler.SimpleUrlHandlerMapping">
    <property name="mappings">
        <props>
            <!-- /hello 路径的请求交给 id 为 helloController 的控制器处理-->
            <prop key="/hello">helloController</prop>
        </props>
    </property>
</bean>
<bean id="helloController" class="controller.HelloController"></bean>
```

#### 3. 控制器

- 一旦选择了合适的控制器， DispatcherServlet 会将请求发送给选中的控制器，到了控制器，请求会卸下其负载（用户提交的请求）等待控制器处理完这些信息

```java
public ModelAndView handleRequest(javax.servlet.http.HttpServletRequest httpServletRequest, javax.servlet.http.HttpServletResponse httpServletResponse) throws Exception {
    // 处理逻辑
    ....
}
```

#### 4. 返回DispatcherServlet

- 当控制器在完成逻辑处理后，通常会产生一些信息，这些信息就是需要返回给用户并在浏览器上显示的信息，它们被称为**模型（Model）**。仅仅返回原始的信息时不够的——这些信息需要以用户友好的方式进行格式化，一般会是 HTML，所以，信息需要发送给一个**视图（view）**，通常会是 JSP
- 控制器所做的最后一件事就是将模型数据打包，并且表示出用于渲染输出的视图名**（逻辑视图名）。它接下来会将请求连同模型和视图名发送回 DispatcherServlet**
- 这样以来，控制器就不会和特定的视图相耦合，传递给 DispatcherServlet 的视图名并不直接表示某个特定的 JSP。（实际上，它甚至不能确定视图就是 JSP）相反，**它传递的仅仅是一个逻辑名称，这个名称将会用来查找产生结果的真正视图**

```java
public ModelAndView handleRequest(javax.servlet.http.HttpServletRequest httpServletRequest, javax.servlet.http.HttpServletResponse httpServletResponse) throws Exception {
    // 处理逻辑
    ....
    // 返回给 DispatcherServlet
    return mav;
}
```

#### 5. 视图解析器

- DispatcherServlet 将会使用视图解析器（view resolver）来将逻辑视图名匹配为一个特定的视图实现，它可能是也可能不是 JSP

#### 6. 视图

- 既然 DispatcherServlet 已经知道由哪个视图渲染结果了，那请求的任务基本上也就完成了
- 它的最后一站是视图的实现，在这里它交付模型数据，请求的任务也就完成了。视图使用模型数据渲染出结果，这个输出结果会通过响应对象传递给客户端

### 使用注解配置SpringMVC

#### 完整的命名空间

```xml
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:context="http://www.springframework.org/schema/context"
       xmlns:mvc="http://www.springframework.org/schema/mvc"
       xsi:schemaLocation="http://www.springframework.org/schema/beans
       http://www.springframework.org/schema/beans/spring-beans-4.2.xsd
    http://www.springframework.org/schema/mvc
    http://www.springframework.org/schema/mvc/spring-mvc-4.2.xsd
    http://www.springframework.org/schema/context
    http://www.springframework.org/schema/context/spring-context-4.2.xsd">
```

#### 主要注解

![](/Users/ng/Documents/spring&mybatis/pictures/SpringMVC注解.png)

### 配置视图解析器

视图解析器负责定位视图，它接受一个由 DispaterServlet 传递过来的逻辑视图名来匹配一个特定的视图。

- **需求：** 有一些页面我们不希望用户用户直接访问到，例如有重要数据的页面，例如有模型数据支撑的页面
- **造成的问题：**
  - 我们可以在【web】根目录下放置一个【test.jsp】模拟一个重要数据的页面，我们什么都不用做，重新启动服务器，网页中输入 `localhost/test.jsp` 就能够直接访问到了，这会造成**数据泄露**
  - 另外我们可以直接输入 `localhost/index.jsp` 试试，根据我们上面的程序，这会是一个空白的页面，因为并没有获取到 `${message}` 参数就直接访问了，这会**影响用户体验**

#### 解决方案

- 我们将我们的 JSP 文件配置在【WEB-INF】文件夹中的【page】文件夹下，【WEB-INF】是 Java Web 中默认的安全目录，是不允许用户直接访问的
  - 也就是你说你通过 `localhost/WEB-INF/` 这样的方式是永远访问不到的

- 但是我们需要将这告诉给视图解析器，我们在 dispatcher-servlet.xml 文件中做如下配置

```java
<bean id="viewResolver"
      class="org.springframework.web.servlet.view.InternalResourceViewResolver">
    <property name="prefix" value="/WEB-INF/page/" />
    <property name="suffix" value=".jsp" />
</bean>
```

这里配置了一个 Spring MVC 内置的一个视图解析器，该解析器是遵循着一种约定：会**在视图名上添加前缀和后缀，进而确定一个 Web 应用中视图资源的物理路径的**

##### 修改HelloController

```java
/**
 * Controller用来声明控制器
 */
@Controller
public class HelloController{

    /**
     * description: RequestMapping注解用来映射 路径请求和方法
     *
     * @param httpServletRequest 请求
     * @param httpServletResponse 回应
     * @return org.springframework.web.servlet.ModelAndView
     */
    @RequestMapping("/hello")
    public ModelAndView handleRequest(javax.servlet.http.HttpServletRequest httpServletRequest, javax.servlet.http.HttpServletResponse httpServletResponse) throws Exception {
        ModelAndView mav = new ModelAndView("index");
        mav.addObject("message", "Hello Spring MVC");
        return mav;
    }
}
```

##### 配置视图解析器

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:context="http://www.springframework.org/schema/context"
       xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd http://www.springframework.org/schema/context http://www.springframework.org/schema/context/spring-context.xsd">

    <!--<bean id="simpleUrlHandlerMapping"-->
                                        <!--class="org.springframework.web.servlet.handler.SimpleUrlHandlerMapping">-->
    <!--<property name="mappings">-->
            <!--<props>-->
                <!--&lt;!&ndash; /hello 路径的请求交给 id 为 helloController 的控制器处理&ndash;&gt;-->
                <!--<prop key="/hello">helloController</prop>-->
            <!--</props>-->
        <!--</property>-->
    <!--</bean>-->
    <!--<bean id="helloController" class="controller.HelloController"></bean>-->

    <!-- 扫描controller下的组件 -->
    <context:component-scan base-package="controller"/>
    <bean id="viewResolver"
          class="org.springframework.web.servlet.view.InternalResourceViewResolver">
        <property name="prefix" value="/WEB-INF/page/" />
        <property name="suffix" value=".jsp" />
    </bean>
</beans>
```

##### 剪切index.jsp文件

### 控制器接收请求数据

##### test.jsp

```xml
<!DOCTYPE html>
<%@ page language="java" contentType="text/html; charset=UTF-8"
         pageEncoding="UTF-8" import="java.util.*" isELIgnored="false"%>
<html>
<head>
    <meta charset="utf-8">
    <title>Spring MVC 传参方式</title>
</head>
<body>
<form action="/param" role="form">
    用户名：<input type="text" name="userName"><br/>
    密码：<input type="text" name="password"><br/>
    <input type="submit" value="提  交">
</form>
</body>
</html>
```

```java
@RequestMapping("/test")
    public ModelAndView handlePost(javax.servlet.http.HttpServletRequest httpServletRequest, javax.servlet.http.HttpServletResponse httpServletResponse) throws Exception {
        return new ModelAndView("test");
    }
```

#### 使用Servlet原生API

```java
@RequestMapping("/param")
public ModelAndView getParam(HttpServletRequest request,
                         HttpServletResponse response) {
    String userName = request.getParameter("userName");
    String password = request.getParameter("password");

    System.out.println(userName);
    System.out.println(password);
    return null;
}
```

#### 使用@RequestParam("前台参数名")注入

- 同名匹配原则

```java
@RequestMapping("/param")
public ModelAndView getParam(@RequestParam("userName") String name, @RequestParam("password") String pwd) {
    System.out.println(name);
    System.out.println(pwd);
    return null;
}
```

- 该注解有三个变量：`value`、`required`、`defaultvalue`

- `value` ：指定 `name` 属性的名称是什么，`value` 属性都可以默认不写

  `required` ：是否必须要有该参数，可以设置为【true】或者【false】

  `defaultvalue` ：设置默认值

#### 使用模型传参

- **要求： 前台参数名字必须和模型中的字段名一样**

##### User模型

```java
package pojo;

public class User {
    
    String userName;
    String password;

    /* getter and setter */
}
```

```java
@RequestMapping("/param")
public ModelAndView getParam(User user) {
    System.out.println(user.getUserName());
    System.out.println(user.getPassword());
    return null;
}
```

#### 中文乱码解决

- **注意：** 跟 Servlet 中的一样，该方法只对 POST 方法有效（因为是直接处理的 request）

我们可以通过配置 Spring MVC 字符编码过滤器来完成，在 web.xml 中添加

```java
<filter>
    <filter-name>CharacterEncodingFilter</filter-name>
    <filter-class>org.springframework.web.filter.CharacterEncodingFilter</filter-class>
    <init-param>
        <param-name>encoding</param-name>
        <!-- 设置编码格式 -->
        <param-value>utf-8</param-value>
    </init-param>
</filter>
<filter-mapping>
    <filter-name>CharacterEncodingFilter</filter-name>
    <url-pattern>/*</url-pattern>
</filter-mapping>
```

### 控制器回显数据

- 在page下创建test2.jsp

```xml
<!DOCTYPE html>
<%@ page language="java" contentType="text/html; charset=UTF-8"
         pageEncoding="UTF-8" import="java.util.*" isELIgnored="false" %>
<html>
<head>
    <title>Spring MVC 数据回显</title>
</head>
<body>
<h1>回显数据：${message}</h1>
</body>
</html>
```

#### 使用Servlet原生API

```java
@RequestMapping("/value")
public ModelAndView showData(HttpServletRequest request,
                                  HttpServletResponse response) {
    request.setAttribute("message","成功！");
    return new ModelAndView("test2");
}
```

#### 使用ModelAndView

```java
@RequestMapping("/value")
public ModelAndView showData(HttpServletRequest request,
                                  HttpServletResponse response) {
    ModelAndView mav = new ModelAndView("test2");
  mav.addObject("message","成功！")
    return mav;
}
```

#### 使用model对象

```java
@ModelAttribute
public void model(Model model) {
    model.addAttribute("message", "注解成功");
}

@RequestMapping("/value")
public String showData() {
    return "test2";
}
```

### 客户端跳转

- 前面不管是地址 `/hello` 跳转到 index.jsp 还是 `/test` 跳转到 test.jsp，这些都是服务端的跳转，也就是 `request.getRequestDispatcher("地址").forward(request, response);`
- 那我们如何进行客户端跳转呢？我们继续在 HelloController 中编写

```java
@RequestMapping("/hello")
public ModelAndView handleRequest(javax.servlet.http.HttpServletRequest httpServletRequest, javax.servlet.http.HttpServletResponse httpServletResponse) throws Exception {
    ModelAndView mav = new ModelAndView("index");
    mav.addObject("message", "Hello Spring MVC");
    return mav;
}

@RequestMapping("/jump")
public ModelAndView jump() {
    ModelAndView mav = new ModelAndView("redirect:/hello");
    return mav;
}
//或者
@RequestMapping("/jump")
public String jump() {
    return "redirect: ./hello";
}
```

- 我们使用 `redirect:/hello` 就表示我们要跳转到 `/hello` 这个路径，我们重启服务器，在地址栏中输入：`localhost/jump` ，会自动跳转到 `/hello` 路径下

### 文件上传

- 需要先导入 `commons-io-1.3.2.jar` 和 `commons-fileupload-1.2.1.jar` 两个包

#### 1. 配置上传解析器

- 在 dispatcher-servlet.xml 中新增一句
  - 开启对上传功能的支持

```java
<bean id="multipartResolver" class="org.springframework.web.multipart.commons.CommonsMultipartResolver"/>
```

#### 2. 编写upload.jsp

```xml
<%@ page contentType="text/html;charset=UTF-8" language="java" %>
<html>
<head>
    <title>测试文件上传</title>
</head>
<body>
<form action="/upload" method="post" enctype="multipart/form-data">
    <input type="file" name="picture">
    <input type="submit" value="上 传">
</form>
</body>
</html>
```

#### 3. 编写控制器

- 在 Package【controller】下新建【UploadController】类

```java
package controller;

import org.springframework.stereotype.Controller;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RequestParam;
import org.springframework.web.multipart.MultipartFile;
import org.springframework.web.servlet.ModelAndView;

@Controller
public class UploadController {

    @RequestMapping("/upload")
    public void upload(@RequestParam("picture") MultipartFile picture) throws Exception {
        System.out.println(picture.getOriginalFilename());
    }

    @RequestMapping("/test2")
    public ModelAndView upload() {
        return new ModelAndView("upload");
    }
}
```

#### 4. 测试



### web.xml

#### 加载流程

![](/Users/ng/Documents/spring&mybatis/pictures/web_xml加载过程.png)

#### 配置

![](/Users/ng/Documents/spring&mybatis/pictures/web_xml配置.png)

### DispatcherServlet

![](/Users/ng/Documents/spring&mybatis/pictures/SpringMVC流程示例.jpg)

### 特殊bean

![](/Users/ng/Documents/spring&mybatis/pictures/SpringMVC特殊bean.png)

### applicationContext.xml配置文件标签

![](/Users/ng/Documents/spring&mybatis/pictures/applicationContext_xml配置文件标签.png)

### 异常处理

![](/Users/ng/Documents/spring&mybatis/pictures/SpringMVC异常处理.png)

### 数据处理

| 数据类型                          | 处理所需注解                                                 |
| --------------------------------- | ------------------------------------------------------------ |
| application/x-www-form-urlencoded | Html表单默认传输编码类型，@RequestParam、@ModelAttribute可以处理，@RequestBody也能处理 |
| multipart/form-data               | 是表单上传的非文本的内容，传递的是文件数据，设置该属性再配合post请求方式从而实现文件上传。需要使用解析器，不能直接处理（即使用@RequestBody不能处理这种格式的数据） |
| 其他格式                          | 其他格式包括application/json, application/xml等。这些格式的数据，必须使用@RequestBody来处理 |



```java
@RestController
@RequestMapping("/test")
public class TestController {

    /**
     * 普通请求
     *
     * http://localhost:8089/test/test1
     */
    @RequestMapping(value = "/test1")
    public String test1() {
        return "test1";
    }

    /**
     * 简单get请求
     * 组合注解 @GetMapping
     * 是@RequestMapping(method = RequestMethod.GET)的缩写
     *
     * http://localhost:8089/test/test2/sun
     */
    @GetMapping(value = "test2/{name}")
    public String test2(@PathVariable String name) {
        // \n不起作用了,那就直接用html中的标签吧
        return "oh you are " + name + "<br> nice to meet you";
    }

    /**
     * 简单get请求2
     * 注解 @PathVariable 映射URL绑定的占位符
     * 1）带占位符的URL是Spring3.0新增的功能，该功能在SpringMVC向REST目标挺进发展过程中具有里程碑的意义。
     * 2）通过@PathVariable可以将URL中占位符参数绑定到控制器处理方法的入参中：
     *    URL中的{xxx}占位符可以通过@PathVariable("xxx")绑定到操作方法的入参中
     *
     * http://localhost:8089/test/test3/admin&admin
     */
    @GetMapping(value = "/test3/{name}&{pwd}")
    public String test3(@PathVariable String name, @PathVariable String pwd) {
        if (name.equals("admin") && pwd.equals("admin")) {
            return "hello welcome admin";
        } else {
            return "oh sorry user name or password is wrong";
        }
    }

    /**
     * get请求
     * 注解 @RequestParam绑定请求参数值
     * 在处理方法入参使用@RequestParam可以把请求参数传递给请求方法，@RequestParam包含的属性值：
     * --- value ：参数名称
     * --- required ：是否必须，默认为true，表示请求参数中必须包含对应的参数，否则抛出异常。
     * --- defaultValue：当请求参数缺少或者有请求参数但值为空时，值采用该设置值
     *
     * http://localhost:8089/test/test4?name=111&pwd=111
     */
    @RequestMapping(value = "/test4", method = RequestMethod.GET)
    public String test4(@RequestParam(value = "name", required = true) String name,
                             @RequestParam(value = "pwd", required = true) String pwd) {
        if (name.equals("admin") && pwd.equals("admin")) {
            return JSON.toJSONString(Result.success());
        } else {
            return JSON.toJSONString(Result.error());
        }
    }

    /**
     * 简单post请求
     * 组合注解 @PostMapping
     * 是@RequestMapping(method = RequestMethod.POST)的缩写。
     *
     * http://localhost:8089/test/test5
     */
    @RequestMapping(value = "/test5", method = RequestMethod.POST)
    public String test5() {
        System.out.println("hello test5");
        return "test5";
    }


    /**
     * post请求
     *
     * http://localhost:8089/test/test6?name=111&pwd=111
     */
    @RequestMapping(value = "/test6", method = RequestMethod.POST)
    public String test6(@RequestParam(value = "name", required = true) String name,
                              @RequestParam(value = "pwd", required = true) String pwd) {
        if (name.equals("admin") && pwd.equals("admin")) {
            return JSON.toJSONString(Result.success());
        } else {
            return JSON.toJSONString(Result.error());
        }
    }

    /**
     * 参数为一个bean对象.spring会自动为我们关联映射
     *
     * get post
     * http://localhost:8089/test/test7?id=1
     *
     * post form-data和 x-www-form-urlencoded
     * http://localhost:8089/test/test7
     * id  1
     */
    @RequestMapping(value = "/test7", method = { RequestMethod.POST, RequestMethod.GET })
    public String test7(BaseEntity entity) {
        if (null != entity && entity.getId() == 1) {
            return JSON.toJSONString(Result.success());
        } else {
            return JSON.toJSONString(Result.error());
        }
    }

    /**
     * 请求内容是一个json串,spring会自动把他和我们的参数bean对应起来
     * 不过要加@RequestBody注解
     *
     * post row(application/json)
     * http://localhost:8089/test/test8
     * id   1
     */
    @RequestMapping(value = "/test8", method = { RequestMethod.POST, RequestMethod.GET })
    public String test8(@RequestBody BaseEntity entity) {
        if (null != entity && entity.getId() == 1) {
            return JSON.toJSONString(Result.success());
        } else {
            return JSON.toJSONString(Result.error());
        }
    }

}
```

#### 处理multipart/form-data

- multipart格式的数据会将一个表单拆分为多个部分（part），每个部分对应一个输入域。在一般的表单输入域中，它所对应的部分中会放置文本型数据，但是如果上传文件的话，它所对应的部分可以是二进制

![](/Users/ng/Documents/spring&mybatis/pictures/multipart_form-data示例.png)

- 配置一个multipart解析器，通过它来告诉DispatcherServlet该如何读取multipart请求

##### StandardServletMultipartResolver

- applicationContext.xml中添加

```xml
<bean id="multipartResolver" class="org.springframework.web.multipart.commons.StandardServletMultipartResolver"/>
```

- 配置类

```java
@Bean(name = "multipartResolver")
public StandardServletMultipartResolver getStandardServletMultipartResolver(){
  	return new StandardServletMultipartResolver();
}
```

- web.xml配置上传参数

```xml
<servlet>
    <servlet-name>dispatcherServlet</servlet-name>
    <servlet-class>org.springframework.web.servlet.DispatcherServlet</servlet-class>
    <init-param>
        <param-name>contextConfigLocation</param-name>
        <param-value>classpath:applicationContext.xml</param-value>
    </init-param>
    <load-on-startup>1</load-on-startup>
    <multipart-config>
        <!--上传到/tmp/upload 目录-->
      <location>/tmp/upload</location>
        <!--文件大小为2M-->
      <max-file-size>2097152</max-file-size>
        <!--整个请求不超过4M-->
      <max-request-size>4194304</max-request-size>
        <!--所有文件都要写入磁盘-->
      <file-size-threshold>0</file-size-threshold>
    </multipart-config>
</servlet>
<servlet-mapping>
  <servlet-name>dispatcherServlet</servlet-name>
  <url-pattern>/</url-pattern>
</servlet-mapping>
```

- 配置类中配置上传参数

```java
@Override
protected void customizeRegistration(ServletRegistration.Dynamic registration) {
  	//上传到/tmp/upload 目录，文件大小为2M，整个请求不超过4M，而且所有文件都要写入磁盘
  	registration.setMultipartConfig(new MultipartConfigElement("E:\\upload_ftp",2097152,4194304,0));
}
```

##### CommonsMultipartResolver

- 在applicationContext.xml中设置

```java
<bean id="multipartResolver" class="org.springframework.web.multipart.commons.CommonsMultipartResolver">
       <!--设置上传目录/tmp/upload;最大的文件容量设置为2M;最大的内存大小设置为0，表示所有文件都会写入磁盘中;无法设定multipart请求整体的最大容量-->
        <property name="uploadTempDir" value="/tmp/upload"/>
        <property name="maxUploadSize" value="2097152"/>
        <property name="maxInMemorySize" value="0"/>
    </bean>
```

- CommonsMultipartResolver 相比较 StandardServletMultipartResolver 来说 就是无法设定multipart请求整体的最大容量。
- CommonsMultipartResolver不会强制要求设置临时文件路径。默认情况下，这个路径就是Servlet容器的临时目录。 StandardServletMultipartResolver 必须设置临时文件路径才能正常执行。（以上所述上传目录均为临时文件路径）

```xml
<form action="/picture" method="post" enctype="multipart/form-data">
  <input type="file" name="picture">
  <input type="submit">
</form>
```

```java
@RequestMapping(value = "/picture",method = RequestMethod.POST)
public String getHome(@RequestPart("picture") MultipartFile picture) throws IOException {
		String name = picture.getName();
		byte[] bytes = picture.getBytes();
  	picture.transferTo(new File("/"+picture.getOriginalFilename())); 
  	//这里保存到文件系统的时候要用相对路径，比如这里配置的是 /。以配置的上传目录为基准。即文件路径E:/upload_ftp/ 是保存的目录
  	return "home";
}
```

- @RequestPart("picture") ：当注册表单提交的时候，picture属性将会给定一个byte数组，这个数组中包含了请求中对应part的数据（通过@RequestPart指定）。如果用户提交表单的时候没有选择文件，那么这个数组会是空（而不是null）
  - 甚至可以用byte[]数组接收Multipart请求而不用 MultipartFile
- 用MultipartFile接收为我们提供了很多的方法以便进行接下来的工作

#### 下载文件

```java
@RequestMapping("download/{fileName:.+}")
    public void downloadZipResource(HttpServletRequest request, 
                                     HttpServletResponse response, 
                                     @PathVariable("fileName") String fileName) 
    {
        ServletContext context = request.getSession().getServletContext();
        String dataDirectory  = context.getRealPath("/downloads") + "/" + fileName;
        File file = new File(dataDirectory);
        if (file.isFile()) 
        {
            String mimeType = context.getMimeType(dataDirectory);   //直接根据文件的路径名来获取它的MineType，这样就可以灵活的根据请求的文件类型来返回输出流了
            response.setContentType(mimeType);
            response.addHeader("Content-Disposition", "attachment; filename="+fileName);
            try 
            {
                OutputStream os = response.getOutputStream();;
                IOUtils.copy(new FileInputStream(file), os);
                os.flush();
            } 
            catch (IOException ex) {
                ex.printStackTrace();
            }
        }
    }
```

