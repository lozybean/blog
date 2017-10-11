Title: JavaScript面向对象
Date: 2015-11-18
Category: Learning
Tags: javascript

## 理解

JavaScript中并没有类的概念, 对象的定义是一系列无序属性的集合, 通过给对象添加各种属性来完成对象的构造:

```javascript
function print(str){
	document.write('<p>' + str + '</p>')
}
var obj = new Object();
obj.name = 'xiaoming';
obj.age = 16;
obj.sayHi = function(){
	print("Hi, this is " + this.name + ", I am " + this.age + " years old.");
};
obj.sayHi();  // Hi, this is xiaoming, I am 16 years old.
```

也通过统一的函数对对象添加统一属性来模拟类的功能:

```javascript
function createPerson(name, age){
	var obj = new Object();
	obj.name = name;
	obj.age = age;
	obj.sayHi = function(){
		print("Hi, this is " + this.name + ", I am " + this.age + " years old.");
	};
	return obj;
}
var xiaoming = createPerson('xiaoming', 16);
xiaoming.sayHi();  // Hi, this is xiaoming, I am 16 years old.
```

但是这样会将对象所有的属性和方法都放到对象本身中, 在内存上有比较大的浪费, 在其他语言(如:Ruby)中, 会将方法放在类中, 属性放在对象本身, 对象调用方法时再到类中调用的方式;

为了实现这些特性, 就要使用JavaScript的`prototype`(原型), 这也是称作原型语言的原因. 其他如`Lua`、`Io`等也是原型语言;

原型模式比一般的类模式更加灵活, 可以控制的内容更多, 并且通过设置属性的特性, 完成非常丰富的面向对象特性;

## 属性特性

对象的属性具有一些特性, 这些特性不能被外部直接访问, 通过设置这些特性, 丰富对象的特性;

属性的特性并不能直接访问, 只能通过`Object.defineProperty()`方法设置, 该方法接受三个参数: 属性所在对象、属性名字、描述符对象(即特性以及对应值的字典); 通过`Object.getOwnPropertyDescription()`方法, 可以查看已有属性的特性, 该方法接受两个参数: 属性所在对象、属性名字;

### 数据属性

数据属性具有以下四个特性:

* `[[configurable]]`: 表示能否通过`delete`删除属性或者重定义属性, 能否修改属性的特性或者能否将属性改为访问器属性; 直接在对象上定义时, 默认为`true`;
* `[[enumerable]]`: 表示能否通过`for-in`循环来返回属性, 直接在对象上定义时, 默认为`true`;
* `[[writable]]`: 表示能否修改属性的值, 直接在对象上定义时, 默认为`true`;
* `[[value]]`: 包含该属性的数据值, 默认为`undefined`;

```javascript
var obj = new Object();
obj.name = 'xiaoming';
obj.age = 16;
Object.defineProperty(obj, 'score', {
	configurable: false,
	enumerable: false,
	value: 59,
});
delete obj.score;
obj.score = 60;
print(obj.score);  // 59
for(p in obj){
	print(p); // name, age
}
```

### 访问器属性

访问器属性即包含`getter`和`setter`函数的属性, 其本身并不包含任何值, 该属性有以下四个特性:

* `[[configurable]]`: 表示能否通过`delete`删除属性或者重定义属性, 能否修改属性的特性或者能否将属性改为数据属性; 用`Object.defineProperty()`定义时, 默认为`false`;
* `[[enumerable]]`: 表示能否通过`for-in`循环来返回属性, 用`Object.defineProperty()`定义时, 默认为`false`;
* `[[get]]`: 表示`getter`方法, 默认为`undefined`;
* `[[set]]`: 表示`setter`方法, 默认为`undefined`;

可以看到, 访问器属性只能通过`Object.defineProperty()`来定义;

