## streams

一个 `java.util.Stream` 代表着能被执行一次或多次操作的元素序列。流操作分为中间操作或者最终操作两种，最终操作返回一特定类型的计算结果，而中间操作返回流本身，这样你就可以将多个操作依次串起来。流的创建需要指定一个数据源，比如 `java.util.Collection` 的子类，List 或者 Set， Map 不支持。Stream 的操作可以串行执行或者并行执行。

我们先来看看顺序流是如何工作的。首先，我们以字符串列表的形式创建一个示例源：

```java
List<String> stringCollection = new ArrayList<>();
stringCollection.add("ddd2");
stringCollection.add("aaa2");
stringCollection.add("bbb1");
stringCollection.add("aaa1");
stringCollection.add("bbb3");
stringCollection.add("ccc");
stringCollection.add("bbb2");
stringCollection.add("ddd1");
```



Java8 扩展了集合类，可以通过 Collection.stream() 或者 Collection.parallelStream() 来创建一个 Stream。下面几节将详细解释常用的 Stream 操作：



#### Filter（过滤）

过滤通过一个 predicate 接口来过滤并只保留符合条件的元素，该操作属于中间操作，所以我们可以再对过滤后的结果来应用其他 Stream 操作（比如 forEach）。 forEach 接收一个函数来对过滤后的元素依次执行。 forEach 是一个最终操作，所以我们不能在 forEach 之后来执行其他 Stream 操作。

```java
stringCollection
	.stream()
	.filter((s) -> s.startsWith("a"))
	.forEach(System.out::println);
	
// "aaa2", "aaa1"
```



forEach 是为 lambda 而设计的，保持了最紧凑的风格。而且 Lambda 表达式本身是可以重用的，非常方便。



#### Sorted（排序）

排序是一个中间操作，返回的是排序好后的 Stream。如果你不指定一个自定义的 Comparator 则会使用默认排序。

```java
// 测试 Sort（排序）
stringCollection
	.stream()
	.sorted()
	.filter((s) -> s.startsWith("a"))
	.forEach(System.out::println);	

// "aaa1", "aaa2"
```



需要注意的是，排序只创建了一个排列好的 Stream，而不会影响原有的数据源，排序之后原数据 stringCollection 是不会被修改的：

```java
System.out.println(stringCollection);
// ddd2, aaa2, bbb1, aaa1, bbb3, ccc, bbb2, ddd1
```



#### Map

中间操作 map 会将元素根据指定的 Function 接口来一次将元素转换为另外的对象。

下面的示例展示了将字符串转换为大写字符串。你也可以通过 map 来将对象转换成其他类型， map 返回的 Stream 类型是根据你 map 传递进去的函数的返回值决定的。

```java
// 测试 Map 操作
stringCollection
	.stream()
	.map(String::toUpperCase)
	.sorted((a, b) -> b.compareTo(a))
	.forEach(System.out::println);

// "DDD2", "DDD1", "CCC", "BBB3", "BBB2", "AAA2", "AAA1"
```



#### Match（匹配）

Stream 提供了多种匹配操作，允许检测指定的 Predicate 是否匹配整个 Stream。所有的匹配操作都是最终操作，并返回一个 boolean 类型的值。

```java
// 测试 Match（匹配）操作
boolean anyStartsWithA = 
	stringCollection
	.stream()
	.anyMatch((s) -> s.startsWith("a"));
	
System.out.println(anyStartsWithA); // true

boolean allStartsWithA = 
	stringCollection
	.stream()
	.allMatch((s) -> s.startsWith("a"));
	
System.out.println(allStartsWithA); // false

boolean noneStartsWithZ = 
	stringCollection
	.stream()
	.noneMatc(h(s) -> s.startsWith("z"));
	
System.out.println(allStartsWithA); // false
```



#### Count（计数）

计数是一个最终操作，返回 Stream 中元素的个数，返回值类型是 long。

```java
// 测试 Count（计数）操作
long startsWithB =
	stringCollection
	.stream()
	.filter((s) -> s.startsWith("b"))
	.count();
	
System.out.println(startsWithB); // 3
```



#### Reduce（规约）

这是一个最终操作，允许通过指定的函数来将 stream 中的多个元素规约为一个元素，规约后的结果是通过 Optional 接口表示的：

```java
// 测试 Reduce（规约）操作
Optional<String> reduced = 
	stringCollection
	.stream()
	.sorted()
	.reduce((s1, s2) -> s1 + "#" + s2);
	
reduced.ifPresent(System.out::println);
// "aaa1#aaa2#bbb1#bbb2#bbb3#ccc#ddd1#ddd2"
```



这个方法的主要作用是把 Stream 元素组合起来。它提供一个起始值（种子），然后依照运算规则（BinaryOperator），和前面 Stream 的第一个、第二个、第 n 个元素组合。从这个意义上说，字符串拼接、数指的 sum、min、max、average 都是特殊的 reduce。例如 Stream 的 sum 就相当于 `Integersum=integers.reduce(0,(a,b)->a+b);` 也有没有起始值的情况，这时会把 Stream 的前面两个元素组合起来，返回的是 Optional。

```java
// 字符串连接，concat = "ABCD"
String concat = Stream.of("A", "B", "C", "D").reduce("", String::concat); 
// 求最小值，minValue = -3.0
double minValue = Stream.of(-1.5, 1.0, -3.0, -2.0).reduce(Double.MAX_VALUE, Double::min); 
// 求和，sumValue = 10, 有起始值
int sumValue = Stream.of(1, 2, 3, 4).reduce(0, Integer::sum);
// 求和，sumValue = 10, 无起始值
sumValue = Stream.of(1, 2, 3, 4).reduce(Integer::sum).get();
// 过滤，字符串连接，concat = "ace"
concat = Stream.of("a", "B", "c", "D", "e", "F").
 filter(x -> x.compareTo("Z") > 0).
 reduce("", String::concat);
```



上面代码例如第一个示例的 reduce()，第一个参数（空白字符）即为起始值，第二个参数（String::concat）为 BinaryOperator。这类有起始值的 reduce() 都返回具体的对象。而对于第四个示例没有起始值的 reduce()，由于可能没有足够的元素，返回的是 Optional，请留意这个区别。

