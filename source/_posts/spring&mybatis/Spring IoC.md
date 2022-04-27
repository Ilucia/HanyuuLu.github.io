# Spring IoC

## IoC

##### IoC的概念

- 简单点儿说，IoC的理念就是，让别人为你服务！
- 通常情况下，被注入对象会直接依赖于被依赖对象。但是，在IoC的场景中，二者之间通过**IoC Service Provider**来打交道，所有的被注入对象和依赖对象现在由IoC Service Provider统一管理。被注入对象需要什么，直接跟IoC Service Provider招呼一声，后者就会把相应的被依赖对象注入到被注入对象中，从而达到IoC Service Provider为被注入对象服务的目的
- IoC Service Provider在这里就是通常的IoC容器所充当的角色。从被注入对象的角度看，**与之前直接寻求依赖对象相比，依赖对象的取得方式发生了反转**，**控制也从被注入对象转到了IoC Service Provider那里**
- **原来是需要什么东西自己去拿，现在是需要什么东西就让别人送过来**
- **IoC是一种可以帮助我们解耦各业务对象间依赖关系的对象绑定方式**

![](/Users/ng/Documents/spring&mybatis/pictures/IoC的角色.png)

### 依赖注入

- **接口注入**
  - 从注入方式的使用上来说，接口注入是现在不甚提倡的一种方式，基本处于“退役状态”。因为它强制被注入对象实现不必要的接口，**带有侵入性**。而构造方法注入和setter方法注入则不需要如此
- **构造方法注入**
  - 这种注入方式的优点就是，**对象在构造完成之后，即已进入就绪状态，可以马上使用**
  - 缺点就是，当依赖对象比较多的时候，构造方法的参数列表会比较长。而通过反射构造对象的时候，对相同类型的参数的处理会比较困难，维护和使用上也比较麻烦。而且在Java中，构造方法**无法被继承，无法设置默认值**。对于非必须的依赖处理，可能需要引入多个构造方法，而参数数量的变动可能造成维护上的不便
- **setter方法注入**
  - 因为方法可以命名，所以setter方法注入在描述性上要比构造方法注入好一些。另外，setter方法**可以被继承，允许设置默认值**，而且有良好的IDE支持
  - 缺点当然就是**对象无法在构造完成后马上进入就绪状态**

## IoC Service Provider

##### 概念

- 它可以指代**任何将IoC场景中的业务对象绑定到一起**的实现方式
- **Spring的IoC容器**就是一个**提供依赖注入服务的IoC Service Provider**
- IoC Service Provider的职责
  - 业务对象的构建管理
  - 业务对象间的依赖绑定

### IoC Service Provider 如何管理对象间的依赖关系

##### 直接编码方式

- 通过**程序编码**的方式**将被注入对象和依赖对象注册到容器中**，并**明确它们相互之间的依赖注入关系**

- 通过**为相应的类指定对应的具体实例**，可以告知IoC容器，当我们要这种类型的对象实例时，将容器中注册的、对应的那个具体实例返回给我们

  `IoContainer container = ...;` 
  `container.register(FXNewsProvider.class,new FXNewsProvider());` 
  `container.register(IFXNewsListener.class,new DowJonesNewsListener());` 
  `...`
  `FXNewsProvider newsProvider = (FXNewsProvider)container.get(FXNewsProvider.class);` 
  `newProvider.getAndPersistNews();`

##### 直接编码方式-接口注入

- 除了注册相应对象，还要**将“注入标志接口”与相应的依赖对象绑定**一下

  `IoContainer container = ...;` 
  `container.register(FXNewsProvider.class,new FXNewsProvider());` 
  `container.register(IFXNewsListener.class,new DowJonesNewsListener());` 
  `...` 
  **`container.bind(IFXNewsListenerCallable.class, container.get(IFXNewsListener.class));`** 
  `...` 
  `FXNewsProvider newsProvider = (FXNewsProvider)container.get(FXNewsProvider.class);` 
  `newProvider.getAndPersistNews();`

- 通过**bind方法**将“被注入对象”（由IFXNewsListenerCallable接口添加标志）所依赖的对象，绑定为容器中注册过的IFXNewsListener类型的对象实例
- 容器在返回FXNewsProvider对象实例之前，会**根据这个绑定信息**，将IFXNewsListener注册到容器中的对象实例**注入到“被注入对象”**——FXNewsProvider中，并最终返回已经组装完毕FXNewsProvider对象

##### 配置文件方式

- 通过XML文件来管理对象注册和对象间依赖关系

##### 注解方式

- 直接在类中**使用元数据信息来标注各个对象之间的依赖关系**，然后**由Guice框架根据这些注解所提供的信息将这些对象组装**后，交给客户端对象使用
- 通过**@Inject**，我们指明需要IoC Service Provider**通过构造方法注入方式**，为FXNewsProvider注入其所依赖的对象。至于余下的**依赖相关信息**，**在Guice中是由相应的Module来提供的**
  - 通过Module指定进一步的依赖注入相关信息之后，我们就可以直接从Guice那里取得最终已经注入完毕，并直接可用的对象了
- **注解最终也要通过代码处理来确定最终的注入关系**，从这点儿来说，注解方式可以算作编码方式的一种特殊情况

## BeanFactory

### BeanFactory和ApplicationContext

- BeanFactory
  - 基础类型IoC容器，提供完整的IoC服务支持
  - 默认采用**延迟初始化策略**（lazy-load）
    - 只有当客户端对象需要访问容器中的某个受管对象的时候，才对该受管对象进行初始化以及依赖注入操作
    - 容器启动初期速度较快，所需要的资源有限
  - 对于**资源有限，并且功能要求不是很严格**的场景，BeanFactory是比较合适的IoC容器选择
- ApplicationContext
  - 在BeanFactory的基础上构建，拥有BeanFactory的所有支持，还提供了其他高级特性
    - ApplicationContext**间接继承自BeanFactory**，所以说它是构建于BeanFactory之上的IoC容器
  - ApplicationContext所管理的对象，在该类型**容器启动之后，默认全部初始化并绑定完成**
    - **要求更多的系统资源**，同时，因为在启动时就完成所有初始化，**容器启动时间较**之BeanFactory也会**长**一些
  - 在那些**系统资源充足，并且要求更多功能**的场景中，ApplicationContext类型的容器是比较合适的选择

### BeanFactory概念

- BeanFactory，顾名思义，就是**生产Bean的工厂**
  - **把每个业务对象看作一个JavaBean对象**
  - BeanFactory就像一个汽车生产厂。你从其他汽车零件厂商或者自己的零件生产部门取得汽车零件送入这个汽车生产厂，最后，只需要从生产线的终点取得成品汽车就可以了。相似地，将应用所需的所有业务对象交给BeanFactory之后，剩下要做的，就是直接从BeanFactory取得最终组装完成并且可用的对象。至于这个最终业务对象如何组装，你不需要关心，BeanFactory会帮你搞定
  - BeanFactory肯定**会公开一个取得组装完成的对象的方法接口**