```javascript
var obj = new Object();
obj.name = 'xiaoming';
obj.age = 16;
Object.defineProperty(obj, '_score',{
	value: 59,
})
Object.defineProperty(obj, 'score',{
	enumerable: true,
	get: function(){
		return this._score;
	},
	set: function(value){
		print('you can not set the score!');
		this._score = this._score;
	}
})
description = Object.getOwnPropertyDescriptor(obj, '_score');
print(description.configurable);  // false
print(description.enumerable);  // false
obj.score = 60;  // you can not set the score!
print(obj.score);  // 59
for(p in obj){
	print(p); // name, age
}
```

在访问器属性中, 可以修改其他的属性, 注意一定不要在`set`中给自身属性赋值, 这相当于是调用`set`, 造成自身调用的死循环;

## 原型

### 构造器模式

JavaScript中除了如开头的工厂模式创建对象之外, 还可以借助`this`来实现构造器模式:

```javascript
function Person(name, age){
	this.name = name;
	this.age = age;
	this.sayHi = function(){
		print("Hi, this is " + this.name + ", I am " + this.age + " years old.");
	};
}
var xiaoming = new Person('xiaoming', 16);
xiaoming.sayHi();  // Hi, this is xiaoming, I am 16 years old.
```

上述过程中, `new Person('xiaoming', 16)`完成以下四个步骤:

1. 创建一个新对象;
2. 将构造函数的作用域赋给新对象(使`this`指向该对象);
3. 执行构造函数;
4. 返回新对象;

同时, 通过以上过程创建的对象会附带一个构造器(`constructor`)属性来指向构造函数; 由该方法创建的对象可以通过`instanceof`关键词的判断(就像类的表现一样):

```javascript
print(xiaoming.constructor == Person);  // true
print(xiaoming instanceof Person);  // true
print(xiaoming instanceof Object);  // true
```

### 引入原型

在构造器模型中, 对象的`constructor`属性正确指向了构造出该对象的函数, 所有该函数实例化的对象都有这个属性, 并且都指向该函数, 并且该属性并不属于对象本身, 这里其实就用到了开头提到的原型:

* 原型`prototype`是函数的一个默认属性, 该属性指向一个原型对象, 并且该对象最初只有一个`constructor`属性, 指向函数;
* 函数实例化的每一个对象都包含了一个内部属性`__proto__`指向函数的原型;
* 在一个对象访问某个属性时, 如果该属性不在对象中, 则会从对象的原型中访问;
* 通过`hasOwnProperty()`可以判断一个属性究竟是从哪里访问的; 而通过`in`操作符则仅仅判断对象是否可以访问该属性, 并不检查是否在原型中; 如上例`xiaoming.constructor`实际上访问的是原型的`constructor`属性;
* 实例对象的指针仅仅指向构造函数的原型, 而不指向构造函数本身;

以下的例子更加清晰地描述了原型的特性;

```javascript
function Foo(){};
print(Foo.prototype.constructor == Foo );  // true
Foo.prototype.name = 'xiaoming';
Foo.prototype.age = 16;
var xiaoming = new Foo();
print(xiaoming.__proto__.name);  // xiaoming
print(xiaoming.name);  // xiaoming
print(xiaoming.hasOwnProperty('name'));  // false
print('name' in xiaoming);  // true
xiaoming.name = 'xiaohong';
print(xiaoming.hasOwnProperty('name'));  // true
print(xiaoming.name);  // xiaohong
print(xiaoming.__proto__.name);  // xiaoming
```

> 注意, 通过`for-in`来遍历对象属性时, 实例中的属性如果设置`enumerable = false`则不会返回, 而在原型中的属性, 无论该特性为何值, 都将被返回; 在IE8及更早版本中除外;

### 原型的特性

进行以下实验:

```javascript
function Foo(){};
old_prototype = Foo.prototype;
var xiaohong = new Foo();
old_prototype.name = 'xiaohong';
print(xiaohong.name);  // xiaohong
print(xiaohong.__proto__ == old_prototype);  // true
Foo.prototype = {
	name: 'xiaoming',
	age: 16,
}
print(old_prototype.constructor == Foo);  // true
print(Foo.prototype.constructor == Foo);  // false
print(Foo.prototype.__proto__.constructor == Object);  // true
var xiaoming = new Foo();
print(xiaohong.constructor == Foo); // true
print(xiaoming.constructor == Foo);  // false
print(xiaoming.constructor == Object);  // true
```

