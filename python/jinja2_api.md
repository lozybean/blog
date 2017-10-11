Title: Jinja2初探之api
Date: 2015-10-18
Category: Learning
Tags: python, jinja2

## 加载模板

最简单的模板使用方式是从字符串中导入模板，如：

```python
from jinja2 import Template
template = Template("this is a {{ value }}")
text = template.render(value='hello')
print text
print text.__class__
```

可以看到模板的使用分为两个部分，首先要加载，然后对其进行渲染，渲染其实就是对模板中的一些变量进行赋值，和自带的模板不同，这里的变量赋值传递的是变量对象本身，而不仅仅是一个字符串。如果对模板中未定义的变量进行赋值，则会直接忽略；如果模板中有未赋值的变量，则默认为空（但是要注意空行）。

同时注意到，渲染后的字符串类型是`unicode`，不仅返回的是unicode，在渲染时，传递的对象也需要是`unicode`（向下兼容`ASCII`），并且需要使用UNIX的换行符`\n`，而事实上，`\n`、`\r`、`\r\n`都会在渲染后变成`\n`，并且如果最后一个字符为`\n`则会默认忽略：

```python
template = Template("\r \n \r\n \n\r \n")
text = template.render()
for c in text:
	print ord(c)
```

结果为：

```
10
32
10
32
10
32
10
10
32
```

在很多情况下，模板渲染的结果需要输出到文件中，则使用`stream`方法创建`TemplateStream()`对象，调用其`dump`方法是一种比较好的方式，并且该对象提供了生成器方法，用以一些较大模板的渲染：

```python
template.stream(value='hello').dump('out.txt')
```

## 更好的方式

一个模板往往存在与模板文件中，如果使用`Template`类直接去构造模板对象，则应该先读取模板文件的内容，并且进行一些IO上的处理，而这些处理jiaja2的`Loader`加载器已经帮我们完成了，在*loader.py*源码文件中，可以看到诸如下面的类：`FileSystemLoader`、`PackageLoader`、`DictLoader`、`FunctionLoader`等等，这些类的工作是提供一个模板名字和模板文件真实路径的对应关系，加载文件时的IO操作等等，也可以编写自己的加载器，只需要继承`BaseLoader`类，并且实现其`get_source()`方法，具体实现方式可以参考源码中其他加载器的实现方式。

更加好的模板加载方式，是通过`Environment`类来构建一个模板，`Environment()`对象是一个环境控制的对象，说白了就是将jinja2里面的各种功能组合在一起，通过`Environment()`对象可以使用加载器加载模板，也可以使用扩展，添加过滤器等等，使用该对象来管理环境内容（如各种标识符，是否使用行语句等）的同时，可以将各个功能更好地结合在一起，如使用一个加载器来加载模板：

```python
from jinja2 import FileSystemLoader,Environment
env = Environment(loader=FileSystemLoader('template'))
template = env.get_template('one_template.txt')
print template.render(value='some value')
```

## 使用扩展

当已经有一个`Environment()`对象之后，就可以把jinja2里面的功能添加上去，扩展便是其中一种，如添加循环控制扩展：

```python
from jinja2 import Environment,FileSystemLoader
env = Environment(loader=FileSystemLoader('template'),
					  extension=['jinja2.ext.loopcontrols'])
```

或者在已有对象上添加扩展：

```python
env.add_extension('jinja2.ext.loopcontrols')
```

个人感觉第二种方式更好，关于jinja2中自带的一些扩展参考[扩展说明文档](http://www.pythonfan.org/docs/python/jinja2/zh/extensions.html)

编写自己的扩展时，应该继承`jinja2.ext.Extension`基类，具体实现参考[模板接口说明](http://www.pythonfan.org/docs/python/jinja2/zh/extensions.html#module-jinja2.ext)

## 自定义过滤器

过滤器的本质是一个函数，该函数至少接收一个位置参数，如：

```python
def datetimeformat(value, format='%H:%M / %d-%m-%y'):
	return value.strftime(format)
```

该函数对一个datetime对象进行格式化，在模板中通过过滤器即可快速实现时间格式的统一，使用自定义的过滤器需要在环境对象中添加绑定：

```python
env.filters['datetimeformat'] = datetimeformat
```

绑定定义的函数到名为`datetimeformat`的过滤器上，此时即可在解析模板中的同名过滤器。

为了在过滤器中获取上下文环境，jinja2提供三个装饰器：

* `@environmentfilter` : 传递环境变量为第一个参数，可以在过滤器中获取环境内容
* `@contextfilter` : 这种过滤器一般接受通用参数`(*args, **kwargs)` ,并通过检查参数的名字做出更灵活的操作，如`map`过滤器
* `@evalcontextfilter` : 传递上下文为第一个参数，通过识别该上下文是否开启自动转义操作，进行更加安全地操作

## 自定义测试

测试和过滤器一样是一个函数，但是测试更加简单，因为其返回值只能是`True`或者`False`，测试往往用来做类型和一致性的检查，所以不能像过滤器一样获取环境以及上下文。如文档中的一个例子：

```python
import math
def is_prime(n):
	if n == 2:
		return True
	for i in xrange(2, int(math.ceil(math.sqrt(n))) + 1):
		if n % i == 0:
			return False
	return True
```

并且通过环境添加该测试的绑定：

```python
env.tests['prime'] = is_prime
```
