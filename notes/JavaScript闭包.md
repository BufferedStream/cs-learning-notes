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


<div align=center>
	<img src="https://github.com/BufferedStream/cs-learning-notes/blob/master/notes/images/js%E9%97%AD%E5%8C%851.jpg"/>
</div>


​		后台的每个执行环境都有一个表示变量的对象——变量对象。全局环境的变量对象始终存在，而像 compare() 函数这样的局部环境的变量对象，则只在函数执行的过程中存在。在创建 compare() 函数时，会创建一个预先包含全局变量对象的作用域链，这个作用域链被保存在内部的 [[Scope]] 属性中。当调用 compare() 函数时，会为函数创建一个执行环境，然后通过复制函数的 [[Scope]] 属性中的对象构建起执行环境的作用域链。此后，又有一个活动对象（在此作为变量对象使用）被创建并被推入执行环境作用域链的前端。对于这个例子中 compare() 函数的执行环境而言，其作用域链中包含两个变量对象：本地活动对象和全局变量对象。显然，作用域链本质上是一个指向变量对象的指针列表，它只引用但不实际包含变量对象。

​		无论什么时候在函数中访问一个变量时，就会从作用域链中搜索具有相应名字的变量。一般来讲，当函数执行完毕后，局部活动对象就会被销毁，内存中仅保存全局作用域（全局执行环境的变量对象）。但是，闭包的情况又有所不同。

​		在另一个函数内部定义的函数会将包含函数（即外部函数）的活动对象添加到它的作用域链中。因此，在 createComparisionFunction() 函数内部定义的匿名函数的作用域链中，实际上将会包含外部函数 createComparisionFunction() 的活动对象。下图展示了当下列代码执行时，包含函数与内部匿名函数的作用域链。


<div align=center>
	<img src="https://github.com/BufferedStream/cs-learning-notes/blob/master/notes/images/js%E9%97%AD%E5%8C%852.jpg"/>
</div>


```js
var compare = createComparisonFunction("name");
var result = compare({name: "zhangsan"}, {name: "lisi"});
```
		

<div align=center>
	<img src="https://github.com/BufferedStream/cs-learning-notes/blob/master/notes/images/js%E9%97%AD%E5%8C%853.jpg"/>
</div>


​		如上图所示，在匿名函数从 createComparisionFunction() 中被返回后，它的作用域链被初始化为包含 createComparisionFunction() 函数的活动对象和全局变量对象（在 compare() 函数中可以访问到 createComparisionFunction() 函数中的 propertyName 参数）。这样，匿名函数就可以访问在 createComparisionFunction() 中定义的所有变量。更为重要的是， createComparisionFunction() 函数在执行完毕后，其活动对象也不会被销毁，因为匿名函数的作用域链仍然在引用这个活动对象。换句话说，当 createComparisionFunction() 函数返回后，其执行环境的作用域链会被销毁，但它的活动对象仍然会留在内存中；直到匿名函数被销毁后， createComparisionFunction() 的活动对象才会被销毁，例如：

```js
//创建函数
var compareNames = createComparisonFunction("name");

//调用函数
var result = compareNames({name: "zhangsan"}, {name: "lisi"});

//解除对匿名函数的引用（以便释放内存）
compareNames = null;
```



​		首先，创建的比较函数被保存在变量 compareNames 中。而通过将 compareNames 设置为等于 null 解除该函数的引用，就等于通知垃圾回收例程将其清除。随着匿名函数的作用域链被销毁，其他作用域（除了全局作用域）也都可以安全地销毁了。调用 compareNames() 的过程中产生的作用域链之间的关系与调用 compare() 函数一样。

​		由于闭包会携带包含它的函数的作用域，因此会比其他函数占用过多的内存。过度使用闭包可能会导致内存占用过多，我们建议读者只在绝对必要时再考虑使用闭包。虽然像 V8 等优化后的 JavaScript 引擎会尝试回收被闭包占用的内存，但请大家还是要慎重使用闭包。



