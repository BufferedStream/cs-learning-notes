## JavaScript闭包

#### 1.闭包

​		有不少开发人员总是搞不清匿名函数和闭包这两个概念，因此经常混用。闭包是指有权访问另一个函数作用域中的变量的函数。创建闭包的常见方式，就是在一个函数内部创建另一个函数，以 createComparisonFunction() 函数为例。

```js
function createComparisonFunction(propertyName) {
	
	return function(object1, object2) {
		var value1 = object1[propertyName];
		var value2 = object2[propertyName];
		
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

​		在这个例子中，赋值的那两行代码是内部函数（一个匿名函数）中的代码。这两行代码访问了外部函数中的变量 propertyname。即使这个内部函数被返回了，而且是在其他地方被调用了，但它仍然可以访问变量 propertyName。之所以还能够访问这个变量，是因为内部函数的作用域链中包含 createComparisonFunction() 的作用域。要彻底搞清楚其中的细节，必须从理解函数第一次被调用的时候都会发生什么入手。

​		前面介绍了作用域链的概念。而有关如何创建作用域链以及作用域链有什么作用的的细节，对彻底理解闭包至关重要。当某个函数第一次被调用时，会创建一个执行环境（execution context）及相应的作用域链，并把作用域链赋值给一个特殊的内部属性（即 [[Scope]]）。然后，使用 this、arguments 和其他命名参数的值来初始化函数的活动对象（activation object）。但在作用域链中，外部函数的活动对象始终处于第二位，外部函数的外部函数的活动对象处于第三位，·······直至作为作用域链终点的全局执行环境。

​		在函数执行过程中，为读取和写入变量的值，就需要在作用域链中查找变量。来看下面的例子。

```js
function compare(value1, value2) {
	if (value1 < value2) {
		return -1;
	} else if (value1 > value2) {
		return 1;
	} else {
		return 0;
	}
}

var result = compare(5, 10);
```

​	

​		以上代码先定义了 compare() 函数，然后又在全局作用域中调用了它。当第一次调用 compare() 时，会创建一个包含 this、arguments、value1 和 value2 的活动对象。全局执行环境的变量对象（包含 this、 result 和 compare）在compare() 执行环境的作用域链中则处于第二位。下图展示了包含上述关系的 compare() 函数执行时的作用域链。



​		后台的每个执行环境都有一个表示变量的对象——变量对象。全局环境的变量对象始终存在，而像 compare() 函数这样的局部环境的变量对象，则只在函数执行的过程中存在。在创建 compare() 函数时，会创建一个预先包含全局变量对象的作用域链，这个作用域链被保存在内部的 [[Scope]] 属性中。当调用 compare() 函数时，会为函数创建一个执行环境，然后通过复制函数的 [[Scope]] 属性中的对象构建起执行环境的作用域链。此后，又有一个活动对象（在此作为变量对象使用）被创建并被推入执行环境作用域链的前端。对于这个例子中 compare() 函数的执行环境而言，其作用域链中包含两个变量对象：本地活动对象和全局变量对象。显然，作用域链本质上是一个指向变量对象的指针列表，它只引用但不实际包含变量对象。

​		无论什么时候在函数中访问一个变量时，就会从作用域链中搜索具有相应名字的变量。一般来讲，当函数执行完毕后，局部活动对象就会被销毁，内存中仅保存全局作用域（全局执行环境的变量对象）。但是，闭包的情况又有所不同。

​		在另一个函数内部定义的函数会将包含函数（即外部函数）的活动对象添加到它的作用域链中。因此，在 createComparisionFunction() 函数内部定义的匿名函数的作用域链中，实际上将会包含外部函数 createComparisionFunction() 的活动对象。下图展示了当下列代码执行时，包含函数与内部匿名函数的作用域链。

```js
var compare = createComparisionFunction("name");
var result = compare({name: "zhangsan"}, {name: "lisi"});
```

​		

​		如上图所示，在匿名函数从 createComparisionFunction() 中被返回后，它的作用域链被初始化为包含 createComparisionFunction() 函数的活动对象和全局变量对象（在 compare() 函数中可以访问到 createComparisionFunction() 函数中的）。这样，匿名函数就可以访问在 createComparisionFunction() 中定义的所有变量。更为重要的是， createComparisionFunction() 函数在执行完毕后，其活动对象也不会被销毁，因为匿名函数的作用域链仍然在引用这个活动对象。换句话说，当 createComparisionFunction() 函数返回后，其执行环境的作用域链会被销毁，但它的活动对象仍然会留在内存中；直到匿名函数被销毁后， createComparisionFunction() 的活动对象才会被销毁，例如：

```
//创建函数
var compareNames = createCom
```











