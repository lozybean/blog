Title: JavaScript基础梳理
Date: 2015-11-09
Category: Learning
Tags: javascript

## 学习环境

1. 浏览器: Safari(9.0.1);
2. 使用Nginx配置最简单的web服务器;
3. 使用vim编辑js文件,并通过html网页显示运行结果;

```javascript
function print(str){
	document.write('<p>' + str + '</p>')
}
```

## 基本变量类型

通过`typeof`**操作符**可以查看变量的类型; JavaScript使用动态类型;

变量类型分基本类型和引用类型;

基本数据类型包括:`Undefined`、`Null`、`Boolean`、`Number`、`String`, 这五种基本数据类型是按值访问的, 并且不可以给这些类型添加属性; 复制基本数据类型时, 复制前后的两个变量保持独立;

引用数据类型即对象`Object`, 引用类型的值按照引用访问, 复制一个对象时, 仅仅是复制其引用地址, 复制前后的两个变量使用相同的内存, 其中一个对象属性的改变将影响另外一个对象; 通过`isinstanceof`操作符可以判断具体的对象类型;

### Undefined

该类型只有一个值: `undefined`; 已经声明但是未定义的变量; 检查一个变量类型是否为`undefined`可以判断该变量是否被声明或者已经定义;

### Null

Null类型也只有一个值: `null`;表示空对象的指针,使用`typeof`检查`null`的类型时,会返回`object`;

### Boolean

该类型有两个值:`true`和`false`;任何非空字符串、非零数字值、任何对象、n/a都会认为是`true`;空字符串、0和NaN、null、undefined将被认为是`false`;

### Number

该类型包含十进制整数、八进制整数、十六进制整数、浮点数、NaN等;

浮点数使用二进制浮点数(IEEE754),存在误差问题; 数值范围在`Number.MIN_VALUE`和`Number.MAX_VALUE`之间,超过下限的转换为`-Infinity`,超过上限的转换为`Infinity`;

NaN表示一个非数值, 任何NaN参与的运算结果都为NaN, `0/0 == NaN`, 将一个变量转换为数值但是无法转换时, 将得到NaN: `Number('NO') == NaN`;

使用`Number()`、`parseInt()`、`parseFloat()`这三个函数可以将非数值转换为数值;

其中`Number()`转换规则为:

1. 如果是数值则直接返回;
2. 如果是Boolean类型, 则true返回1，false返回0;
3. 如果是null, 返回0;
4. 如果是undefined, 返回NaN;
5. 如果是字符串, 如果是空字符串, 返回0; 如果只包含数字, 返回该数字, 并忽略前面的0; 如果包含十六进制格式, 返回对应十进制值;其他字符串返回NaN;
6. 如果是对象, 则调用`valueOf()`方法, 再依照前面的规则转换;

```javascript
function print(str){
    document.write('<p>' + str + '</p>')
}
function testNumber(obj){
    var n = Number(obj);
    print(n);
}
testNumber(10);				//10
testNumber(null);			//0
testNumber(undefined);		//NaN
testNumber("");				//0
testNumber("01243.3aa");	//NaN
testNumber("01243.3");		//1243.3
testNumber("0x13a");		//314
```

如果是处理整数或者浮点数, 应该使用更好的`parseInt()`和`parseFloat()`函数:

1. parseInt()转换字符串时, 从字符串开头开始转换, 直到遇到非数字字符; parseFloat()类似;
2. 当parseFloat()返回的是整数时, 则返回一个整数;
3. parseInt()可以指定第二个参数表示转换时的基数;
4. 对空字符串返回NaN;

```javascript
function testParseInt(obj,base){
    var n = parseInt(obj,base);
    print(n);
}
function testParseFloat(obj){
    var n = parseFloat(obj);
    print(n);
}
testParseInt(null);				//NaN
testParseInt(undefined);		//NaN
testParseInt("");				//NaN
testParseInt("0012");			//12
testParseInt("01243.3aa",8);	//675
testParseInt("0x13a",16);		//314
testParseFloat("0.2131");		//0.2131
testParseFloat("1231ab");		//1231
```