上述过程描述了以下事实:

* 修改或者添加原型的某一个属性后, 实例对象如果访问的是原型中的属性, 则会被动态刷新;
* 函数和原型是两个独立的对象, 仅仅通过各自的属性指向对方产生关联;
* 可以重写函数的原型, 但是此时只保留了函数通过`prototype`与原型的关联, 而不具备原型通过`constructor`与函数的关联;
* 由于原型是一个对象, 当新原型自身不存在的`constructor`方法时, 将会访问其原型对象的`constructor`方法, 即`Object()`的原型的`constructor`;
* 旧的原型对象如果仍然被使用而没有回收, 则依然拥有指向函数的`constructor`属性;
* 重写原型之前的实例对象指向了旧的原型; 重写原型之后的实例对象指向了新的原型;

根据以上事实, 可以想到(为方便描述, 称上述函数为类型):

* 修改或者添加已有类型的原型的属性, 而不修改原型本身, 则可以调整或者添加已有类型的功能;
* 修改已有类型的原型本身, 则会切断旧原型和类型的关系;
* 如果原型的属性是可变值, 如数组、对象等, 则某个实例对数组中的某个值、对象中的某个属性做出的修改会在原型的基础上进行, 从而改变所有实例的属性;
* 如果原型的属性是不可变值, 则某个实例修改该属性时, 实际上是在实例自身中创建了一个新的同名属性, 该修改不会影响其他实例的属性;
* 基于以上两点, 应该只在原型中保存可共享的属性, 避免存放可变值;

### 构造模式和原型组合

为了避免原型中的一些不应该共享的属性被共享, 应该组合构造模式和原型的特点, 如:

```javascript
function Person(name, age){
	this.name = name;
	this.age = age;
	this.friends = []
}
Person.prototype = {
	constructor: Person,
	sayHi: function(){
		print("Hi, this is " + this.name + ", I am " + this.age + " years old.");
	}
}
var xiaohong = new Person('xiaohong', 15);
xiaohong.friends = ['xiaoming','xiaopang'];
var xiaoming = new Person('xiaoming', 16);
xiaoming.friends = ['xiaohong','xiaopang'];
print(xiaohong.friends);  // xiaoming,xiaopang
print(xiaoming.friends);  // xiaohong,xiaopang
```

上面的特性也基于类的面向对象语言已经很相似, 即类中保存了方法等共享属性, 实例对象中保存其他属性; 可以将原型看做是类, 那么在原型中的属性也就是类属性, 当实例对象中没有该属性时, 会从类属性中调用, 如果类属性是可变对象, 则实例对其进行的更改也是在类属性的基础上, 这些描述看起来比较复杂, 结合Python等语言的属性查找方式则可以很好理解;

### 动态原型模式

构造模式和原型的组合很好解决了属性共享的问题, 但是需要分两块进行, 通过动态原型模式将这两个部分全部封装到构造函数中:

```javascript
function Person(name, age){
	this.name = name;
	this.age = age;

	if (typeof this.sayHi != 'function'){
		Person.prototype.sayHi = function(){
			print("Hi, this is " + this.name + ", I am " + this.age + " years old.");
		}
	}
}
var xiaoming = new Person('xiaoming', 16);
xiaoming.sayHi();
```

上述代码中, 在构造函数中添加了一个判断, 该判断的作用是检查构造函数是否是初次调用, 如果是初次调用, 则`this`中还没有某些应该有的方法(使用任意一个即可完成检查), 此时, 需要往原型中添加需要被共享的属性; 如果不是初次调用, 则由于原型中已经存在共享的属性, 该判断不会通过, 也不会再次修改原型中的共享属性;

### 寄生构造函数模型

