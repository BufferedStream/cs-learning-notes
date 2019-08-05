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
		console.log(this.name);
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
		console.log(this.name);
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
	this.sayName = new Function("console.log(this.name);");	//与声明函数在逻辑上是等价的
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
    console.log(this.name);
}

var person1 = new Person("zhangsan", 18, "Software Engineer");
var person2 = new Person("lisi", 19, "Docotor");
```

​		在这个例子中，我们把`sayName()`函数的定义转移到了构造函数外部。而在构造函数内部，我们将`sayName`属性设置成等于全局的`sayName`函数。这样一来，由于`sayName`包含的是一个指向函数的指针，因此`person1`和`person2`对象就共享了在全局作用域中定义的同一个`sayName()`函数。这样做确实解决了两个函数做同一件事的问题，可是新问题又来了：在全局作用域中定义的函数实际上只能被某个对象调用，这让全局作用域有带你名不副实。而更让人无法接受的是：如果对象需要定义很多方法，那么就要定义很多个全局函数，于是我们这个自定义的引用类型就丝毫没有封装性可言了。好在，这些问题可以通过使用原型模式来解决。



#### 3.原型模式

​		我们创建的每个函数都有一个prototype（原型）属性，这个属性是一个指针，指向一个对象，而这个对象的用途是包含可以由特定类型的所有实例共享的属性和方法。如果按照字面意思来理解，那么prototype就是通过调用构造函数而创建的哪个对象实例的原型对象。使用原型对象的好处是可以让所有实例共享它所包含的属性和方法。换句话说，不必在构造函数中定义对象实例的信息，而是可以将这些信息直接添加到原型对象中，如下面的例子所示。

```js
function Person() {
}

Person.prototype.name = "zhangsan";
Person.prototype.age = 19;
Person.prototype.job = "Software Engineer";
Person.prototype.sayName = function() {
  console.log(this.name);  
};

var person1 = new Person();
person1.sayName();		//"zhangsan"

var person2 = new Person();
person2.sayName();		//"zhangsan"
console.log(person1.sayName == person2.sayName);	//true
```

​		在此，我们将`sayName()`方法和所有属性直接添加到了Person的prototype属性中，构造函数变成了空函数。即使如此，也仍然可以通过调用构造函数来创建新对象，而且新对象还会具有相同的属性和方法。但与构造函数模式不同的是，新对象的这些属性和方法是由所有实例共享的。换句话说，`person1`和`person2`访问的都是同一组属性和同一个`sayName()`函数。要理解原型模式的工作原理，必须先理解`ECMAScript`中原型对象的性质。

##### 		1.理解原型对象

​		无论什么时候，只要创建了一个新函数，就会根据一组特定的规则为该函数创建一个prototype属性，这个属性指向函数的原型对象。在默认情况下，所有原型对象都会自动获得一个constructor（构造函数）属性，这个属性包含一个指向prototype属性所在函数的指针。就拿前面的例子来说，`Person.prototype.constructor`指向Person。而通过这个构造函数，我们还可继续为原型对象添加其他属性和方法。

```js
Person == Person.prototype.constructor	//true
typeof Person.prototype		//"object"
```

​		创建了自定义的构造函数之后，其原型对象默认只会取得constructor属性：至于其他方法，则都是从Object继承而来的。当调用构造函数创建一个新实例后，该实例的内部将包含一个指针（内部属性），指向构造函数的原型对象。`ECMA-262`第五版中管这个指针叫[[Prototype]]。虽然在脚本中没有标准的方式访问[[Prototype]]，但Firefox、Safari和Chrome在每个对象上都支持一个属性`_proto_`；而在其他实现中，这个属性对脚本则是完全不可见的。不过，要明确的真正重要的一点就是，这个连接存在于实例与构造函数的原型对象之间，而不是存在于实例与构造函数之间。

```js
Person1._proto_ == Person;	//false
Person1._proto_ == Person.prototype;	//true
```

​		如下图所示，展现了各个对象之间的关系。



​		上图展示了Person构造函数、Person的原型属性以及Person现有的两个实例之间的关系。在此，`Person.prototype`指向了原型对象，而`Person.prototype.constructor`又指向了Person。原型对象中出了包含constructor属性之外，还包括后来添加的其他属性。Person的每个实例——`person1`和`person2`都包含一个内部属性，该属性仅仅指向了`Person.prototype`；换句话说，**它们与构造函数没有直接的关系**。此外，要格外注意的是，虽然这两个实例都不包含属性和方法，但我们却可以调用`person1.sayName()`。这是通过查找对象属性的过程来实现的。

​		`isPrototypeOf()`方法和`Object.getPrototypeOf()`方法是用来确定[[Prototype]]属性的方法。使用`Object.getPrototypeOf()`可以很方便地取得一个对象的原型，而这在利用原型实现继承的情况下是非常重要的。

​		每当代码读取某个对象的某个属性时，都会执行一次搜索，目标是具有给定名字的属性。	搜索首先从对象实例本身开始。如果在实例中找到了具有给定名字的属性，则返回该属性的值；如果没有找到，则继续搜索指针指向的原型对象，在原型对象中查找具有给定名字的属性。如果在原型对象中找到了这个属性，则返回该属性的值。

​		JavaScript高级程序设计里说，"虽然可以通过对象实例访问保存在原型中的值，但却不能通过对象实例重写原型中的值"。但是在chrome控制台里还是可以实现的，如下所示：

```js
person2.__proto__.name = "lisi";
person1.name == "lisi";		//true
```



##### 2.更简单的原型语法

​		读者大概注意到了，前面例子中每添加一个属性和方法就要敲一遍`Person.prototype`。为减少不必要的输入，也为了从视觉上更好地封装原型的功能，更常见的做法是用一个包含所有属性和方法的对象字面量来重写整个原型对象，如下面的例子所示。

```js
function Person() {
}

