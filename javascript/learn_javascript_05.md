Title: JavaScript之DOM
Date: 2015-11-27
Category: Learning
Tags: javascript

**DOM**(文档对象模型)是针对XML和HTML文档的一个API, 分为3级, 其中:

* DOM1: 页面结构及查询标准;
* DOM2: 扩展DOM1, 在已有类型基础上添加方法和属性, 引入更多的交互能力;
* DOM3: 扩展DOM1, 增强既有类型的基础上, 添加了新的类型用以扩展API;

由于*IE浏览器*使用COM对象的形式实现DOM, 与其他浏览器中的行为特点并不一致;

## DOM 节点层次

DOM将一个XML或者HTML描述成由多层节点构成的结构, 所有页面都会标记为一个有特定根节点(document)的树形结构;

示例文档:

```html
<html>
	<head>
		<title>
			Title Node
		</title>
	</head>
	<body>
		<p class='paragraph'> Paragraph Node </p>
		<ul>
			<li>item 1</li>
			<li>item 2</li>
			<li>item 3</li>
		</ul>
		<div class="first" id="demo">
			<p> first div </p>
		</div>
		<div class="secend">
			<p> second div </p>
		</div>
		<script src="learn_dom.js"></script>
	</body>
</html>
```

### Node

其中所有的节点类型都继承自相同的`Node`接口, 该接口主要有三个属性:

* `nodeType`: 节点类型;
* `nodeName`: 节点名称;
* `nodeValue`: 节点值;

该接口定义了一系列的节点类型, 保存在每个节点的`nodeType`属性中, 所有节点类型都必须是其中之一:

* Node.ELEMENT_NODE(1): 元素类型, 也是最常用的类型;
* Node.ATTRIBUTE_NODE(2);
* Node.TEXT_NODE(3);
* Node.CDATA_SECTION_NODE(4);
* Node.ENTITY_REFERENCE_NODE(5);
* Node.ENTITY_NODE(6);
* Node.PROCESSION_INSTRUCTION_NODE(7);
* Node.COMMENT_NODE(8);
* Node.DOCUMENT_NODE(9);
* Node.DOCUMENT_TYPE_NODE(10);
* Node.DOCUMENT_FRAGMENT_NODE(11);
* Node.NOTATION_NODE(12);

所有上述元素类型可以有字符串常量和整数两种形式表示, 但是在*IE浏览器*中, 只能使用整数值, 所以使用整数值是一种比较好的方式;

文档中的所有节点存在文档树的关系:

* 文档树的根节点必然是`document`;
* 每个节点都有`childNodes`属性, 保存一个`NodeList`对象, 但并不是所有类型的节点都有子节点;
* `NodeList`对象虽然有`length`属性, 也可使用下标访问其中元素, 但是并不是`Array`的实例, 只能将某个时刻的`NodeList`对象快照转化为`Array`实例;
* 每个节点都有`parentNode`属性, 表示父节点, `document`节点的父节点为`null`;
* 每个节点都有`nextSibling`和`previousSibling`, 表示其后一个同辈节点以及前一个同辈节点;
* 所有节点都有`ownerDocument`属性, 表示该节点指向的文档节点, `document`节点该属性为`null`;

获取当前`NodeList`快照的方式:

```javascript
function convertToArray(nodes){
	var array = null;
	try {
		array = Array.prototype.slice.call(nodes, 0);
	}catch(ex){
		array = new Array();
		for (var i=0, len=nodes.length; i<len; i++){
			array.push(nodes[i]);
		}
	}
	return array;
}
```

```javascript
// learn_dom.js
var html = document.childNodes[0];
alert(html);  // [object HTMLHtmlElement]
alert(html.nodeType);  // 1
alert(html.nodeName);  // HTML
alert(html.nodeValue);  // null
alert(html.parent);  // [object HTMLDocument]
alert(html.childNodes);  // [object NodeList]
html_children = convertToArray(html.childNodes);
print(html_children);  // [object HTMLHeadElement],[object Text],[object HTMLBodyElement]
head = html_children[0];
alert(head.nextSibling);  // [object Text]
alert(head.previousSibling);  // null
print(head.ownerDocument);  // [object HTMLDocument]
```