寄生构造函数是工厂模式的一个扩展, 其写法和工厂模式一样, 用于给已有的类添加一些属性来得到一个特殊的类, 由于该类是通过`return`返回得到, 所以创建的对象和构造函数(以及其原型)之间没有任何关系, 所以并不能通过`instance of`等操作来确定对象的类型;

该模式和工厂模式没有本质区别, 只不过在工厂中新建了一个比较具象的类, 而不再是Object基类而已;

```javascript
function SpecialArray(){
	var values = new Array();
	values.push.apply(values, arguments);
	values.toPipedString = function(){
		return this.join("|");
	};
	return values;
}
var colors = new SpecialArray("red", "blue", "green");
```

### 稳妥构造函数模型

和寄生构造函数模型类似, 稳妥构造函数也是通过返回对象的方式来创建对象, 但是比寄生构造函数模型增加了一些限制:

* 不创建公共属性;
* 新创建的对象的实例方法不引用this;
* 不使用new操作符调用构造函数;

该模型主要应用于一些不能够使用this以及new的安全环境中;

## 继承

### 原型链

在原型的特性中, 讨论了原型和构造函数之间的关系, 并且知道当访问一个对象的属性时, 会先从对象中查找, 如果找不到则会从对象的原型中查找, 而由于原型也是一个对象, 如果仍然找不到, 则会从原型的原型中查找, 依次类推, 有些类似于Python中查找方式;

这种由原型串联起来的关系称为**原型链**, JavaScript中通过原型链实现继承的过程;

通过原型的`__proto__`指向另一个原型的方式实现继承的结构, 该链的终端是`Object.prototype`; 注意不应该直接使用`__proto__`属性指向另一个原型, 某些浏览器中并没有该属性, 并且这种方法并不直观, 而应该使用`new`;

```javascript
function Person(name, age){
	this.name = name;
	this.age = age;
}
Person.prototype.sayHi = function(){
	print("Hi, this is " + this.name + ", I am " + this.age + " years old.");
}
function Student(id){
	this.id = id;
}
Student.prototype = new Person();
Student.prototype.sayHi = function(){
	print("Hi, this is " + this.name + ", I am a student, " + this.age + " years old.");
}
var xiaoming = new Student();
print(xiaoming instanceof Student);  // true
print(xiaoming instanceof Person);  // true
print(xiaoming instanceof Object);  // true
print(Student.prototype.isPrototypeOf(xiaoming));  // true
print(Person.prototype.isPrototypeOf(xiaoming));  // true
print(Object.prototype.isPrototypeOf(xiaoming));  // true
```

由于原型之间通过原型链连接在一起, 所以不应该使用字面量的方式来定义一个原型, 也不能将原型改成其他的对象(不能修改指针), 之前讨论过, 修改原型对象会破坏原型与已有对象的联系, 而在原型链中, 已有对象就是原型, 改变原型链中的原型对象, 会破坏整个原型链;

* 不能使用字面量方式或者任何修改原型对象指针的操作, 不然会破坏原型链;
* 给原型对象添加修改属性不会破坏原型链, 但是要注意原型链查找的方向, 在链前端修改的属性并不能在链后端访问得到;
* 如果是不可变属性, 则修改该属性实际上是删除已有的属性并且添加新的同名属性, 此时如果原本属性在链后端, 则会在当前位置添加一个新的属性对象;
* 如果是可变属性, 应该避免放到原型链中, 如果确实需要这么做, 那么一些不会改变该属性的操作将不会新建新的属性对象, 而是在原本的对象上完成修改;

整个继承过程和Python、Ruby等语言非常相似, 按照当前对象 -> 当前对象的类 -> 当前对象的父类 -> 父类的父类 ... -> 基本类 的顺序完成属性的查找, 也就是继承的过程:

> 向右一步, 向上查找

### 借用构造函数

在最基本的原型链上, 子类的构造函数的参数难以传递到父类的构造函数中, 通过借用父类的构造函数可以完成参数的传递:

```javascript
function Person(name, age){
	this.name = name;
	this.age = age;
}
function Student(name, age, id){
	Person.call(this, name, age);

	this.id = id;
}
```

