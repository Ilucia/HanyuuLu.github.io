# Spring Cloud

SpringCloud的**基础功能**：

- 服务治理： Spring Cloud Eureka
- 客户端负载均衡： Spring Cloud Ribbon
- 服务容错保护： Spring Cloud Hystrix
- 声明式服务调用： Spring Cloud Feign
- API网关服务：Spring Cloud Zuul
- 分布式配置中心： Spring Cloud Config

## Spring Cloud Eureka

#### 使用场景及架构

从分布式/微服务的角度而言：就是把我们一**大**的项目，**分解**成多个**小**的模块。这些小的模块组合起来，完成功能

拆分出多个模块以后，就会出现**各种各样**的问题

首当其冲的就是子系统之间的**通讯**问题。子系统与子系统之间不是在同一个环境下，那就需要**远程调用**。远程调用可能就会想到httpClient，WebService等等这些技术来实现。

既然是远程调用，就必须知道ip地址，我们可能有以下的场景。

- 功能实现一：A服务需要调用B服务

- - 在A服务的代码里面调用B服务，**显式通过IP地址调用**：`http://123.123.123.123:8888/java3y/3`

- 功能实现二：A服务调用B服务，B服务调用C服务，C服务调用D服务

- - 在A服务的代码里面调用B服务，显式通过IP地址调用：`http://123.123.123.123:8888/java3y/3`，(同样地)B->C，C->D

- 功能实现三：D服务调用B服务，B服务调用C服务

- - 在D服务的代码里面调用B服务，显式通过IP地址调用：`http://123.123.123.123:8888/java3y/3`，(同样地)B->C

- .....等等等等

万一，我们**B服务的IP地址变了**，想想会出现什么问题：A服务,D服务(等等)需要**手动更新**B服务的地址

- 在服务多的情况下，手动来维护这些静态配置就是噩梦！

> 为了解决微服务架构中的**服务实例维护问题(ip地址)**， 产生了大量的**服务治理**框架和产品。 这些框架和产品的实现都围绕着服务注册与服务发现机制来完成对微服务应用实例的**自动化管理**。

在SpringCloud中我们的服务治理框架一般使用的就是Eureka。

我们的问题：

- 现在有A、B、C、D四个服务，它们之间会互相调用(而且IP地址很可能会发生变化)，一旦某个服务的IP地址变了，那服务中的代码要跟着变，手动维护这些静态配置(IP)非常麻烦！

Eureka是这样解决上面所说的情况的：

- 创建一个E服务，将A、B、C、D四个服务的信息都**注册**到E服务上，E服务维护这些已经注册进来的信息

![](/Users/ng/Documents/spring&mybatis/pictures/Eureka服务注册架构.jpg)

A、B、C、D四个服务都可以**拿到**Eureka(服务E)那份**注册清单**。A、B、C、D四个服务互相调用不再通过具体的IP地址，而是**通过服务名来调用**！

- 拿到注册清单--->注册清单上有服务名--->自然就能够拿到服务具体的位置了(IP)。
- 其实简单来说就是：代码中通过**服务名找到对应的IP地址**(IP地址会变，但服务名一般不会变)