### String

String类型表示由0个或者多个16位unicode字符组成的字符串; 可以由`" "`或者`' '`表示;

字符串是不可变对象, 改变某个字符串必须先销毁再创建;

字符串使用`+`拼接, 访问其`length`属性返回长度;

通过任何类型(除了`null`和`undefined`)都有的`toString()`方法可以转换为字符串; 或者通过`String()`函数可以将一个变量转为字符串, 该方法会将`null`和`undefined`以字符串形式返回, 其他类型则调用`toString()`方法;

### Object

对象是一组数据和功能的集合, 通过`new Object()`来创建一个对象; 使用`.`来访问对象的属性和方法, 或者使用`[ ]`下标来访问会引发编译错误的属性名称;

`Object`类型作为所有对象的基本基础, 所有的对象都拥有该对象的属性方法, 包括:

* `constructor`: 该属性保存用于创建该对象的构造函数;
* `hasOwnProperty(propertyName)`: 检查指定的属性是否在当前对象中存在, 参数以字符串形式传入;
* `isPrototypeOf(object)`: 检查传入的对象是否是传入对象的原型;
* `propertyIsEnumerable(propertyName)`: 检查该属性是否能够使用`for-in`语句枚举;
* `toLocaleString()`: 返回对象的字符串表示, 与执行环境的地区对应;
* `toString()`: 返回对象的字符串表示;
* `valueOf()`: 返回对象的字符串、数值、布尔值表示, 通常和toString()的返回值相同;

## 操作符

1. `++`、`--`等操作符和C语言相同;
2. `+`操作符会将变量转换(使用`Number()`转换)为数值, `-`操作符在将变量转换为数值的基础上取负数;
3. `~`按位非、`|`按位或、`&`按位与、`^`按位异或、`<<`左移、`>>`右移、`>>>`无符号右移等和C语法类似;
4. `!`逻辑非、`&&`逻辑与、`||`逻辑非等和习惯用法相同;
5. 加性操作符`+`: 如果有字符串则按照字符串连接, 否则先转换为数值再相加;
6. 加性操作符`-`: 先转换为数值在做减法, 如果是对象则调用`valueOf()`方法, 如果没有则调用`toString()`先转为字符串;
7. 乘性操作符: `*`、`/`、`%`都先将操作数转为数值再进行计算;
8. 比较操作符: `>`、`>=`、`<`、`<=`: 如果有字符串, 则执行字符串比较, 否则转为数值比较;
9. 相等操作符: `==`、`===`, 前者不判断类型是否相同, 后者将比较类型; `!=`表示不相等;
10. 三目操作符: ` ? :`, 赋值操作符等都和常规使用类似;

数值计算时将使用以下规则:

1. 如果有NaN, 则结果为NaN;
2. 如果有Infinity, 则按照以下规则进行;

```javascript
print(Infinity + 1);				//Infinity
print(Infinity * 0);				//NaN
print(Infinity * -1);				//-Infinity
print(Infinity * 2);				//Infinity
print(Infinity * Infinity);		//Infinity
print(Infinity / 0);				//Infinity
print(-Infinity / 0);				//-Infinity
print(Infinity / Infinity);		//NaN
print(Infinity + Infinity);		//Infinity
print(Infinity + -Infinity);		//Nan
```

## 语句

`if`、`for`、`while`、`do-while`等和一般规则相同;

`for-in`语句用来对可迭代对象进行枚举,相当于`foreach`等相关语句, 其语法为: `for ( var property in iterable )`;

`label`语句: 用于循环控制中`break`和`continue`跳转的位置, 和`Java`中的特性相同, 只能用于循环跳转, 是弱化的`TODO`;

`with`语句: 将某段代码的作用域置于一个特定对象中, 如:

```javascript
obj = new Object()
obj.name = 'xiaoming';
obj.age = 16;
with(obj){
    print(name);		// xiaoming
    print(age);		// 16
}
```

`switch`语句和C语言中类似, 每个statment后面需要有`break`退出, 否则将合并多个条件执行;

## 函数

