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
var name = "The Window";

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



#### 2.模仿块级作用域

​		如前所述， JavaScript 没有块级作用域的概念。这意味着在块语句中定义的变量，实际上是在包含函数中而非语句中创建的，来看下面的例子。

```js
function outputNumbers(count) {
	for (var i=0; i<count; i++) {
		console.log(i);
	}
	console.log(i);		//计数
}
```

​		这个函数中定义了一个 for 循环。在 Java、C++ 等语言中，变量 i 只会在 for 循环的语句块中有定义，循环一旦结束，变量 i 就会被销毁。可是在 JavaScript 中，变量 i 是定义在 outputNumbers() 的活动对象中的，因此从它有定义开始，就可以在函数内部随处访问它。即使像下面这样错误地重新声明同一个变量，也不会改变它的值。

```js
function outputNumbers(count) {
	for (var i=0; i<count; i++) {
		console.log(i);
	}
	
	var i;
	console.log(i);		//计数
}
```

​		

​		JavaScript 从来不会告诉你是否多次声明了同一个变量：遇到这种情况，它只会对后续的声明视而不见（不过，它会执行后续声明中的变量初始化）。匿名函数可以用来模仿块级作用域并避免这个问题。

​		用作块级作用域（通常称为私有作用域）的匿名函数的语法如下所示。

```js
(function(){
	//这里是块级作用域
})();
```



​		无论在什么地方，只要临时需要一些变量，就可以使用私有作用域，例如：

```js
function outputNumbers(count) {
	(function() {
		for (var i=0; i<count; i++) {
			console.log(i);
		}
	})();
	
	console.log(i);		//计数
}
```



​		在这个重写后的 outputNumbers() 函数中，我们在 for 循环外部插入了一个私有作用域。在匿名函数中定义的任何变量，都会在执行结束时被销毁。因此，变量 i 只能在循环中使用，使用后即被销毁。而在私有作用域中能够访问变量 count，是因为这个匿名函数是一个闭包，它能够访问包含作用域中的所有变量。

​		这种技术经常在全局作用域中被用在函数外部，从而限制向全局作用域中添加过多的变量和函数。一般来说，我们都应该尽量少向全局作用域中添加变量和函数。在一个由很多开发人员共同参与的大型应用程序中，过多的全局变量和函数很容易导致命名冲突。而通过创建私有作用域，每个开发人员既可以使用自己的变量，又不必担心搞乱全局作用域。例如：

```js
(function() {

	var now = new Date();
    for (now.getMonth() == 0 && now.getDate() == 1) {
    	console.log("Happy new year!");
    }
    
})();
```



​		把上面这段代码放在全局作用域中，可以用来确定哪一天是 1 月 1 日；如果到了这一天，就会向用户显示一条祝贺新年的消息。其中的变量 now 现在是匿名函数中的局部变量，而我们不必在全局作用域中创建它。

​		这种做法可以减少闭包占用内存的问题，因为没有指向匿名函数的引用。只要函数执行完毕，姐可以立即销毁其作用域链了。



#### 3.私有变量

​		严格来讲，JavaScript 中没有私有成员的概念；所有对象属性都是公有的。不过，倒是有一个私有变量的概念。任何在函数中定义的变量，都可以认为是私有变量，因为不能在函数的外部访问这些变量。私有变量包括函数的参数、局部变量和在函数内部定义的其他函数。来看下面的例子：

```js
function add(num1, num2) {
	var sum = num1 + num2;
	return sum;
}
```



​		在这个函数内部，有 3 个私有变量： num1、 num2 和 sum。在函数内部可以访问这几个变量，但在函数外部则不能访问它们。如果在这个函数内部创建一个闭包，那么闭包通过自己的作用域链也可以访问这些变量。而利用这一点，就可以创建用于访问私有变量的共有方法。

​		我们把有权访问私有变量和私有函数的共有方法称为特权方法（privileged method）。有两种在对象上创建特权方法的方式。第一种是在构造函数中定义特权方法，基本模式如下。

```js
function MyObject() {
	
	//私有变量和私有函数
	var privateVariable = 10;
	
	function privateFunction() {
		return false;
	}
	
	//特权方法
	this.publicMethod = function() {
		privateVariable++;
		return privateFunction();
	};
}
```

​		

