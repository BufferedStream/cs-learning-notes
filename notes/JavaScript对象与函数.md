## JavaScript对象与函数

&emsp;&emsp;`ECMAScript`中的对象其实就是一组数据和功能的集合。对象可以通过执行new操作符后跟要创建的对象类型的名称来创建。而创建Object类型的示例并为其提添加属性和（或）方法，就可以创建自定义对象，如下所示：

```javascript
	var o = new Object();
```

​	仅仅创建Object的实例并没有什么好处，但关键是要要理解一个重要的思想：即在`ECMAScript`中，（就像Java中的`java.lang.Object`对象一样）Object类型是所有它的实例的基础。换句话说，Object类型所具有的任何属性和方法也同样存在于更具体地对象中。

Object的每个实例都具有下列属性和方法。

- Constructor：保存着用于创建当前对象的函数。对于前面的例子而言，构造函数（constructor）就是Object()。

- `hasOwnProperty（propertyName）`：用于检查给定的属性在当前对象实例中（而不是在实例的原型中）是否存在。其中，作为参数的属性名（`propertyName`）必须以字符串形式指定（例如：`o.hasOwnProperty("name")`）。

- `isProtorypeOf(object)`：用于检查传入的对象是否是另一个对象的原型。

- `propertyIsEnumerable(propertyName)`：用于检查给定的属性是否能够使用for-in语句来枚举。与`hasOwnProperty()`方法一样，作为参数的属性名必须以字符串形式指定。

- `toLocaleString()`：返回对象的字符串表示，该字符串与执行环境的地区对应。

- `toString()`：返回对象的字符串表示。

- `valueOf()`：返回对象的字符串、数指或布尔值表示。通常与`toString()`方法的返回值相同。

  

![image1](https://github.com/BufferedStream/cs-learning-notes/blob/master/notes/images/js%E5%AF%B9%E8%B1%A1%E4%B8%8E%E5%87%BD%E6%95%B01.png)

```javascript
function person(){};
var p = new person();
p.name = 'tom';

p.constructor >> ƒ person(){}
person.constructor >> ƒ Function() { [native code] }

p.hasOwnProperty("name") >> true
person.hasOwnProperty("name") >> true

person.prototype.isPrototypeOf(p) >> true
Object.prototype.isPrototypeOf(p) >> true

p.propertyIsEnumerable("name") >> false
person.propertyIsEnumerable("name") >> false

p.toLocaleString() >> "[object Object]"
person.toLocaleString() >> "function person () {}"

p.toString() >> "[object Object]"
person.toLocaleString() >> "function person () {}"

p.valueOf() >> person {} 
person.valueOf() >> ƒ person () {}

typeof p.valueOf() >> "object"
typeof person.valueOf() >> "function"
```



**从技术角度讲，`ECMA-262`中对象的行为不一定适用于JavaScript中的其他对象。浏览器环境中的对象，比如`BOM`和`DOM`中的对象，都属于宿主对象，因为它们是由宿主实现提供和定义的。`ECMA-262`不负责定义宿主对象，因此宿主对象可能会也可能不会继承Object。**



函数对任何语言来说都是一个核心的概念。通过函数可以封装任意多条语句，而且可以在任何地方、任何时候调用执行。`ECMAScript`中的函数使用function关键字来声明，后跟一组参数以及函数体。函数的基本语法如下所示：

```js
function functionName(arg0, arg1,...,argN) {
	statements
}
```

`ECMAScript`函数的参数与大多数其他语言中函数的参数有所不同。`ECMAScript`函数不介意传递进来多少个参数，也不在乎传进来的参数是什么数据类型。也就是说，即使你定义的函数只接受两个参数，在调用这个函数时也未必一定要传递两个参数。可以传递一个、三个甚至不传递参数，而解析器永远不会有什么怨言。之所以会这样，原因是`ECMAScript`中的参数在内部是用一个数组来表示的。函数接受到的始终都是这个数组，而不关心数组中包含哪些参数（如果有参数的话）。如果这个数组中不包含任何元素，无所谓；如果包含多个元素，也没有问题。实际上，在函数体内可以通过arguments对象来访问这个参数数组，从而获取传递给函数的每一个参数。

其实，arguments对象只是与数组类似（它并不是Array的实例），因为可以使用方括号语法访问它的每一个元素，使用length属性来确定传递进来多少个参数。



**`ECMAScript`中的所有参数传递的都是值，不可能通过引用传递参数。**



`ECMAScript`函数不能像传统意义上那样实现重载。而在其他语言（如Java）中，可以为一个函数编写两个定义，只要这两个定义的签名（接受的参数的类型和数量）不同即可。如前所述，`ECMAScript`函数没有签名，因为其参数是由包含零或多个值的数组来表示的。而没有函数签名，真正的重载是不可能做到的。（通过检查传入函数中参数的类型和数量并作出不同的反应，可以模仿方法的重载。）

如果在`ECMAScript`中定义了两个名字相同的函数，则该名字只属于后定义的函数。



在`ECMAScript`中，函数实际上是对象。每个函数都是Function类型的实例，而且都与其他引用类型一样具有属性和方法。由于函数是对象，因此函数名实际上也是要给指向函数对象的指针，不会与某个函数绑定。函数通常是使用函数声明语法定义的，如下所示：

```js
function sum (num1, num2) {
	return num1 + num2;
}
```

这与下面使用函数表达式定义函数的方式几乎相差无几。

```js
var sum = function (num1, num2) {
	return num1 + num2;
}
```

最后一种定义函数的方式是使用Function构造函数。Function构造函数可以接受任意数量的参数，但最后一个参数始终都被看成是函数体，而前面的参数则枚举出了新函数的参数。如下：

```js
var sum = new Function("num1", "num2", "return num1 + num2");	//不推荐
```

从技术上讲，这是一个函数表达式。但是，我们不推荐读者使用这种方法定义函数，因为这种语法会导致解析两次代码（第一次是解析常规`ECMAScript`代码，第二次是解析传入构造函数中的字符串），从而影响性能。不过，这种语法对于理解“函数是对象，函数名是指针”的概念倒是非常直观的。

由于函数名仅仅是指向函数的指针，因此函数名与包含对象指针的其他变量没有什么不同。换句话说，一个函数可能会有多个名字，如下例所示。

```js
function sum(num1, num2) {
	return num1 + num2;
}
alert(sum(10, 10));			//20

var anotherSum = sum;
alert(anotherSum(10, 10));	//20

sum = null;
alert(anotherSum(10, 10));	//20
```

将函数名想象为指针，也有助于理解为什么`ECMAScript`中没有函数重载的概念。



因为`ECMAScript`中的函数名本身就是变量，所以函数也可以作为值来使用。也就是说，不仅可以像传递参数一样把一个函数传递给另一个函数，而且可以将一个函数作为另一个函数的结果返回。



`ECMAScript`中的函数是对象，因此函数也有属性和方法。每个函数都包含两个属性：length和prototype。其中，length属性表示函数希望接收的命名参数的个数。

在`ECMAScript`核心所定义的全部属性中，最耐人寻味的就要数prototype属性了。对于`ECMAScript`中的引用类型而言，prototype是保存它们所有实例方法的真正所在。换句话说，诸如`toString()`和`valueOf()`等方法实际上都保存在prototype名下，只不过是通过各自对象的实例访问罢了。在创建自定义引用类型以及实现继承时，prototype属性的作用是极为重要的。在`ECMAScript`中，prototype属性是不可枚举的，因此使用for-in无法实现。