使用`function`关键字声明函数;函数名不可以是`eval`和`arguments`;

函数的参数通过`arguments`对象来传递, 同时也可以指定参数, 但是不能出现同名的命名参数;

函数在传递参数时, 是按值传递的, 基本类型和引用类型的传递和对应的复制操作一样, 即基本数据类型传入独立的变量, 引用数据类型传入地址; 和Python中的表现基本相同;

## 作用域

JavaScript的作用域表现和Python类似, 将按照作用域链逐级搜索标识符;

通过`with`语句可以为代码块添加作用域; 通过`try-catch`语句可以延长作用域到catch语句块;

没有块级作用域, 即在`if`、`for`等语句中定义的变量将会添加到最近的环境中, 在一个函数中, 最接近的环境就是函数的局部环境, 在with语句中, 最接近的环境是函数环境, 在顶级作用域中, 最接近的环境是全局环境. 没有使用`var`申明的变量将进入全局环境. 如:

```javascript
if(true){
    var a = 10;
}
print(a);       // 10
for(var i=0;i<10;i++){

}
print(i);       // 10
function foo(){
    var f = 10;
}
//print(f);  无法访问
var obj = new Object();
function foo(){
    with(obj){
        var f = 10;
    }
    print(f);  // 10
}
foo();
```

## 引用类型

### Object

大多数引用类型都是Object类型的实例, 通过两种方式可以创建一个Object对象:

```javascript
var obj = new Object();
var obj = {};
```

同时也可以使用`.`或者`[ ]`来访问某个属性, 一般使用`.`访问属性, 当属性名称包含空格等可能引起编译错误的问题时, 使用`[ ]`下标访问该属性;

### Array

JavaScript中使用Array类型来表示一个数组, 数组元素可以包含不同类型的值, 并且没有数组大小的限制, 数组大小会动态调整, 数组的表现和Perl中的数组类似, 但是提供更多的方法;

通过两种方式可以创建一个数组:

```javascript
var array = new Array(1,2,3);
var array = [1, 2, 3]
```

访问未定义的元素将返回`undefined`, 对超过数组长度元素进行赋值将自动扩展数组长度;

使用`Array.isArray()`方法可以检查一个值是不是数组, 并且不管该值在哪个全局环境下创建;

数组的`toString()`方法和`toLocaleString()`方法会对每个元素调用相应的方法, 并且用`,`连接成字符串; 使用`join()`方法可以指定连接符;

数组中的`pop()`、`push()`、`shift()`、`unshift()`四个方法和Perl语言中的表现一致, 但是从函数变为数组的方法, `push()`、`unshift()`允许一次传入多个值;

数组中的`sort()`方法默认使用字符串排序, 通过传入比较函数改变排序规则;

支持切片`slice()`以及`splice()`方法;

`splice()`方法可以接收2个或者以上参数, 其中第一个参数表示起始位置, 第二个参数表示要删除的项数, 第三个参数开始表示插入的元素;

```javascript
var array = ['red','blue','green',13,48,true];
print(array);  // red,blue,green,13,48,true
print(array[0]);  // red
print(array.length);  // 6
if( Array.isArray(array) ){
    print('array is Array');  // array is Array
}
print(array.join("##"));  // red##blue##green##13##48##true
array.push('tail');
print(array);    // red,blue,green,13,48,true,tail
print(array.pop());  // tail
array.unshift('head');
print(array);  // head,red,blue,green,13,48,true
print(array.shift());  // head
print(array.sort());  // 13,48,blue,green,red,true
print(array.reverse()); // true,red,green,blue,48,13
function cmp(a, b){
    if(a < b){
        return -1;
    }else if(a > b){
        return 1;
    }else{
        return 0;
    }
}
print(array.sort(cmp));  // true,blue,green,red,13,48
array = array.concat('value',['a1','a2'],'value2');
print(array);  // true,blue,green,red,13,48,value,a1,a2,value2
print(array.slice(5)); // 48,value,a1,a2,value2
print(array.slice(1,3));  // blue,green
print(array.splice(0,1,'false')) // true
print(array) // false,blue,green,red,13,48,value,a1,a2,value2
print(array.indexOf('blue'))  // 1
```

