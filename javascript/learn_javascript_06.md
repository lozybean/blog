Title: JavaScript事件
Date: 2015-12-3
Category: Learning
Tags: javascript

事件实现了HTML和JavaScript之间的交互工作;

## 事件流

当一个事件发生时, 不仅仅发生在某个元素上, 同时也在该元素的容器上发生, 为了描述接收事件的顺序, 引入了**事件流**的概念, 由于历史原因, IE和Netscape浏览器在事件流的描述上采用完全不同的顺序: **事件冒泡**和**事件捕获**:

* 事件冒泡: 表示事件开始由具体元素接收, 然后逐级向上传播;
* 事件捕获: 表示上级元素最先接收到事件, 最具体的元素最后接收;

而DOM事件流则将事件过程描述为三个阶段:

1. 事件捕获阶段: 该阶段发生事件捕获, 从上级元素开始逐级往下传递;
2. 处于目标阶段: 实际目标在该阶段接收到事件;
3. 事件冒泡阶段: 该阶段发生事件响应, 从实际目标开始逐级冒泡;

## 事件处理

用户或者浏览器执行的某种动作称为事件, 常用的如: click、mouseover、load等; 响应这些事件的函数就成为事件处理函数, 这些函数以`on`开头, 后面跟着事件名称, 如: `onclick`就是一个对click事件的事件处理函数;

### HTML事件处理

在HTML元素中添加事件处理函数特性, 即可完成事件处理, 如:

```html
<input type="button" value="Click Me" onclick="alert('Clicked')">
```

上述HTML定义一个按钮, 并且具有onclick特性, 该特性即是一个事件处理函数, 该函数可以如上述直接描述函数体, 也可以引用其他页面定义的函数;

当引用其他页面(如js脚本)中的函数时, 如果该页面没有加载完成, 就会引发错误, 为了隐藏错误, 需要对其进行异常处理:

```html
<input type="button" value="Click Me" onclick="try{showMessage();}catch(ex){}">
```

HTML事件处理比较直接, 但是缺点非常明显, 除了以上缺点之外, 由于不同js引擎对标识符的处理差异等也会引发一些错误;

更加严重的问题是: 通过HTML做事件处理, 使得HTML和JavaScript形成紧密耦合, 一旦该事件需要跟新, 则同时需要更新HTML和JavaScript两份代码, 不利于维护;

### DOM级处理程序

为了解决上述不足, 通过JavaScript指定事件处理是比较好的方法;

DOM实现事件处理时, 其实就是操作元素的事件处理程序属性:

```javascript
var element_input = document.getElementById("click");
element_input.onclick = function () {
    alert(this.id)
};
```

如上实例, 通过操作元素的属性来为元素添加事件处理程序, 如果要删除某个事件, 也仅仅需要将事件处理程序设为`null`即可;

DOM2级事件处理在此基础上为元素定义了两个方法:

* `addEventListener(element, function, boolean)`: 该方法为指定事件添加指定的处理程序, 第三个参数为`true`时表示在捕获阶段调用, `false`时表示在冒泡阶段调用;
* `removeEventListener(element, function, boolean)`: 该方法移除指定事件的指定处理程序;

通过第一个方法添加的事件处理程序, 只能通过第二个方法删除, 并且需要接收相同的函数对象, 这意味着如果是匿名处理程序, 则会导致事件处理程序无法删除;

一般而言, 在冒泡阶段处理事件能够有更好的兼容性, 除非必须在事件到达目标之前就截获时才使用捕获阶段处理;

```javascript
function showMessage() {
    alert('I have been clicked')
}
var element_input = document.getElementById("click");
element_input.addEventListener("click", showMessage, false);
element_input.removeEventListener("click", showMessage, false);
```

而在IE浏览器中, 则实现了类似上述的事件处理程序: `attachEvent()`和`detachEvent()`, 由于IE8之前版本只支持事件冒泡, 这两个方法接收两个参数;

为了在不同浏览器中能够兼容使用事件处理, 最好通过以下方法做功能检查:

```javascript
var EventUtil = {
    addHandler: function (element, type, handler) {
        if (element.addEventListener) {
            element.addEventListener(type, handler, false);
        } else if (element.attachEvent) {
            element.attachEvent("on" + type, handler);
        } else {
            element["on" + type] = handler;
        }
    },
    removeHandler: function (element, type, handler) {
        if (element.removeEventListener()) {
            element.removeEventListener(type, handler, false);
        } else if (element.detachEvent) {
            element.detachEvent("on" + type, handler);
        } else {
            element["on" + type] = null;
        }
    }
};
```

上述方法将按照: DOM2 -> IE -> DOM 的顺序选择事件处理方法;

## 事件对象

使用DOM处理事件时, 无论使用哪种方法, 都会传入一个event对象:

```javascript
function showMessage(event) {
    alert(event)
}
var element_input = document.getElementById("click");
EventUtil.addHandler(element_input, 'click', showMessage);  // [object MouseEvent]
```

这些event对象根据事件类型不同而具有不同的方法, 但是都会具有以下属性的方法:

* `type`: 触发事件的类型;
* `bubbles`: 表名事件是否冒泡;
* `cancelable`: 表名是否可以取消事件的默认行为;
* `currentTarget`: 当前正在处理事件的元素;
* `target`: 事件的目标元素;
* `defaultPrevented`: 表名是否已经调用了preventDefault()方法;
* `detail`: 与事件相关的细节信息;
* `eventPhase`: 表示事件处理的阶段: 1,捕获阶段; 2,处于阶段; 3,冒泡阶段;
* `trusted`: true表示该事件是浏览器生成的, false表示是开发人员通过JavaScript创建的;
* `view`: 与事件关联的抽象视图, 相当于发生事件的window对象;
* `preventDefault()`: 取消事件的默认行为;
* `stopImmediatePropagation()`: 取消事件的进一步获取或者冒泡, 同时阻止任何事件处理程序被调用;
* `stopPropagation()`: 取消事件的进一步获取或者冒泡;

以上属性方法都是只读的, 通过访问某些属性可以获取到事件的详细信息, 便于做出处理;

如, 当一个元素具有多个事件时, 可以通过`type`属性判断, 并分别处理:

```javascript
var handler = function (event) {
    switch (event.type) {
        case "click":
            alert("Clicked");
            break;
        case "mouseover":
            event.target.style.backgroundColor = 'red';
            break;
        case "mouseout":
            event.target.style.backgroundColor = '';
            break;
    }
};
var element_input = document.getElementById("click");
EventUtil.addHandler(element_input, 'click', handler);
EventUtil.addHandler(element_input, 'mouseover', handler);
EventUtil.addHandler(element_input, 'mouseout', handler);
```

由于事件的处理是通过冒泡的方式进行的, 所以事件处理程序可能并不属于元素本身, 而可能是元素的上一级元素;

```javascript
function showMessage(event) {
    alert(event.currentTarget + ' ' + event.target)
}
EventUtil.addHandler(document.body, 'click', showMessage);  // [object HTMLBodyElement] [object HTMLInputElement]
```

在事件处理时, input元素并没有事件处理程序, 于是向上冒泡, 直到在body中找到该事件的处理程序, 此时, this对象为body, 而真实处理的对象target则是input;

当input和body中都存在事件处理程序时, 通过事件冒泡会依次执行, 通过`stopPropagation()`方法可以停止进一步冒泡;

同样的, IE浏览器在处理事件对象时, 对这些属性方法的使用有所不同, 和前文相同, 需要对不同浏览器进行功能检查并且适配;

更加完整的`EventUtil`:

```javascript
var EventUtil = {
    addHandler: function (element, type, handler) {
        if (element.addEventListener) {
            element.addEventListener(type, handler, false);
        } else if (element.attachEvent) {
            element.attachEvent("on" + type, handler);
        } else {
            element["on" + type] = handler;
        }
    },
    removeHandler: function (element, type, handler) {
        if (element.removeEventListener()) {
            element.removeEventListener(type, handler, false);
        } else if (element.detachEvent) {
            element.detachEvent("on" + type, handler);
        } else {
            element["on" + type] = null;
        }
    },
    getEvent: function (event) {
        return event ? event : window.event;
    },
    getTarget: function (event) {
        return event.target || evnet.srcElement;
    },
    preventDefault: function (event) {
        if (event.preventDefault) {
            event.preventDefault();
        } else {
            event.returnValue = false;
        }
    },
    stopPropagation: function (event) {
        if (event.stopPropagation) {
            event.stopPropagation();
        } else {
            event.cancelBubble = true;
        }
    }
};
```

