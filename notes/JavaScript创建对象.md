## JavaScript创建对象

#### 1.工厂模式

​		虽然Object构造函数或对象字面量都可以用来创建单个对象，但这些方式有个明显的缺点：使用同一个接口创建很多对象，会产生大量的重复代码。为解决这个问题，人们开始使用工厂模式的一种变体。

​		工厂模式是软件工程领域一种广为人知的设计模式，这种模式抽象了创建具体对象的过程。考虑到在`ECMAScript`中无法创建类，开发人员就发明了一种函数，用函数来封装以特定接口创建对象的细节，如下例所示。

```js
function createPerson(name, age, job) {
	var o = new Object();
	o.name = name;
	o.age = age;
	o.job = job;
	o.sayName = function() {
		alert(this.name);
	};
	return o;
}

var person1 = createPerson("zhangsan", 18, "Software Engineer");
var person2 = createPerson("lisi", 19, "Docotor");
```

​		函数`createPerson()`能够根据接收的参数来构建一个包含所有必要信息的Person对象。可以无数次地调用这个函数，而每次它都会返回一个包含三个属性一个方法地对象。工厂模式虽然解决了创建多个相似对象的问题，但却没有解决对象识别的问题（即怎样知道一个对象的类型）。



#### 2.构造函数模式

​		`ECMAScript`中的构造函数可用来创建特定类型的对象。像Object和Array这样的源生构造函数，在运行时会自动出现在执行环境中。此外，也可以创建自定义的构造函数，从而定义自定义对象类型的属性和方法。例如，可以使用构造函数模式将前面的例子重写如下。

```js
function Person(name, age, job) {
	this.name = name;
	this.age = age;
	this.job = job;
	this.sayName = function() {
		alert(this.name);
	}
}

var person1 = new Person("zhangsan", 18, "Software Engineer");
var person2 = new Person("lisi", 19, "Docotor");
```

`Person()`与`createPeson()`相比，存在以下不同之处：

- 没有显式地创建对象；
- 直接将属性和方法赋给了this对象；
- 没有return语句。



​    要创建Person的新实例，必须使用new操作符。以这种方式调用构造函数实际上会经历以下四个步骤：

1. 创建一个新对象；
2. 将构造函数的作用域赋给新对象（因此this就指向了这个新对象）；
3. 执行构造函数中的代码（为这个新对象添加属性）；
4. 返回新对象。		

```js
person1.constructor == Person	//true
person2.constructor == Person	//true
```

​		对象的constructor属性最初是用来标识对象类型的。但是，提到检测对象类型，还是`instanceof`操作符要更可靠一些。

​		创建自定义的构造函数意味着将来可以将它的实例标识为一种特定的类型：而这正式构造函数模式胜过工厂模式的地方。



​		构造函数与其他函数的唯一区别，就在于调用他们的方式不同。不过，构造函数毕竟也是函数，不存在定义构造函数的特殊语法。任何函数，只要通过new操作符来调用，那它就可以作为构造函数；而任何函数，若果不通过new操作符来调用，那它跟普通函数也不会有什么两样。例如，前面例子中定义的Person()函数可以通过下列任何一种方式来调用。

```js
//当作构造函数使用
var person = new Person("zhangsan", 18, "Software Engineer");
person.sayName();	//"zhangsan"

//作为普通函数调用
Person("zhangsan", 18, "Software Engineer");
window.sayName();	//"zhangsan"

//在另一个对象的作用域中调用
var o = new Object();
Person.call(o, "lisi", 19, "doctor");
o.sayName();	//"lisi"
```

​		这个例子中的前两行代码展示了构造函数的典型用法，即使用new操作符来创建一个新对象。接下来的两行代码展示了不使用new操作符调用Person()会出现什么结果：属性和方法都被添加给window对象了。



​		构造函数的问题，构造函数虽然好用，但也并非没有缺点。使用构造函数的主要问题，就是每个方法都要在每个实例上重新创建一遍。在前面的例子中，`person1`和`person2`都有一个名为	`sayName()`的方法，但那两个方法不是同一个Function的实例。不要忘了——`ECMAScript`中的函数是对象，因此每定义一个函数，也就是实例化了一个对象。从逻辑角度讲，此时的构造函数也可以这样定义。

```js
function Person(name, age, job) {
	this.name = name;
	this.age = age;
	this.job = job;
	this.sayName = new Function("alert(this.name);");	//与声明函数在逻辑上是等价的
}
```

​		从这个角度上来看构造函数，更容易明白每个Person实例都包含一个不同的Function实例（以显示name属性）的本质。说明白些，以这种方式创建函数，会导致不同的作用域链和标识符解析，但创建Function新实例的机制仍然是相同的。因此，不同实例上的同名函数是不相等的，以下代码可以证明这一点。

```
person1.sayName == person2.sayName	//false
```

​		然而，创建两个完成同样任务的Function实例的的确没有必要；况且有this对象在，根本不用在执行代码前就把函数绑定到特定对象上面。因此，大可像下面这样，通过把函数定义转移到构造函数外部来解决这个问题。

```js
function Person(name, age, job) {
	this.name = name;
	this.age = age;
	this.job = job;
	this.sayName = sayName;
}

function sayName() {
    alert(this.name);
}

var person1 = new Person("zhangsan", 18, "Software Engineer");
var person2 = new Person("lisi", 19, "Docotor");
```

​		在这个例子中，我们把`sayName()`函数的定义转移到了构造函数外部。而在构造函数内部，我们将`sayName`属性设置成等于全局的`sayName`函数。这样一来，由于`sayName`包含的是一个指向函数的指针，因此`person1`和`person2`对象就共享了在全局作用域中定义的同一个`sayName()`函数。这样做确实解决了两个函数做同一件事的问题，可是新问题又来了：在全局作用域中定义的函数实际上只能被某个对象调用，这让全局作用域有带你名不副实。而更让人无法接受的是：如果对象需要定义很多方法，那么就要定义很多个全局函数，于是我们这个自定义的引用类型就丝毫没有封装性可言了。好在，这些问题可以通过使用原型模式来解决。



#### 3.原型模式

​		我们创建的每个函数都有一个prototype（原型）属性，这个属性是一个指针，指向一个对象，而这个对象的用途是包含可以由特定类型的所有实例共享的属性和方法。如果按照字面意思来理解，那么prototype就是通过调用构造函数而创建的哪个对象实例的原型对象。使用原型对象的好处是可以让所有实例共享它所包含的属性和方法。换句话说，不必在构造函数中定义对象实例的信息，而是可以将这些信息直接添加到原型对象中，如下面的例子所示。

​		









