## JavaScript闭包

#### 1.闭包

​		有不少开发人员总是搞不清匿名函数和闭包这两个概念，因此经常混用。闭包是指有权访问另一个函数作用域中的变量的函数。创建闭包的常见方式，就是在一个函数内部创建另一个函数，以 createComparisonFunction() 函数为例。

```js
function createComparisonFunction(propertyName) {
	
	return function(object1, object2) {
		var value1 = object1(propertyName);
		var value2 = object2(propertyName);
		
		if (value1 < value2) {
			return -1;
		} else if (value1 > value2) {
			return 1;
		} else {
			return 0;
		}
	};
}
```

​		在这个例子中，