Person.prototype = {
	name : "zhangsan",
	age : "19",
	job : "Software Engineer",
	sayName : function() {
		console.log(this.name);
	}
};
```

​		在上面的代码中，我们将`Person.prototype`设置为等于一个以对象字面量形式创建的新对象。最终结果相同，但有一个例外：constructor属性不再指向Person了。前面曾经介绍过，每创建一个函数，就会同时创建它的prototype对象，这个对象也会自动获得constructor属性。而我们在这里使用的语法，本质上完全重写了默认的prototype对象，因此constructor属性也就变成了新对象的constructor属性（指向Object构造函数），不再指向Person函数。此时，尽管	`instanceof`操作符还能返回正确的结果，但通过constructor已经无法确定对象的类型了，如下所示。

```js
var friend = new Person();

console.log(friend instanceof Object);	//true
console.log(friend instanceof Person);	//true
console.log(friend.constructor == Object);	//true
console.log(friend.constructor == Person);	//false
```

​		在此，用`instanceof`操作符测试Object和Person仍然返回true，但constructor属性则等于Object而不等于Person了。如果constructor属性真的很重要，可以像下面这样特意将它设置回适当的值。

```js
function Person() {
}

Person.prototype = {
	constructor : Person,
	name : "zhangsan",
	age : "19",
	job : "Software Engineer",
	sayName : function() {
		console.log(this.name);
	} 
};
```



##### 3.原型的动态性

​		由于在原型中查找值得过程是一次搜索，因此我们对原型对象所做的任何修改都能够立即从实例上反映出来——即使是先创建了实例后修改原型也照样如此。如下例。

```js
var friend = new Person();
Person.prototype.sayHi = function() {
	console.log("hi");
};
friend.sayHi();	//"hi"
```

​		以上代码先创建了Person的一个实例，并将其保存在person中。然后，下一条语句在`Person.prototype`中添加了一个方法`sayHi()`。即使person实例是在添加新方法之前创建的，但它仍然可以访问这个新方法。其原因可以归结为实例与原型之间的松散连接关系。当我们调用`person.sayHi()`时，首先会在实例中搜索名为`sayHi`的属性，在没找到的情况下，会继续搜索原型。因为实例与原型之间的连接不过是一个指针，而非一个副本，因此就可以在原型中找到新的`sayHi`属性并返回保存在那里的函数。

​		



##### 4.原型对象的问题

​		原型模式也不是没有缺点。首先，它省略了为构造函数传递初始化参数这一环节，结果所有实例在默认情况下都将取得相同的属性值。虽然这会在某种程度上带来一些不方便，但还不是原型的最大问题。原型模式的最大问题是由其共享的本性所导致的。

​		原型中所有属性是被很多实例共享的，这种共享对于函数非常合适。对于那些包含基本值的属性倒也说得过去，毕竟，通过在实例上添加一个同名属性，可以隐藏原型中得对象属性。然而，对于包含引用类型值得属性来说，问题就比较突出了。如下例：

```js
function Person() {
}

