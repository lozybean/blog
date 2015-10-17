Title: Jinja2初探之模板语法
Date: 2015-10-17
Category: pages
Tags: python, jinja2

在使用模板之前，首先要有一个模板，最常见的就是网页模板，因为这是Jinja2的起源(我觉得在脚本生成和组织上肯定也能够有很好地应用)，详细的模板语法可以参考[模板设计者文档](http://www.pythonfan.org/docs/python/jinja2/zh/templates.html)。

Jinja2默认规定了几个分割符：

1. 变量插入分隔符：`{{ ... }}`，里面的内容会被直接插入到模板中；
2. 语句声明分隔符：`{% ... %}`，
3. 注释分隔符：`{# ... #}`

相关内容在*default.py*中可以看到，如果有必要，可以修改。

变量插入分隔符中的变量将会直接被插入到模板中，与最简单的模板功能一样。模板中的变量定义保留他们在Python中的对象属性，而块定义中则可以使用Jinja2的各种语句。

同样在*parser.py*中，可以看到支持的几个关键字：

```python
_statement_keywords = frozenset(['for', 'if', 'block', 'extends', 'print',
                                 'macro', 'include', 'from', 'import',
                                 'set'])
_compare_operators = frozenset(['eq', 'ne', 'lt', 'lteq', 'gt', 'gteq'])
```

## 1. 变量

jinja2中的变量在渲染时，可以将变量的属性同时传递进来，并且可以和python中一样使用`.`来访问属性，或者也可以用下标的方式来访问属性，如：

```django
{{ foo.bar }}
{{ foo['bar'] }}
```

上面两种方式执行的效果是一样的，这很容易带来困惑，因为这种做法将变量的属性和一个字典变量混合了，而在jinja2内部中，这两种方式的区别仅仅是首先寻找属性还是首先寻找项，为了区分真正的属性，jinja2提供一个`attr()`的过滤器，该过滤器只会取出变量的属性，而会忽略同名的项。如：

```django
{{ foo | attr("bar") }}
{{ foo['bar'] }}
```

上面两种方式则有些区别，第一种只会寻找`foo`的`bar`，而第二种则先寻找`foo['bar']`，如果找不到，再寻找`foo`的`bar`属性。关于过滤器的内容见下文。

jinja2中可以通过`set`来定义一个变量，该变量可以是数值、字符串、列表、元组、字典等。

## 2. 逻辑控制

如使用for关键字可以对一个可遍历的对象进行遍历，其用法为：

```django
{% for item in items %}
# do something
{% else %}
# do something else
{% endfor %}
```

真实操作后可以发现，在`{% for item in items %}`的后面和`{% endfor %}`的前面会多加一个空行，可以通过在`%`前加`-`去掉，如：`{% for item in items -%}`以及`{%- endfor %}`

Jinja2中，可以在循环中使用一些特殊的对象，如：

* `loop.index` : 当前循环迭代的次数，从1开始
* `loop.index0` : 当前循环迭代的次数，从0开始
* `loop.revindex` : 到当前循环结束需要迭代的次数，从1开始
* `loop.revindex0` : 到当前循环结束需要迭代的次数，从0开始
* `loop.first` : 是否是第一次迭代
* `loop.last` : 是否是最后一次迭代
* `loop.length` : 序列中的项目数
* `loop.cycle` : 使用辅助函数取值

在默认的for语句中，并没有`continue`、`break`等循环控制，可以使用开启扩展来支持该功能，如：

```django
{% for item in items -%}
    {% if loop.first -%}
        {% continue %}
    {% endif -%}
    {% if loop.index == 5 -%}
        {% break %}
    {% endif -%}
    {{ item -}}
{% endfor %}
```

使用过程为：

```python
from jinja2 import FileSystemLoader,Environment
env = Environment(loader=FileSystemLoader('template'),extensions=['jinja2.ext.loopcontrols'])
template = env.get_template('template.txt')
items = [1,2,3,4,5,6,7]
print template.render(items=items)
```

有关其他的内置扩展，以及扩展的编写参考[扩展说明文档](http://www.pythonfan.org/docs/python/jinja2/zh/extensions.html#module-jinja2.ext)

## 3. 宏

宏可以看做是程序语言中的函数，使用`macor`关键字定义一个宏：

```django
{% macro foo(*args, **kwargs) %}
	# do something
{% endmacro %}
```

在宏的内部，可以使用以下变量：

* varargs : 该变量以列表形式保存超出宏定义中位置参数个数之外的参数
* kwargs : 该变量以字典形式保存未使用的关键字参数
* caller : 该变量表示以call关键字被调用时，调用对象中的内容

同时，宏对象也提供一些属性：

* name : 宏的名称
* arguments : 宏接受的参数名的元组
* defaults : 默认值的元组
* catch_kwargs : 是否有未使用的关键字参数
* catch_varargs : 是否有超出个数的位置参数
* caller : 是否有caller变量，且由call标签调用

以下是两种调用方式的例子：

```django
{% macro with_caller(string) -%}
    get a {{ string }},
    This is a caller {{ caller() }}
{%- endmacro %}
{% call with_caller('hello') %}
    from caller
    {{ with_caller.caller }}
{%- endcall %}

{% macro without_caller(string) -%}
    This is a {{ string }}
{%- endmacro %}
{{ without_caller('hello') }}
{{ without_caller.caller }}
```

## 4. 过滤器

过滤器很像是**nix*中的管道，可以快速地对变量进行一些格式化操作，如对一个句子的每个单词首字母大写：`{{ title|title }}`，其过程为返回`title(title)`的值，外面的`title`对应`|`之后的`title`，可以看到过滤器的名字不会和变量名冲突。

另外一种过滤器的用法：

```django
{% filter title -%}
{{ title|lower }}
{%- endfilter %}
```

以语句方式的过滤器会在最后执行，上面的过程和`title|lower|title`表示相同的含义。

过滤器中接受参数，当过滤器使用参数时，被过滤的内容会作为第二个参数传入，如：`{{ ['a','b','c']|join(',') }}`表示的内容为`a,b,c`。

Jinja2内置了很多过滤器，详细可以参考[内置过滤器清单](http://www.pythonfan.org/docs/python/jinja2/zh/templates.html#builtin-filters)，使用过滤器可以对字符串做出快速的格式化操作，在模板的层面就可以保证一些字符串格式的一致性。

## 5. 测试

jinja2中可以使用`if`和`is`进行测试，测试的含义是`if`后面的变量是否能够使`is`后面的函数返回`True`，如：

```django
{% if value is defined %}
# do something with value
{% end if %}
```

测试的作用和过滤器比较类似，但是测试决定的是**是**与**否**，而过滤器则是将一个变量通过若干的运算变成新的变量来完成格式化。

更多的测试可以参考[内置测试清单](http://www.pythonfan.org/docs/python/jinja2/zh/templates.html#builtin-tests)

## 6. 模板组合

和编程语言一样，jinja2的模板语言也提供两种代码复用方式：组合、继承。jinja2中有两种组合的方式：

* 使用`include`语句可以将一个模板包含到当前的模板中，并且返回该模板的渲染结果
* 使用`import`语句可以将一个模板导入到当前模板中，使得当前模板可以使用被导入模板中的内容，如宏以及一些通过`set`关键字设定的变量等

`include`更像是将两个模板文件拼接在一起，并且默认可以访问到当前的一些变量，如：

有一个*small.txt*模板：

```django
this is {{ value }} in small.txt
```

另外一个*big.txt*模板：

```django
this is in big.txt
{% set values = [1,2,3] %}
{% for value in values -%}
    {% include 'small.txt' %}
{% endfor %}
```
从jinja2.1开始，被包含的模板可以访问模板定义中的变量。

而`import`则和Python中的`import`行为类似，将一个模板中的一些可复用的宏导入到当前模板中，并且被当前模板使用。如：

```django
{% import 'macros.txt' as macro %}
# do something using function in macro
{% from 'macros.txt' import foo as footest %}
# do something
```

而名称以一个或者更多的下划线开头的宏和变量是私有的，不能被导入。

包含和导入在某个角度看，更加像是一个相反的过程：包含用当前的名称空间对被包含的模板进行渲染并且拼接，而导入则是使得当前模板可以使用被导入模板中定义的宏和变量，而被导入模板中的其他内容则不会被当前模板拼接。

包含默认会传递上下文，而导入不会，这一点可以通过`with context`或者`without context`来改变，如：`{% from 'macros.txt' import foo with context %}`或者`{% include 'header.txt' without context %}`

## 7. 模板继承

模板继承指的是子模板继承基本模板的骨架，并且针对更加特殊的情况，对骨架进行填充。而需要被填充的内容，则由`block`标签定义，没有在`block`中的内容会原样复制到子模板中。

如： 

有一个*base.txt*模板：

```django
this is from base
{% block head -%}
    block content from base
    {% block inside -%}
    {%- endblock %}
{%- endblock %}
end string from base
```

另外一个*sub.txt*模板：

```django
{% extends 'base.txt' %}
{% block head -%}
    {{ super() }}
{%- endblock %}
{% block inside -%}
    this is from sub.txt
{%- endblock %}
```

*sub.txt*渲染的结果为：

```
this is from base
block content from base
    this is from sub.txt
end string from base
```

注意，在*sub.txt*中，每一个`block`都应该是独立填充的，并且不应该有重复填充，如果使用沿用*base.txt*中的嵌套方式，则会发生重复填充的情况，如：

*sub.txt*中内容为：

```django
{% extends 'base.txt' %}
{% block head -%}
    {{ super() }}
    {% block inside -%}
        this is from sub.txt
    {%- endblock %}
{%- endblock %}
```

则渲染的结果为

```
this is from base
block content from base
    this is from sub.txt
    this is from sub.txt
end string from base
```

在继承的过程中，由于`super()`已经包含了一个`inside`块，所以`head`块被继承为具有两个`inside`块的块。

上面的过程中，使用到了`super()`，即调用父级块定义内容，类似地，`self`则表示当前模块对象，`self.header()`可以在当前位置调用`header`块。

注意*base.txt*决定了整个骨架，*sub.txt*只是对其中的一些`block`定义，但是不能改变整个骨架，就像是用*sub.txt*中定义的`block`来填充*base.txt*一样。如果一些块没有被重新定义，则使用父级中的块定义。

如果父级模板中使用了嵌套块，如：

```django
{% for item in items %}
	{% block loop_item %}
		{{ item }}
	{% endblock %}
{% endfor %}
```

上述写法并不能有任何作用，原因是`loop_item`默认情况下不能访问作用域外的变量，理由是如果该块被子模板替换，则该变量无法被传递。Jinja2.2开始，可以通过`scoped`修饰词将块设定到作用域中，如：

```django
{% for item in items %}
	{% block loop_item scoped %}
		{{ item }}
	{% endblock %}
{% endfor %}
```

