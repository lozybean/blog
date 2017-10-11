Title: JavaScript表单
Date: 2015-12-7
Category: Learning
Tags: javascript

## 理解表单

表单在HTML中使用&lt;form&gt;元素表示, 对应在JavaScript中为HTMLFormElement类型, 该类型继承HTMLElement, 并且拥有以下特有属性或方法:

* `acceptCharset`: 服务器能够处理的字符集;
* `action`: 接受请求的URL;
* `elements`: 表单中所有控件的集合;
* `enctype`: 请求的编码类型;
* `length`: 表单中控件的数量;
* `method`: 要发送的HTTP类型;
* `name`: 表单的名称;
* `target`: 用于发送请求和接收响应的窗口名称;
* `reset()`: 将所有表单域重置为默认值;
* `submit()`: 提交表单;

除了使用`getElementById()`等方式获取表单元素之外, `document.forms`可以一次性获取页面中的所有表单;

有两种方式可以提交表单, 一种是通过创建`submit`按钮,
