## JavaScript对象与函数

​	`ECMAScript`中的对象其实就是一组数据和功能的集合。对象可以通过执行new操作符后跟要创建的对象类型的名称来创建。而创建Object类型的示例并为其提添加属性和（或）方法，就可以创建自定义对象，如下所示：

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

```
function functionName(arg0, arg1,...,argN) {
	statements
}
```

`ECMAScript`函数的参数与大多数其他语言中函数的参数有所不同。`ECMAScript`函数不介意传递进来多少个参数，也不在乎传进来的参数是什么数据类型。也就是说，即使你定义的函数只接受两个参数，在调用这个函数时也未必一定要传递两个参数。可以传递一个、三个甚至不传递参数，而解析器永远不会有什么怨言。之所以会这样，原因是`ECMAScript`中的参数在内部是用一个数组来表示的。函数接受到的始终都是这个数组，而不关心数组中包含哪些参数（如果有参数的话）。如果这个数组中不包含任何元素，无所谓；如果包含多个元素，也没有问题。实际上，在函数体内可以通过arguments对象来访问这个参数数组，从而获取传递给函数的每一个参数。

其实，arguments对象只是与数组类似（它并不是Array的实例），因为可以使用方括号语法访问它的每一个元素，使用length属性来确定传递进来多少个参数。



**`ECMAScript`中的所有参数传递的都是值，不可能通过引用传递参数。**



`ECMAScript`函数不能像传统意义上那样实现重载。而在其他语言（如Java）中，可以为一个函数编写两个定义，只要这两个定义的签名（接受的参数的类型和数量）不同即可。如前所述，`ECMAScript`函数没有签名，因为其参数是由包含零或多个值的数组来表示的。而没有函数签名，真正的重载是不可能做到的。（通过检查传入函数中参数的类型和数量并作出不同的反应，可以模仿方法的重载。）

如果在`ECMAScript`中定义了两个名字相同的函数，则该名字只属于后定义的函数。
