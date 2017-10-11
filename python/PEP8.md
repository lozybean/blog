Title: PEP8
Date: 2015-09-27
Category: Learning
Tags: python, PEP8

[**PEP**](https://www.python.org/dev/peps/)(Python Enhancement Proposals)，是一系列关于Python的改进建议，其中的第8篇建议，即**PEP8**（Title: Style Guide for Python Code），则是针对Python的代码编写规范提出的一些建议。本文记录一些比较个人比较在意的点。

> Readability Counts

一份编码建议旨在提高编码的可读性，统一的代码编写规范对代码可读性来说非常重要，而且从PEP中，可以看出Python的设计哲学，从而更好地理解Python。

## 代码编排

#### 1. 使用4个空格缩进，避免使用tab缩进

强制使用缩进是Python代码的一大特点，但是同时也是很多人对Python绝望的特性，这是由于很多不好的缩进习惯造成的，甚至在一些知名实验室中的代码都存在这些问题（如The Huttenhower Lab的LEfSe的[源码](https://bitbucket.org/nsegata/lefse)中，混合使用了空格和tab的缩进，如果只从代码角度上讲，这份源码是烂透了)。

为什么不使用tab（制表符）进行缩进呢？这是因为不同环境下，对制表符的转换有所不同，而空格则可以永远保持相同的外观。使用tab键作为缩进的代码，如果对其做复制操作，则多半会出缩进问题。

使用tab缩进的好处是，只需要按一次键盘，但是基本上主流IDE和编辑器都支持expand tab，如：

* vim中，设置`expandtab`、`softtabstop=4`即可在输入tab时自动转换为4个空格，使用`autocmd FileType python setlocal`设置，只在编辑python文件时生效。
* sublime中，设置	`"tab_size": 4`,`"translate_tabs_to_spaces": true`这两项，即可在输入tab时进行转换。
* pycharm等IDE在设置中都有indent选项，pycharm默认为4个空格。

在python3中，混合使用tab和空格将会报错，python2中，使用-t选项可以显示警告，使用-tt选项可以直接报错。

#### 2. 代码一行最大长度为79

在编程时，应该注意以下两点，来保证代码一行长度不过长：

* 避免在一行中写出复杂的生成式或者列表推导式，有时候为了显示能够熟练使用一些技巧，会写出很长的推导式，这样的代码可读性较差，在性能上也不一定具有提升，这种写法是**complicated**的，为了代码可读性，平凡的**simple**要比装逼的**complicated**好的多。
* 不要使用`;`将多条语句强行写到同一行上。
* 不可避免地需要较长代码时，应该使用`\`符号进行分隔，分隔的同时应该注意缩进规则，不可使用和下一级相同的缩进，参考：[https://www.python.org/dev/peps/pep-0008/#indentation](https://www.python.org/dev/peps/pep-0008/#indentation)；
* 参数列表过长时，应该换行，并且遵守上述缩进规则。

对一行代码的最大长度做出限制，看似是比较顽固保守的，但是其中透露出`The Zen of Python`中的重要思想：


>Simple is better than complex.

>Complex is better than complicated.

#### 3. 空行与空格的使用

空行的含义在于分割逻辑，空行使用原则为：

* 类和顶级元素之间空两行
* 方法定义之间空一行
* 同一个函数内逻辑上无关的段落空一行，并且尽量避免这种情况，一个函数应该实现最小功能
* 其他地方不要使用空行

空格用来在一个句子中，分隔独立含义的元素。

* 避免不必要的空格，如：

	1.	在括号后面不要使用空格，如：<del>spam( ham[ 1 ], { eggs: 2 } )</del>，正确的写法为：`spam(ham[1], {eggs: 2})`；
	2. 在分割符号（`,`，`;`，`:`）前面不要用空格，如：<del>if x == 4 : print x , y ; x , y = y , x</del>，正确的写法为：`if x == 4: print x, y; x, y = y, x`；
	3. 在具有独立含义的元素中，不使用空格，如在切片操作中，两个冒号两侧都不应该使用空格；
	4. 在函数调用的小括号前、索引的中括号前不应该使用空格；
	5. 不要为了让`=`对齐，而在`=`两侧添加多余的空格；

* 在操作符两侧使用一个空格，当操作符有优先级关系时，只再低优先级的操作符两侧添加空格，如：

	1. `i = i + 1`
	2. `submitted += 1`
	3. `x = x*2 - 1`
	4. `hypot2 = x*x + y*y`
	5. `c = (a+b) * (a-b)`

* 在表示字典参数时，`=`两侧不添加空格，如：`complex(real, imag=0.0)`，但是，当出现`:`的参数注释以及`->`的返回值注释时，则应该使用空格，如：`def munge(sep: AnyStr = None):`，`def munge() -> AnyStr:`；

不论`if`、`try`后面有几条语句，都应该分行显示；不论语句多么简短，不同的语句都应该分行显示，而不要使用`;`连接，如：

```python
if foo == 'blah':
    do_blah_thing()
do_one()
do_two()
do_three()
```

而不能写作<del>if foo == 'blah': do_blah_thing()</del>，<del>do_one(); do_two(); do_three()</del>等；

## 文档编排

#### 1. 编排顺序

按照：*docstring* -> *import* -> *globals* & *constance* -> 其他定义的顺序编排。

[PEP257](https://www.python.org/dev/peps/pep-0257)给出了文档描述编写时的建议，只需要为`public`的模块、类、方法添加文档描述，非`public`的内容不需要添加描述，意味着不希望使用者使用或者修改这部分代码，以免更新后的代码不可控，虽然Python没有提供直接`private`，在提供了更大使用空间的同时，对使用者也有了更高的要求。

在import部分，应当以*标准库* -> *三方库* -> *自己编写的库*的顺序导入，并在中间以空行隔开，python2中如果需要导入`__future__`，则应该写在最开头；在import时，每个独立的模块都应该使用单独的`import`，而不要使用`,`一次导入多个模块，如：<del>import sys, os</del>，但是同一个模块的不同类、方法则可以使用`,`一次导入多个，如正确的写法示例：

```python
import sys
import os
from subprocess import Popen, PIPE
```

#### 2. 注释

>Comments that contradict the code are worse than no comments

不要添加过多不必要的注释，在代码更新时，必须更新注释内容。注释的内容必须使用完整的英文句子。

块注释时，应该以`#`并且紧跟一个空格开头，空行应该以单独的`#`表示，如：

```python
# It is a block comment.
#
# Something else.
#
# It is the end line.
```

单行注释应该**尽量避免**，如确实要使用，则在代码和注释之间添加至少两个空格。

注释的内容不应该是语句的翻译，而是解释语句的意义，如：

这是一个无谓的注释，这种注释不如没有：

```python
x = x + 1                 # Increment x
```

这是一个有意义的注释，解释该语句的意义：

```python
x = x + 1                 # Compensate for border
```

## 命名规范

1. 没有必要给同一个模块的函数添加统一的前缀，因为Python中函数将始终受到模块的名称保护，同样的方法也受到所在对象的名称保护；
2. 使用单个下划线`_`开头的方法，表示不建议直接使用，有些模块中习惯使用单个下划线`_`结尾，用以区分关键字；使用双下划线`__`开头，无法正常直接导入，如果非要使用，则需要用`_ClassName__boo`这种方式；在开头和结尾使用双下划线`__`表示一些魔术方法，如`__str__`、`__class__`等等；
3. 避免单独使用小写`l`、大写`O`、大写`I`等容易搞错的字母；
4. 模块名称应该使用简短的、全小写的单词，可以使用下划线来增加可读性，而包名中则不应该使用下划线；由于模块名称对应文件名称，应该考虑多平台下的文件命名规范；
5. 类名应该使用`CapWords`的方式，如果是内部类，则使用`_CapWords`;
6. 异常类的命名，应该使用：`CapWordsError`的方式；
7. 函数和变量命名应该全部使用小写字母，用下划线来增加可读性；
8. 常量命名应该全部使用大写字母，用下划线来增加可读性；

## 编码优化建议

PEP8给出了很多具体的建议，详细可以参考[官方文档](https://www.python.org/dev/peps/pep-0008/#programming-recommendations)

1. 对于不同的Python，应该考虑到实现的过程，从而使用最好的代码来编写，如字符串连接操作时：在CPython(标准Python解释器)中，`+`的效率非常高，而Jython中使用`.join()`效率比较高；
2. 使用`is`、`is not`来代替`==`、`!=`；避免对非布尔变量直接判断，而应该使用`if x is not None`的形式；避免使用`if not x is None`这种容易混肴的语句；如果是布尔变量，则应该直接用来判断，而不要再和布尔值比较，如当`greeting`是布尔变量时，应该使用`if greeting`而不是`if greeting == True`或者更差的`if greeting is True`；
3. 尽量避免使用匿名函数`lambda`，而对每个函数都进行定义；
4. 使用模块和包中的异常类，避免直接使用`Exception`；更加不要使用裸露的`except`来获取所有的异常，而应该紧跟具体的异常；`try`的中的代码应该尽量少，从而利于异常的准确定位，不要在`try`后面直接`return`一个过程,应该将计算的过程代码独立出来；
5. 使用`raise ValueError('message')`来代替`raise ValueError,'message'`；
6. 应该避免空的`return`，而使用`return None`；并且在分支结构中，确保每个分支都有返回值，而不应该置空一些可能不重要的分支；
7. 检查对象类型时，应该使用`isinstanceof(obj,int)`，而不是`type(obj) is type(1)`，这样会造成一次额外的运算；
8. 同样的，检查序列是否为空时，应该直接判断，如`if seq`,而不要使用`if len(seq)`先求长度；
9. 使用内置的方法，而不是另外重新实现，如判断字符串的后缀时，应该使用`if foo.endswith('gz')`而不应该使用`if foo[:-2] == 'gz'`；