## 事件类型

以下讨论具体的各种事件类型, 其具体类型以及触发条件为:

* UI事件, 当用户与页面的元素交互时触发;
* 焦点事件, 当元素获得或者失去焦点时触发;
* 鼠标事件, 当使用鼠标在页面上执行操作时触发;
* 滚轮事件, 当使用滚轮或者类似设备时触发;
* 文本事件, 当文档中输入文本时触发;
* 键盘事件, 当用户通过键盘在页面上执行操作时触发;
* 合成事件, 当为IME(输入法编辑器)输入字符时触发;
* 变动事件, 当底层DOM结构发生变化时触发;

这些事件在DOM2和DOM3级别可能有所不同, 具体需要结合文档查看最新支持的情况;

### UI事件

UI事件往往和window对象或者表单控件有关, 包括文档的加载与卸载事件、错误事件、窗口大小改变、窗口滚动带动元素、选择文本框等等;

#### 1. load

当页面完成加载后(包括图像、CSS、JS等外部资源), load事件将被触发, 该事件常用来检查页面是否完整加载, 以便脚本操作其中元素, DOM2中规定该事件应该从document上触发, 可是为了兼容性等问题, 往往浏览器都会在window上实现该事件:

```javascript
EventUtil.addHandler(window, "load", function () {
    var image = document.createElement("img");
    EventUtil.addHandler(image, "load", function (event) {
        event = EventUtil.getEvent(event);
        alert(EventUtil.getTarget(event).src);
    });
    document.body.appendChild(image);
    image.src = "smile.gif";
});
```

以上例程中, 首先为window添加load事件, 并当其加载完成后添加image元素到DOM中;

需要注意的是, 图像和其他资源加载的条件并不相同:

* `<img>`元素在指定src属性之后就会立即开始图像下载, 而不一定需要先添加到文档中, 因此, src属性应该在事件处理程序之后添加;
* `<script>`、`<link>`元素指定外部资源时, 只有当该元素被添加到文档, 并且设置了src属性后才会开始加载, 因此, src属性和事件处理程序的先后顺序没有必要区分;

所以上述例程将src属性放到最后指定;

#### 2. unload

unload事件当整个文档被卸载后触发(从一个页面切换到另外一个页面), 利用这个事件可以进行引用的清除, 以免内存泄露;

而由于该事件是文档卸载后才触发, 此时页面加载后的对象已经不一定存在, 如果直接操作DOM节点或者样式将会出现错误;

#### 3. resize

resize事件当窗口大小被调整到新的宽度和高度时触发, 多数浏览器会在窗口变化1像素时就触发, 由于触发可能非常频繁, 因此不可以在该事件的处理程序中添加计算量较大的代码, 以免浏览器反应变慢;

#### 4. scroll

scroll事件当文档被滚动期间触发, 通过监控&lt;body&gt;元素的scrollLeft和scrollTop这两个属性可以监控当前位置; 除了Safari之外的浏览器都将通过&lt;html&gt;元素来描述该变化, 为了浏览器间兼容, 应该做以下判断:

```javascript
EventUtil.addHandler(window, "scroll", function () {
    if (document.compatMode == 'CSS1Compat') {
        alert(document.documentElement.scrollTop);
    } else {
        alert(document.body.scrollTop);
    }
});
```

和resize事件一样, scroll事件也会被频繁触发, 因此应该尽量保持事件处理程序简单;

### 焦点事件

该事件当元素在页面元素获得或者失去焦点时触发, 和document.hasFocus()方法以及document.activeElement属性配合使用, 可以跟踪用户在页面上的行为; 主要包括以下六个事件:

1. `focusout`: 元素失去焦点时触发, 并向上冒泡;
2. `focusin`: 元素获得焦点时触发, 并向上冒泡;
3. `blur`: 元素失去焦点时触发, 不会冒泡;
4. `DOMFocusOut`: 元素失去焦点时触发, 只有Opera支持, DOM3中废弃;
5. `focus`: 元素获取焦点时触发, 不会冒泡;
6. `DOMFocusIn`: 元素获取焦点时触发, 只有Opera支持, DOM3中废弃;

当焦点从一个元素转向另外一个元素时, 这六个事件依次触发;

通过以下方法可以检查浏览器是否支持这些事件:

```javascript
var isSupported = document.implementation.hasFeature("FocusEvent", "3.0");
```

### 鼠标和滚轮事件

鼠标作为最重要的定位设备, 其事件也因此非常重要, 主要有以下事件:

* `mousedown`: 按下鼠标时触发;
* `mouseup`: 松开鼠标时触发;
* `click`: 单击主鼠标按钮时触发; 可以通过键盘触发;
* `dblclick`: 双击主鼠标按钮时触发; DOM3支持;
* `mouseenter`: 鼠标光标从元素外部移动到元素范围内触发; 进入子元素不触发; 该事件不冒泡; DOM3支持;
* `mouseleave`: 鼠标光标从元素内部移动到元素范围外触发; 离开子元素不触发; 该事件不冒泡; DOM3支持;
* `mouseover`: 鼠标光标从元素外部移动到元素或者其子元素时触发;
* `mouseout`: 鼠标光标从元素或者其子元素移动到元素外部时触发;
* `mousemove`: 鼠标指针在元素内部移动时, 反复触发;

其中, 只有mouseenter和mouseleave不冒泡, 只有click通过键盘可以触发;

#### 按钮和点击

由于鼠标上有多个按钮, 为了判断哪个按钮被按下, DOM给`mousedown`和`mouseup`事件添加了`button`属性, 该属性只有三个值: 0:鼠标主按钮, 1:鼠标中间按钮, 2:鼠标副按钮; 而IE浏览器中, 该属性的值就比较复杂:

* 0: 没有按下任何按钮;
* 1: 按下主按钮;
* 2: 按下副按钮;
* 3: 同时按下主、副按钮;
* 4: 按下中间按钮;
* 5: 同时按下主按钮和中间按钮;
* 6: 同时按下副按钮和中间按钮;
* 7: 同时按下三个按钮;

通过检查是否支持DOM2来判断使用那一个模型;

```javascript
var isDOM2Supported = document.implementation.hasFeatrue("MouseEvents", "2.0");
```

按钮点击的同时, 事件对象具有**修改键**属性:

* `shiftKey`: 表示鼠标事件发生时shift按键是否被按下;
* `ctrlKey`: 表示鼠标事件发生时ctrl按键是否被按下;
* `altKey`: 表示鼠标事件发生时alt按键是否被按下;
* `metaKey`: 表示鼠标事件发生时, meta按键(windows下为win键, mac下为cmd键)是否被按下;

只有在同一个元素上相继触发mousedown和mouseup, click事件才会被触发; 也只有当触发两次click之后, dblclick才会触发; 这四个事件的触发顺序为:

1. mousedown
2. mouseup
3. click
4. mousedown
5. mouseup
6. click
7. dblclick

如果中间有中断, 则后续事件将不再触发;

#### 鼠标移动

判断鼠标是否移入元素有四个方法, 这四个方法分成两对, 其中mouseenter和mouseleave只在移动到元素中或者移除元素外才触发, 而mouseover和mouseout则当鼠标移动到元素或者子元素时触发;

mouseover和mouseout事件具有额外的相关元素属性: `relatedTarget`, 当mouseover触发时, 主目标是获得光标的元素, 而相关元素就是失去光标的元素; mouseout反之;该属性在IE8之前不支持, 并使用`fromElement`和`toElement`属性保存, 兼容获取该属性的方法为:

```javascript
var getRelatedTarget = function(event){
	if (event.relatedTarget){
		return event.relatedTarget;
	} else if (event.toElement){
		return event.toElement;
	} else if (event.fromElement){
		return event.fromElement;
	} else {
		return null;
	}
}
```

通过以下方法检查鼠标事件是否被支持:

```javascript
var isDOM2Supported = document.implementation.hasFeature("MouseEvents", "2.0");
var isDOM3Supported = document.implementation.hasFeature("MouseEvent", "3.0");
```

注意DOM3中事件名称为MouseEvent;

鼠标事件触发后, 可以通过以下属性获取光标的位置:

* `clientX`, `clientY`: 这两个属性表示鼠标光标相对浏览器的水平和垂直坐标;
* `pageX`, `pageY`: 这两个属性表示鼠标光标相对文档的水平和垂直坐标; IE8及更早版本不支持;
* `screenX`, `screenY`: 这两个属性表示鼠标光标相对整个屏幕的水平和垂直坐标;

以上属性的水平坐标都是从左边缘计算的位置, 垂直坐标都是从上边缘计算的位置; 和scrollLeft和scrollTop对应;

虽然IE8以及早期版本不支持文档位置属性, 但是可以通过浏览器位置以及滚动的信息计算出来:

```javascript
EventUtil.addHandler(element, "click", function(event){
	event = EventUtil.getEvent(event);
	var pageX = event.pageX;
	var pageY = event.pageY;
	if (pageX == undefined){
		pageX = event.clientX + (document.body.scrollLeft || document.documentElement.scrollLeft);
	}
	if (pageY == undefiend){
		pageY = event.clientY + (document.body.scrollTop || document.documentElement.scrollTop);
	}
});
```

#### 滚轮

不同浏览器对鼠标滚轮的方法支持并不相同, 通过以下方法可以做到兼容:

```javascript
var getWheelDelta = function(event){
	if (event.wheelDelta){
		if (client.engine.opera && client.engine.opera < 9.5){
			// opera 9.5 之前版本方向和其他浏览器相反
			return -event.wheelDelta
		}else{
			// IE、Chrome、Safari、opera 9.5之后版本
			return event.wheelDelta
		}
	} else {
		// firefox
		return -event.detail * 40;
	}
}
```

#### 其他注意点

由于移动设备的特殊性, 在给iOS的Safari开发时, 应该注意以下几点:

* 不支持dblclick;
* 轻击可单击的元素可以触发mousemove事件; 如果屏幕因此变化, 则不再有任何事件发生, 否则一次触发mousedown、mouseup、click;
* 轻击不可单机的元素不会触发任何事件;
* mousemove事件也会触发mouseover和mouseout事件;
* 两个手指滚动页面时, 触发mousewheel和scroll事件;

尽量选择使用`click`事件;

### 键盘与文本事件

当用户使用键盘时会触发该事件, 不同浏览器的表现差异较大;

主要有三个事件:

* `keydown`: 按下任意键时触发, 如果不松开, 则重复触发;
* `keypress`: 按下任意字符按键时触发, 如果不松开, 则重复触发;
* `keyup`: 释放键盘上的按键时触发;

同时也支持修改键属性(IE不支持metaKey);

当触发keydown和keyup事件时, 事件可以获得一个keyCode属性表示键盘上对应按键的编码;

而当keypress事件触发时, 还会获得charCode编码表示字符对应的ASCII编码, 而IE和opera早期浏览器则仍然在keyCode属性中保存其ASCII编码;

DOM3中新增key和char属性来替换charCode, 但是不同浏览器之间支持的差异较大;

DOM3还新增了`textInput`事件, 该事件当用户在可输入区域中输入字符时触发; 该事件具有两个属性:

* `data`: 保存输入字符;
* `inputMethod`: 保存输入的来源;

### 复合事件

复合事件是DOM3中添加的事件, 主要用于处理IME文本复合系统, 浏览器支持较少, 检查支持方式为:

```javascript
var isSupported = document.implementation.hasFeature("CompositionEvent", "3.0");
```

### 变动事件

变动事件在DOM的某一部分发生变化时触发, 有很多变动事件已经在DOM3中作废并且不推荐使用, 仍然支持的事件主要有:

* `DOMNodeRemoved`: 当子节点在父节点中被移除时触发, 事件属性relatedNode为父节点;
* `DOMNodeRemovedFromDocument`: 当节点在文档中移除时触发, 该事件不冒泡;
* `DOMNodeInserted`: 在一个子节点插入到父节点时触发, 事件属性relatedNode为父节点;
* `DOMInsertedIntoDocument`: 当节点被插入到文档中时触发, 该事件不冒泡;
* `DOMSubtreeModified`: 在DOM结构中发生任何变化时触发, 任何以上事件触发后都会触发该事件;

### HTML5事件

HTML5给出了很多浏览器应该支持的事件, 有一些已经得到完善支持, 有一些还没有; 使用前应该做好文档查阅和功能检查的工作;

#### 1. contextmenu

上下文菜单事件, 该事件当鼠标点击或者修饰键加鼠标点击时触发, 属于鼠标事件, 该事件中包含鼠标事件的属性; 通过设置该事件的处理程序, 可以自定义上下文菜单;

以书上例子为例:

```html
<!DOCTYPE HTML PUBLIC "-//W3C//DTD HTML 4.01//EN" "http://www.w3.org/TR/html4/strict.dtd">
<html lang="en">
<head>
    <meta http-equiv="Content-Type" content="text/html;charset=UTF-8">
    <title>ContextMenu Event Example</title>
</head>
<body>
<div id="myDiv">Right click or Ctrl+click me to get a custom context menu.
    Click anywhere else to get the default context menu.
</div>
<ul id="myMenu" style="position:absolute; visibility: hidden; background-color: silver">
    <li><a href="http://www.nczonline.net">Nicholas' site</a></li>
    <li><a href="http://www.wrox.com">Wrox site</a></li>
    <li><a href="http://www.yahoo.com">Yahoo!</a></li>
</ul>
<script src="learn_event.js"></script>
</body>
</html>
```

```javascript
// include EventUtil;
EventUtil.addHandler(window, "load", function (event) {
    var div = document.getElementById("myDiv");
    EventUtil.addHandler(div, "contextmenu", function (event) {
        event = EventUtil.getEvent(event);
        EventUtil.preventDefault(event);
        var menu = document.getElementById("myMenu");
        menu.style.left = event.clientX + "px";
        menu.style.top = event.clientY + "px";
        menu.style.visibility = "visible";
    });
    EventUtil.addHandler(document, "click", function (event) {
        document.getElementById("myMenu").style.visibility = "hidden";
    });
});
```

#### 2. beforeunload

beforeunload在页面卸载之前触发, 该事件会弹出一个对话框, 询问是否确定离开, 事件的returnValue属性表示对话框显示的文字内容;

#### 3. DOMContentLoaded

DOMContentLoaded事件在形成完整DOM树时即可触发, 而不必要等到所有的外部资源加载完毕, 所以该事件发生在load事件之前;

#### 4. readystatechange

readystatechange事件在某个元素或者文档加载状态改变时触发, 支持该事件的元素有一个readyState属性, 用以表示该元素的当前状态, 这些状态包括:

1. `uninitialized`: 未初始化, 表示对象存在当未完成初始化;
2. `loading`: 正在加载对象数据;
3. `loaded`: 对象数据加载完毕;
4. `interactive`: 交互状态, 可以操作对象, 但是没有完全加载;
5. `complete`: 对象加载完成;

这些属性全部属性元素, 而事件本身并没有提供任何信息;

并非所有的对象都会经历所有阶段, 有可能会跳过中间的某个阶段; 如有些元素会一直停留在load状态, 而有些元素则会跳过load直接进入complete, 而这两个状态都表示该元素已经可用;

进行动态脚本或者动态样式加载外部资源时, 可以判断加载的状态, 以便调用;

```javascript
// 动态脚本
EventUtil.addHandler(window, "load", function () {
    var script = document.createElement("script");

    EventUtil.addHandler(script, "readystatechange", function (event) {
        event = EventUtil.getEvent(event);
        var target = EventUtil.getTarget(event);

        if (target.readyState == "loaded" || target.readyState == "complete") {
            EventUtil.removeHandler(target, "readystatechange", arguments.callee);
            alert("script loaded");
        }
    });
    script.src = "example.js";
    document.body.appendChild(script);
});

// 动态样式
EventUtil.addHandler(window, "load", function () {
    var link = document.createElement("link");
    link.type = "text/css";
    link.rel = "stylesheet";

    EventUtil.addHandler(link, "readystatechange", function (event) {
        event = EventUtil.getEvent(event);
        var target = EventUtil.getTarget(event);

        if (target.readyState == "loaded" || target.readyState == "complete") {
            EventUtil.removeHandler(target, "readystatechange", arguments.callee);
            alert("CSS loaded");
        }
    });
    link.src = "example.css";
    document.head.appendChild(link);
});
```

