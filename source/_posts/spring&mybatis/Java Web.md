## Servlet

- Servlet其实就是一个**遵循Servlet开发的java类**
- Servlet是**由服务器调用的，运行在服务器端**

- Servlet带给我们最大的作用就是**能够处理浏览器带来的HTTP请求，并返回一个响应给浏览器，从而实现浏览器和服务器的交互**


### Java Web目录结构

![Java Web目录结构](/Users/ng/Documents/spring&mybatis/pictures/Java Web目录结构.png)

- bbs目录代表一个web应用
- bbs目录下的html,jsp文件可以直接被浏览器访问
- WEB-INF目录下的资源是不能直接被浏览器访问的
- web.xml文件是web程序的主要配置文件
- 所有的classes文件都放在classes目录下
- jar文件放在lib目录下

### 编写Servlet程序

1. 创建一个自定义类，实现Servlet接口
2. 我们发现有5个方法需要重写，有init【初始化】，destroy【销毁】,service【服务】,ServletConfig【Servlet配置】,getServletInfo【Servlet信息
3. 调用ServletResponse对象的方法向浏览器输出HelloWorld
4. 配置xml文件，光写了Servlet是不行的，Tomcat还要知道浏览器怎么访问这个Servlet


#### Servlet生命周期

1. **加载Servlet。当Tomcat第一次访问Servlet的时候，Tomcat会负责创建Servlet的实例**
2. **初始化。当Servlet被实例化后，Tomcat会调用init()方法初始化这个对象**
3. **处理服务。当浏览器访问Servlet的时候，Servlet 会调用service()方法处理请求**
4. **销毁。当Tomcat关闭时或者检测到Servlet要从Tomcat删除的时候会自动调用destroy()方法，让该实例释放掉所占的资源。一个Servlet如果长时间不被使用的话，也会被Tomcat自动销毁**
5. **卸载。当Servlet调用完destroy()方法后，等待垃圾回收。如果有需要再次使用这个Servlet，会重新调用init()方法进行初始化操作**
6. 简单总结：只要访问Servlet，service()就会被调用。init()只有第一次访问Servlet的时候才会被调用。destroy()只有在Tomcat关闭的时候才会被调用

![Servlet调用图](/Users/ng/Documents/spring&mybatis/pictures/Servlet调用图.png)

#### 继承HttpServlet编写Servlet程序

- 实现Servlet接口，要实现5个方法
- 而HttpServlet类已经实现了Servlet接口的所有方法，编写Servlet时，只需要继承HttpServlet，重写你需要的方法即可，并且它在原有Servlet接口上添加了一些与HTTP协议处理方法，它比Servlet接口的功能更为强大
- idea创建Servlet的时候已经重写好了doGet()和doPost()方法


#### HTTP协议

- **HTTP协议是客户端和服务器交互的一种通迅的格式**
- 当在浏览器中点击这个链接的时候，**浏览器会向服务器发送一段文本**，**告诉服务器请求打开的是哪一个网页。服务器收到请求后，就返回一段文本给浏览器，浏览器会将该文本解析，然后显示出来。**这段「文本」就是遵循HTTP协议规范的