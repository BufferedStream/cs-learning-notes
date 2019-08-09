## JavaScript继承

&emsp;&emsp;继承是 OO 语言中的一个最为人津津乐道的概念。许多 OO 语言都支持两种继承方式：接口继承和实现继承。接口继承只继承方法签名，而实现继承则继承实际的方法。如前所述，由于函数没有签名，在 ECMAScript 中无法实现接口继承。ECMAScript 只支持实现继承，而且其实现继承主要是依靠原型链来实现的。



#### 1.原型链

&emsp;&emsp; ECMAScript 中描述了原型链的概念，并将原型链作为实现继承的主要方法。其基本思想是利用原型让一个引用类型继承另一个引用类型的属性和方法。简单回顾一下构造函数、原型和实例的关系：每个构造函数都有一个原型对象，原型对象都包含一个指向构造函数的指针，而实例都包含一个指向原型对象的内部指针。那么，假如我们让原型对象等于另一个类型的实例，结果会怎么样呢？显然，此时的原型对象将包含一个指向另一个原型的指针，相应地，另一个原型中也包含着一个指向另一个构造函数的指针。假如另一个原型又是另一个类型的实例，那么上述关系依然成立，如此层层递进，就构成了实例与原型的链条。这就是所谓原型链的基本概念。

&emsp;&emsp;实现原型链有一种基本模式，其代码大致如下。

```js
function SuperType() {
	this.property = true;
}

SuperType.prototype.getSuperValue = function() {
	return this.property;
};

function SubType() {
	this.subproperty = false;
}

//继承了SuperType
SubType.prototype = new SuperType();

SubType.prototype.getSubValue = function() {
	return this.subproperty;
};

var instance = new SubType();
console.log(instance.getSuperValue());
```

&emsp;&emsp;instance 的结构如下图所示。



&emsp;&emsp;以上代码定义了两个类型： SuperType 和 SubType。每个类型分别有一个属性和一个方法。它们的主要区别是 SubType 继承了 SuperType，而继承是通过创建 SuperType 的实例，并将该实例赋给 SubType.prototype 实现的。实现的本质是重写原型对象，代之以一个新类型的实例。换句话说，原来存在于 SuperType 的实例中的所有属性和方法，现在也存在于  SubType.prototype 中了。在确立了继承关系之后，我们给 SubType.prototype 添加了一个方法，这样就在继承了 SuperType 的属性和方法的基础上又添加了一个新方法。这个例子中的实例以及构造构造函数和原型之间的关系如下图所示。

​		

&emsp;&emsp;在上面的代码中，我们没有是用 SubType 默认提供的原型，而是给它换了一个新原型；这个新原型就是 SuperType 的实例。于是，新原型不仅具有作为一个 SuperType 的实例所拥有的全部属性和方法，而且其内部还有一个指针，指向了 SuperType 的原型。最终结果就是这样的：instance 指向 SubType 的原型， SubType 的原型又指向  SuperType 的原型（通过 [[][][Prototype]] 指向的）。getSuperValue() 方法仍然还是 SuperType.prototype 中，但 property 则位于 SubType.prototype 中。这是因为 property 是一个实例属性，而 getSuperValue() 则是一个原型方法。既然 SubType.prototype 现在是 SuperType 的实例，那么 property 当然就位于该实例中了。此外，要注意 instance.constructor 现在指向的是 SuperType，这是因为原来 SubType.prototype 中的 constructor 被重写了的缘故。（实际上，不是 SubType 的 constructor 属性被重写了，而是 SubType 的原型通过 SuperType 的匿名对象指向了另一个对象——SuperType 的原型，而这个原型对象的 constructor 属性指向的是 SuperType）。

&emsp;&emsp;通过实现原型链，本质上扩展了本章前面介绍的原型搜索机制。读者大概还记得，当以读取模式访问一个实例属性时，首先会在实例中搜索该属性。如果没有找到该属性，则会继续搜索实例的原型。在通过原型链实现继承的情况下，搜索过程就得以沿着原型链继续向上。就拿上面的例子来说，调用 instance.getSuperValue() 会经历三个搜索步骤：1）搜索实例； 2）搜索 SubType.prototype； 3）搜索 SuperType.prototype，最后一步才会找到该方法。在找不到属性或方法的情况下，搜索过程总是要一环一环地前行到原型链末端才会停下来。

##### 		1.别忘记默认的原型

&emsp;&emsp;事实上，前面例子中展示的原型链还少一环。我们知道，所有引用类型都继承了 Object，而这个继承也是通过原型链实现的。大家要记住，所有函数的默认原型都是 Object 的实例，因此默认原型都会包含一个内部指针，指向 Object.prototype。这也正是所有自定义类型都会继承 toString()、valueOf() 等默认方法的根本原因。所以，我们说上面例子展示的原型链中还应该包括另外一个继承层次。下图为我们展示了该例子中完整的原型链。



&emsp;&emsp;一句话，SubType 继承了 SuperType，而 SuperType 继承了 Object。当调用 instance.toString() 时，实际上调用的是保存在 Object.prototype中的那个方法。



##### 		2.确定原型和实例的关系

&emsp;&emsp;可以通过两种方式来确定原型和实例之间的关系。第一种方式是使用 instanceof 操作符，只要用这个操作符来测试实例与原型链中出现过的构造函数，结果就会返回 true。以下几行代码就说明了这一点。

```js
console.log(instance instanceof Object);	//true
console.log(instance instanceof SuperType);	//true
console.log(instance instanceof SubType);	//true
```

