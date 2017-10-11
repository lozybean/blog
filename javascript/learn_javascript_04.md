Title: JavaScript浏览器
Date: 2015-11-24
Category: Learning
Tags: javascript

在Web应用中, 浏览器作为客户端, 浏览器功能的访问和控制、浏览器类型的检测等等, 都是非常重要的知识;

* **BOM**(browser object model)浏览器对象模型;
* 客户端检测;

## window对象

`window`对象是**BOM**的核心, 表示浏览器的一个实例, 再浏览器中, window对象具有双重身份:

* 浏览器对象接口;
* 全局对象;

即, 所有在全局作用域中声明的变量都会成为window对象的属性或者方法, 但是存在以下区别:

1. 全局变量不可使用`delete`删除, 而在window对象上定义的属性可以; (IE9以下将都直接报错)
2. 不可访问未定义变量, 但是可以访问window上的可能不存在的属性;

```javascript
function print(str){
	document.write('<p>' + str + '</p>')
}
var name = 'env';
function foo(){
	return 'foo';
}
print(window.name);  // env
print(window.foo);  // foo
window.color = 'red';
delete name;  // false
delete window.color; // true
var newValue = oldValue;  // 错误! oldValue未定义;
var newValue = window.oldValue;
print(newValue)  // undefined
```

*注: 为了方便调试, 仍然和之前一样使用了自定义的print函数, 这样会覆盖window.print调用打印机方法*

运行该特性, 可以用来检查一个可能未定义的变量是否存在;

### 框架

当一个页面中包含框架时, 每个框架都会有一个window对象:

```html
<!DOCTYPE html>
<html lang="cn">
<head>
    <title>Learn JavaScript</title>
    <meta charset="utf-8">
</head>
<frameset rows="250,*">
	<frame src="top_frame.html" name="topFrame"></frame>
	<frameset cols="50%,50%">
		<frame src="left_frame.html" name="leftFrame"></frame>
		<frame src="right_frame.html" name="rightFrame"></frame>
	</frameset>
</frameset>
</html>
```

框架网页显示以及每个框架的访问方式:

![frame](http://ww1.sinaimg.cn/large/95202659gw1eybcdk9b5kj20qk0gr75e.jpg)

其中, 由于`top`始终指向最高层框架, 使用`top`来访问是最好的方式;

### 窗口

window对象中提供的窗口属性有很多, 但是不同的浏览器可能具有不同的使用方式, 操作窗口之前确定浏览器的类型比较重要;

通过`window.open()`可以弹出一个新的窗口, 初始化时可传入窗口属性, 通过`close()`方法可以关闭窗口;

窗口有如下属性方法:

* `screenLeft`、`screenTop`: 获取窗口边缘位置; 或者为`screenX`、`screenY`;
* `moveTo`: 移动到某个位置, 接受两个参数为新位置的坐标值;
* `moveBy`: 移动, 接受两个参数为水平向右和垂直向上

一般而言, 浏览器会默认阻止窗口弹出, 而且体验也不好, 尽量别用这个;

### 延迟调用和间歇调用

* `setTimeout()`方法可以设置延迟调用, 接受两个参数, 第一个为调用的函数, 第二个为延迟毫秒数, 该函数执行后返回一个`id`;
* `clearTimeout()`接受一个延迟调用的`id`并且取消延迟调用;
* `setInterval()`方法可以设置间歇调用, 和延迟调用类似, 但是循环执行;
* `clearInterval()`和取消延迟调用类似;

应该避免使用间歇调用, 而使用延迟调用来模拟间歇调用;

### 系统对话框

* `alert()`方法弹出一个文档;
* `confirm()`方法弹出一个确认框, 并且获取确认值;
* `prompt()`方法弹出一个对话框, 返回用户输入值;

不管是弹出窗口还是对话框, 感觉都不是很舒服, 希望以后不要用到这些;

## location对象

`location`对象提供当前窗口中加载的文档有关的信息, 该对象既是`window`的属性, 也是`document`的属性;

其属性有:

* `hash`, 返回URL中的hash(#后跟零或多个字符串);
* `host`, 返回服务器名称和端口号;
* `port`, 返回URL中的端口号;
* `hostname`, 返回不带端口号的服务器名称;
* `href`, 返回当前加载页面的完整URL;
* `pathname`, 返回URL中的目录或者文件名;
* `protocol`, 返回页面使用协议;
* `search`, 返回URL查询字符串(?后跟一个赋值, 多个以&分割)

根据以上属性, 可以获取到当前页面URL的具体情况, 修改这些属性可以将页面引导至其他位置, 每次修改其中的属性, 页面都会重新加载(hash除外);

通过修改`location`的`hash`可以完成导航, 也可以修改`pathname`来引导至服务器其他位置等等;

常用操作为:

* `assign()`, 改变浏览器位置并且生成历史记录, 与修改`href`属性效果相同;
* `replace()`, 改变浏览器位置并且不生成历史记录;
* `reload()`, 重新加载页面, 若传入一个参数`true`则会强制从服务器加载, 否则有可能使用缓存; 由于`reload()`之后的代码不一定执行, 所以最好将该方法放到代码最后一行;

## 其他对象

* `navigator`对象主要用于检测浏览器类型, 不同的浏览器对该对象中属性的支持并不相同;
* `screen`对象用来表示客户端能力, 包括浏览器窗口外部的显示器信息, 如像素宽度、高度、DPI等等; 不同浏览器支持不同;
* `history`对象用于保存上网历史记录, 以及历史记录之间的跳转;

## 客户端检测

由于不同浏览器之间, 相同浏览器的不同版本之间对某些函数方法的支持都可能不同, 能力检测的目标不局限于识别浏览器的类型, 而是直接识别浏览器对某种方法是否支持:

```javascript
// 作者: Peter Michaux
function isHostMethod(object, property){
	var t = typeof object[property];
	return t == 'function' ||
			(!!(t=='object' && object[property])) ||
			t == 'unnknown';
}
```

通过检测一个对象中是否存在某个方法, 如果不存在则使用其他替代方法来完成代码在不同浏览器中的适配;

如检测浏览器是否支持`document.getElementById()`方法, 如果不支持, 则可能为IE早期版本, 使用`document.all`方法代替, 如:

```javascript
function getElement(id){
	if (isHostMethod(document, getElementById)){
		return doument.getElementById(id);
	}else if (isHostMethod(document, all)){
		return document.all[id];
	}else{
		throw new Error("No way to retrieve element!")
	}
}
```

上述代码完成一个更加可靠的`getElement`方法用以适配早期IE浏览器;

注意浏览器能力检测并不是浏览器检测, 一个浏览器中的某个能力特色也并不足以证明就是该浏览器, 不应该只检测某一个比较典型的能力就认为是某种浏览器从而决定其他能力, 在逻辑上再清晰不过的错误, 很多网站上都存在(来自js高程书中原话);

除了浏览器能力的检测, 类似方法可以用来检测浏览器的缺陷(bug), 实现方式就是用一个判断函数来检查是否具有缺陷, 并尽可能解决; bug检测往往涉及运行代码, 所以应该仅仅检测关键bug, 并且在脚本开头就执行检测工作;

除了直接检查浏览器能力和bug, 还可以通过`navigator`对象中的各种浏览器类型属性, 检测出浏览器的具体类型和版本等信息;

由于浏览器代理字串可以被轻易修改(爬虫中往往就会修改代理来仿照浏览器), 所以应该优先选择能力检测, 仅仅在能力检测等不可用或者需要使用浏览器代理检测来决定表现能力等必须使用代理检测的情况下, 才选择代理检测;