```java
public interface BeanFactory { 
    String FACTORY_BEAN_PREFIX = "&"; 
    Object getBean(String name) throws BeansException;
    Object getBean(String name, Class requiredType) throws BeansException; 
    /** 
    * @since 2.5 
    */ 
    Object getBean(String name, Object[] args) throws BeansException; 
    boolean containsBean(String name); 
    boolean isSingleton(String name) throws NoSuchBeanDefinitionException; 
    /** 
    * @since 2.0.3 
    */ 
    boolean isPrototype(String name) throws NoSuchBeanDefinitionException; 
    /** 
    * @since 2.0.1 
    */ 
    boolean isTypeMatch(String name, Class targetType) throws NoSuchBeanDefinitionException; 
    Class getType(String name) throws NoSuchBeanDefinitionException; 
    String[] getAliases(String name);
}
```

- 上面代码中的方法基本上都是**查询相关的方法**，例如，取得某个对象的方法（getBean）、查询某个对象是否存在于容器中的方法（containsBean），或者取得某个bean的状态或者类型的方法等
  - 因为通常情况下，对于独立的应用程序，只有主入口类才会跟容器的API直接耦合。

- 使用BeanFactory**前后唯一的不同，就是对象之间依赖关系的解决方式改变了**

##### FX新闻应用设计和实现框架代码

```java
//设计FXNewsProvider类用于普遍的新闻处理
public class FXNewsProvider 
{ 
... 
} 
//设计IFXNewsListener接口抽象各个新闻社不同的新闻获取方式
public interface IFXNewsListener 
{ 
... 
} 
//给出相应实现类
public class DowJonesNewsListener implements IFXNewsListener
{ 
... 
} 
//设计IFXNewsPersister接口抽象不同数据访问方式
public interface IFXNewsPersister 
{ 
... 
} 
//实现相应的实现类
public class DowJonesNewsPersister implements IFXNewsPersister 
{ 
... 
}
```

- **BeanFactory**会**通过常用的XML文件来注册并管理各个业务对象之间的依赖关系**

### BeanFactory的对象注册与依赖绑定方式

- 为了能够明确**管理各个业务对象以及业务对象之间的依赖绑定关系**，需要某种途径来记录和管理这些信息
  - 直接编码，配置文件，元数据

#### 直接编码

- BeanFactory只是一个接口，我们最终**需要一个该接口的实现来进行实际的Bean的管理**
- **BeanFactory接口只定义如何访问容器内管理的Bean的方法**，各个BeanFactory的具体实现类负责具体Bean的注册以及管理工作
- **BeanDefinitionRegistry接口才是在BeanFactory的实现中担当Bean注册管理的角色**
  - 通常情况下，具体的BeanFactory实现类会实现这个接口来管理Bean的注册 
- 打个比方说，**BeanDefinitionRegistry就像图书馆的书架**，所有的书是放在书架上的。虽然你还书或者借书都是跟**图书馆**（也就是**BeanFactory**，或许BookFactory可能更好些）打交道，但书架才是存放各类图书的地方

```java
public static void main(String[] args) 
{ 
    DefaultListableBeanFactory beanRegistry = new DefaultListableBeanFactory(); 
    BeanFactory container = (BeanFactory)bindViaCode(beanRegistry); 
    FXNewsProvider newsProvider = ➥
    (FXNewsProvider)container.getBean("djNewsProvider"); 
    newsProvider.getAndPersistNews(); 
} 
public static BeanFactory bindViaCode(BeanDefinitionRegistry registry) 
{ 
    AbstractBeanDefinition newsProvider = ➥
    new RootBeanDefinition(FXNewsProvider.class,true); 
    AbstractBeanDefinition newsListener = ➥
    new RootBeanDefinition(DowJonesNewsListener.class,true); 
    AbstractBeanDefinition newsPersister = ➥
    new RootBeanDefinition(DowJonesNewsPersister.class,true); 
    // 将bean定义注册到容器中
    registry.registerBeanDefinition("djNewsProvider", newsProvider); 
    registry.registerBeanDefinition("djListener", newsListener); 
    registry.registerBeanDefinition("djPersister", newsPersister); 
    // 指定依赖关系
    // 1. 可以通过构造方法注入方式
    ConstructorArgumentValues argValues = new ConstructorArgumentValues(); 
    argValues.addIndexedArgumentValue(0, newsListener); 
    argValues.addIndexedArgumentValue(1, newsPersister); 
    newsProvider.setConstructorArgumentValues(argValues); 
    // 2. 或者通过setter方法注入方式
    MutablePropertyValues propertyValues = new MutablePropertyValues(); 
    propertyValues.addPropertyValue(new ropertyValue("newsListener",newsListener)); 
    propertyValues.addPropertyValue(new PropertyValue("newPersistener",newsPersister)); 
    newsProvider.setPropertyValues(propertyValues); 
    // 绑定完成
    /**
      因为传入的DefaultListableBeanFactory同时实现了BeanFactory和
      BeanDefinitionRegistry接口，所以，这样做强制类型转换不会出现问题。但需要注意的是，单纯
      的BeanDefinitionRegistry是无法强制转换到BeanFactory类型的
    **/
    return (BeanFactory)registry; 
}
```

- 在 main 方法中，**首先构造一个 DefaultListableBeanFactory 作 为 BeanDefinition Registry**，**然后将其交给bindViaCode方法进行具体的对象注册和相关依赖管理**，**然后通过bindViaCode返回的BeanFactory取得需要的对象**，**最后执行相应逻辑**。在我们的实例里，当然就是取得FXNewsProvider进行新闻的处理
- 在bindViaCode方法中，**首先针对相应的业务对象构造与其相对应的BeanDefinition**，使用了 RootBeanDefinition 作 为 BeanDefinition 的实现类。构造完成后，**将这些BeanDefinition注册到通过方法参数传进来的BeanDefinitionRegistry中**。之后，因为我们的FXNewsProvider是采用的构造方法注入，所以，需要**通过ConstructorArgumentValues为其注入相关依赖**。在这里为了同时说明setter方法注入，也同时展示了在Spring中如何使用代码实现setter方法注入。如果要运行这段代码，需要把setter方法注入部分的4行代码注释掉。**最后，以BeanFactory的形式返回已经注册并绑定了所有相关业务对象的BeanDefinitionRegistry实例**

#### 配置文件

- 采用外部配置文件时，Spring的IoC容器有一个统一的处理方式。通常情况下，需要根据不同的外
  部配置文件格式，**给出相应的BeanDefinitionReader实现类**，**由BeanDefinitionReader的相应实**
  **现类负责将相应的配置文件内容读取并映射到BeanDefinition**，然后**将映射后的BeanDefinition注**
  **册到一个BeanDefinitionRegistry**，之后，**BeanDefinitionRegistry即完成Bean的注册和加载**