Person.prototype = {
	constructor : Person,
	name : "zhangsan",
	age : "19",
	friends : ["lisi", "wangwu"],
	sayName : function() {
		console.log(this.name);
	} 
};

var person1 = new Person();
var person2 = new Person();

person1.friends.push("zhaoliu");

console.log(person2.friends);	//["lisi,wangwu,zhaoliu"]
```

​		假如我们得初衷就是像这样在所有实例中共享一个数组，那么对这个结果我没有话可说。可是，实例一般都是要有属于自己的全部属性的。而这个问题正是我们很少看到有人单独使用原型模式的原因所在。

​		尽管可以随时为原型添加属性和方法，并且修改能够立即在所有对象实例中反映出来，但如果时重写整个原型对象，那么情况就不一样了。我们知道，调用构造函数时会为实例添加一个指向最初原型的[[Prototype]]指针，而把原型修改为另外一个对象就等于切断了构造函数与最初原型之间的联系。



#### 4.组合使用构造函数模式和原型模式

​		创建自定义类型的最常见方式，就是组合使用构造函数模式与原型模式。构造函数模式用于定义实例属性，而原型模式用于定义方法和共享的属性。结果，每个实例都会有自己的一份实例属性的副本，但同时又共享着对方法的引用，最大限度地节省了内存。另外，这种混成模式还支持向构造函数传递参数；可谓是集两种模式之长。下面的代码重写了前面的例子。

```js
function Person(name, age, job) {
	this.name = name;
	this.age = age;
	this.job = job;
	this.friends = ["c", "d"];
}

Person.prototype = {
	constructor : Person,
	sayName : function() {
		console.log(this.name);
	}
}

var person1 = new Person("a", 19, "Software Engineer");
var person2 = new Person("b", 18, "Doctor");

person1.friends.push("e");
console.log(person1.friends);	//["c", "d", "e"]
console.log(person2.friends);	//["c", "d"]
console.log(person1.friends == person2.friends);	//false
console.log(person1.sayName == person2.sayName);	//true
```

​		在这个例子中，实例属性都是在构造函数中定义的，而由所有实例共享的属性constructor和方法`sayName()`则都是在原型中定义的。而修改了`person1.friends`（向其中添加一个新字符串），并不会影响到`person2.friends`。因为它们分别引用了不同的数组。

​		这种构造函数与原型混成的模式，是目前在`ECMAScript`中使用最广泛、认同度最高的一种创建自定义类型的方法。可以说，这是用来定义引用类型的一种默认模式。



#### 5.动态原型模式

​		有其他`OO`语言经验的开发人员在看到独立的构造函数和原型时，很可能会感到非常困惑。动态原型模式正是致力于解决这个问题的一个方案，它把所有信息都封装在了构造函数中，而通过在构造函数中初始化原型（仅在必要的情况下），又保持了同时使用构造函数和原型的优点。换句话说，可以通过检查某个应该存在的方法是否有效，来决定是否需要初始化原型。如下例所示。

```js
function Person(name, age, job) {
	//属性
	this.name = name;
	this.age = age;
	this.job = job;
	//方法
    if (typeof this.sayName != "function") {
        
        Person.prototype.syaName = function() {
          console.log(this.name);  
        };
    }
}

var friend = new Person("lisi", 29, "Software Engineer");
friend.sayName();
```

​		注意构造函数中方法的部分。这里只在`sayName()`方法不存在的情况下，才会将它添加到原型中。这段代码只会在初次调用构造函数时才会执行。此后，原型已经完成初始化，不需要再做什么修改了。不过要记住，这里对原型所作的修改，能够立即再所有实例中得到反映。因此，这种方法确实可以说非常完美。其中，if语句检查的可以是初始化之后应该存在的任何属性或方法——不必用一大堆if语句检查每个属性和每个方法；只要检查其中一个即可。对于采用这种模式创建的对象，还可以使用`instanceof`操作符确定它的类型。

​		使用动态原型模式时，不能使用对象字面量重写原型。前面已经解释过了，如果在已经创建了实例的情况下重写原型，那么就会切断现有实例与新原型之间的联系。如下例所示。

```js
function Person(name, age, job) {
	//属性
	this.name = name;
	this.age = age;
	this.job = job;
	//方法
    if (typeof this.sayName != "function") {
        
        Person.prototype= {
          sayName : function() {console.log(this.name);}
        };
    }
}

var friend = new Person("lisi", 29, "Software Engineer");
friend.sayName();	//Uncaught TypeError: friend.sayName is not a function
```