节点的关系指针是只读的, 所以若要操作节点, 并不能直接修改节点关系, 只能通过内置的几个操作方法:

* `appendChild()`: 在childNodes的最后添加一个节点, 该方法执行后返回新增的节点;
* `insertBefore()`: 在某个节点之前插入节点, 成为该节点的前一个同辈节点, 同样会返回该节点;
* `replaceChild()`: 替换某个节点, 被替换掉的节点仍然存在文档中, 只是没有了在文档中的位置;
* `removeChild()`: 移除某个节点, 移除的节点只是没有了在文档中的位置;
* `cloneNode()`: 复制一个节点, 如果传入一个参数为`true`, 则会同时复制节点以及整个子节点树; 返回的节点没有父节点, 必须通过其他方法添加到节点树中;
* `normalize()`: 删除空的文本节点, 合并相邻的文本节点;

上述前四个方法都是针对子节点进行操作, 但有些类型的节点并没有子节点, 此时将会发生错误;

```javascript
var body = document.childNodes[0].childNodes[2];
var ul = body.childNodes[3];
var ul_2 = ul.cloneNode(true);
var new_ul = body.appendChild(ul_2);
alert(new_ul == ul_2);  // true
alert(body.lastChild == new_ul);  // true
```

### Document

在浏览器中, `document`对象是`window`对象的一个属性, 是`HTMLDocument`(继承自`Document`)的一个实例, 是浏览器文档中其他节点的根节点, 各项属性为:

* `nodeType`: DOCUMENT_NODE(9);
* `nodeName`: #document;
* `nodeValue`: null;
* `parentNode`: null;
* `ownerDocument`: null;
* 子节点可能为: 最多一个DocumentType, 最多一个Element, ProcessingInstruction或Comment;

在上面的例子中, `document`的唯一Element子节点就是一个`html`元素, 通过`document.childNodes[0]`访问, 同时也可以通过`document.documentElement`属性访问得到; 后面一种访问方式可以保证获得Element子节点, 而当存在`<!DOCUTYPE>`标签时, 则DocumentType节点可能为第一个子节点;

DocumentType子节点可以通过`document.doctype`属性可靠访问, 除了`IE8`以及之前版本会将其认为是Comment; 该属性在不同的浏览器中表现各异;

除了上述属性外, `document`具有表现页面信息的属性:

* `title`: 文档标题;
* `URL`: 页面完整URL;
* `domain`: 页面域名;
* `referrer`: 页面来源URL;

`domain`属性可以修改为当前域名的更加*松散*的方式, 从而完成两个子域之间的跨域通信;

```javascript
// document.domain == 'p2p.wrox.com'
document.domain = 'wrox.com';  // 设置后, 该域名下的不同子域, 如: p2p.wrox.com, c2c.wrox.com可以互相通信;
document.domain = 'p2p.wrox.com';  // 错误, 只能将域名修改为更加松散的方式;
```

document更常见的应用在于查找元素, 通过以下两个方法:

* `getElementById()`: 通过元素id查找, 得到相应元素, 如果没有该id, 则返回null;
* `getElementByTagName()`: 通过标签名查找, 得到一个HTMLCollection;
* `HTMLCollection`对象中的元素可以通过下标索引来获取元素, 如果元素有`name`属性, 则可以通过该值访问;

这些方法不同浏览器的表现不同, IE7以及更低版本使用`getElementById()`查找时会忽略大小写, 并且`name`属性也可以被`getElementById()`获取, 等等; 在html中`getElementByTagName()`将忽略大小写, 但是xml中大小写敏感;

`getElementByTagName("*")`可以获得所有元素;

```javascript
var first_div = document.getElementById('demo');
alert(first_div);  // [object HTMLDivElement]
var divs = document.getElementsByTagName('div');
alert(divs);  //  [object HTMLCollection]
alert(divs[0] == divs['demo']);  // true
```

除了使用上述两个方法之外, document对象还有一些特殊的集合(`HTMLCollection`对象):

* `document.anchors`: 文档中所有带name特性的&lt;a&gt;元素;
* `document.links`: 文档中所有带href特性的&lt;a&gt;元素;
* `document.forms`: 文档中所有的&lt;form&gt;元素;
* `document.images`: 文档中所有的&lt;img&gt;元素;