数组支持以下迭代方法, 其中给定的函数需要接收三个参数: 数组项的值、该项在数组中的位置、数组本身;

* `every()`: 对数组的每一项运行给定函数, 当所有项都返回true时, 返回true;
* `some()`: 对数组的每一项运行给定函数, 当其中有一个返回true时, 即返回true;
* `filter()`: 对数组的每一项运行给定函数, 并返回能够返回true的项组成的数组;
* `forEach()`: 对数组的每一项运行给定函数, 不返回内容;
* `map()`: 对数组的每一项运行给定函数, 并返回函数返回结果组成的数组;

```javascript
function test_iter(item, index, array){
    return( item > 2)
}
var array = [1,5,3,4,6,2,1,3,4,5];
print( array.every(test_iter) )  // false
print( array.some(test_iter) )  // true
print( array.filter(test_iter) )  // 5,3,4,6,3,4,5
array.forEach(function(item,index,array){
    print( index + ',' + item )   //
});
print( array.map(function(item,index,array){
    return item * 2;
}) );   // 2,10,6,8,12,4,2,6,8,10
```

以及`reduce()`方法和一个从右侧开始的版本`reduceRight()`, 该方法传入的函数需要指定4个参数: 前一项, 当前项, 索引值, 数组本身, 该方法对数组做一个归并操作, 返回值将赋值给当前变量, 然后在下一个循环中作为前一项参与运算:

```javascript
print( array.reduce(function(prev,cur,index,array){
    print(prev); // 查看内部运算细节
    return prev + cur;  // 34
}) );
```

相比python的`map()`和`reduce()`, JavaScript中的这两个方法暴露更加丰富的细节, 也更加容易理解;

### Date

Date类型主要用于时间和日期的处理, 和其他语言不同的是, 不同的浏览器对日期的处理会很不一样, 但是基本方式大同小异;

Date类可以通过两种方式实例化:

* 通过传入Unix时间戳(从1970前开始的毫秒数)来实例化;
* Date.parse()方法可以从一个特定格式字符串来解析出时间戳;
* Date.UTC()通过依次传入:年份、基于0的月份、基于1的天数、小时数、分钟、秒、毫秒等值来获取时间戳, 注意该时间是GMT时间, 前两个参数为必须;
* Date.now()获取调用该方法时的时间戳;
* 通过依次传入和UTC()方法的参数, 可以直接通过Date()实例化, 此时, 使用的时间是当地时间;

```javascript
var date = new Date(Date.parse('Nov 9 2015'));
print(date);  // Mon Nov 09 2015 00:00:00 GMT+0800 (CST)
print(date.toLocaleString()); // 2015年11月9日 GMT+8 00:00:00
print(date.valueOf()); // 1446998400000
var date = new Date(Date.UTC(2015, 10, 9));
print(date); // Mon Nov 09 2015 08:00:00 GMT+0800 (CST)
var date = new Date(Date.now())
print(date); // Thu Nov 12 2015 22:39:07 GMT+0800 (CST)
var date = new Date(2015, 10, 9);
print(date); // Mon Nov 09 2015 00:00:00 GMT+0800 (CST)
```

Date()对象的两种字符串方法: `toString()`, `toLocaleString()`在不同浏览器中返回的格式并不相同, 默认使用的`toString()`方法返回GMT格式, `toLocaleString()`则会显示浏览器设置的地区相适应的格式, `valueOf()`方法返回时间戳, 该值可以用来比较两个时间的大小;

日期类型有许多格式化方法以及获取时间组件的方法, 以下举一些例子:

```javascript
var date = new Date(Date.now());
print(date.toDateString()); // Thu Nov 12 2015
print(date.toTimeString()); // 22:52:04 GMT+0800 (CST)
print(date.getFullYear()); // 2015
print(date.getMonth()); // 10
print(date.getDay()); // 4
print(date.getHours()); // 22
print(date.getMinutes()); // 52
// More
```

### RegExp

