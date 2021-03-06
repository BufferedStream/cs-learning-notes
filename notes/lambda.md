## Lambda 

我们先用一个小例子来看一下以前的 Java 版本是怎么给字符串数组排序的。

```java
List<String> names = Arrays.asList("peter", "anna", "mike", "xenia");
Collections.sort(names, new Comparator<String>()
	@Override
	public int compare(String a, String b) |{
		return b.compareTo(a);
	}
});

```



静态的实用方法 `Collections.sort` 接受了一个 list 和一个比较器对象来对指定 list 排序。你会发现你经常需要创建匿名对象传递给 sort 方法。

与创建匿名对象相比，Java 8提供了一种更简便的语法，lambda表达式：

```
Collections.sort(names, (String a, String b) -> {
    return b.compareTo(a);
});
```



你可以看到，代码变得更加简单易读了。但还可以简化：

```
Collections.sort(names, (String a, String b) -> b.compareTo(a));
```



对于只有一行代码的方法，你可以同时省略 `{}` 和 `return` 关键字。但我们还能继续简化：

```
names.sort((a, b) -> b.compareTo(a));
```



names 序列接收了排序的方法。Java 编译器也知道入参类型所以你可以跳过它们。接下来我们深入了解 lambda 表达式有哪些不同寻常的用法。