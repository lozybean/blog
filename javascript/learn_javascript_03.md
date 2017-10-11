Title: JavaScript函数表达式
Date: 2015-11-22
Category: Learning
Tags: javascript

## 函数的定义

JavaScript中函数可以通过两种形式定义:

* 使用`function`关键字声明, 函数的声明会在代码执行之前进行;
* 使用函数表达式声明, 可以选择使用一个对象保存函数, 也可以选择不保存而作为匿名函数使用;

```javascript
// 使用函数声明
function foo(){

}
// 使用函数表达式
var foo = function(){

}
```

上面两种方式的最大区别是, 第一种方式会在代码运行前进行, 从而可以在声明之前调用; 第二种方式只会在代码运行过程中进行, 在函数表达式之前调用该函数将会发生错误;

另外一个区别是, 由于函数的声明会在代码运行前进行, 所以不可以声明两个同名函数, 哪怕是在分支结构中, 此时浏览器只会返回其中一个申明, 具体返回的函数根据浏览器不同而不同;

```javascript
// 以下代码将产生不可预知错误！
if(condition){
	function foo(){

	}
}else{
	function foo(){

	}
}
// 应该使用函数表达式正确完成:
if(condition){
	var foo = function(){

	}
}else{
	var foo = function(){

	}
}
```

## 递归

由于函数名称只是一个对象指针, 如果在函数中使用函数名调用本身, 则如果在外部更改函数的内容会引发调用错误;

如, 一个阶乘函数的递归实现中:

```javascript
function factorial(num){
	if(num <= 1){
		return 1;
	}else{
		return num * factorial(num-1);
	}
}
print(factorial(4));  // 24
var anotherFactorial = factorial;
factorial = null;
anotherFactorial(4);  // 错误!在递归中无法调用factorial()
```

为了消除该错误, 应该避免在递归中使用函数名, 而使用`argument.callee()`来获取当前函数代替; 但是某些情况下, 该函数并不能被使用, 并且在文档中明确指出该方法将在未来取消; 更好的做法是使用函数表达式来完成:

```javascript
var factorial = (function f(num){
	if(num <= 1){
		return 1;
	}else{
		return num * f(num-1);
	}
});
print(factorial(4));  // 24
var anotherFactorial = factorial;
factorial = null;
anotherFactorial(4);  // 24
```

## 作用域链

函数包含两个部分: **执行环境**和**代码对象**;

执行环境包含一个作用域链, 该作用域链本质上是一个指向变量对象的列表, 按顺序依次为: 当前活动对象 -> 上一级活动对象 -> ... -> 全局变量对象; 其中, 上一级活动对象是指当函数包含在其他函数中时, 可以访问其他函数的活动对象, 此时称内部的函数为外部函数的*闭包*;

```javascript
var envValue = 'envirValue';
function outerFunction(){
	var outerValue = 'out1';
	function innerFunction(){
		var innerValue = envValue + ' ' + outerValue + 'in innerFunction';
		return innerValue;
	}
	return innerFunction;
}
innerFunctionCopy = outerFunction();
innerFunction = null;
print(innerFunctionCopy());  // envirValue out1in innerFunction
```

上面的例子中, `innerFunction`是`outerFunction`的一个闭包, `innerFunction`的作用域链包含:

1. 当前活动对象: innerValue;
2. 上一级活动对象: outerValue, innerFunction;
3. 全局变量对象: envValue, outerFunction, innerFunctionCopy, innerFunction, ...;

全局变量对象之所以包含一个省略号, 因为后面在全局环境中添加的内容仍然会包含到作用域链中;

而由于作用域链访问时的顺序, 即使`innerFunctionCopy`也使用`innerFunction`的名字, 或者将`innerFunction`改成其他的对象, 内部函数的仍然可以正常进行;

这是因为, 当函数执行完毕后, 如果包含闭包, 则该闭包的作用域链中包含该函数的局部对象的引用, 此时, 函数的局部对象并不会被销毁, 也就是会将`innerFunction`保留在内部作用域链中; 外部函数的作用域链则仍然会被销毁;

但是要注意的是, 上例中不能修改`outerFunction`为其他对象, 因为outerFunction在全局变量对象中, 一旦修改, 则真正修改了作用域链中的该函数, 就会造成递归中提到的错误;

由于闭包的这个特性, 为了避免占用内存, 应该释放对其他函数作用域的引用, 如:

```javascript
innerFunctionCopy = null;
```

## 定义时与运行时

一个函数在被声明或者定义时, 其代码对象并不会被执行, 只有当使用`()`调用函数时, 其代码对象才会被执行;

```javascript
function createFunctions(){
	var result = new Array();
	for(var i=0; i<10; i++){
		result[i] = function(){
			return i;
		};
	}
	return result;
}
var results = createFunctions()
print(results.map(function(f){
	return f();
}))  // 10,10,10,10,10,10,10,10,10,10
```

1. 执行外部函数createFunction(), 返回一个数组;
2. 在外部函数中, 给数组的每个元素赋值为一个函数对象;
3. 执行每个函数对象, 在作用域链中寻找变量`i`, 并且在上一级中找到, 此时, `i == 10`;

