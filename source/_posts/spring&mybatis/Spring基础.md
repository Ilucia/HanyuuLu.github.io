# Spring基础

## Core模块

### 注解方式配置

⾃从jdk5有了注解这个新特性，我们可以看到Struts2框架、Hibernate框架都⽀持使⽤注解来配置信
息...
通过注解来配置信息就是为了简化IOC容器的配置，注解可以把对象添加到IOC容器中、处理对象依赖
关系，我们来看看怎么⽤吧：
使⽤注解步骤：
1）先引⼊context名称空间
xmlns:context="http://www.springframework.org/schema/context

2）开启注解扫描器
<context:component-scan base-package=""></context:component-scan>
第⼆种⽅法:也可以通过⾃定义扫描类以@CompoentScan修饰来扫描IOC容器的bean对象。
如下代码:

```java
//表明该类是配置类
@Configuration
//启动扫描器，扫描bb包下的
//也可以指定多个基础包
//也可以指定类型
@ComponentScan("bb")
public class AnnotationScan {
}
```

- 在使⽤`@ComponentScan()`这个注解的时候，在测试类上需要`@ContextConfiguration`这个注解来加载配置类
- `@ContextConfiguration`这个注解⼜在Spring的test包下
- 创建对象以及处理对象依赖关系，相关的注解：
  - `@ComponentScan`扫描器
  - `@Configuration`表明该类是配置类
  - `@Component` 指定把⼀个对象加⼊IOC容器--->@Name也可以实现相同的效果【⼀般少⽤】
  - `@Repository` 作⽤同@Component； 在持久层使⽤
  - `@Service` 作⽤同@Component； 在业务逻辑层使⽤
  - `@Controller` 作⽤同@Component； 在控制层使⽤
  - `@Resource` 依赖关系
    - 如果`@Resource`不指定值，那么就根据类型来找，相同的类型在IOC容器中不能有两个
      如果`@Resource`指定了值，那么就根据名字来找

#### UserDao

```
package aa;
import org.springframework.stereotype.Repository;
/**
* Created by ozc on 2017/5/10.
*/
//把对象添加到容器中,⾸字⺟会⼩写
@Repository
public class UserDao {
public void save() {
System.out.println("DB:保存⽤户");
}
}
```

#### userService

```
import org.springframework.stereotype.Service;
import javax.annotation.Resource;
//把UserService对象添加到IOC容器中,⾸字⺟会⼩写
@Service
public class UserService {
//如果@Resource不指定值，那么就根据类型来找--->UserDao....当然了，IOC容器不能有两个
UserDao类型的对象
//@Resource
//如果指定了值，那么Spring就在IOC容器找有没有id为userDao的对象。
@Resource(name = "userDao")
private UserDao userDao;
public void save() {
userDao.save();
}
}
```

#### userAction

```
*/
//把对象添加到IOC容器中,⾸字⺟会⼩写
@Controller
public class UserAction {
@Resource(name = "userService")
private UserService userService;
public String execute() {
userService.save();
return null;
}
}
```

#### 测试

```
public class App {
public static void main(String[] args) {
// 创建容器对象
ApplicationContext ac = new
ClassPathXmlApplicationContext("aa/applicationContext.xml");
UserAction userAction = (UserAction) ac.getBean("userAction");
userAction.execute();
}
}
```

#### Bean创建细节

- 参数指定

##### 单例/多例

- `scope="singleton"` 默认值， 即 默认是单例 【service/dao/⼯具类】

* `scope="prototype"` 多例
  * Action对象

##### 创建时间点

- `scope="prototype"` 在⽤到对象的时候，才创建对象

* `scope="singleton"` 在启动(容器初始化之前)， 就已经创建了bean，且整个应⽤只有⼀
个

##### 是否延迟创建

- `lazy-init="false"` 默认为false, 不延迟创建，即在启动时候就创建对象
- `lazy-init="true"` 延迟初始化, 在⽤到对象的时候才创建对象
  - 只对单例有效

##### 创建对象之后的初始化/销毁

- `init-method="init_user"` 对应对象的init_user⽅法，在对象创建之后执⾏

* `destroy-method="destroy_user"` 在调⽤容器对象的destroy⽅法时候执⾏，(容器⽤
实现类)