document对象还提供了以下写入文档的方法:

* `write()`: 原样写入;
* `writeln()`: 在末尾添加`\n`换行符;
* `open()`, `close()`: 打开和关闭网页的输出流;

之前的几篇实例中改写的`print()`方法就用到了文档的写入;

### Element

`Element`类型就是网页中的元素, 上述通过`getElementById()`等得到的就是`Element`对象; 该对象的节点属性为:

* `nodeType`: ELEMENT_NODE(1);
* `nodeName`: 元素的标签名;
* `nodeValue`: null;
* `parentNode`: 可能为Document或者Element;
* 子节点可能为: Element, Text, Comment, ProcessingInstruction, CDATASection 或 EntityReference;

除了使用`nodeName`访问元素的标签名, `tagName`属性也会返回相同值, 元素标签名查找时忽略大小写, 但是实际返回时在HTML中始终为大写;

```javascript
var first_div = document.getElementById('demo');
alert(first_div.tagName);  // DIV
alert(first_div.nodeName == first_div.tagName);  // true
alert(first_div.tagName == 'div');  // false
alert(first_div.tagName.toLowerCase() == 'div');  // true
```

所有的HTML元素都由HTMLElement类型(继承自Element)及其子类型表示, 该类型在Element基础上添加了以下属性:

* `id`: 元素在文档中的唯一标识符;
* `className`: 元素的class特性;
* `title`: 元素的附加说明信息, title特性;
* `lang`: 语言代码;
* `dir`: 语言方向, "ltr"(left-to-right)或者"rtl"(right-to-left);

这些值都可以被修改(虽然并不一定在客户端体现), 修改后会立即应用对应的CSS样式;

除了上述属性描述了一部分HTML元素的特性之外, 还可以通过以下方法来访问或者修改这些特性:

* `getAttribute()`: 获取特性;
* `setAttribute()`: 修改特性;
* `removeAttribute()`: 删除特性;

```javascript
var first_div = document.getElementById('demo');
alert(first_div.id);  // demo
alert(first_div.className == first_div.getAttribute('class'));  // true
first_div.setAttribute('class','another');
alert(first_div.className)  // another
```

同时, Element元素是唯一使用`attributes`属性的节点类型, 该属性包含一个NamedNodeMap, 元素的每个特性都由一个Attr节点表示, 并且这些节点都保存再NamedNodeMap中, 通过以下方法获取或者操作属性:

* `getNamedItem(name)`: 返回html元素的对应特性(Attr节点)的值, 也可通过下标直接访问;
* `removeNamedItem(name)`: 移除该Attr节点;
* `setNamedItem(node)`: 添加新Attr节点;
* `item(pos)`: 返回在pos位置出的Attr节点;

上述方式获取操作属性并不如之前的方法方便, 但是使用这些方法可以对属性进行遍历:

```javascript
function outputAttributes(element){
	var pairs = new Array();
	for(var i=0,len=element.attributes.length; i<len; i++){
		var attrName = element.attributes[i].nodeName;
		var attrValue = element.attributes[i].nodeValue;
		if(element.attributes[i].specified){  // for IE7 or lower;
			pairs.push(attrName + '="' + attrValue + '"');
		}
	}
	return pairs.join(' ');
}

```

通过`document.createElement()`方法可以创建一个元素节点, 该方法接收一个参数指定元素标签名, 同时为该元素设置ownerDocument属性, 并通过Element元素的相应方法设置属性, 但是此时该元素在文档中并没有其位置, 需要通过`node.appendChild`等方法将其添加到文档树中;

Element类型也具有`getElementById()`以及`getElementByTagName()`方法, 此时, 这两个方法只返回子节点下的相应元素;

### Text

Text类型表示文本节点, 该节点不包含HTML代码, 只包含转义后的HTMl字符, 该类型节点属性为:

* `nodeType`: TEXT_NODE(3);
* `nodeName`: #text;
* `nodeValue`: 具体的文本内容;
* `parentNode`: Element;
* 不支持子节点;

在html中, 除了IE浏览器之外, 两个标签之间都会有一个空的文本节点;

通过`nodeValue`属性可以修改文本内容, 也可以通过以下方法修改:

* `appendData(text)`: 在末尾添加text;
* `deleteData(offset, count)`: 从offset位置开始删除count个字符;
* `insertData(offset, text)`: 在offset指定位置插入text;
* `replaceData(offset, count, text)`: 用text替换从offset指定位置开始到offset+count出的文本;
* `splitText(offset)`: 从offset位置将文本分为两个文本节点, 前一个作为原节点的值, 后一个为新节点的值, 返回新节点;
* `subsringData(offset, count)`: 提取offset开始count个字符的文本;
* `createTextNode(text)`: 创建文本节点, 传入参数为文本值;

在DOM文档中, 相邻的文档节点容易导致混乱, 为了合并相邻文档节点, 可以使用前文介绍的`document.normalize()`方法; 而`splitText()`方法则会产生完全相反的结果;

```javascript
var first_div = document.getElementById('demo');
var p = first_div.childNodes[1];
var text = p.childNodes[0];
alert(text.nodeValue);  // first div
alert(text.data);  // first div
text.appendData(' append');
alert(text.data);  // first div append
text.insertData(6, ' insert');
alert(text.data);  // first insert div append
another_text = text.substringData(6,7);
alert(another_text);  // insert
var new_text = text.splitText(6);
alert(text.data);  // first
alert(new_text.data);  // insert div append
another_text = document.createTextNode(another_text);
p.appendChild(another_text);
alert(convertToArray(p.childNodes));  // [object Text],[object Text],[object Text]
p.normalize();
alert(convertToArray(p.childNodes));  // [object Text]
```

### 其他类型

其他并不常用的类型有:

* `Comment`类型: 表示注释节点;
* `CDATASection`类型: 只针对XML, 表示CDATA区域;
* `DocumentType`类型: 表示&lt;!DOCTYPE&gt;标签指定的节点;
* `DocumentFragment`类型: 唯一没有对应标记的类型, 是一种轻量级文档, 可以包含和控制节点;
* `Attr`类型: 元素的特性节点;

## DOM 操作

### 动态脚本

通过在DOM插入&lt;script&gt;元素, 可以在javascript中创建动态脚本, 和脚本在html中的两种存在方式一样, 动态脚本也有两种创建方式:

* 动态加载外部javascript脚本;
* 直接在页面中插入脚本代码;

动态插入脚本需要访问script元素以及修改其属性等, 在不同浏览器中表现不同, 下面展示js高程书上的两个实例函数:

```javascript
// 动态加载外部脚本
function loadScript(url){
	var script = document.createElement('script');
	script.type = 'text/javascript';
	script.src = url;
	document.body.appendChild(script);
}
// 动态插入脚本代码
function loadScriptString(code){
	var script = document.createElement('script');
	script.type = 'text/javascript';
	try{
		script.appendChild(document.createTextNode(code));
	}catch (ex){
		script.text = code;
	}
	document.body.appendChilde(script);
}
```

上述两种加载方式有些区别, 第二种方式可以立即加载完成, 相当于将字符串传递给`evel()`, 而第一种方式由于需要加载文件, 并没有标准方法可以探知合适加载完成;

### 动态样式

和动态脚本类似, 在javascript中创建css样式, 称为动态样式, 同样也有两种方式:

* 在link元素中添加外部css引用;
* 在style元素中添加css代码;

对应函数为:

```javascript
// 在link元素中动态加载外部css
function loadStyle(url){
	var link = document.createElement('link');
	link.rel = 'stylesheet';
	link.type = 'text/css';
	link.href = url;
	document.head.appendChild(link);
}
// 在style元素中动态插入css
function loadStyleString(css){
	var style = document.createElement('style');
	style.type = 'text/css';
	try{
		style.appendChild(document.createTextNode(css));
	}catch (ex){
		style.styleSheet.cssText = css;
	}
	document.head.appendChild(style);
}
```

需要指出: 在文档中`document.head`的Note中描述:

> `document.head` is read-only. Trying to assign a value to this property will fail silently or, when in *ECMAScript Strict Mode* in a Gecko browser, throw a *TypeError*.

然而在实际测试中, firefox 和 safari 均可以往`document.head`中添加子节点, 并且正常显示, 可能添加子节点并没有对该元素做出修改, 但是没有深入探究, 如果出问题, 则应该使用`document.getElementByTagName('head')[0]`来获取;