##### properties文件

```properties
# Properties格式表达的依赖注入配置内容
djNewsProvider.(class)=..FXNewsProvider 
# ----------通过构造方法注入的时候------------- 
djNewsProvider.$0(ref)=djListener 
djNewsProvider.$1(ref)=djPersister 
# ----------通过setter方法注入的时候--------- 
# djNewsProvider.newsListener(ref)=djListener 
# djNewsProvider.newPersistener(ref)=djPersister 
djListener.(class)=..impl.DowJonesNewsListener 
djPersister.(class)=..impl.DowJon
esNewsPersister
```

- **djNewsProvider作为beanName，后面通过.(class)表明对应的实现类是什么**，实际上使用djNewsProvider.class=...的形式也是可以的，但Spring 1.2.6之后不再提倡使用，而**提倡使用.(class)的形式**
- **通过在表示beanName的名称后添加.$[number]后缀的形式，来表示当前beanName对应的对象需要通过构造方法注入的方式注入相应依赖对象**。在这里，我们分别将构造方法的第一个
  参数和第二个参数对应到djListener和djPersister。需要注意的一点，就是**$0和$1后面的(ref)，(ref)用来表示所依赖的是引用对象，而不是普通的类型**。如果不加(ref)，
  PropertiesBeanDefinitionReader会将djListener和djPersister作为简单的String类型进行注入，异常自然不可避免啦
- FXNewsProvider采用的是构造方法注入，而为了演示setter方法注入在Properties配置文件中又 5 
  是一个什么样子，以便于你更全面地了解基于Properties文件的配置方式，我们在下面增加了
  **setter方法注入**的例子，不过进行了注释。实际上，**与构造方法注入最大的区别就是，它不使用数字顺序来指定注入的位置，而使用相应的属性名称来指定注入**。newsListener和
  newPersistener恰好就是我们的FXNewsProvider类中所声明的属性名称。这印证了之前在
  比较构造方法注入和setter方法注入方式不同时提到的差异，**即构造方法注入无法通过参数名称来标识注入的确切位置，而setter方法注入则可以通过属性名称来明确标识注入**。与在
  Properties中表达构造方法注入一样，同样需要注意，如果属性名称所依赖的是引用对象，那
  么一定不要忘了(ref)

```java
//加载Properties配置的BeanFactory的使用演示
public static void main(String[] args) 
{
  DefaultListableBeanFactory beanRegistry = new DefaultListableBeanFactory(); 
  BeanFactory container = (BeanFactory)bindViaPropertiesFile(beanRegistry); 
  FXNewsProvider newsProvider = (FXNewsProvider)container.getBean("djNewsProvider");
  newsProvider.getAndPersistNews(); 
} 

public static BeanFactory bindViaPropertiesFile(BeanDefinitionRegistry registry) 
{ 
  PropertiesBeanDefinitionReader reader = 
  new PropertiesBeanDefinitionReader(registry); 
  reader.loadBeanDefinitions("classpath:../../binding-config.properties"); 
  return (BeanFactory)registry; 
}
```

##### xml文件

- Properties配置文件格式提供PropertiesBeanDefinitionReader相对应，**Spring同样为XML格式的配置文件提供了现成的BeanDefinitionReader实现，即XmlBeanDefinitionReader**。**XmlBeanDefinitionReader负责读取Spring指定格式的XML配置文件并解析，之后将解析后的文件内容映射到相应的BeanDefinition，并加载到相应的BeanDefinitionRegistry中**（在这里是DefaultListableBeanFactory）。这时，整个BeanFactory就可以放给客户端使用了
- Spring在DefaultListableBeanFactory的基础上构建了简化XML格式配置加载的XmlBeanFactory实现
  - XmlBeanFactory现已不再使用

#### 元数据

- **@Autowired的存在将告知Spring容器需要为当前对象注入哪些依赖对象**。而**@Component则是配合Spring 2.5中新的classpath-scanning功能使用的**。现在我们只要再向Spring的配置文件中增加一个“触发器”，使用@Autowired和@Component标注的类就能获得依赖对象的注入了

```xml
<?xml version="1.0" encoding="UTF-8"?> 
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:context="http://www.springframework.org/schema/context"
       xmlns:tx="http://www.springframework.org/schema/tx"
       xsi:schemaLocation="http://www.springframework.org/schema/beans
       		http://www.springframework.org/schema/beans/spring-beans-2.5.xsd
          http://www.springframework.org/schema/context
          http://www.springframework.org/schema/context/spring-context-2.5.xsd
          http://www.springframework.org/schema/tx
          http://www.springframework.org/schema/tx/spring-tx-2.5.xsd"> 
  	<context:component-scan base-package="cn.spring21.project.base.package"/> 
</beans>
```

- `<context:component-scan/>`会到指定的包（package）下面**扫描标注有@Component的类**，如果找到，则将它们**添加到容器进行管理**，并根据它们所标注的@Autowired为**这些类注入符合条件的依赖对象**

## IoC容器的实现

- Spring的IoC容器会**以某种方式加载Configuration Metadata**（通常也就是XML格式的配置信息），然后**根据这些信息绑定整个系统的对象**，最终**组装成一个可用的基于轻量级容器的应用系统**

- Spring的IoC容器实现以上功能的过程，基本上可以按照类似的流程划分为两个阶段，即**容器启动阶段**和**Bean实例化阶段**

![](/Users/ng/Documents/spring&mybatis/pictures/容器功能实现的各个阶段.png)

##### 容器启动阶段

- 容器启动伊始，首先会**通过某种途径加载Configuration MetaData**。除了代码方式比较直接，在大部分情况下，容器需要依赖某些工具类（BeanDefinitionReader）对加载的Configuration MetaData进行解析和分析，并**将分析后的信息编组为相应的BeanDefinition**，最后**把这些保存了bean定义必要信息的BeanDefinition，注册到相应的BeanDefinitionRegistry**，这样容器启动工作就完成了

##### Bean实例化阶段

- 经过第一阶段，现在所有的bean定义信息都通过BeanDefinition的方式注册到了BeanDefinitionRegistry中。**当某个请求方通过容器的getBean方法明确地请求某个对象，或者因依赖关系容器需要隐式地调用getBean方法时，就会触发第二阶段的活动**
- 该阶段，容器会首先检查所请求的对象之前是否已经初始化。**如果没有，则会根据注册的BeanDefinition所提供的信息实例化被请求对象，并为其注入依赖**。如果该对象实现了某些回调接口，也会根据回调接口的要求来装配它。**当该对象装配完毕之后，容器会立即将其返回请求方使用**。如果说第一阶段只是根据图纸装配生产线的话，那么第二阶段就是使用装配好的生产线来生产具体的产品了

