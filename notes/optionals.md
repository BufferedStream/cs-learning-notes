## Optionals

Optionals 不是函数式接口，而是用于防止 NullPointerException 的漂亮工具。这是下一节的一个重要概念，让我们快速了解一下 Optionals 的工作原理。

Optional 是一个简单的容器，其值可能是 null 或者不是 null。在 Java 8 之前一般某个函数应该返回非空对象但是有时却什么也没有返回，而在 Java 8 中，你应该返回 Optional 而不是 null。

```java
//of()：为非null的值创建一个Optional
Optional<String> optional = Optional.of("bam");

// isPresent()： 如果值存在返回true，否则返回false
optional.isPresent(); // true

//get()：如果Optional有值则将其返回，否则抛出NoSuchElementException
optional.get(); // "bam"

//orElse()：如果有值则将其返回，否则返回指定的其它值
optional.orElse("fallback"); // "bam"

//void ifPresent(Consumer<? super T> consumer)
//ifPresent()：如果Optional实例有值则为其调用consumer，否则不做处理
optional.ifPresent((s) -> System.out.println(s.charAt(0))); // "b" 
```

