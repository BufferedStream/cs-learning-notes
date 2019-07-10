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