### 干预容器启动阶段

- Spring提供了一种叫做**BeanFactoryPostProcessor**的容器扩展机制。该机制**允许我们在容器实例化相应对象之前，对注册到容器的BeanDefinition所保存的信息做相应的修改**。这就相当于在容器实现的第一阶段最后加入一道工序，让我们对最终的BeanDefinition做一些额外的操作，比如修改其中bean定义的某些属性，为bean定义增加其他信息等
- 对于BeanFactory来说，我们需要用手动方式应用所有的BeanFactoryPostProcessor
- 对于ApplicationContext来说，因为**ApplicationContext会自动识别配置文件中的BeanFactoryPostProcessor并应用它**，所以，相对于BeanFactory，在ApplicationContext中加载并应用BeanFactoryPostProcessor，**仅需要在XML配置文件中将这些BeanFactoryPostProcessor简单配置一下即可**

```xml
<beans> 
<bean class="org.springframework.beans.factory.config.PropertyPlaceholderConfigurer"> 
<property name="locations"> 
<list> 
<value>conf/jdbc.properties</value> 
<value>conf/mail.properties</value> 
</list>
</property> 
</bean> 
... 
</beans>
```

#### Spring提供的BeanFactoryPostProcessor实现

##### PropertyPlaceholderConfigurer

- PropertyPlaceholderConfigurer**允许我们在XML配置文件中使用占位符（Placeholder）**，并**将这些占位符所代表的资源单独配置到简单的properties文件中来加载**

```xml
<property name="url"> 
<value>${jdbc.url}</value> 
</property> 
<property name="driverClassName"> 
<value>${jdbc.driver}</value> 
</property> 
<property name="username"> 
<value>${jdbc.username}</value> 
</property> 
<property name="password"> 
<value>${jdbc.password}</value> 
</property>
```

- **当BeanFactory在第一阶段加载完成所有配置信息时，BeanFactory中保存的对象的属性信息还只是以占位符的形式存在**，如${jdbc.url}、${jdbc.driver}。**当PropertyPlaceholderConfigurer作为BeanFactoryPostProcessor被应用时，它会使用properties配置文件中的配置信息来替换相应BeanDefinition中占位符所表示的属性值**。这样，当**进入容器实现的第二阶段实例化bean时，bean定义中的属性值就是最终替换完成的了**
- PropertyPlaceholderConfigurer不单会从其配置的properties文件中加载配置项，同时还**会检查Java的System类中的Properties**，可以通过setSystemPropertiesMode()或者setSystemPropertiesModeName()来控制是否加载或者覆盖System相应Properties的行为

##### PropertyOverrideConfigurer

- 可以**通过PropertyOverrideConfigurer对容器中配置**的任何你想处理的**bean定义的property信息进行覆盖替换**
- PropertyOverrideConfigurer的**properties文件中的配置项，覆盖掉了原来XML中的bean定义的property信息**
- PropertyOverrideConfigurer使用的配置文件示例：
  `beanName.propertyName=Value`
  也就是说，properties文件中的键是**以XML中配置的bean定义的beanName为标志开始**的（通常就是id指定的值），后面**跟着相应被覆盖的property的名称**
- 当容器中配置的多个PropertyOverrideConfigurer对同一个bean定义的同一个property值进
  行处理的时候，**最后一个将会生效**

##### CustomEditorConfigure

- CustomEditorConfigure只是**辅助性地将后期会用到的信息注册到容器，对BeanDefinition没有做任何变动**
- 不管对象是什么类型，也不管这些对象所声明的依赖对象是什么类型，**通常都是通过XML（或者properties）文件格式来配置这些对象类型**。但XML所记载的，都是String类型，即**容器从XML格式的文件中读取的都是字符串形式，最终应用程序却是由各种类型的对象所构成**。要想**完成这种由字符串到具体对象的转换**，**都需要这种转换规则相关的信息**，而CustomEditorConfigurer就是帮助我们传达类似信息的
- Spring内部**通过JavaBean的PropertyEditor来帮助进行String类型到其他类型的转换工作**。只要**为每种对象类型提供一个 PropertyEditor ，就可以根据该对象类型取得与其相对应的PropertyEditor来做具体的类型转换**。Spring容器内部在做具体的类型转换的时候，会**采用JavaBean框架内默认的PropertyEditor搜寻逻辑**，从而**继承了对原生类型以及java.lang.String.java.awt.Color和java.awt.Font等类型的转换支持**。同时，Spring框架还提供了自身实现的一些PropertyEditor，这些PropertyEditor大部分都位于org.springframework. beans.propertyeditors包下
  - **StringArrayPropertyEditor**。该PropertyEditor会**将符合CSV格式的字符串转换成String[]数组的形式，默认是以逗号（,）分隔的字符串**，但可以指定自定义的字符串分隔符
  - ClassEditor。**根据String类型的class名称，直接将其转换成相应的Class对象，相当于通**
    **过Class.forName(String)完成的功效**。可以通过String[]数组的形式传入需转换的值，以
    达到与提供的ClassArrayEditor同样的目的
    - ClassEditor，FileEditor，LocaleEditor，PatternEditor等均**默认加载使用**

##### 给出针对特定对象类型的PropertyEditor实现

- 假设需要**对yyyy/MM/dd形式的日期格式转换提供支持**。虽然可以直接让PropertyEditor实现类去实现java.beans.PropertyEditor接口，不过，我们可以**直接继承java.beans.PropertyEditorSupport类以避免实现java.beans.PropertyEditor接口的所有方法**。就好像这次，我们仅仅让DatePropertyEditor完成从 String 到 java.util.Date 的转换，**只需要实现setAsText(String)方法**，而其他方法一概不管

![image-20201026162419885](/Users/ng/Library/Application Support/typora-user-images/image-20201026162419885.png)

- **通过CustomEditorConfigurer将刚实现的DatePropertyEditor注册到容器，以告知容器按照DatePropertyEditor的形式进行String到java.util.Date类型的转换工作**
- 如果容器是**BeanFactory的实现**，比如XmlBeanFactory，则**需要通过编码手动应用**

```java
XmlBeanFactory beanFactory = new XmlBeanFactory(new ClassPathResource
// 
CustomEditorConfigurer ceConfigurer = new CustomEditorConfigurer(); 
Map customerEditors = new HashMap(); 
customerEditors.put(java.util.Date.class, new DatePropertyEditor())；
ceConfigurer.setCustomEditors(customerEditors); 
// 
ceConfigurer.postProcessBeanFactory(beanFactory);
```

- **ApplicationContext相应实现**，因为ApplicationContext会**自动识别BeanFactoryPostProcessor并应用**，所以**只需要在相应配置文件中配置一下**

### Bean的生命周期

