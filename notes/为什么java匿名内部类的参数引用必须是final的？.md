## Lambda 

我们先用一个小例子来看一下以前的Java版本是怎么给字符串数组排序的。

```java
List<String> names = Arrays.asList("peter", "anna", "mike", "xenia");
Collections.sort(names, new Comparator<String>()
	@Override
	public int compare(String a, String b) |{
		return b.compareTo(a);
	}
});

```

静态的实用方法 `Collections.sort` 接受了一个list和一个comparator来对指定list排序。你会发现你经常需要new匿名类传递给sort方法。

与new匿名类相比，Java 8提供了一种更简便的语法，lambda表达式：

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

names序列接受了排序的方法。Java编译器也知道入参类型所以你可以跳过它们。接下来我们深入了解lambda表达式有哪些不同寻常的用法。