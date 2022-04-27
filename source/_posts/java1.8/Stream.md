### Stream 简介

- Java 8 中，引入了流（Stream）的概念，这个流和以前我们使用的 IO 中的流并不太相同
- 所有继承自 `Collection` 的接口都可以转换为 `Stream`

#### 例

假设我们有一个 `List` 包含一系列的 `Person`，`Person` 有姓名 `name` 和年龄 `age` 连个字段。现要求这个列表中年龄大于 20 的人数

通常按照以前我们可能会这么写：

```java
long count = 0;
for (Person p : persons) {
	if (p.getAge() > 20) {
		count ++;
	}
}
```

但如果使用 `stream` 的话，则会简单很多:

```java
long count = persons.stream()
					.filter(person -> person.getAge() > 20)
 					.count();
```

这只是 `stream` 的很简单的一个用法。现在链式调用方法算是一个主流，这样写也更利于阅读和理解编写者的意图，一步方法做一件事

### Stream 常用操作

- `Stream` 的方法分为两类。一类叫惰性求值，一类叫及早求值。
- 如果返回值是 Stream，那么是惰性求值
  - 如果调用惰性求值方法，`Stream` 只是记录下了这个惰性求值方法的过程，并没有去计算，等到调用及早求值方法后，就连同前面的一系列惰性求值方法顺序进行计算，返回结果

通用形式为：

```java
Stream.惰性求值.惰性求值. ... .惰性求值.及早求值
```

整个过程和建造者模式有共通之处。建造者模式使用一系列操作设置属性和配置，最后调 用一个 build 方法，这时，对象才被真正创建

#### collect(toList())

`collect(toList())` 方法由 `Stream` 里的值生成一个列表，是一个及早求值操作，可以理解为 `Stream` 向 `Collection` 的转换

> 注意这边的 `toList()` 其实是 `Collectors.toList()`，因为采用了静态倒入，看起来显得简洁。

```java
List<String> collected = Stream.of("a", "b", "c")
							   .collect(Collectors.toList());
assertEquals(Arrays.asList("a", "b", "c"), collected);
```

#### map

如果有一个函数可以将一种类型的值转换成另外一种类型，`map` 操作就可以使用该函数，将一个流中的值转换成一个新的流。