&emsp;&emsp;由于原型链的关系，我们可以说 instance 是 Object、SuperType 或 SubType 中任何一个类型的实例。因此，测试这三个构造函数的结果都返回了 true。

&emsp;&emsp;第二种方式是使用 isPrototypeof() 方法。同样，只要是原型链中出现过的原型，都可以说是该原型链所派生的实例的原型，因此 isPrototypeOf() 方法也会返回true，如下所示。

```js
console.log(Object.prototype.isPrototypeOf(instance));	//true
console.log(SuperType.prototype.isPrototypeOf(instance));	//true
console.log(SubType.prototype.isPrototypeOf(instance));	//true
```



##### 		3.谨慎地定义方法

&emsp;&emsp;子类型有时候需要重写超类型中的某个方法，或者需要添加超类型中不存在的某个方法。但不管怎样，给原型添加方法的代码一定要放在替换原型的语句之后。来看下面的例子。

```js
function SuperType() {
	this.property = true;
}

SuperType.prototype.getSuperValue = function() {
	return this.property;
};

function SubType() {
	this.subproperty = false;
}

//继承了SuperType
SubType.prototype = new SuperType();

//添加新方法
SubType.prototype.getSubValue = function() {
	return this.subproperty;  
};

//重写超类型中的方法
SubType.prototype.getSuperValue = function() {
  	return false;  
};

var instance = new SubType();
console.log(instance.getSuperValue());
```

如上所示，SubType.prototype 中的同名方法会覆盖 SuperType.prototype 中的同名方法。



##### 		4.原型链的问题

&emsp;&emsp;原型链虽然很强大，可以用它来实现继承，但它也存在一些问题。其中，最主要的问题来自包含引用类型值的原型。想必大家还记得，我们前面介绍过包含引用类型值的原型属性会被所有实例共享；而这也正是为什么要在构造函数中，而不是在原型对象中定义属性的原因。在通过原型来实现继承时，原型实际上会变成另一个类型的实例。于是，原先的实例属性也就顺理成章地变成了现在的原型属性了。于是，原型的实例属性也就顺理成章地变成了现在的原型属性了。下列代码可以用来说明这个问题。

```js
function SuperType() {
	this.colors = ["red", "blue", "green"];
}

function SubType() {
}

//继承了 SuperType
SubType.prototype = new SuperType();

var instance1 = new SubType();
instance1.colors.push("black");
console.log(instance1.colors);		//["red", "blue", "green", "black"]

var instance2 = new SubType();
console.log(instance1.colors);		//["red", "blue", "green", "black"]
```

&emsp;&emsp;原型链的第二个问题是：在创建子类型的实例时，不能向超类型的构造函数中传递参数。实际上，应该说是没有办法在不影响所有对象实例的情况下，给超类型的构造函数传递参数。有鉴于此，再加上前面刚刚讨论过的由于原型中包含引用类型值所带来的问题，实践中很少会单独使用原型链。

​	

#### 2.借用构造函数

&emsp;&emsp;在解决原型中包含引用类型值所带来问题的过程中，开发人员开始使用一种叫做借用构造函数（constructor stealing）的技术（有时候也叫做伪造对象或经典继承）。这种技术的基本思想非常简单，即在子类型构造函数的内部调用超类型构造函数。别忘了，函数只不过是在特定环境中执行代码的对象，因此通过使用 apply() 和 call() 方法也可以在（将来）新创建的对象上执行构造函数，如下所示：

```js
function SuperType() {
	this.colors = ["red", "blue", "green"];
}

function SubType() {
	//继承了 SuperType
	SuperType.call(this);
}

var instance1 = new SubType();
instance1.colors.push("black");
console.log(instance1.colors);

var instance2 = new SubType();
console.log(instance2.colors);	
```

&emsp;&emsp;代码中加背景的那一行代码“借调”了超类型的构造函数。通过使用 call()方法（或 apply()方法也可以）。我们实际上是在（未来将要）新创建的 SubType 实例的环境下调用了 SuperType 构造函数。这样一来，就会在新 SubType对象上执行 SuperType() 函数中定义的所有对象初始化代码。结果，SubType 的每个实例都会具有自己的 colors 属性的副本了。

​		

##### 				1.传递参数

&emsp;&emsp;相对于原型链而言，借用构造函数有一个很大的优势，即可以在子类型构造函数中向超类型构造函数传递参数。看下面这个例子。

```js
function SuperType(name) {
	this.name = name;
}

function SubType() {
	//继承了 SuperType，同时还传递了参数
	SuperType.call(this, "zhangsan");
	
	//实例属性
	this.age = 29;
}

var instance = new SubType();
console.log(instance.name);		//"zhangsan"
console.log(instance.age);		//29
```

&emsp;&emsp;以上代码中的 SuperType 只接受一个参数 name，该参数会直接赋给一个属性。在 SubType 构造函数内部调用 SuperType 构造函数时，实际上是为 SubType 的实例设置了 name 属性。为了确保 SuperType 构造函数不会重写子类型的属性，可以在调用超类型构造函数后，再添加应该再子类型中定义的属性。

​		

##### 		2.借用构造函数的问题

&emsp;&emsp;如果仅仅是借用构造函数，那么也将无法避免构造函数模式存在的问题——方法都在构造函数中定义，因此函数复用就无从谈起了。而且，在超类型的原型中定义的方法，对子类型而言也是不可见的，结果所有类型都只能使用构造函数模式。考虑到这些问题，借用构造函数的技术也是很少单独使用的。