通过以上方式即可完成参数的传递, 如果想要添加方法或者其他共享属性, 则应该组合使用原型链:

### 组合继承

```javascript
function Person(name, age){
	this.name = name;
	this.age = age;
}
Person.prototype.sayHi = function(){
	print("Hi, this is " + this.name + ", I am " + this.age + " years old.");
}
function Student(name, age, id){
	Person.call(this, name, age);
	this.id = id;
}
Student.prototype = new Person();
Student.prototype.sayHi = function(){
	print("Hi, this is " + this.name + ", I am a student, " + this.age + " years old.");
}
var xiaoming = new Student('xiaoming', 16, 1);
xiaoming.sayHi();  // Hi, this is xiaoming, I am a student, 16 years old.
```

在组合继承和原型链的两个例子中`Student.prototype = new Person();`进行了修改原型的操作, 这样产生的原型中并没有`constructor`属性, 但是由于原型链是完备的, 所以仍然可以通过`instanceOf()`等函数的检查, `constructor`属性在原型链中并不起到实际作用, 指定该属性为构造函数本身有利于查找对象的构造函数;

在组合继承的过程中, 子类型的原型调用了一次父类型的构造函数, 子类型的构造函数中再一次调用父类型的构造函数, 在原型中首先会得到父类型的属性, 然后再在子类型的构造函数中覆盖这些属性或者直接调用原型中的属性;

### 原型继承

下面实现一种轻量级的继承方法:

```javascript
function object(o){
	function F(){};
	F.prototype = o;
	return new F();
}
var xiaoming = {
	name: 'xiaoming',
	age: 16,
	sayHi: function(){
		print("Hi, this is " + this.name + ", I am a student, " + this.age + " years old.");
	}
}
var xiaohong = object(xiaoming);
xiaohong.name = 'xiaohong';
xiaohong.age = 15;
xiaohong.sayHi();  // Hi, this is xiaohong, I am a student, 15 years old.
```

上面的方法没有创建新的类型, 甚至不需要构造函数, 通过给一个对象添加原型完成了继承的操作, 这是一种比较轻便的继承实现, 在比较新的浏览器中对该方式进行了规范:

```javascript
var xiaoming = {
	name: 'xiaoming',
	age: 16,
	sayHi: function(){
		print("Hi, this is " + this.name + ", I am a student, " + this.age + " years old.");
	}
}
var xiaohong = Object.create(xiaoming, {
	name:{
		value: 'xiaohong'
	},
	age:{
		value: 15
	}
});
xiaohong.sayHi();  // Hi, this is xiaohong, I am a student, 15 years old.
```

`Object.create()`也可仅仅指定一个参数来指定原型对象, 然后在外部实现属性的定义;

在这种继承方式中, 所有的属性都是共享的, 当包含可变属性时, 子类型会对父类型造成影响;

### 寄生组合继承

以上介绍的继承方式, 组合继承比较常用, 但是会在子类型对象的原型中添加一些不必要的属性, 为了解决这个问题, 需要使用下面的方式:

```javascript
function inheritPrototype(subType, superType){
	var prototype = Object(superType.prototype);
	prototype.constructor = subType;
	subType.prototype = prototype;
}
function Person(name, age){
	this.name = name;
	this.age = age;
}
Person.prototype.sayHi = function(){
	print("Hi, this is " + this.name + ", I am " + this.age + " years old.");
}
function Student(name, age, id){
	Person.call(this, name, age);
	this.id = id;
}
inheritPrototype(Student, Person);
Student.prototype.sayHi = function(){
	print("Hi, this is " + this.name + ", I am a student, " + this.age + " years old.");
}
print(Student.prototype.name);  // undefined
var xiaoming = new Student('xiaoming', 16, 1);
xiaoming.sayHi();  // Hi, this is xiaoming, I am a student, 16 years old.
```

在上面的过程中, `Student.prototype`指向了父类型原型, 而没有将父类型中的属性添加到自身, 这种方式是比较理想的继承范式;