##### 1.1 闭包与变量

​		作用域链的这种配置机制引出了一个值得注意的副作用，即闭包只能取得包含函数中任何变量的最后一个值。别忘了闭包所保存的是整个变量对象，而不是某个特殊的变量。下面这个例子可以清晰地说明这个问题。

```js
function createFunctions() {
	var result = new Array();
	
	for (var i=0; i<10; i++) {
		result[i] = function() {
			return i;
		};
	}
	
	return result;
}
```

​		这个函数会返回一个函数数组。表面上看，似乎每个函数都应该返回自己的索引值，即位置 0 的函数返回 0，位置 1 的函数返回 1，以此类推。但实际上，每个函数都返回 10。因为每个函数的作用域链中都保存着 createFunctions() 函数的活动对象，所以它们引用的都是同一个变量 i。当 createFunctions() 函数返回后，变量 i 的值是 10，此时每个函数都引用这保存变量 i 的同一个变量对象，所以在每个函数内部 i 的值都是 10。但是，我们可以通过创建另一个匿名函数强制让闭包的行为符合预期，如下所示。

```js
function createFunctions() {
	var result = new Array();
	
	for (var i=0; i<10; i++) {
		result[i] = function(num) {
			return function() {
            	return num;    
            };
		}(i);
	}
	
	return result;
}
```

​		在重写了前面的 createFunctions() 函数后，每个函数就会返回各自不同的索引值了。在这个版本中，我们没有直接把闭包赋值给数组，而是定义了一个匿名函数，并将立即执行该匿名函数的结果赋给数组。这里的匿名函数有一个参数 num，也就是最终的函数要返回的值。在调用每个匿名函数时，我们传入了变量 i。由于函数参数是按值传递的，所以就会将变量 i 的当前值复制给参数 num。而在这个匿名函数内部，又创建并返回了一个访问 num 的闭包。这样一来， result 数组中的每个函数都有自己 num 变量的一个副本，因此就可以返回各自不同的数值了。



##### 1.2 关于 this 对象

​		在闭包中使用 this 对象也可能会导致一些问题。我们知道， this 对象是在运行时基于函数的执行环境绑定的：在全局函数中， this 等于 window，而当函数被作为某个对象的方法调用时， this 等于那个对象。不过，匿名函数的执行环境具有全局性，因此其 this 对象通常指向 window。但有时候由于编写闭包的方式不同，这一点可能不会那么明显。下面来看一个例子。

```js
var name = "The Window";

var object = {
	name : "My Object",
	
	getNameFunc : function() {
		return function() {
			return this.name;
		};
	}
};

console.log(object.getNameFunc()());	//"The Window"(在非严格模式下)，注意需要两组括号
```

​		

​		以上代码先创建了一个全局变量 name，又创建了一个包含 name 属性的对象。这个对象还包含了一个方法——getNameFunc()，它返回一个匿名函数，而匿名函数又返回 this.name。由于 getNameFunc() 返回一个函数，因此调用 obejct.getNameFunc()() 就会立即调用它返回的函数，结果就是返回一个字符串。然而，这个例子返回的字符串是 “The Window”，即全局 name 变量的值。为什么匿名函数没有取得其包含作用域（或外部作用域）的 this 对象呢？

​		前面曾经提到过，每个函数在被调用时，其活动对象都会自动取得两个特殊变量： this 和 arguments。内部函数在搜索这两个变量时，只会搜索到其活动对象为止，因此永远不可能直接访问外部函数中的这两个变量。不过，把外部作用域中的 this 对象保存在一个闭包能够访问到的变量里，就可以让闭包访问该对象了，如下所示。

```js
var name = "The Window";s

var object = {
	name : "My Object",
	
	getNameFunc : function() {
        var that = this;
		return function() {
			return that.name;
		};
	}
};

console.log(object.getNameFunc()());	//"My Object"
```

​		把 this 对象赋值给 that 变量是一种常用的做法。







