## 依赖注入

Spring提供了好⼏种的⽅式来给属性赋值

1. 通过构造函数
2. 通过set⽅法给属性注⼊值
3. p名称空间
4. ⾃动装配(了解)
5. 注解

### 使用注解实现自动装配

- @Autowired注解来实现⾃动装配
  - 可以在构造器上修饰，也可以在setter⽅法上修饰
  - 来⾃java的@Inject的和@AutoWired有相同的功能
  - 如果没有匹配到bean，为了避免异常的出现，我们可以使⽤required属性上设置为false

UserService中使⽤userDao变量来维护与Dao层之间的依赖关系，UserAction中使⽤userService变量
来维护与Service层之间的依赖关系。

```java
public class UserDao {
public void save() {
System.out.println("DB:保存⽤户");
}
}
public class UserService {
private UserDao userDao;
public void save() {
userDao.save();
}
}
public class UserAction {
private UserService userService;
public String execute() {
userService.save();
return null;
}
}
```

##### applicationContext.xml配置⽂件

```xml
<!--创建userDao对象-->
<bean id="userDao" class="UserDao"/>
<!--创建userService对象-->
<bean id="userService" class="UserService">
<!--要想在userService层中能够引⽤到userDao，就必须先创建userDao对象-->
<constructor-arg index="0" name="userDao" type="UserDao" ref="userDao">
</constructor-arg>
</bean>
```

## AOP

### 注解⽅式实现AOP编程

我们之前⼿动的实现AOP编程是需要⾃⼰来编写代理⼯⼚的，现在有了Spring，就不需要我们⾃⼰写代
理⼯⼚了。Spring内部会帮我们创建代理⼯⼚。也就是说，不⽤我们⾃⼰写代理对象了

因此，我们只要关⼼**切⾯类、切⼊点、编写切⼊表达式指定拦截什么⽅法**就可以了

#### 在配置⽂件中开启AOP注解⽅式

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:p="http://www.springframework.org/schema/p"
       xmlns:context="http://www.springframework.org/schema/context"
       xmlns:aop="http://www.springframework.org/schema/aop"
       xsi:schemaLocation="http://www.springframework.org/schema/beans
                           http://www.springframework.org/schema/beans/spring-beans.xsd
                           http://www.springframework.org/schema/context
                           http://www.springframework.org/schema/context/spring-context.xsd
                           http://www.springframework.org/schema/aop
                           http://www.springframework.org/schema/aop/spring-aop.xsd">
  <context:component-scan base-package="aa"/>
  
  <!-- 开启aop注解⽅式 -->
  <aop:aspectj-autoproxy></aop:aspectj-autoproxy>