- ApplicationContext启动之后会实例化所有的bean定义，但**ApplicationContext在实现的过程中依然遵循Spring容器实现流程的两个阶段**，只不过它会**在启动阶段的活动完成之后，紧接着调用注册到该容器的所有bean定义的实例化方法getBean()**。这就是为什么当你得到ApplicationContext类型的容器引用时，容器内所有对象已经被全部实例化完成
- 说getBean()方法是有可能触发Bean实例化阶段的活动，是因为**只有当对应某个bean定义的getBean()方法第一次被调用时**，不管是显式的还是隐式的，**Bean实例化阶段的活动才会被触发**，**第二次被调用则会直接返回容器缓存的第一次实例化完的对象实例**（prototype类型bean除外）。当**getBean()方法内部发现该bean定义之前还没有被实例化之后，会通过createBean()方法来进行具体的对象实例化**

![](/Users/ng/Documents/spring&mybatis/pictures/Bean的实例化过程.png)

- Spring容器将对其所管理的对象**全部给予统一的生命周期管理**

#### Bean的实例化与BeanWrapper

- 容器在内部实现的时候，**采用“策略模式（Strategy Pattern）”来决定采用何种方式初始化bean实例**。通常，可以通过反射或者CGLIB动态字节码生成来初始化相应的bean实例或者动态生成其子类。InstantiationStrategy是实例化策略的抽象接口，其直接子类**SimpleInstantiationStrategy实现了简单的对象实例化功能**，可以通过反射来实例化对象实例，但不支持方法注入方式的对象实例化
- **CglibSubclassingInstantiationStrategy**继承了SimpleInstantiationStrategy的以反射方式实例化对象的功能，并且通过CGLIB的动态字节码生成功能，该策略实现类**可以动态生成某个类的子类，进而满足了方法注入所需的对象实例化需求**。默认情况下，容器内部采用的是CglibSubclassingInstantiationStrategy

##### 第一步：实例化Bean对象

- 容器只要根据相应bean定义的BeanDefintion取得实例化信息，结合CglibSubclassingInstantiationStrategy以及不同的bean定义类型，就可以返回实例化完成的对象实例。但是，**不是直接返回构造完成的对象实例，而是以BeanWrapper对构造完成的对象实例进行包裹，返回相应的BeanWrapper实例**

##### 第二步：设置对象属性

- 在第一步结束后返回BeanWrapper实例而不是原先的对象实例，就是为了第二步“设置对象属性”
- **BeanWrapper定义继承了PropertyAccessor接口，可以以统一的方式对对象属性进行访问**；BeanWrapper定义同时又直接或者间接**继承了PropertyEditorRegistry和TypeConverter接口**。**在第一步构造完成对象之后，Spring会根据对象实例构造一个BeanWrapperImpl实例**，然后**将之前CustomEditorConfigurer注册的PropertyEditor复制一份给BeanWrapperImpl实例**（这就是BeanWrapper同时又是PropertyEditorRegistry的原因）。这样，当BeanWrapper转换类型、设置对象属性值时，就不会无从下手了
- 使用BeanWrapper对bean实例操作很方便，可以**免去直接使用Java反射API操作对象实例的烦琐**

```java
Object provider = Class.forName("package.name.FXNewsProvider").newInstance(); 
Object listener = Class.forName("package.name.DowJonesNewsListener").newInstance(); 
Object persister = Class.forName("package.name.DowJonesNewsPersister").newInstance(); 
BeanWrapper newsProvider = new BeanWrapperImpl(provider); 

newsProvider.setPropertyValue("newsListener", listener); 
newsProvider.setPropertyValue("newPersistener", persister); 

assertTrue(newsProvider.getWrappedInstance() instanceof FXNewsProvider); 
assertSame(provider, newsProvider.getWrappedInstance()); 
assertSame(listener, newsProvider.getPropertyValue("newsListener")); 
assertSame(persister, newsProvider.getPropertyValue("newsPersister")); 
```

#### Aware接口

- 当对象实例化完成并且相关属性以及依赖设置完成之后，Spring容器会检查当前对象实例是否实现了一系列的以Aware命名结尾的接口定义。如果是，则将这些Aware接口定义中规定的依赖注入给当前对象实例
- BeanFactory的Aware接口
  - BeanNameAware。如果Spring容器检测到当前对象实例实现了该接口，会**将该对象实例的bean定义对应的beanName设置到当前对象实例**
  - BeanClassLoaderAware。如果容器检测到当前对象实例实现了该接口，会**将对应加载当前bean的Classloader注入当前对象实例**。默认会使用加载org.springframework.util.ClassUtils类的Classloader
  - BeanFactoryAware。在介绍方法注入的时候，我们提到过使用该接口以便每次获取prototype类型bean的不同实例。**如果对象声明实现了BeanFactoryAware接口，BeanFactory容器会将自身设置到当前对象实例**。这样，**当前对象实例就拥有了一个BeanFactory容器的引用，并且可以对这个容器内允许访问的对象按照需要进行访问**
- ApplicationContext的Aware接口
  - ResourceLoaderAware。ApplicationContext**实现了Spring的ResourceLoader接口**。当容器检测到当前对象实例实现了ResourceLoaderAware接口之后，会将当前ApplicationContext自身设置到对象实例，这样**当前对象实例就拥有了其所在ApplicationContext容器的一个引用**
  - ApplicationEventPublisherAware。ApplicationContext作为一个容器，同时还实现了ApplicationEventPublisher接口，这样，它就**可以作为ApplicationEventPublisher来使用**。所以，**当前ApplicationContext容器如果检测到当前实例化的对象实例声明了ApplicationEventPublisherAware接口，则会将自身注入当前对象**
  - MessageSourceAware。**ApplicationContext通过MessageSource接口提供国际化的信息支持**，即I18n（Internationalization）。它自身就实现了MessageSource接口，所以**当检测到当前对象实例实现了MessageSourceAware接口，则会将自身注入当前对象实例**
  - org.springframework.context.ApplicationContextAware。 **如果ApplicationContext容器检测到当前对象实现了ApplicationContextAware接口，则会将自身注入当前对象实例**

#### BeanPostProcessor

- **BeanPostProcessor**存在于**对象实例化阶段**，而**BeanFactoryPostProcessor**则存在于**容器启动阶段**
- 与BeanFactoryPostProcessor通常会处理容器内所有符合条件的BeanDefinition类似，BeanPostProcessor**会处理容器内所有符合条件的实例化后的对象实例**
- **postProcessBeforeInitialization()方法**是图4-10中BeanPostProcessor**前置处理**这一步将会执行的方法，**postProcessAfterInitialization()**则是对应图4-10中BeanPostProcessor**后置处理**那一步将会执行的方法。BeanPostProcessor的两个方法中**都传入了原来的对象实例的引用**，这为我们扩展容器的对象实例化过程中的行为提供了极大的便利，我们几乎可以对传入的对象实例执行任何的操作
- 通常比较常见的使用BeanPostProcessor的场景，是**处理标记接口实现类，或者为当前对象提供代理实现**。在图4-10的第三步中，ApplicationContext对应的那些**Aware接口实际上就是通过BeanPostProcessor的方式进行处理的**