#### 5. pageshow, pagehide

浏览器为了加快前进和后退时的速度, 添加了往返缓存的特性, 将页面数据保存在缓存中(back-forward cache, bfcache), 当页面从往返缓存中读取时, 页面的load事件并不会触发, 而pageshow事件则始终会触发; 相反地, 如果一个页面没有显式处理unload事件, 则在其卸载时会保存到缓存中, 从而触发pagehide事件;

pageshow事件包含一个persisted属性, 表示该页面是否保存在bfcache中;

这两个事件目标虽然是document, 但由于浏览器实现时为了兼容性通常会将这两个事件添加到window中;

```javascript
(function () {
    var showCount = 0;

    EventUtil.addHandler(window, "load", function () {
        alert("Load fired");
    });

    EventUtil.addHandler(window, "pageshow", function (event) {
        showCount++;
        alert("Show has been fired " + showCount +
            " times. Persisted? " + event.persisted);
    });
})();

EventUtil.addHandler(window, "pagehide", function (event) {
    alert("Hidding. Persisted? " + event.persisted);
});
```

#### 6. hashchange

该事件当URL中的hash值改变时触发, 通常用于Ajax应用中利用URL参数保存导航信息;

该事件包含两个属性: oldURL和newURL, 分别表示改变之前和改变之后的URL, 但是IE和firefox浏览器不支持这两个属性, 这时需要通过location.hash确定当前hash;

以下代码检测浏览器是否支持该事件:

```javascript
var isSupported = ("onhashchange" in window) &&
					 (document.documentMode === undefined || document.documentMode > 7);
```

### 设备事件以及触摸手势事件

设备事件应用于智能手机和平板电脑中, 主要当设备横向模式、纵向模式切换时触发, 不同浏览器对该事件的处理不同, 与设备支持也紧密相关;

触摸事件和手势事件在iOS的Safari中应用, 当触摸屏幕或者在屏幕中产生手势时相应触发;

## 事件性能

每个事件处理程序都会占用内存, 内存中事件越多, 性能就会越差, 并且事件程序的处理会增加DOM的访问次数, 从而降低性能;

为了提升事件的性能, 有以下两种方式来优化事件:

1. 利用事件冒泡, 将相同类型的事件统一使用同一个事件处理程序, 只在程序内部判断目标元素类型从而做出相应操作;
2. 即使移除内存中过时不用的空事件处理程序, 另外当使用innerHTML元素替换元素时, 应该首先移除事件处理程序, 否则会导致无法正确回收;

应该避免一些无意义的事件, 以及避免给会重复触发的事件添加复杂处理, 从而提高浏览器性能;

## 事件模拟

在IE之外的浏览器中, 使用`document.createEvent(type)`方法, 并且传入相应事件类型, 经过相应的初始化操作之后, 通过`dispatchEvent(event)`方法模拟事件的触发;

这些事件类型包括:

* UIEvents, 对应DOM3中为UIEvent;
* MouseEvents, 对应DOM3中为MouseEvent;
* MutationEvents, 对应DOM3中为MutationEvent;
* HTMLEvents, DOM3中将该类型分散到其他事件类型中;
* KeyboardEvent, 只在DOM3中存在;

当浏览器中不支持某些事件时, 也可以通过`document.createEvent("Events")`方法创建通用事件对象, 使用`initEvent()`初始化, 然后给该对象赋予相应的属性, 再通过`dispatchEvent()`方法触发;

而在IE浏览器中, 则需要调用`document.createEventObject()`方法创建event对象, 然后直接给该对象赋值相应的属性, 直到在触发事件时, 使用`fireEvent(type, event)`才指定事件类型;
