## Lambda Scopes

lambda 表达式访问外部作用域变量的权限与匿名内部类的对象非常类似。你可以正常访问作用域外的常量（final variables）、实例变量以及静态变量。



#### 访问局部变量

我们可以直接在 lambda 表达式中访问外部的局部变量：

```java
final int num = 1;
Converter<Integer, String> stringConverter = (from) -> String,valueOf(from + num);

StringConverter.convert(2);	//3
```



但是和匿名内部类对象不同的是，这里的变量 num 可以不用声明为 final，该代码同样正确。（ java 8 的匿名内部类对象的语法和这个一致了！）

```java
int num = 1;
Converter<Integer, String> stringConverter =
 (from) -> String.valueOf(from + num);

stringConverter.convert(2); // 3
```



不过这里的 num 不能被后面的代码修改（即隐性的具有 final 的语义），例如下面的就无法编译：

```java
int num = 1;
Converter<Integer, String> stringConverter =
 (from) -> String.valueOf(from + num);
num = 3;	//在lambda表达式中试图修改num同样是不允许的。
```



#### 访问字段和静态变量

与局部变量相比，我们对 lambda 表达式中的实例字段和静态变量都有读写访问权限。这特性和匿名内部类对象是一致的。

```java
class Lambda4 {
	static int outerStaticNum;
	int outerNum;
	
	void testScopes() {
		Converter<Integer, String> stringConverter1 = (from) -> {
			outerNum = 23;
			return String.valueOf(from);
		};
		
		Converter<Integer, String> stringConverter2 = (from) -> {
			outerStaticNum = 72;
			return String.valueOf(from);
		};
	}
}
```



#### 访问默认接口方法

还记得第一节中的 formula 示例吗？ `Formula` 接口定义了一个默认方法 `sqrt`，可以从包含匿名对象的每个 formula 实例访问该方法。 这不适用于 lambda 表达式。

无法从 lambda 表达式中访问默认方法，故以下代码无法编译：

```java
Formula formula = (a) -> sqrt(a * 100);
```