上面的例子可以看到, 将一个函数赋值给一个变量, 并不能执行函数, 而当执行函数时, 作用域链中的对象可能已经发生改变, 为了避免这个错误, 应该赋值函数的执行结果而非代码对象:

```javascript
function createFunctions(){
	var result = new Array();

	for(var i=0; i<10; i++){
		result[i] = function(){
			return i;
		}();   // 返回执行结果;
	}
	return result;
}
var results = createFunctions()
print(results);  // 0,1,2,3,4,5,6,7,8,9
```

同样的问题在使用`this`对象时也会发生:

```javascript
var name = 'env';
var object = {
	name: 'object',
	getNameFunc: function(){
		return function(){
			return this.name;
		};
	}
}
print(object.getNameFunc()());  // env
```

上述代码中, 完成下面的操作:

1. getNameFunc()执行结果返回一个子函数, 该子函数返回`this.name`
2. 再跟一个括号来执行该子函数, 此时, 子函数的作用域链被暴露在全局环境中, 并不再是外部函数的闭包;

类似于:

```javascript
print((object.getNameFunc())());
```

而如果想要返回`object`则应该将子函数暴露在外部函数中, 或者将外部作用域的`this`变量保存到其他对象中, 更改内部函数的作用域链;

改变执行顺序:

```javascript
var name = 'env';
var object = {
	name: 'object',
	getNameFunc: function(){
		return function(){
			return this.name;
		};
	}()
}
print(object.getNameFunc());  // object
```

保存`this`:

```javascript
var name = 'env';
var object = {
	name: 'object',
	getNameFunc: function(){
		var that = this;
		return function(){
			return that.name;
		};
	}
}
print(object.getNameFunc()());  // object
```

上述两种处理都可以得到相同的效果, 但是原理却并不相同; 第一种方式是改变了两个函数的执行顺序, 第二种方式是改变了函数的作用域链;

PS: 有一个书上指明但是个人认为会引起混肴的地方:

```javascript
var name = 'env';
var object = {
	name: 'object',
	getNameFunc: function(){
		return this.name;
	}
}
print(object.getNameFunc());  // object
print((object.getNameFunc)());  // object
print((object.getNameFunc = object.getNameFunc)());  // env
print(object.getNameFunc = object.getNameFunc); // function (){ return this.name; }
```

引起上面的问题主要是函数的特性引起, 而是赋值操作的返回值是函数本身, 此时再调用该函数, 自然就将其暴露在全局环境中了; 而再其他语言中, 赋值操作返回值一般为是否成功的状态;

## 模仿私有作用域

JavaScript中没有块级作用域, 也就是说, 在`if`和`for`等语句中的变量在出了该语句块之后依然生效; 但是可以借助函数的作用域链的形式, 模仿封装一个块级作用域的功能:

```javascript
(function(){
	// 块级作用域
})();
```

这里要注意的一点是, `function`会被当做关键字, 会被当成函数声明处理, 函数声明后面不可以跟括号; 用括号将整个函数体括起来, 才能使其成为匿名函数表达式;

当一些地方需要创建一个私有作用域, 防止一些零时变量污染作用域, 则可以使用以上的方式模拟一个私有作用域;

该匿名没有被其他对象引用, 在执行完毕后并不会造成内存占用的问题;

## 私有变量

由于在函数中定义的局部变量不可以在函数外被使用, 同样的在构造函数中, 对象也不可访问到构造函数中的变量, 只能通过`this`给对象添加属性, 根据这个特性, 可以模拟私有属性出来:

```javascript
function Person(name){
	var name = name;

	this.getName = function(){
		return name;
	}
}
var xiaoming = new Person('xiaoming');
print(xiaoming.name);  // undefined
print(xiaoming.getName); //
```

此时, 实例化对象无法访问到函数中的私有变量`name`, 只能通过对象共有属性`getName`来访问;

通过下面的方式可以模拟静态属性:

```javascript
(function(){

	var name = "";

	Person = function(value){
		name = value;
	};

	Person.prototype.getName = function(){
		return name;
	}

	Person.prototype.setName = function(value){
		name = value;
	}

})();
var xiaoming = new Person('xiaoming');
print(xiaoming.getName());  // xiaoming
var xiaohong = new Person();
xiaohong.setName('xiaohong');
print(xiaohong.getName());  // xiaohong
print(xiaoming.getName());  // xiaohong
```

通过以上模拟, 一个实例对象对静态变量的修改会影响到所有的实例对象;

## 模块模式

一个单例对象可以用一个字面量对象来表示:

```javascript
var singleton = {
	name: '',
	method: function(){
		// ...
	}
}
```

通过模拟私有化变量, 可以对单例模式进行增强, 为其添加私有属性和方法, 甚至使其成为某个类型的实例:

```javascript
var singleton = function(){

	var privateValue = "";

	function privateFunction(){
		return false;
	}

	var object = new CustomType();  // 创建特定类型的实例

	object.publicValue = "";

	object.publicMethod = function(){

	};

	return object;
}();
```

上述代码中, 创建了一个单例对象`singleton`, 并且构建了私有环境为其添加私有属性, 通过返回一个对象的方式, 完成其公有属性的定义;