</beans>
```

#### 类代码

##### 切⾯类

```java
@Component
@Aspect//指定为切⾯类
public class AOP {
    //⾥⾯的值为切⼊点表达式
    @Before("execution(* aa.*.*(..))")
    public void begin() {
        System.out.println("开始事务");
    }
    @After("execution(* aa.*.*(..))")
    public void close() {
        System.out.println("关闭事务");
    }
}
```

##### UserDao

```java
@Component
public class UserDao implements IUser {
    @Override
    public void save() {
        System.out.println("DB:保存⽤户");
    }
}
```

##### IUser接口

```java
public interface IUser {
    void save();
}
```

##### 测试

```java
public class App {
    public static void main(String[] args) {
        ApplicationContext ac = new ClassPathXmlApplicationContext("aa/applicationContext.xml");
        //这⾥得到的是代理对象....
        IUser iUser = (IUser) ac.getBean("userDao");
        System.out.println(iUser.getClass());
        iUser.save();
    }
}
```

#### AOP注解API

- **@Aspect**
  - 指定⼀个类为切⾯类
- **@Pointcut("execution(* cn.itcast.e_aop_anno..(..))")** 
  - 指定切⼊点表达式
- **@Before("pointCut_()")**
  - 前置通知，⽬标⽅法之前执⾏
- **@After("pointCut_()")**
  - 后置通知，⽬标⽅法之后执⾏（始终执⾏）
- **@AfterReturning("pointCut_()")**
  - 返回后通知，执⾏⽅法结束前执⾏(异常不执⾏)
- **@AfterThrowing("pointCut_()")**
  - 异常通知，出现异常时候执⾏
- **@Around("pointCut_()")**
  - 环绕通知，环绕⽬标⽅法执⾏

```java
// 指定切⼊点表达式，拦截哪个类的哪些⽅法
@Pointcut("execution(* aa.*.*(..))")
//Point签名
public void pointCut_() {
}
// 前置通知 : 在执⾏⽬标⽅法之前执⾏
@Before("pointCut_()")
public void begin(){
  System.out.println("开始事务/异常");
}
// 后置/最终通知：在执⾏⽬标⽅法之后执⾏，⽆论是否出现异常最终都会执⾏
@After("pointCut_()")
public void after(){
  System.out.println("提交事务/关闭");
}
// 返回后通知： 在调⽤⽬标⽅法结束后执⾏ 【出现异常不执⾏】
@AfterReturning("pointCut_()")
public void afterReturning() {
  System.out.println("afterReturning()");
}
// 异常通知： 当⽬标⽅法执⾏异常时候执⾏此关注点代码
@AfterThrowing("pointCut_()")
public void afterThrowing(){
  System.out.println("afterThrowing()");
}
// 环绕通知：环绕⽬标⽅式执⾏
@Around("pointCut_()")
public void around(ProceedingJoinPoint pjp) throws Throwable{
  System.out.println("环绕前....");
  pjp.proceed(); // 执⾏⽬标⽅法
  System.out.println("环绕后....");
}
```

#### @Pointcut

在Spring 2.0中，Pointcut的定义包括两个部分：**Pointcut表示式**(expression)和**Pointcut签名**(signature)

```java
//Pointcut表示式
@Pointcut("execution(* com.savage.aop.MessageSender.*(..))")
//Point签名
private void log(){} 
```

然后要使用所定义的Pointcut时，可以指定Pointcut签名

```java
@Before("og()")
```

这种使用方式等同于以下方式，直接定义execution表达式使用

```java
@Before("execution(* com.savage.aop.MessageSender.*(..))")
```

Pointcut定义时，还**可以使用&&、||、! 这三个运算**

```java
@Pointcut("execution(* com.savage.aop.MessageSender.*(..))")
private void logSender(){}

@Pointcut("execution(* com.savage.aop.MessageReceiver.*(..))")
private void logReceiver(){}

@Pointcut("logSender() || logReceiver()")
private void logMessage(){}
```

这个例子中，**logMessage()将匹配任何MessageSender和MessageReceiver中的任何方法**

还可以**将公用的Pointcut放到一个类中，以供整个应用程序使用**，如下：

```java
package com.savage.aop;

import org.aspectj.lang.annotation.*;

public class Pointcuts {
@Pointcut("execution(* *Message(..))")
public void logMessage(){}

@Pointcut("execution(* *Attachment(..))")
public void logAttachment(){}

@Pointcut("execution(* *Service.*(..))")
public void auth(){}
}
```

在使用上面定义Pointcut时，指定完整的类名加上Pointcut签名就可以了，如：

```java
package com.savage.aop;

import org.aspectj.lang.JoinPoint;
import org.aspectj.lang.annotation.*;

@Aspect
public class LogBeforeAdvice {
@Before("com.sagage.aop.Pointcuts.logMessage()")
public void before(JoinPoint joinPoint) {
System.out.println("Logging before " + joinPoint.getSignature().getName());
}
}
```

#### 切入点表达式

```
execution(modifiers-pattern? ret-type-pattern declaring-type-pattern? name-pattern(param-pattern) throws-pattern?)
```

- **?** 号代表0或1，可以不写
- *** **号代表任意类型，0或多
- ⽅法参数为 **..** 表示为可变参数

- **modifiers-pattern?**【修饰的类型，可以不写】
- **ret-type-pattern**【⽅法返回值类型，必写】
- **declaring-type-pattern?**【⽅法声明的类型，可以不写】
- **name-pattern(param-pattern)**【要匹配的名称，括号⾥⾯是⽅法的参数】
- **throws-pattern?**【⽅法抛出的异常类型，可以不写】

##### 例

![](/Users/ng/Documents/spring&mybatis/pictures/切入点表达式.png)