![img](https://images2015.cnblogs.com/blog/1046505/201610/1046505-20161020160120326-1422995090.png)

```java
List<String> collected = Stream.of("a", "b", "hello")
							   .map(string -> string.toUpperCase())
							   .collect(toList());
assertEquals(asList("A", "B", "HELLO"), collected);
```

`map` 方法就是接受的一个 `Function` 的匿名函数类，进行的转换。

![img](https://images2015.cnblogs.com/blog/1046505/201610/1046505-20161020160134904-1768291565.png)

#### filter

遍历数据并检查其中的元素时，可尝试使用 `Stream` 中提供的新方法 `filter`。

![img](https://images2015.cnblogs.com/blog/1046505/201610/1046505-20161020160150935-2057482897.png)

```java
List<String> beginningWithNumbers = 
		Stream.of("a", "1abc", "abc1")
			  .filter(value -> isDigit(value.charAt(0)))
			  .collect(toList());
assertEquals(asList("1abc"), beginningWithNumbers);
```

`filter` 方法就是接受的一个 `Predicate` 的匿名函数类，判断对象是否符合条件，符合条件的才保留下来。

![img](https://images2015.cnblogs.com/blog/1046505/201610/1046505-20161020160211779-1658294349.png)

#### flatMap

`flatMap` 方法可用 `Stream` 替换值，然后将多个 `Stream` 连接成一个 `Stream`。

![img](https://images2015.cnblogs.com/blog/1046505/201610/1046505-20161020160225482-436563893.png)

```java
List<Integer> together = Stream.of(asList(1, 2), asList(3, 4))
							   .flatMap(numbers -> numbers.stream())
							   .collect(toList());
assertEquals(asList(1, 2, 3, 4), together);
```

`flatMap` 最常用的操作就是合并多个 `Collection`。

#### max和min

`Stream` 上常用的操作之一是求最大值和最小值。Stream API 中的 `max` 和 `min` 操作足以解决这一问题。

```java
List<Integer> list = Lists.newArrayList(3, 5, 2, 9, 1);
int maxInt = list.stream()
				 .max(Integer::compareTo)
				 .get();
int minInt = list.stream()
				 .min(Integer::compareTo)
				 .get();
assertEquals(maxInt, 9);
assertEquals(minInt, 1);
```

这里有 2 个要点需要注意：

1. `max` 和 `min` 方法返回的是一个 `Optional` 对象（和 Google Guava 里的 Optional 对象是一样的）。`Optional` 对象封装的就是实际的值，可能为空，所以保险起见，可以先用 `isPresent()` 方法判断一下。`Optional` 的引入就是为了解决方法返回 `null` 的问题。
2. `Integer::compareTo` 也是属于 Java 8 引入的新特性，叫做 **方法引用（Method References）**。在这边，其实就是 `(int1, int2) -> int1.compareTo(int2)` 的简写，可以自己查阅了解，这里不再多做赘述。

#### reduce

`reduce` 操作可以实现从一组值中生成一个值。在上述例子中用到的 `count`、`min` 和 `max` 方法,因为常用而被纳入标准库中。事实上，这些方法都是 `reduce` 操作。

![img](https://images2015.cnblogs.com/blog/1046505/201610/1046505-20161020160243217-1836446247.png)

上图展示了 `reduce` 进行累加的一个过程。具体的代码如下：

```java
int result = Stream.of(1, 2, 3, 4)
				   .reduce(0, (acc, element) -> acc + element);
assertEquals(10, result);
```

注意 `reduce` 的第一个参数，这是一个初始值。`0 + 1 + 2 + 3 + 4 = 10`。

如果是累乘，则为：

```java
int result = Stream.of(1, 2, 3, 4)
				   .reduce(1, (acc, element) -> acc * element);
assertEquals(24, result);
```

### 数据并行化操作

`Stream` 的并行化也是 Java 8 的一大亮点。数据并行化是指将数据分成块，为每块数据分配单独的处理单元。这样可以充分利用多核 CPU 的优势。

并行化操作流只需改变一个方法调用。如果已经有一个 `Stream` 对象，调用它的 `parallel()` 方法就能让其拥有并行操作的能力。如果想从一个集合类创建一个流，调用 `parallelStream()` 就能立即获得一个拥有并行能力的流。

```java
int sumSize = Stream.of("Apple", "Banana", "Orange", "Pear")
					.parallel()
					.map(s -> s.length())
					.reduce(Integer::sum)
					.get();
assertEquals(sumSize, 21);
```

这里求的是一个字符串列表中各个字符串长度总和。

![img](https://images2015.cnblogs.com/blog/1046505/201610/1046505-20161020160307935-1086007264.png)

如果你去计算这段代码所花的时间，很可能比不加上 `parallel()` 方法花的时间更长。这是因为数据并行化会先对数据进行分块，然后对每块数据开辟线程进行运算，这些地方会花费额外的时间。并行化操作只有在 **数据规模比较大** 或者 **数据的处理时间比较长** 的时候才能体现出有事，所以并不是每个地方都需要让数据并行化，应该具体问题具体分析。

### 其他

#### 收集器

`Stream` 转换为 `List` 是很常用的操作，其他 `Collectors` 还有很多方法，可以将 `Stream` 转换为 `Set`, 或者将数据分组并转换为 `Map`，并对数据进行处理。也可以指定转换为具体类型，如 `ArrayList`, `LinkedList` 或者 `HashMap`。甚至可以自定义 `Collectors`，编写自己的收集器。

#### 元素顺序

另外一个尚未提及的关于集合类的内容是流中的元素以何种顺序排列。一些集合类型中的元素是按顺序排列的，比如 List；而另一些则是无序的，比如 HashSet。增加了流操作后，顺序问题变得更加复杂。

总之记住。如果集合本身就是无序的，由此生成的流也是无序的。一些中间操作会产生顺序，比如对值做映射时，映射后的值是有序的，这种顺序就会保留 下来。如果进来的流是无序的，出去的流也是无序的。

如果我们需要对流中的数据进行排序，可以调用 `sorted` 方法：

```java
List<Integer> list = Lists.newArrayList(3, 5, 1, 10, 8);
List<Integer> sortedList = list.stream()
							   .sorted(Integer::compareTo)
							   .collect(Collectors.toList());
assertEquals(sortedList, Lists.newArrayList(1, 3, 5, 8, 10));
```

#### @FunctionalInterface

我们讨论过函数接口定义的标准，但未提及 @FunctionalInterface 注释。事实上，每个用作函数接口的接口都应该添加这个注释。

但 Java 中有一些接口，虽然只含一个方法，但并不是为了使用 Lambda 表达式来实现的。比如，有些对象内部可能保存着某种状态，使用带有一个方法的接口可能纯属巧合。

该注释会强制 javac 检查一个接口是否符合函数接口的标准。如果该注释添加给一个枚举类型、类或另一个注释，或者接口包含不止一个抽象方法，javac 就会报错。重构代码时，使用它能很容易发现问题。