#### InitializingBean和init-method

- InitializingBean是容器内部广泛使用的一个**对象生命周期标识接口**
  - 其作用在于，在对象实例化过程调用过“BeanPostProcessor的前置处理”之后，会接着检测当前对象是否实现了InitializingBean接口，如果**是，则会调用其afterPropertiesSet()方法进一步调整对象实例的状态**。比如，在有些情况下，某个业务对象实例化完成后，还不能处于可以使用状态。这个时候就可以让该业务对象实现该接口，并在方法afterPropertiesSet()中完成对该业务对象的后续处理
  - 让业务对象实现这个接口，则显得Spring容器比较具有侵入性
- Spring还提供了另一种方式来指定自定义的对象初始化操作，那就是在XML配置的时候，使用`<bean>`的init-method属性
  - 通过init-method，系统中业务对象的自定义初始化操作可以以任何方式命名，而不再受制于InitializingBean的afterPropertiesSet()。如果系统开发过程中规定：所有业务对象的自定义初始化操作都必须以init()命名，为了省去挨个`<bean>`的设置init-method这样的烦琐，我们还**可以通过最顶层的`<beans>`的default-init-method统一指定这一init()方法名**
  - 一般，我们是在集成第三方库，或者其他特殊的情况下，才会需要使用该特性

#### DisposableBean与destroy-method

- 当所有的一切，该设置的设置，该注入的注入，该调用的调用完成之后，容器**将检查singleton类型的bean实例，看其是否实现了DisposableBean接口**。**或者其对应的bean定义是否通过`<bean>`的destroy-method属性指定了自定义的对象销毁方法**。如果**是，就会为该实例注册一个用于对象销毁的回调**（Callback），**以便在这些singleton类型的对象实例销毁之前，执行销毁逻辑**
- 与InitializingBean和init-method用于对象的自定义初始化相对应，**DisposableBean和destroy-method为对象提供了执行自定义销毁逻辑的机会**
- 最常见到的该功能的使用场景就是**在Spring容器中注册数据库连接池**
- 不过，这些**自定义的对象销毁逻辑，在对象实例初始化完成并注册了相关的回调方法之后，并不会马上执行**。**回调方法注册后，返回的对象实例即处于使用状态，只有该对象实例不再被使用的时候，才会执行相关的自定义销毁逻辑**，此时**通常也就是Spring容器关闭的时候**。但**Spring容器在关闭之前，不会自动调用这些回调方法。所以，需要我们告知容器，在哪个时间点来执行对象的自定义销毁方法**
- 对于BeanFactory容器来说。我们**需要在独立应用程序的主程序退出之前**，或者其他被认为是合适的情况下（依照应用场景而定），**调用ConfigurableBeanFactory提供的destroySingletons()方法销毁容器中管理的所有singleton类型的对象实例**
- 对于ApplicationContext容器来说，它为我们提供了**registerShutdownHook()方法**，该方法**底层使用标准的Runtime类的addShutdownHook()方式来调用相应bean对象的销毁逻辑**，从而**保证在Java虚拟机退出之前，这些singtleton类型的bean对象实例的自定义销毁逻辑会被执行**

## ApplicationContext

- 作为Spring提供的较之BeanFactory更为先进的IoC容器实现，ApplicationContext除了拥有BeanFactory支持的所有功能之外，还进一步扩展了基本容器的功能，包括BeanFactoryPostProcessor、BeanPostProcessor以及其他特殊类型bean的自动识别、容器启动后bean实例的自动初始化、国际化的信息支持、容器内事件发布等

### 统一资源加载策略

##### 使用原因

- java SE提供的URL全名是Uniform Resource Locator（统一资源定位器）。首先，说是统一资源定位，但基本实现却**只限于网络形式发布的资源的查找和定位工作**，基本上只提供了基于HTTP、FTP、File等协议的资源定位功能
- 资源这个词的范围比较广义，资源**可以任何形式存在**，如以**二进制对象**形式存在、以**字节流**形式存在、以**文件**形式存在等；而且，资源也**可以存在于任何场所**，如存在于**文件系统**、存在于**Java应用的Classpath**中，甚至存在于**URL可以定位的地方**

#### Spring中的Resource