![img](https://pic1.zhimg.com/50/v2-5913ec6dc3604a877fca57cf77e35247_hd.jpg?source=1940ef5c)![img](https://pic1.zhimg.com/80/v2-5913ec6dc3604a877fca57cf77e35247_1440w.jpg?source=1940ef5c)

### Eureka服务治理机制

Eureka专门用于给其他服务注册的称为Eureka Server(服务注册中心)，其余注册到Eureka Server的服务称为Eureka Client

在Eureka Server一般我们会这样配置：

```yaml
register-with-eureka: false     #false表示不向注册中心注册自己。
    fetch-registry: false     #false表示自己端就是注册中心，我的职责就是维护服务实例，并不需要去检索服务
```

Eureka Client**分为服务提供者和服务消费者**。

- 但很可能，某服务**既是服务提供者又是服务消费者**。

如果在网上看到SpringCloud的**某个服务配置没有"注册"到Eureka-Server也不用过于惊讶**(但是它是可以获取Eureka服务清单的)

- 很可能只是作者把该服务认作为**单纯的服务消费者**，单纯的服务消费者无需对外提供服务，也就无须注册到Eureka中了

```yaml
eureka:
  client:
    register-with-eureka: false  # 当前微服务不注册到eureka中(消费端)
    service-url: 
      defaultZone: http://eureka7001.com:7001/eureka/,http://eureka7002.com:7002/eureka/,http://eureka7003.com:7003/eureka/
```

下面是Eureka的治理机制：

- 服务提供者

  - **服务注册：启动的时候会通过发送REST请求的方式将自己注册到Eureka Server上**，同时带上了自身服务的一些元数据信息。
  - **服务续约：**在注册完服务之后，**服务提供者会维护一个心跳**用来持续告诉Eureka Server: "我还活着 ” 、
  - **服务下线：当服务实例进行正常的关闭操作时，它会触发一个服务下线的REST请求**给Eureka Server, 告诉服务注册中心：“我要下线了 ”。

- 服务消费者

  - **获取服务：当我们启动服务消费者**的时候，它会发送一个REST请求给服务注册中心，来获取上面注册的服务清单
  - **服务调用：服务消费者在获取服务清单后，通过服务名**可以获得具体提供服务的实例名和该实例的元数据信息。在进行服务调用的时候，**优先访问同处一个Zone中的服务提供方**。

- Eureka Server(服务注册中心)：

  - **失效剔除：**默认每隔一段时间（默认为60秒） 将当前清单中超时（默认为90秒）**没有续约的服务剔除出去**。
  - **自我保护：**。EurekaServer 在运行期间，会统计心跳失败的比例在15分钟之内是否低于85%(通常由于网络不稳定导致)。 Eureka Server会将当前的**实例注册信息保护起来**， 让这些实例不会过期，尽可能**保护这些注册信息**。

最后，我们就有了这张图：![img](https://pic1.zhimg.com/80/v2-22ac91d35e2f2da97c6e8fd70c359f84_1440w.jpg?source=1940ef5c)

举个例子：

- 3y跟女朋友去东站的东方宝泰逛街，但不知道东方宝泰有什么好玩的。于是就去**物业**搜了一下**东方宝泰商户清单**，发现一楼有优衣库，二楼有星巴克，三楼有麦当劳。
- 在优衣库旁边，有新开张的KFC，在墙壁打上了很大的标识“欢迎KFC**入驻**东方宝泰”。
- 商家们需要定时**交物业费**给物业。
- **物业维持**东方宝泰的稳定性。如果某个商家不想在东方宝泰运营了，告诉了物业。物业自然就会将其在东方宝泰商户清单去除。

### 服务注册与发现

#### 创建“服务注册中心”

创建一个基础的Spring Boot工程，命名为`eureka-server`，并在`pom.xml`中引入需要的依赖内容：

```xml
<parent>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-parent</artifactId>
    <version>1.5.4.RELEASE</version>
    <relativePath/>
</parent>

<dependencies>
    <dependency>
        <groupId>org.springframework.cloud</groupId>
        <artifactId>spring-cloud-starter-eureka-server</artifactId>
    </dependency>
</dependencies>

<dependencyManagement>
    <dependencies>
        <dependency>
           <groupId>org.springframework.cloud</groupId>
           <artifactId>spring-cloud-dependencies</artifactId>
           <version>Dalston.SR1</version>
           <type>pom</type>
           <scope>import</scope>
        </dependency>
    </dependencies>
</dependencyManagement>
```

通过`@EnableEurekaServer`注解启动一个服务注册中心提供给其他应用进行对话。这一步非常的简单，只需要在一个普通的Spring Boot应用中添加这个注解就能开启此功能，比如下面的例子：

```java
@EnableEurekaServer
@SpringBootApplication
public class Application {

    public static void main(String[] args) {
        new SpringApplicationBuilder(Application.class)
                    .web(true).run(args);
    }
}
```

在默认设置下，该服务注册中心也会将自己作为客户端来尝试注册它自己，所以我们需要禁用它的客户端注册行为，只需要在`application.properties`配置文件中增加如下信息：

```
spring.application.name=eureka-server
server.port=1001

eureka.instance.hostname=localhost
eureka.client.register-with-eureka=false
eureka.client.fetch-registry=false
```

为了与后续要进行注册的服务区分，这里将服务注册中心的端口通过server.port属性设置为1001。启动工程后，访问：http://localhost:1001/，可以看到下面的页面，其中还没有发现任何服务。

[![img](http://blog.didispace.com/content/images/posts/spring-cloud-starter-dalston-1-1.png)](http://blog.didispace.com/content/images/posts/spring-cloud-starter-dalston-1-1.png)

#### 创建“服务提供方”

下面我们创建提供服务的客户端，并向服务注册中心注册自己。本文我们主要介绍服务的注册与发现，所以我们不妨在服务提供方中尝试着提供一个接口来获取当前所有的服务信息。

首先，创建一个基本的Spring Boot应用。命名为`eureka-client`，在`pom.xml`中，加入如下配置：

```xml
<parent> 
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-parent</artifactId>
    <version>1.5.4.RELEASE</version>
    <relativePath/> <!-- lookup parent from repository -->
</parent>

<dependencies>
    <dependency>
        <groupId>org.springframework.cloud</groupId>
        <artifactId>spring-cloud-starter-eureka</artifactId>
    </dependency>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-web</artifactId>
    </dependency>
</dependencies>

<dependencyManagement>
    <dependencies>
        <dependency>
           <groupId>org.springframework.cloud</groupId>
           <artifactId>spring-cloud-dependencies</artifactId>
           <version>Dalston.SR1</version>
           <type>pom</type>
           <scope>import</scope>
        </dependency>
    </dependencies>
</dependencyManagement>
```

其次，实现/dc请求处理接口，通过DiscoveryClient对象，在日志中打印出服务实例的相关内容。

```java
@RestController
public class DcController {

    @Autowired
    DiscoveryClient discoveryClient;

    @GetMapping("/dc")
    public String dc() {
        String services = "Services: " + discoveryClient.getServices();
        System.out.println(services);
        return services;
    }

}
```

最后在应用主类中通过加上`@EnableDiscoveryClient`注解，该注解能激活Eureka中的DiscoveryClient实现，这样才能实现Controller中对服务信息的输出。

```java
@EnableDiscoveryClient
@SpringBootApplication
public class Application {
    public static void main(String[] args) {
        new SpringApplicationBuilder(
            ComputeServiceApplication.class)
            .web(true).run(args);
    }
}
```

我们在完成了服务内容的实现之后，再继续对`application.properties`做一些配置工作，具体如下：

```properties
spring.application.name=eureka-client
server.port=2001
eureka.client.serviceUrl.defaultZone=http://localhost:1001/eureka/
```

通过`spring.application.name`属性，我们可以指定微服务的名称后续在调用的时候只需要使用该名称就可以进行服务的访问。`eureka.client.serviceUrl.defaultZone`属性对应服务注册中心的配置内容，指定服务注册中心的位置。为了在本机上测试区分服务提供方和服务注册中心，使用`server.port`属性设置不同的端口。

启动该工程后，再次访问：http://localhost:1001/。可以如下图内容，我们定义的服务被成功注册了。

[![img](http://blog.didispace.com/content/images/posts/spring-cloud-starter-dalston-1-2.png)](http://blog.didispace.com/content/images/posts/spring-cloud-starter-dalston-1-2.png)

当然，我们也可以通过直接访问`eureka-client`服务提供的`/dc`接口来获取当前的服务清单，只需要访问：http://localhost:2001/dc，我们可以得到如下输出返回：

```
Services: [eureka-client]
```

其中，方括号中的`eureka-client`就是通过Spring Cloud定义的`DiscoveryClient`接口在eureka的实现中获取到的所有服务清单。由于Spring Cloud在服务发现这一层做了非常好的抽象，所以，对于上面的程序，我们可以无缝的从eureka的服务治理体系切换到consul的服务治理体系中区。

我们已经成功地将服务提供者：eureka-client或consul-client注册到了Eureka服务注册中心或Consul服务端上了，同时我们也通过`DiscoveryClient`接口的`getServices`获取了当前客户端缓存的所有服务清单，那么接下来我们要学习的就是：如何去消费服务提供者的接口？

### 使用LoadBalancerClient

在Spring Cloud Commons中提供了大量的与服务治理相关的抽象接口，包括`DiscoveryClient`、这里我们即将介绍的`LoadBalancerClient`等。对于这些接口的定义我们在上一篇介绍服务注册与发现时已经说过，Spring Cloud做这一层抽象，很好的解耦了服务治理体系，使得我们可以轻易的替换不同的服务治理设施。

从`LoadBalancerClient`接口的命名中，我们就知道这是一个负载均衡客户端的抽象定义，下面我们就看看如何使用Spring Cloud提供的负载均衡器客户端接口来实现服务的消费。

下面的例子，我们将利用上一篇中构建的eureka-server作为服务注册中心、eureka-client作为服务提供者作为基础。

- 我们先来创建一个服务消费者工程，命名为：`eureka-consumer`。并在`pom.xml`中引入依赖（这里省略了parent和dependencyManagement的配置）：

```xml
<dependencies>
    <dependency>
        <groupId>org.springframework.cloud</groupId>
        <artifactId>spring-cloud-starter-eureka</artifactId>
    </dependency>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-web</artifactId>
    </dependency>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-actuator</artifactId>
    </dependency>
</dependencies>
```

- 配置`application.properties`，指定eureka注册中心的地址：

```properties
spring.application.name=eureka-consumer
server.port=2101

eureka.client.serviceUrl.defaultZone=http://localhost:1001/eureka/
```

- 创建应用主类。初始化`RestTemplate`，用来真正发起REST请求。`@EnableDiscoveryClient`注解用来将当前应用加入到服务治理体系中。

```java
@EnableDiscoveryClient
@SpringBootApplication
public class Application {

	@Bean
	public RestTemplate restTemplate() {
		return new RestTemplate();
	}

	public static void main(String[] args) {
		new SpringApplicationBuilder(Application.class).web(true).run(args);
	}
}
```

- 创建一个接口用来消费eureka-client提供的接口：

```java
@RestController
public class DcController {

    @Autowired
    LoadBalancerClient loadBalancerClient;
    @Autowired
    RestTemplate restTemplate;

    @GetMapping("/consumer")
    public String dc() {
        ServiceInstance serviceInstance = loadBalancerClient.choose("eureka-client");
        String url = "http://" + serviceInstance.getHost() + ":" + serviceInstance.getPort() + "/dc";
        System.out.println(url);
        return restTemplate.getForObject(url, String.class);
    }
}
```

可以看到这里，我们注入了`LoadBalancerClient`和`RestTemplate`，并在`/consumer`接口的实现中，先通过`loadBalancerClient`的`choose`函数来负载均衡的选出一个`eureka-client`的服务实例，这个服务实例的基本信息存储在`ServiceInstance`中，然后通过这些对象中的信息拼接出访问`/dc`接口的详细地址，最后再利用`RestTemplate`对象实现对服务提供者接口的调用。

在完成了上面你的代码编写之后，读者可以将eureka-server、eureka-client、eureka-consumer都启动起来，然后访问http://localhost:2101/consumer ，来跟踪观察eureka-consumer服务是如何消费eureka-client服务的`/dc`接口的

## Spring Cloud Ribbon

Spring Cloud Ribbon是基于Netflix Ribbon实现的一套客户端负载均衡的工具。它是一个基于HTTP和TCP的客户端负载均衡器。它可以通过在客户端中配置ribbonServerList来设置服务端列表去轮询访问以达到均衡负载的作用。

当Ribbon与Eureka联合使用时，ribbonServerList会被DiscoveryEnabledNIWSServerList重写，扩展成从Eureka注册中心中获取服务实例列表。同时它也会用NIWSDiscoveryPing来取代IPing，它将职责委托给Eureka来确定服务端是否已经启动。

而当Ribbon与Consul联合使用时，ribbonServerList会被ConsulServerList来扩展成从Consul获取服务实例列表。同时由ConsulPing来作为IPing接口的实现。

我们在使用Spring Cloud Ribbon的时候，不论是与Eureka还是Consul结合，都会在引入Spring Cloud Eureka或Spring Cloud Consul依赖的时候通过自动化配置来加载上述所说的配置内容，所以我们可以快速在Spring Cloud中实现服务间调用的负载均衡。

负载均衡有两种类型：

- 客户端负载均衡(Ribbon)

- - 服务实例的**清单在客户端**，客户端进行负载均衡算法分配。
  - (从上面的知识我们已经知道了：客户端可以从Eureka Server中得到一份服务清单，在发送请求时通过负载均衡算法，**在多个服务器之间选择一个进行访问**)

- 服务端负载均衡(Nginx)

- - 服务实例的**清单在服务端**，服务器进行负载均衡算法分配

所以，我们的图可以画成这样：



![img](https://pic2.zhimg.com/50/v2-3cebcf81da382b55f4e9e740b48c727e_hd.jpg?source=1940ef5c)![img](https://pic2.zhimg.com/80/v2-3cebcf81da382b55f4e9e740b48c727e_1440w.jpg?source=1940ef5c)



Ribbon是支持负载均衡，默认的负载均衡策略是轮询，我们也是可以根据自己实际的需求自定义负载均衡策略的。

```java
@Configuration
public class MySelfRule
{
	@Bean
	public IRule myRule()
	{
		//return new RandomRule();// Ribbon默认是轮询，我自定义为随机
		//return new RoundRobinRule();// Ribbon默认是轮询，我自定义为随机
		
		return new RandomRule_ZY();// 我自定义为每台机器5次
	}
}
```

实现起来也很简单：继承AbstractLoadBalancerRule类，重写`public Server choose(ILoadBalancer lb, Object key)`即可。

SpringCloud 在CAP理论是选择了AP的，在Ribbon中还可以配置**重试机制**的

下面我们通过具体的例子来看看如何使用Spring Cloud Ribbon来实现服务的调用以及客户端均衡负载。

### 项目修改

我们将利用之前构建的`eureka-server`作为服务注册中心、`eureka-client`作为服务提供者作为基础。而基于Spring Cloud Ribbon实现的消费者，我们可以根据`eureka-consumer`实现的内容进行简单改在就能完成，具体步骤如下：

- 根据`eureka-consumer`复制一个服务消费者工程，命名为：`eureka-consumer-ribbon`。在`pom.xml`中增加下面的依赖：

```xml
<dependencies>
    ...
    <dependency>
        <groupId>org.springframework.cloud</groupId>
        <artifactId>spring-cloud-starter-ribbon</artifactId>
    </dependency>
</dependencies>
```

- 修改应用主类。为`RestTemplate`增加`@LoadBalanced`注解：

```java
@EnableDiscoveryClient
@SpringBootApplication
public class Application {

	@Bean
	@LoadBalanced
	public RestTemplate restTemplate() {
		return new RestTemplate();
	}

	public static void main(String[] args) {
		new SpringApplicationBuilder(Application.class).web(true).run(args);
	}
}
```

- 修改Controller。去掉原来通过`LoadBalancerClient`选取实例和拼接URL的步骤，直接通过RestTemplate发起请求。

```java
@RestController
public class DcController {

    @Autowired
    RestTemplate restTemplate;

    @GetMapping("/consumer")
    public String dc() {
        return restTemplate.getForObject("http://eureka-client/dc", String.class);
    }

}
```

可以看到这里，我们除了去掉了原来与`LoadBalancerClient`相关的逻辑之外，对于`RestTemplate`的使用，我们的第一个url参数有一些特别。这里请求的host位置并没有使用一个具体的IP地址和端口的形式，而是采用了服务名的方式组成。那么这样的请求为什么可以调用成功呢？因为Spring Cloud Ribbon有一个拦截器，它能够在这里进行实际调用的时候，自动的去选取服务实例，并将实际要请求的IP地址和端口替换这里的服务名，从而完成服务接口的调用。

将eureka-server、eureka-client、eureka-consumer-ribbon都启动起来，然后访问http://localhost:2101/consumer ，来跟踪观察eureka-consumer-ribbon服务是如何消费eureka-client服务的`/dc`接口的，并且也可以通过启动多个eureka-client服务来观察其负载均衡的效果。

## Spring Cloud Feign

Spring Cloud Feign是一套基于Netflix Feign实现的声明式服务调用客户端。它使得编写Web服务客户端变得更加简单。我们只需要通过创建接口并用注解来配置它既可完成对Web服务接口的绑定。它具备可插拔的注解支持，包括Feign注解、JAX-RS注解。它也支持可插拔的编码器和解码器。Spring Cloud Feign还扩展了对Spring MVC注解的支持，同时还整合了Ribbon和Eureka来提供均衡负载的HTTP客户端实现。

下面，我们通过一个例子来展现Feign如何方便的声明对eureka-client服务的定义和调用。

### 项目修改

下面的例子，我们将利用之前构建的`eureka-server`作为服务注册中心、`eureka-client`作为服务提供者作为基础。而基于Spring Cloud Ribbon实现的消费者，我们可以根据`eureka-consumer`实现的内容进行简单改在就能完成，具体步骤如下：

- 根据`eureka-consumer`复制一个服务消费者工程，命名为：`eureka-consumer-feign`。在`pom.xml`中增加下面的依赖：

```xml
<dependencies>
    ...
    <dependency>
        <groupId>org.springframework.cloud</groupId>
        <artifactId>spring-cloud-starter-feign</artifactId>
    </dependency>
</dependencies>
```

- 修改应用主类。通过`@EnableFeignClients`注解开启扫描Spring Cloud Feign客户端的功能：

```java
@EnableFeignClients
@EnableDiscoveryClient
@SpringBootApplication
public class Application {

	public static void main(String[] args) {
		new SpringApplicationBuilder(Application.class).web(true).run(args);
	}
}
```

- 创建一个Feign的客户端接口定义。使用`@FeignClient`注解来指定这个接口所要调用的服务名称，接口中定义的各个函数使用Spring MVC的注解就可以来绑定服务提供方的REST接口，比如下面就是绑定`eureka-client`服务的`/dc`接口的例子：

```java
@FeignClient("eureka-client")
public interface DcClient {

    @GetMapping("/1")
    String consumer1();
  
    @GetMapping("/2")
    String consumer2();
}
```

- 修改Controller。通过定义的feign客户端来调用服务提供方的接口：

```java
@RestController
public class DcController {

    @Autowired
    DcClient dcClient;

    @GetMapping("/consumer1")
    public String dc() {
        return dcClient.consumer1();
    }
  
   @GetMapping("/consumer2")
    public String dc() {
        return dcClient.consumer2();
    }

}
```

通过Spring Cloud Feign来实现服务调用的方式更加简单了，通过`@FeignClient`定义的接口来统一的声明我们需要依赖的微服务接口。而在具体使用的时候就跟调用本地方法一点的进行调用即可。由于Feign是基于Ribbon实现的，所以它自带了客户端负载均衡功能，也可以通过Ribbon的IRule进行策略扩展。另外，Feign还整合的Hystrix来实现服务的容错保护，在Dalston版本中，Feign的Hystrix默认是关闭的。

将eureka-server、eureka-client、eureka-consumer-feign都启动起来，然后访问http://localhost:2101/consumer ，来跟踪观察eureka-consumer-feign服务是如何消费eureka-client服务的`/dc`接口的，并且也可以通过启动多个eureka-client服务来观察其负载均衡的效果。

### Feign 传输文件

在Spring Cloud封装的Feign中并不直接支持传文件，但可以通过引入Feign的扩展包来实现，本来就来具体说说如何实现。

#### 服务提供方（接收文件）

服务提供方的实现比较简单，就按Spring MVC的正常实现方式即可，比如：

```java
@EnableFeignClients
@EnableDiscoveryClient
@SpringBootApplication
public class Application {

    @RestController
    public class UploadController {

        @PostMapping(value = "/uploadFile", consumes = MediaType.MULTIPART_FORM_DATA_VALUE)
        public String handleFileUpload(@RequestPart(value = "file") MultipartFile file) {
            return file.getName();
        }

    }

    public static void main(String[] args) {
        new SpringApplicationBuilder(Application.class).web(true).run(args);
    }

}
```

#### 服务消费方（发送文件）

在服务消费方由于会使用Feign客户端，所以在这里需要在引入feign对表单提交的依赖，具体如下：

```xml
<dependency>
   <groupId>io.github.openfeign.form</groupId>
   <artifactId>feign-form</artifactId>
   <version>3.0.3</version>
</dependency>
<dependency>
   <groupId>io.github.openfeign.form</groupId>
   <artifactId>feign-form-spring</artifactId>
   <version>3.0.3</version>
</dependency>
<dependency>
   <groupId>commons-fileupload</groupId>
   <artifactId>commons-fileupload</artifactId>
   <version>1.3.3</version>
</dependency>
```

定义文件上传方的应用主类和FeignClient，假设服务提供方的服务名为`eureka-feign-upload-server`

```java
@EnableFeignClients
@EnableDiscoveryClient
@SpringBootApplication
public class Application {

    public static void main(String[] args) {
        new SpringApplicationBuilder(Application.class).web(true).run(args);
    }

}

@FeignClient(value = "upload-server", configuration = UploadService.MultipartSupportConfig.class)
public interface UploadService {
 
    @PostMapping(value = "/uploadFile", consumes = MediaType.MULTIPART_FORM_DATA_VALUE)
    String handleFileUpload(@RequestPart(value = "file") MultipartFile file);
 
    @Configuration
    class MultipartSupportConfig {
        @Bean
        public Encoder feignFormEncoder() {
            return new SpringFormEncoder();
        }
    }
 
}
```

在启动了服务提供方之后，尝试在服务消费方编写测试用例来通过上面定义的Feign客户端来传文件，比如：

```java
@Slf4j
@RunWith(SpringJUnit4ClassRunner.class)
@SpringBootTest
public class UploadTester {

    @Autowired
    private UploadService uploadService;

    @Test
    @SneakyThrows
    public void testHandleFileUpload() {

        File file = new File("upload.txt");
        DiskFileItem fileItem = (DiskFileItem) new DiskFileItemFactory().createItem("file",
                MediaType.TEXT_PLAIN_VALUE, true, file.getName());

        try (InputStream input = new FileInputStream(file); OutputStream os = fileItem.getOutputStream()) {
            IOUtils.copy(input, os);
        } catch (Exception e) {
            throw new IllegalArgumentException("Invalid file: " + e, e);
        }

        MultipartFile multi = new CommonsMultipartFile(fileItem);

        log.info(uploadService.handleFileUpload(multi));
    }

}
```

## Spring Cloud Config

Spring Cloud Config是Spring Cloud团队创建的一个全新项目，用来为分布式系统中的基础设施和微服务应用提供集中化的外部配置支持，它分为服务端与客户端两个部分。其中服务端也称为分布式配置中心，它是一个独立的微服务应用，用来连接配置仓库并为客户端提供获取配置信息、加密/解密信息等访问接口；而客户端则是微服务架构中的各个微服务应用或基础设施，它们通过指定的配置中心来管理应用资源与业务相关的配置内容，并在启动的时候从配置中心获取和加载配置信息。Spring Cloud Config实现了对服务端和客户端中环境变量和属性配置的抽象映射，所以它除了适用于Spring构建的应用程序之外，也可以在任何其他语言运行的应用程序中使用。由于Spring Cloud Config实现的配置中心默认采用Git来存储配置信息，所以使用Spring Cloud Config构建的配置服务器，天然就支持对微服务应用配置信息的版本管理，并且可以通过Git客户端工具来方便的管理和访问配置内容。当然它也提供了对其他存储方式的支持，比如：SVN仓库、本地化文件系统。

在本文中，我们将学习如何构建一个基于Git存储的分布式配置中心，并对客户端进行改造，并让其能够从配置中心获取配置信息并绑定到代码中的整个过程。

### 准备配置仓库

- 准备一个git仓库，可以在码云或Github上创建都可以。
  - https://gitee.com/ilucia/config_demo
- 假设我们读取配置中心的应用名为`config-client`，那么我们可以在git仓库中创建该项目的默认配置文件`config-client.yml`：

```yaml
info:
  profile: default
```

- 为了演示加载不同环境的配置，我们可以在git仓库中再创建一个针对dev环境的配置文件`config-client-dev.yml`：

```yaml
info:
  profile: dev
```

### 构建配置中心

通过Spring Cloud Config来构建一个分布式配置中心非常简单，只需要三步：

- 创建一个基础的Spring Boot工程，命名为：`config-server-git`，并在`pom.xml`中引入下面的依赖（省略了parent和dependencyManagement部分）：

```xml
<dependencies>
	<dependency>
		<groupId>org.springframework.cloud</groupId>
		<artifactId>spring-cloud-config-server</artifactId>
	</dependency>
</dependencies>
```

- 创建Spring Boot的程序主类，并添加`@EnableConfigServer`注解，开启Spring Cloud Config的服务端功能。

```java
@EnableConfigServer
@SpringBootApplication
public class Application {

	public static void main(String[] args) {
		new SpringApplicationBuilder(Application.class).web(true).run(args);
	}

}
```

- 在`application.yml`中添加配置服务的基本信息以及Git仓库的相关信息，例如：

```yaml
spring: 
  application:
    name: config-server
  cloud:
    config:
      server:
        git:
          uri: https://gitee.com/ilucia/config_demo
          username: ilucia
          password: 321324543x
server:
  port: 1201
```

到这里，使用一个通过Spring Cloud Config实现，并使用Git管理配置内容的分布式配置中心就已经完成了。我们可以将该应用先启动起来，确保没有错误产生，然后再尝试下面的内容。

> 如果我们的Git仓库需要权限访问，那么可以通过配置下面的两个属性来实现；
> spring.cloud.config.server.git.username：访问Git仓库的用户名
> spring.cloud.config.server.git.password：访问Git仓库的用户密码

完成了这些准备工作之后，我们就可以通过浏览器、POSTMAN或CURL等工具直接来访问到我们的配置内容了。访问配置信息的URL与配置文件的映射关系如下：

- /{application}/{profile}[/{label}]
- /{application}-{profile}.yml
- /{label}/{application}-{profile}.yml
- /{application}-{profile}.properties
- /{label}/{application}-{profile}.properties

上面的url会映射`{application}-{profile}.properties`对应的配置文件，其中`{label}`对应Git上不同的分支，默认为master。我们可以尝试构造不同的url来访问不同的配置内容，比如，要访问master分支，config-client应用的dev环境，就可以访问这个url：`http://localhost:1201/config-client/dev/master`，并获得如下返回：

```json
{
    "name": "config-client",
    "profiles": [
        "dev"
    ],
    "label": "master",
    "version": null,
    "state": null,
    "propertySources": [
        {
            "name": "https://gitee.com/ilucia/config_demo/config-client-dev.yml",
            "source": {
                "info.profile": "dev"
            }
        },
        {
            "name": "https://gitee.com/ilucia/config_demo/config-client.yml",
            "source": {
                "info.profile": "default"
            }
        }
    ]
}
```

我们可以看到该Json中返回了应用名：config-client，环境名：dev，分支名：master，以及default环境和dev环境的配置内容。

### 构建客户端

在完成了上述验证之后，确定配置服务中心已经正常运作，下面我们尝试如何在微服务应用中获取上述的配置信息。

- 创建一个Spring Boot应用，命名为`config-client`，并在`pom.xml`中引入下述依赖：

```xml
<dependencies>
	<dependency>
		<groupId>org.springframework.boot</groupId>
		<artifactId>spring-boot-starter-web</artifactId>
	</dependency>
	<dependency>
		<groupId>org.springframework.cloud</groupId>
		<artifactId>spring-cloud-starter-config</artifactId>
	</dependency>
</dependencies>
```

- 创建Spring Boot的应用主类，具体如下：

```java
@SpringBootApplication
public class Application {

	public static void main(String[] args) {
		new SpringApplicationBuilder(Application.class).web(true).run(args);
	}

}
```

- 创建`bootstrap.yml`配置，来指定获取配置文件的`config-server-git`位置，例如：

```yaml
spring:
  application:
    name: config-client
  cloud:
    config:
      uri: http://localhost:1201/
      profile: default
      label: master

server:
  port: 2001
```

上述配置参数与Git中存储的配置文件中各个部分的对应关系如下：

- spring.application.name：对应配置文件规则中的`{application}`部分
- spring.cloud.config.profile：对应配置文件规则中的`{profile}`部分
- spring.cloud.config.label：对应配置文件规则中的`{label}`部分
- spring.cloud.config.uri：配置中心`config-server`的地址

**这里需要格外注意：上面这些属性必须配置在bootstrap.properties中，这样config-server中的配置信息才能被正确加载。**

在完成了上面你的代码编写之后，读者可以将config-server-git、config-client都启动起来，然后访问http://localhost:2001/info ，我们可以看到该端点将会返回从git仓库中获取的配置信息：

```json
{
    "profile": "default"
}
```

另外，我们也可以修改config-client的profile为dev来观察加载配置的变化。