​		这个模式在构造函数内部定义了所有私有变量和函数。然后，又继续创建了能够访问这些私有成员的特权方法。能够在构造函数中定义特权方法，是因为特权方法作为闭包有权访问在构造函数中定义的所有变量和函数。对这个例子而言，变量 privateVariable 和函数 privateFunction() 只能通过特权方法 publicMethod 来访问。在创建 MyObject 的实例后，除了使用 publicMehod() 这一个途径外，没有任何方法可以直接访问 privateVariable 和 privateFunction()。

​		利用私有和特权成员，可以隐藏那些不有个被直接修改的数据，例如：

```js
function Person(name) {
	
	this.getName = function() {
		return name;
	};
	
	this.setName = function(value) {
		name = value;
	};
}

var person = new Person("zhangsan");
console.log(person.getName());		//"zhangsan"
person.setName("lisi");
console.log(person.getName());		//"lisi"
```



​		以上代码的构造函数中定义了两个特权方法： getName() 和 setName()。这两个方法都可以在构造函数外部使用，而且都有权访问私有变量 name。但在 Person 构造函数外部，没有任何办法访问 name。由于这两个方法是在构造函数内部定义的，它们作为闭包能够通过作用域链访问 name。私有变量 name 在 Person 的每一个实例中都不相同，因为每次调用构造函数都会重新创建这两个方法。不过，在构造函数中定义特权方法也有一个缺点，那就是你必须使用构造函数模式来达到这个目的。前面曾经讨论过，构造函数模式的缺点是针对每个实例都会创建同样一组新方法，而使用静态私有变量来实现特权方法就可以避免这个问题。



##### 3.1 静态私有变量

​		通过在私有作用域中定义私有变量或函数，同样也可以创建特权方法，其基本模式如下所示。

```js
(function() {
	
	//私有变量和私有函数
	var privateVariable = 10;
	
	function privateFunction() {
		return false;
	}
	
	//构造函数
	MyObject = function() {
	};
	
	//共有/特权方法
	MyObject.prototype.publicMethod = function() {
		privateVariable++;
		return privateFunction();
	};

})();
```



​		这个模式创建了一个私有作用域，并在其中封装了一个构造函数及相应的方法。在私有作用域中，首先定义了私有变量和私有函数，然后又定义了构造函数及其公有方法。公有方法是在原型上定义的，这一点体现了典型的原型模式。需要注意的是，这个模式在定义构造函数时并没有使用函数声明，而是使用了函数表达式。函数声明只能创建局部函数，但那并不是我们想要的。出于同样的原因，我们也没有在声明 MyObject 时使用 var 关键字。记住：初始化未经声明的变量，总是会创建一个全局变量。因此， MyObject 就成了一个全局变量，能够在私有作用域之外被访问到。但也要知道，在严格模式下给未经声明的变量赋值会导致错误。

​		这个模式与在构造函数中定义特权方法的主要区别，就在于私有变量和函数是由实例共享的。由于特权方法是在原型上定义的，因此所有实例都使用同一个函数。而这个特权方法，作为一个闭包，总是保存着对包含作用域的引用。来看一看下面的代码。

```js
(function() {

	var name = "";
	
	Person = function(value) {
		name = value;
	};
	
	Person.prototype.getName = function() {
		return name;
	};
	
	Person.prototype.setName = function(value) {
		name = value;
	};
})();

var person1 = new Person("zhangsan");
console.log(person1.getName());		//"zhangsan"
person1.setName("lisi");
console.log(person1.getName());		//"lisi"

var person2 = new Person("wangwu");
console.log(person1.getName());		//"wangwu"
console.log(person2.getName());		//"wangwu"
```



​		这个例子中的 Person 构造函数与 getName() 和 setName() 方法一样，都有权访问私有变量 name。在这种模式下，变量 name 就变成了一个静态的、由所有实例共享的属性。也就是说，在一个实例上调用 setName() 会影响所有实例。而调用 setName() 或新建一个 Person 实例都会赋予 name 属性一个新值。结果就是所有实例都会返回相同的值。

​		以这种方式创建静态私有变量会因为使用原型而增进代码复用，但每个实例都没有自己的私有变量。到底是使用实例变量，还是静态私有变量，最终还是要视具体需求而定。

​		多查找作用域链中的一个层次，就会在一定程度上影响查找速度。而这正是使用闭包和私有变量的一个显明的不足之处。



##### 3.2 模块模式

​		前面的模式是用于为自定义类型创建私有变量和特权方法的。而道格拉斯所说的模块模式（module pattern）则是为单利创建私有变量和特权方法。所谓单利（singleton），指得就是只有一个实例的对象。按照惯例， JavaScript 是以对象字面量的方式来创建单例对象的。