### 表格

由于&lt;table&gt;元素结构比较复杂, 在动态添加表格时, 往往需要编辑大量代码, 并且不够直观, 为此, HTML DOM为&lt;table&gt;、&lt;tbody&gt;、&lt;tr&gt;元素添加更多的属性和方法:

在&lt;table&gt;中:

* `rows`: 表格中所有行的HTMLColletion;
* `insertRow(pos)`: 在rows集合指定位置插入一行;
* `deleteRow(pos)`: 删除指定位置行;
* `caption`: 保存着&lt;caption&gt;元素的指针;
* `createCaption()`: 创建&lt;caption&gt;元素并返回其引用;
* `deleteCaption()`: 删除&lt;caption&gt;元素;
* `tFoot`: &lt;tfoot&gt;元素的指针;
* `createTFoot()`: 创建&lt;tfoot&gt;元素并返回其引用;
* `deleteTFoot()`: 删除&lt;tfoot&gt;元素;
* `tHead`: &lt;thead&gt;元素的指针;
* `createTHead()`: 创建&lt;thead&gt;元素并返回其引用;
* `deleteTHead()`: 删除&lt;thead&gt;元素;
* `tBodies`: &lt;tbody&gt;元素的HTMLCollection;

在&lt;tbody&gt;中:

* `rows`: &lt;tbody&gt;中所有行的HTMLCollection;
* `deleteRow(pos)`: 删除指定位置的行;
* `insertRow(pos)`: 向rows集合的指定位置插入行并返回其引用;

在&lt;tr&gt;中:

* `cells`: &lt;tr&gt;中所有单元格的HTMLCollection;
* `deleteCell(pos)`: 删除指定位置单元格;
* `insertCell(pos)`: 向cells集合的指定位置插入单元格并返回其引用;

```javascript
var table = document.createElement('table');
table.border = 1;
table.width = '100%';

var tbody = document.createElement('tbody');
table.appendChild(tbody);

var row1 = tbody.insertRow(0);
var cell_1_1 = row1.insertCell(0);
var text_1_1 = document.createTextNode('cell 1,1');
cell_1_1.appendChild(text_1_1);
var cell_1_2 = row1.insertCell(1);
var text_1_2 = document.createTextNode('cell 1,2');
cell_1_2.appendChild(text_1_2);

var row2 = tbody.insertRow(1);
var cell_2_1 = row2.insertCell(0);
var text_2_1 = document.createTextNode('cell 2,1');
cell_2_1.appendChild(text_2_1);
var cell_2_2 = row2.insertCell(1);
var text_2_2 = document.createTextNode('cell 2,2');
cell_2_2.appendChild(text_2_2);

document.body.appendChild(table);
```

以上实例代码将表格操作更加易于理解;

### NodeList

NodeList对象以及NamedNodeMap和HTMLCollection等都是动态扩展的, 随着DOM实时更新, 所以当访问这些元素时, 都会有一次基于文档的查询, 为了尽量减少访问次数, 最好查看其当时的快照(如前文给出的转换方法);

在遍历NodeList等元素时, 应该将其`length`属性提前取出, 否则有可能由于动态扩展的问题造成死循环;

## DOM扩展

DOM最主要的扩展是Selectors API(选择符API)以及HTML5, 这些扩展为DOM添加了更多的属性;

### Selector API

Select API主要实现使用**CSS选择符**来获取元素, 主要有以下两种方法:

* `querySelector()`: 接收一个CSS选择符, 并且返回与该模式匹配的第一个元素;
* `querySelectorAll()`: 接收一个CSS选择符, 并且返回与该模式匹配的NodeList;

个别比较老旧的浏览器并不支持该扩展, 在使用之前应该做好功能检查工作;

```javascript
var first_div = document.querySelector("body>div.first#demo");
alert(first_div);  // [object HTMLDivElement]
var divs = document.querySelectorAll("div");
alert(divs);  // [object NodeList]
```

使用CSS选择符给获取元素带来很大便利, 更便利的方式应该还是jQuery上关于DOM的操作;

### 元素遍历

如果直接遍历Node, 则需要判断节点类型; 如果需要直接遍历元素, 则可以用到以下的属性:

* `childElementCount`: 返回子元素的个数;
* `firstElementChild`: 指向第一个元素;
* `lastElementChild`: 指向最后一个元素;
* `previousElementSibling`: 前一个同辈元素;
* `nextElementSibling`: 后一个同辈元素;

和Node的属性可以对应, 可以省略节点类型判断;

### HTML5中的DOM扩展

由于class属性的大量使用, HTML5对类的功能进行了扩充, 主要添加了以下两个属性方法:

* `getElementsByClassName()`: 该方法通过接受一个或者多个类名的字符串, 返回NodeList;
* `classList`: 该属性针对具有多个class属性的元素, 提供比className更加强大的类处理;

通过以上方法和属性的扩展, 大大强化了对类的操作;

HTML5在DOM中添加了焦点功能, 用来获取DOM的焦点元素以及焦点的管理等, 主要通过以下属性和方法:

* `document.activeElement`: 该属性获取当前DOM中的焦点元素, 当页面加载完成时, 一般保存body元素, 通过tab键等方式可以改变焦点;
* `document.hasFocus()`: 该方法用来检测文档是否获得焦点, 由于页面未加载完成时没有焦点, 该方法可以用以判断;
* `focus()`: 元素调用该方法可以获得焦点;

HTML5还为HTMLDocument添加了以下属性:

* `document.readyState`: 该属性有两个可能的值: loading和complete, 用以描述文档加载状态;
* `document.compatMode`: 该属性有两个可能的值: CSS1Compat和BackCompat, 用以区分渲染模式;
* `document.head`: 返回head元素;
* `document.charset`: 返回文档使用的字符集;
* `document.defaultCharset`: 返回文档默认字符集;

在HTML5中可以为元素添加非标准属性, 只需要添加前缀`data-`, 这些属性会被添加到元素的`dataset`属性中作为DOMStringMap存在, 但是键名会做一些处理(去掉前缀、去掉非字符、转换为小写等)使用:

```javascript
document.write('<div id="myDiv" data-myname="someValue"></div>');

var div = document.querySelector("#myDiv");
print(div.dataset.myname)  // someValue
```

为了方便给文档插入大量HTML标记, HTML5加入了以下属性:

* `innerHTML`: 返回调用元素的所有子节点对应的HTML标记;
* `outerHTML`: 返回调用元素本身以及其所有子节点的HTML标记;
* `insertAdjacentHTML()`: 该方法接收两个参数, 第二个参数为HTML字符串, 第一个参数表示插入位置, 必须是以下四个值之一:
	* beforebegin: 在当前元素之前插入紧邻的同辈元素;
	* afterbegin: 在当前元素之下插入一个子元素, 该子元素作为第一个子元素;
	* beforeend: 在当前元素之下插入一个子元素, 该子元素作为最后一个元素;
	* afterend: 在当前元素之后插入紧邻的同辈元素;

利用以上属性方法, 可以在文档中方便地插入大量的标记, 如前文的插入表格, 使用以上属性方法可以简化为:

```javascript
var ul = document.querySelector("body>ul");
alert(ul.innerHTML);
alert(ul.outerHTML);
ul.insertAdjacentHTML("beforeend", "<li> item 4 </li>")
var table_string =
	`<table width="100%" border="1">
		<tbody>
			<tr>
				<td>cell 1,1</td>
				<td>cell 1,2</td>
			</tr>
			<tr>
				<td>cell 2,1</td>
				<td>cell 2,2</td>
			</tr>
		</tbody>
	</table>`
document.body.insertAdjacentHTML("beforeend",table_string)
```

上述多行字符串需要ES6支持;

四个位置对应的是标签出现的先后, 如beforebegin表示调用元素开始之前, 也就是上一个同辈元素; afterbegin则是调用元素开始之后, 也就是第一个子元素;

使用上述属性方法替换或者插入元素时, 如果其中带有事件处理, 则有可能导致事件处理的内存占用, 应该手动释放事件处理有关的绑定;

HTML5添加了`scrollIntoView()`方法, 使得元素在滚动页面时一直保持可见;

除了上述扩展之外, 不同浏览器之间还有一些没有被HTML5标准化的扩展, 就算已经标准化的扩展, 最好也需要查看一下浏览器的支持情况, 最新情况总是可以在文档中找到;