- Spring框架内部使用**Resource接口作为所有资源的抽象和访问接口**，我们之前在构造BeanFactory的时候已经接触过它，如下代码：
  - ``BeanFactory beanFactory = new XmlBeanFactory(new ClassPathResource("..."));` 
  - 其中**ClassPathResource**就是Resource的一个特定类型的实现，**代表的是位于Classpath中的资源**
- Resource接口可以根据资源的不同类型，或者资源所处的不同场合，给出相应的具体实现。Spring框架在这个理念的基础上，提供了一些实现类（可以在org.springframework.core.io包下找到这些实现类）
  - **ByteArrayResource**。将字节（byte）数组提供的数据作为一种资源进行封装，如果通过InputStream形式访问该类型的资源，该实现**会根据字节数组的数据，构造相应的ByteArrayInputStream并返回**
  - ClassPathResource。该实现**从Java应用程序的ClassPath中加载具体资源并进行封装**，可以使用指定的类加载器（ClassLoader）或者给定的类进行资源加载
  - FileSystemResource。对java.io.File类型的封装，所以，我们**可以以文件或者URL的形式对该类型资源进行访问**，只要能跟File打的交道，基本上跟FileSystemResource也可以
  - UrlResource。**通过java.net.URL进行的具体资源查找定位的实现类**，内部委派URL进行具体的资源操作
  - InputStreamResource。将给定的InputStream视为一种资源的Resource实现类，较为少用。可能的情况下，以ByteArrayResource以及其他形式资源实现代之
- 想实现自定义的Resource，我们可以继承AbstractResource抽象类，然后根据当前具体资源特征，覆盖相应的方法就可以了

#### “更广义的URL”——ResourceLoader

- 资源是有了，但如何去查找和定位这些资源，则应该是ResourceLoader的职责所在了

- **ResourceLoader接口是资源查找定位策略的统一抽象**，具体的资源查找定位策略则由相应的ResourceLoader实现类给出

  - 把ResourceLoader称作**统一资源定位器**或许才更恰当一些

- ResourceLoader定义如下

  ```java
  public interface ResourceLoader { 
      String CLASSPATH_URL_PREFIX = ResourceUtils.CLASSPATH_URL_PREFIX; 
      Resource getResource(String location); 
      ClassLoader getClassLoader(); 
  } 
  ```

  - 其中**最主要的就是Resource getResource(String location)方法**，通过它，我们就**可以根据指定的资源位置，定位到具体的资源实例**

##### DefaultResourceLoader

- ResourceLoader有一个默认的实现类，即DefaultResourceLoader，该类默认的资源查找处理逻辑如下
  1. 首先**检查资源路径是否以classpath:前缀打头**，如果是，则**尝试构造ClassPathResource类型资源并返回**
  2. **否则尝试通过URL，根据资源路径来定位资源**，如果没有抛出MalformedURLException，**有则会构造UrlResource类型的资源并返回**
     1. 如果还是无法根据资源路径定位指定的资源，则**委派getResourceByPath(String) 方法来定位**， DefaultResourceLoader 的getResourceByPath(String)方法默认实现逻辑是，构造ClassPathResource类型的资源并返回

```java
Resource fakeFileResource = resourceLoader.getResource("D:/spring21site/README"); 
assertTrue(fakeFileResource instanceof ClassPathResource); 
assertFalse(fakeFileResource.exists()); 
Resource urlResource1 = resourceLoader.getResource("file:D:/spring21site/README"); 
assertTrue(urlResource1 instanceof UrlResource); 
Resource urlResource2 = resourceLoader.getResource("http://www.spring21.cn"); 
assertTrue(urlResource2 instanceof UrlResource); 
try{ 
fakeFileResource.getFile();
fail("no such file with path["+fakeFileResource.getFilename()+"] exists in classpath"); 
} 
catch(FileNotFoundException e){ 
// 
} 
try{ 
  urlResource1.getFile();
} 
catch(FileNotFoundException e){ 
  fail();
}
```

- **fakeFileResource资源的类型**，并不是我们所预期的FileSystemResource类型，而**是ClassPathResource类型**，这是由DefaultResourceLoader的资源查找逻辑所决定的。**如果最终没有找到符合条件的相应资源，getResourceByPath(String)方法就会构造一个实际上并不存在的资源并返回**。而指定有协议前缀的资源路径，则通过URL能够定位，所以，**返回的都是UrlResource类型**

##### FileSystemResourceLoader

- 为了**避免DefaultResourceLoader在最后getResourceByPath(String)方法上的不恰当处理**，我们可以使用**FileSystemResourceLoader，它继承自DefaultResourceLoader，但覆写getResourceByPath(String)方法**，**使之从文件系统加载资源并以FileSystemResource类型返回**。这样，我们就可以取得预想的资源类型

```java
public void testResourceTypesWithFileSystemResourceLoader() 
{ 
ResourceLoader resourceLoader = new FileSystemResourceLoader(); 
Resource fileResource = resourceLoader.getResource("D:/spring21site/README"); 
assertTrue(fileResource instanceof FileSystemResource); 
assertTrue(fileResource.exists()); 
Resource urlResource = resourceLoader.getResource("file:D:/spring21site/README"); 
assertTrue(urlResource instanceof UrlResource); 
}
```

- FileSystemResourceLoader在ResourceLoader家族中的兄弟**FileSystemXmlApplicationContext**，也是**覆写了getResourceByPath(String)方法的逻辑**，以改变DefaultResourceLoader的默认资源加载行为，**最终从文件系统中加载并返回FileSystemResource类型的资源**

##### ResourcePatternResolver ——批量查找的ResourceLoader

- ResourcePatternResolver是ResourceLoader的扩展，**ResourceLoader每次只能根据资源路径返回确定的单个Resource实例**，而**ResourcePatternResolver则可以根据指定的资源路径匹配模式，每次返回多个Resource实例**。接口ResourcePatternResolver定义如下：

```java
public interface ResourcePatternResolver extends ResourceLoader { 
    String CLASSPATH_ALL_URL_PREFIX = "classpath*:"; 
    Resource[] getResources(String locationPattern) throws IOException; 
} 
```

- ResourcePatternResolver在继承ResourceLoader原有定义的基础上，又引入了Resource[] getResources(String)方法定义，以支持根据路径匹配模式返回多个Resources的功能。它同时还引入了一种新的协议前缀classpath\*:，针对这一点的支持，将由相应的子类实现给出
- ResourcePatternResolver最常用的一个实现是**PathMatchingResourcePatternResolver**，该实现类**支持ResourceLoader级别的资源加载，支持基于Ant风格的路径匹配模式（类似于\*\*/\*.suffix之类的路径形式），支持ResourcePatternResolver新增加的classpath\*:前缀等**，基本上集所有技能于一身
- 在构造PathMatchingResourcePatternResolver实例的时候，可以指定一个ResourceLoader，如果不指定的话，则PathMatchingResourcePatternResolver内部会默认构造一个DefaultResourceLoader实例。PathMatchingResourcePatternResolver内部会将匹配后确定的资源路径，委派给它的ResourceLoader来查找和定位资源。这样，**如果不指定任何ResourceLoader的话，PathMatchingResourcePatternResolver在加载资源的行为上会与DefaultResourceLoader基本相同，只存在返回的Resource数量上的差异**。如下代码表明了二者在资源加载行为上的一致性：

```java
ResourcePatternResolver resourceResolver = new PathMatchingResourcePatternResolver(); 
Resource fileResource = resourceResolver.getResource("D:/spring21site/README"); 
assertTrue(fileResource instanceof ClassPathResource); 
assertFalse(fileResource.exists()); 
... 
```

- 不过，**可以通过传入其他类型的ResourceLoader来替换PathMatchingResourcePatternResolver内部默认使用的DefaultResourceLoader，从而改变其默认行为**
  - 使用FileSystemResourceLoader替换默认的DefaultResourceLoader，从而使得PathMatchingResourcePatternResolver的行为跟使用FileSystemResourceLoader一样

```java
public void testResourceTypesWithPathMatchingResourcePatternResolver() 
{ 
ResourcePatternResolver resourceResolver = new PathMatchingResourcePatternResolver(); 
Resource fileResource = resourceResolver.getResource("D:/spring21site/README"); 
assertTrue(fileResource instanceof ClassPathResource); 
assertFalse(fileResource.exists()); 
resourceResolver = new PathMatchingResourcePatternResolver(new ➥
FileSystemResourceLoader()); 
fileResource = resourceResolver.getResource("D:/spring21site/README"); 
assertTrue(fileResource instanceof FileSystemResource); 
assertTrue(fileResource.exists()); 
}
```

#### ApplicationContext与ResourceLoader

- **ApplicationContext**继承了ResourcePatternResolver，当然就**间接实现了ResourceLoader接口**。所以，**任何的ApplicationContext实现都可以看作是一个ResourceLoader甚至ResourcePatternResolver**。而**这就是ApplicationContext支持Spring内统一资源加载策略的真相**

![image-20201104112529095](/Users/ng/Library/Application Support/typora-user-images/image-20201104112529095.png)

##### 扮演ResourceLoader的角色

- 既然ApplicationContext可以作为ResourceLoader或者ResourcePatternResolver来使用，那么，很显然，我们可以通过ApplicationContext来加载任何Spring支持的Resource类型

```java
ResourceLoader resourceLoader = new ClassPathXmlApplicationContext("配置文件路径"); 
// 或者
// ResourceLoader resourceLoader = new FileSystemXmlApplicationContext("配置文件路径"); 
Resource fileResource = resourceLoader.getResource("D:/spring21site/README"); 
assertTrue(fileResource instanceof ClassPathResource); 
assertFalse(fileResource.exists()); 
Resource urlResource2 = resourceLoader.getResource("http://www.spring21.cn"); 
assertTrue(urlResource2 instanceof UrlResource);
```

##### ResourceLoader类型的注入

- 在大部分情况下，如果**某个bean需要依赖于ResourceLoader来查找定位资源**，我们可以为其**注入容器中声明的某个具体的ResourceLoader实现**，该bean也无需实现任何接口，**直接通过构造方法注入或者setter方法注入规则声明依赖即可**，这样处理是比较合理的。不过，如果你不介意你的bean定义依赖于Spring的API，那不妨考虑用一下Spring提供的便利
- ApplicationContext容器本身就是一个ResourceLoader，我们为了该类还需要单独提供一个resourceLoader实例就有些多于了，**直接将当前的ApplicationContext容器作为ResourceLoader注入**

```java
public class FooBar implements ResourceLoaderAware{ 
private ResourceLoader resourceLoader; 
public void foo(String location) 
{ 
System.out.println(getResourceLoader().getResource(location).getClass()); 
}
public ResourceLoader getResourceLoader() { 
return resourceLoader;
}
  public void setResourceLoader(ResourceLoader resourceLoader) { 
this.resourceLoader = resourceLoader; 
} 
} 
public class FooBar implements ApplicationContextAware{ 
private ResourceLoader resourceLoader; 
public void foo(String location) 
{ 
System.out.println(getResourceLoader().getResource(location).getClass()); 
} 
public ResourceLoader getResourceLoader() { 
return resourceLoader; 
} 
public void setApplicationContext(ApplicationContext ctx) ➥
throws BeansException { 
this.resourceLoader = ctx; 
} 
}
```

- 剩下的就是直接将FooBar配置到bean定义文件即可，如下所示

```xml
<bean id="fooBar" class="...FooBar"> 
</bean>
```

- 现在，**容器启动的时候，就会自动将当前ApplicationContext容器本身注入到FooBar中**，因为ApplicationContext类型容器可以自动识别Aware接口

##### Resource类型的注入

- 我们之前讲过，容器可以将bean定义文件中的字符串形式表达的信息，正确地转换成具体对象定义的依赖类型。对于那些Spring容器提供的默认的PropertyEditors无法识别的对象类型，我们可以提供自定义的PropertyEditor实现并注册到容器中，以供容器做类型转换的时候使用。默认情况下，BeanFactory容器不会为Resource类型提供相应的PropertyEditor，所以，如果我们想注入Resource类型的bean定义，就需要注册自定义的PropertyEditor到BeanFactory容器。不过，对于ApplicationContext来说，我们无需这么做，因为**ApplicationContext容器可以正确识别Resource类型并转换后注入相关对象**
- **直接在配置文件中以String形式指定template所在位置，ApplicatonContext就可以正确地转换类型并注入依赖**

```xml
<bean id="mailer" class="...XMailer"> 
<property name="template" value="..resources.default_template.vm"/> 
... 
</bean>
```

- ApplicationContext启动伊始，**会通过一个ResourceEditorRegistrar来注册Spring提供的针对Resource类型的PropertyEditor实现到容器中**，这个PropertyEditor叫做 ResourceEditor。这样，ApplicationContext就可以正确地识别Resource类型的依赖了。ResourceEditor怎么实现：**把配置文件中的路径让ApplicationContext作为ResourceLoader定位一下**

- 如果应用对象需要**依赖一组Resource**，与ApplicationContext注册了ResourceEditor类似，**Spring提供了ResourceArrayPropertyEditor实现，我们只需要通过CustomEditorConfigurar告知容器即可**

##### 在特定情况下，ApplicationContext的Resource加载行为

- 我们下面主要讨论两种类型的ApplicationContext容器，即ClassPathXmlApplicationContext和FileSystemXmlApplicationContext

- 我们知道，对于URL所接受的资源路径来说，通常开始都会有一个协议前缀，比如file:、http:、ftp:等。既然Spring使用UrlResource对URL定位查找的资源进行了抽象，那么，同样也支持这样类型的资源路径，而且，在这个基础上，Spring还扩展了协议前缀的集合。ResourceLoader中增加了一种新的资源路径协议——classpath:，ResourcePatternResolver又增加了一种——classpath*:。这样，我们就可以**通过这些资源路径协议前缀，明确地告知Spring容器要classpath中加载资源**

  `// 代码中使用协议前缀
  ResourceLoader resourceLoader = new ➥
  FileSystemXmlApplicationContext("classpath:conf/container-conf.xml"); 
  // 配置中使用协议前缀
  <bean id="..." class="..."> 
  ean id="..." class="..
  <value>classpath:resource/template.vm</value> 
  </property> 
  </bean>`

- **classpath*:与classpath:的唯一区别就在于，如果能够在classpath中找到多个指定的资源，则返回多个**。我们可以通过这两个前缀改变某些ApplicationContext实现类的默认资源加载行为

- ClassPathXmlApplicationContext和FileSystemXmlApplicationContext在处理资源加载的默认行为上有所不同

  - **当ClassPathXmlApplicationContext在实例化的时候，即使没有指明classpath:或者classpath*:等前缀，它会默认从classpath中加载bean定义配置文件**

  - 而**FileSystemXmlApplicationContext则有些不同，它会尝试从文件系统中加载bean定义文件**

  - **通过在资源路径之前增加classpath:前缀，明确指定**
    **FileSystemXmlApplicationContext从classpath中加载bean定义的配置文件**

    `ApplicationContext ctx = new FileSystemXmlApplicationContext("classpath:conf/appContext.xml");`

  - 这时，**FileSystemXmlApplicationContext就是从Classpath中加载配置，而不是从文件系统中加载**。也就是说，它现在对应的是ClassPathResource类型的资源，而不是默认的FileSystemResource类型资源。FileSystemXmlApplicationContext之所以如此，是因为它与FileSystemResourceLoader一样，也**覆写了DefaultResourceLoader的getResourceByPath(String)方法**，逻辑跟 FileSystemResourceLoader一模一样

  - 对于ClassPathXmlApplicationContext来说，如果我们**不指定路径之前的前缀**，它也不会像资源路径所表现的那样，从文件系统加载资源，而是像实例化时候的行为一样，**从Classpath中加载这种没有路径前缀的资源**