JavaScript中的正则和Perl非常相似, 可以使用和Perl一样的格式生成正则表达式, 也可以用RegExp()来生成, 但是既然支持Perl里生动的写法, 甚至沿用Perl里的若干魔术变量, 对比Perl只是缺少一些较少使用的功能, 感觉非常强大！

```javascript
var string = 'Thu 2015-11-13 00:18:14';
var reg = /thu+\s+\b(\d{4})-(\d{1,2})-(\d{1,2})\b/i;
print(reg.exec(string));  // Thu 2015-11-13,2015,11,13
print(reg.exec(string)[1])  // 2015
print(RegExp.$1);  // 2015
if(reg.test(string)){
	print("this string is matched!")
}
```

用Perl的方式写正则就是舒服啊~

### Function

函数也是一种对象, 所以除了使用`function`关键字后面跟上函数名称来定义函数之外, 也可以使用`var foo = function(){}`的方式, 或者使用`new Function()`的方式, 其中最后一个参数为代码块; 不推荐使用第三种方式, 其他两种方式示例为:

```javascript
function sum(num1, num2){
	return num1 + num2;
}
var sum = function(num1, num2){
	return num1 + num2;
};
```

由于和其他动态语言一样, 函数名只是一个函数对象的指针, 所以并没有重载的概念, 同名函数将直接覆盖已有函数, 也和其他动态语言一样使用松耦合的方式传递参数和返回值; 解释器会先进行函数的申明, 但是具体的表达式则在执行时才会最终确定; 函数同样可以作为一个参数传入到其他函数中, 这些都和其他的动态语言表现一致;

JavaScript中的函数比较特殊的对象:

* `argument`: 该对象保存函数的所有参数, 具有`callee`方法, 用来调用拥有该对象的函数本身;
* `this`: 该对象保存调用该函数的对象, 如果没有指定, 则使用全局对象(网页全局域中为`window`);
* `caller`: 该对象保存调用当前函数的函数的引用, 如果在全局域中, 则为`null`;

```javascript
window.color = 'red';
var obj = {'color': 'blue'};
function sayColor(){
	print(this.color);
}
sayColor();  // red
obj.sayColor = sayColor;
obj.sayColor();  // blue

function outer(){
	inner()
}
function inner(){
	print(inner.caller)
}
outer()
```

注意, `inner.caller`代表一个函数对象, 如果调用该函数会产生死循环;

函数的其他属性和方法:

* `length`属性: 表示函数希望接收的**命名参数**的个数;
* `protetype`属性: 表示保存该函数的原型;
* `apply()`方法: 用来调用该函数, 需要传入两个参数: 运行函数的作用域、函数参数数组;
* `call()`方法: 用来调用该函数, 需要一个以上参数: 运行函数的作用域、一系列直接传入函数的参数;

使用`apply()`和`call()`可以灵活调整函数的作用域:

```javascript
window.color = 'red';
var obj = {'color': 'blue'};
function sayColor(){
	print(this.color);
}
sayColor.apply(this,[]);
sayColor.call(window);
sayColor.call(obj);
```

### Number

和Java中一样, 除了基本类型之外, 还提供了包装类型, 由于使用了堆内存存储, 相比基本类型效率较低, 相比基本类型提供更长的生命周期;

区分转型函数`Number()`以及创建引用类型`new Number()`; 不是必要情况下, 不应该使用`new Number()`的方式;

Number类型主要提供以下属性和方法:

* `toFixed()`方法, 接受一个参数用于控制显示的小数位数, 将使用四舍五入的方式取近似值;
* `toExponential()`方法, 接受一个参数用于显示科学计数法显示时的小数位数;
* `toPrecision()`方法, 接受一个参数表示有效数字, 根据有效数字位数选择显示方式;

```javascript
var num = 99.333333;
print(num.toFixed(2));  // 99.33
print(num.toExponential(2));  // 9.93e+1
print(num.toPrecision(2));  // 99
print(num.toPrecision(1));  // 1e+2
```

### String

String包装类型的基本属性和方法有:

* `charAt()`、`charCodeAt()`、`[ ]`通过这三种方式可以访问字符串的某个字符; 其中第一个和第三个方法直接返回指定位置的字符, 第二个方法返回字符的编码;
* `concat()`: 接受多个参数, 将参数拼接到字符串后面, 大多数情况下使用`+`操作符更加方便;
* `slice()`、`substr()`、`substring()`通过这三个方法创建子字符串, `slice()`切片方法和Python中表现相同; `substr()`接受一个参数时, 表示从该位置开始到字符串末尾的子字符串, 可选第二个参数表示子字符串的长度; `substring()`方法和`slice()`方法区别在于会将负数转为`0`, 并自动使用两个参数中的较小值作为开始;
* `indexOf()`、`lastIndexOf()`方法用于查找子字符串在字符串中的位置; 可以接受第二个参数表示开始搜索的位置;
* `trim()`: 删除字符串头尾的空白;
* `toLowerCase()`、`toUpperCase()`等大小写转换方法;
* `match()`、`search()`、`replace()`、`split()`等字符串常用方法使用时, 都支持正则表达式;
* `localeComopare()`: 字符串比较方法, 即Perl中的`cmp`;
* `fromCharCode()`: 传入一系列编码, 返回组成的字符串;

下面是一些示例:

```javascript
var str = 'hello world, it is 2015-11-18 20:29:04 +0800';
print(str.charAt(1));  // e
print(str.charCodeAt(1));  // 101
print(str[1]);  // e
print(str.concat('!'));  // hello world, it is 2015-11-18 20:29:04 +0800!
print(str.slice(1,3));  // el
print(str.slice(1,-3));  // ello world, it is 2015-11-18 20:29:04 +0
print(str.substr(1,3));  // ell
print(str.substr(-3));  // 800
print(str.substr(1,-3));  //
print(str.substring(1,3));  // el
print(str.substring(1,-3));  // h
print(str);  // hello world, it is 2015-11-18 20:29:04 +0800
print(str.indexOf('i', 5));  // 13
print(str.lastIndexOf('i'));  // 16
print(str.trim());  // hello world, it is 2015-11-18 20:29:04 +0800
print(str.toLowerCase());  // hello world, it is 2015-11-18 20:29:04 +0800
print(str.toUpperCase());  // HELLO WORLD, IT IS 2015-11-18 20:29:04 +0800

var pattern = /it is (\d{4}-\d{1,2}-\d{1,2})/;
var matches = str.match(pattern);
print(matches[0]);  //  it is 2015-11-18
print(matches[1]);  //  2015-11-18
print(matches.index);  // 13
print(str.search(pattern));  // 13
print(str.replace(pattern, "$1"));  // hello world, 2015-11-18 20:29:04 +0800
print(str.split(/\s+/));  // hello,world,,it,is,2015-11-18,20:29:04,+0800
function htmlEscape(text){
	return text.replace(/[<>"&]/g, function(match, pos, originalText){
		switch (match) {
			case '<':
				return "&lt;";
			case '>':
				return "&gt;";
			case '&':
				return "&amp;";
			case '"':
				return "&quot;";
		}
	});
}
print(htmlEscape('<p class="paragraph"> Hello world! </p>'))  // <p class="paragraph"> Hello world! </p>
```

最后使用之前定义的`print`函数会再次将内容做转换;

### Global

不属于任何其他对象的属性和方法都是全局对象的属性和方法, 许多不需要其他对象便可以使用的函数都是全局对象的方法, 在全局作用域中定义的函数和变量也是全局对象的方法和属性; 在浏览器中, 一般将全局对象作为`window`对象的一部分加以实现, 即全局对象的属性和方法都是`window`对象的属性和方法

全局变量主要有以下常用属性和方法:

* `encodeURL()`、`encodeURLComponent()`方法用于URL的编码;
* `eval()`方法用于执行某段字符串表示的代码, 慎用该方法, 避免**代码注入攻击**;

### Math

Math对象提供一些简单的数学功能, 包括一些常用的常量以及常用计算方法, 如Math.E、Math.PI、min()、ceil()、floor()、round()、random()、sin()、sqrt()等等;
