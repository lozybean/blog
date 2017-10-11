Title: Python3元编程技巧
Date: 2016-10-08
Category: pages
Tags: python, metaprogramming


本文参考[David Beazley 13年pycon的演讲](https://www.youtube.com/watch?time_continue=1&v=sPiWg5jSoZI)整理. 
大师写了35年的程序, 依然对新技术保持高度热情, 走在技术的最前面. 甚至现在好多年轻人都执着于py2, 迟迟不肯学习py3, 惧怕新技术带来挑战, 想想真是不应该.
大师是我辈的楷模, 感谢大师的精彩演讲!

## 什么是元编程

元(Meta)又译为后设, 简单来说就是"关于什么的什么", "超越什么的什么", 具体可以参考[wiki百科上的解读](https://zh.wikipedia.org/wiki/%E5%BE%8C%E8%A8%AD).

元编程就是"描述编程的编程", 即操作代码的代码.

## DRY

DRY(don't repeat yourself)是元编程最初的动机, 为了避免重复代码, 就需要通过一些手段进行代码复用.

为什么不要编写重复代码:

1. 高度重复的代码很差劲;
2. 编写这种代码很沉闷(到处复制粘贴);
3. 代码结构混乱难以阅读;
4. 维护时也需要到处修改, 难以维护;

通过元编程, 可以避免写重复代码, 并且更重要的是, 元编程很好玩, 她令编程充满乐趣.

## 装饰器

### debugging

我们经常用一些print语句来进行debugging, 如:

```python
def add(x, y):
    print('add')
    return x + y
```

但是当有很多函数需要debugging的时候, 看起来就会像是这样:

```python
def add(x, y):
    print('add'):
    return x + y

def sub(x, y):
    print('sub')
    return x - y

def mul(x, y):
    print('mul')
    return x * y

def div(x, y):
    print('div')
    return x / y
```

我们使用了四条雷同的print语句, 这就是重复代码, 一个简单的方式去除这些重复代码, 就是使用装饰器(decorator).

装饰器就是一个返回另外一个函数的封装的函数, 这个返回的函数封装在原来的函数基础上进行一些额外的修饰语句.

### 使用装饰器

下面是一个使用装饰器的例子:

```python
from functools import wraps

def debug(func):
    msg = func.__qualname__
    @wraps(func)
    def wrapper(*args, **kwargs):
        print(msg)
        return func(*args, **kwargs)
    return wrapper
```

上面的`debug`函数中, 最终返回的`wrapper`就是对`func`的封装, 并且在func运行之前首先打印函数的名字(就像上文中重复的print语句一样).

接下来我们就通过`@`符号来使用装饰器, 将之前的四个函数改写为:

```python
@debug
def add(x, y):
    return x + y

@debug
def sub(x, y):
    return x - y

@debug
def mul(x, y):
    return x * y

@debug
def div(x ,y):
    return x / y
```

虽然看起来代码量没有减少, 但是程序的逻辑却发生很大的变化: 

1. debugging代码被孤立到单独的位置;
2. 易于修改debugging, 或者取消debug;
3. 被装饰的函数不需要考虑debug的细节;

### 函数元信息

`functools.wraps`将`func`的元信息拷贝出来, 并将其替换到`wrapper`函数中. 如果不这样做, 被修饰后的函数就不会继续保留原来的元信息:

```
## 如果没有@wraps(func)这一行:
>>> add.__qualname__
'debug.<locals>.wrapper'

## 加上@wraps(func)这一行后:
>>> add.__qualname__
'add'
```

可以看到, 保留了被修饰函数的元信息后, 整个函数看上去就和原来一样.

### 带参数的装饰器

在debug的时候, 能打印一些前缀信息就好了:

```python
def add(x, y):
    print('***add')
    return x + y
```

这就需要用到带参数的装饰器, 看起来是这样的:

```python
from functools import wraps

def debug(prefix=''):
    def decorate(func):
        msg = prefix + func.__qualname__
        @wraps(func)
        def wrapper(*args, **kwargs):
            print(msg)
            return func(*args, **kwargs)
        return wrapper
    return decorate
```

用起来是这样的:

```python
@debug(prefix='xxx')
def add(x, y):
    return x + y
```

这个装饰器有点难看, 可以通过`functools.partial`重写:

```python
from functools import wraps, partial

def debug(func=None, prefix=''):
    if func is None:
        return partial(debug, prefix=prefix)

    msg = prefix + func.__qualname__
    @wraps(func)
    def wrapper(*args, **kwrags):
        print(msg)
        return func(*args, **kwargs)
    return wrapper
```

`functools.partial`函数可以生成固定某个参数的子函数, 由于被装饰的函数永远作为最后一个参数传入, 作为装饰器的`debug`的第一个参数`func`永远是`None`, 但是返回的`partail(debug, prefix=prefix)`相当于只有一个参数`func`的函数, 就正常完成装饰工作.

### 类装饰器

当需要给一个类里所有的方法都加debug时, 看起来会像是这样:

```python
class Spam:
    @debug
    def grok(self):
        pass

    @debug
    def bar(self):
        pass

    @debug
    def foo(self):
        pass
```

我们又看到了重复性代码, 在每个方法前面都加了一行装饰, 虽然少, 但是秉着**DRY**的原则, 我们可以用类装饰器来统一给所有的方法添加装饰:

```python
def debugmethods(cls):
    for name, val in vars(cls).items():
        if callable(val):
            setattr(cls, name, debug(val))
    return cls
```

以上代码的思路是:

1. 遍历类字典中所有的对象;
2. 辨别是否callable;
3. 将callable的对象添加装饰;

用起来是这样的:

```python
@debugmethods
class Spam:
    def grok(self):
        pass
    def bar(self):
        pass
    def foo(self):
        pass
```

仅仅装饰一次, 就给类里所有的方法添加了装饰. 

这种方法有一个限制: 仅对实例方法有效, 对静态方法和类方法无效. 其原因是: 在类定义完成之前, 不能创建类的引用. 所以在类定义完成之前, 类方法和静态方法都不是callable的.

### 访问debug

为了在每次访问对象属性时进行debug, 需要重写类:

```python
def debugattr(cls):
    orig_getattribute = cls.__getattribute__

    def __getattribute__(self, name):
        print('Get:', name)
        return orig_getattribute(self, name)
    cls.__getattribute__ = __getattribute__

    return cls
```

此时, 若对一个类进行上述装饰, 其效果会是这样的:

```python
class Point:
    def __init__(self, x, y):
        self.x = x
        self.y = y

>>> p = Point(2, 3)
>>> p.x
Get: x
2
>>> p.y
Get: y
3
>>>
```

## 元类(MetaClass)

### 创建类的类

设想现在有一系列的类需要被装饰, 大概是这样子:

```python
@debugmethods
class Base:
    pass

@debugmethods
class Spam(Base):
    pass

@debugmethods
class Grok(Base):
    pass

@debugmethods
class Mondo(Base):
    pass
```

好吧, 重复性代码又出现了, 我们需要一个元类(metaclass)来解决.

```python
class debugmeta(type):
    def __new__(cls, clsname, bases, clsdict):
        clsobj = super().__new__(cls, clsname,
                                 bases, clsdict)
        clsobj = debugmethods(clsobj)
        return clsobj

class Base(metaclass=debugmeta):
    pass

class Spam(Base):
    pass
```

在上述元类的定义中, 首先使用super().__new__(cls, clsname, bases, clsdict)语句按照一般方式创建类, 然后在通过下一个clsobj = debugmethods(clsobj)语句来装饰该类.

可以看出, 元类实际上控制的是类的创建过程.

### 创建类时发生了什么

所有的对象都有类/类型, 如:

```
>>> type(3)
<class 'int'>
>>> type('xxx')
<class 'str'>
>>> type(2.1)
<class 'float'>
```

类/类型同样也是对象, 所以也有类型, 类/类型的类型是`type`类:

```
>>> class A:
...     pass
...
>>> type(A)
<class 'type'>
>>> type(type)
<class 'type'>
>>> type
<class 'type'>
```

当有一个如下的类:

```python
class Spam(Base):
    def __init__(self, name):
        self.name = name

    def bar(self):
        print("I'm Spam.bar")
```

由三个部分组成:

1. 类名: Spam
2. 父类: Base
3. 方法(类主体): __init__, bar

当这个类被定义时, 其内部依次进行下面几步工作:

1.分离类主体:

```python
body = '''
def __init__(self, name):
    self.name = name

def bar(self):
    print("I'm Spam.bar")
'''
```
2.创建类字典, 默认只是创建一个一般的字典, 作为类在主体声明时的本地命名空间;

```python
clsdict = type.__prepare__('Spam', (Base, ))

>>> clsdict
{}
```

3.运行类主体, 并返回更新后的字典:

```python
exec(body, globals(), clsdict)
```

返回的更新后字典就会将本体中的内容添加进来:

```python
>>> clsdict
{'__init__': <function __main__.__init__>, 'bar': <function __main__.bar>}
```

4.使用类名、父类名、类字典三个部分构造类:

```python
Spam = type('Spam', (Base,), clsdict)

>>> Spam
__main__.Spam
>>> s = Spam('Guido')
>>> s.name
Guido
>>> s.bar()
I'm Spam.bar
```

### 元类干了啥

上文中提到, 元类控制了类的创建过程, 也就是"创建类的类". 

在上述类建立过程的描述中, 默认的元类就是`type`类, Spam类定义时, 相当于:

```python
class Spam(metaclass=type):
    pass
```

指定不同于type的其他类作为元类, 就可以修改类的创建方式, 如上文中的`debugmeta`:

```python
class debugmeta(type):
    def __new__(cls, clsname, bases, clsdict):
        clsobj = super().__new__(cls, clsname,
                                 bases, clsdict)
        clsobj = debugmethods(clsobj)
        return clsobj
```

此时, 若Spam的元类指定为debugmeta: class Spam(metaclass=debugmeta), 则在创建的第四步, 发生的将会是:

```
Spam = debugmeta('Spam', (Base,), clsdict)
```

于是, 创建出的类就自动加上了`debugmethods`的类装饰.

### 为什么要用元类

元类只是在创建类的过程中添加了类修饰, 其本质上和类修饰是差不多的, 但是元类可以通过继承, 就像遗传性变异一样, 影响所有的子类(而不需要在所有的类前面添加类装饰).

在本节开始之前, 使用元类优化后的将会变成这样:

```python
class Base(metaclass=debugmeta):
    pass

class Spam(Base):
    pass

class Grok(Base):
    pass

class Mondo(Base):
    pass
```

### 结构问题

设想现在有以下几个类:

```
class Stock:
    def __init__(self, name, share, price):
        self.name = name
        self.share = share
        self.price = price

class Point:
    def __init__(self, x, y):
        self.x = x
        self.y = y

class Host:
    def __init__(self, address, port):
        self.address = address
        self.port = port
```

如果你时刻记得**DRY**原则的话, 上述的代码看起来会很难看: 每个init函数里面的内容都是雷同的.

继承一个Structure类可以消除这些重复性代码:

```python
class Structure:
    _fields = []
    def __init__(self, *args):
        if len(args) != self._fields:
            raise TypeError('Wrong # args')
        for name, val in zip(self._fields, args):
            setattr(self, name, val)

class Stock(Structure):
    _fields = ['name', 'share', 'price']

class Point(Structure):
    _fields = ['x', 'y']

class Host(Structure):
    _fields = ['address', 'port']
```

看起来可以用:

```python
>>> s = Stock('ACME', 5, 123.45)
>>> s.name
'ACME'
>>> s.shares
50
>>> s.price
123.45
```

但是其实存在很多问题:

* 不支持键值参数

```
>>> s = Stock('ACME', price=123.45, shares=50)
Traceback (most recent call last):
  File "<stdin>", line 1, in <module>
TypeError: __init__() got an unexpected keyword argument 'shares'
```

* 缺失运行时签名:

```
>>> import inspect
>>> inspect.signature(Stock)
<Signature (*args)>
```

签名中只有*args, 而不是具体的属性名称.

### 加上签名

创建签名:

```python
from inspect import Parameter, Signature

fields = ['name', 'shares', 'price']
params = [Parameter(name,
                    Parameter.POSITIONAL_OR_KEYWORD)
                    for name in fields]
sig = Signature(params)
```

签名并不只是一些元数据, 而是封装了这些内容的具体对象.

绑定签名:

```python
def func(*args, **kwargs):
    bound_args = sig.bind(*args, **kwrags)
    for name, val in bound_args.arguments.items():
        print(name, '=', val)
```

通过`sig.bind`方法将位置参数和键值参数绑定到sig对象中, 其返回的`bound_args.arguments`对象为包含传入参数的OrderedDict.

使用起来就像这样:

```
>>> func('ACME', 50, 91.1)
name = ACME
shares = 50
price = 91.1
>>> func('ACME', price=91.1, shares=50)
name = ACME
shares = 50
price = 91.1
```

可以看出, 绑定后就支持了键值参数和位置参数.

看起来可以用.

### 使用签名重写

利用签名来解决Structrue中存在的问题:

```python
from inspect import Parameter, Signature

def make_signature(names):
    return Signature(Parameter(name, 
                               Parameter.POSITIONAL_OR_KEYWORD)
                     for name in names)

class Structure:
    __signatue__ = make_signature([])
    def __init__(self, *args, **kwargs):
        bound = self.__signature__.bind(*args, **kwargs)
        for name, val in bound.arguments.items():
            setattr(self, name, val)
```

使用改写后的Structure:

```python
class Stock(Structure):
    __signature__ = make_signature(['name', 'shares', 'price'])

class Point(Structure):
    __signature__ = make_signature(['x', 'y'])

class Host(Structure):
    __signature__ = make_signature(['address', 'port'])
```

之前的问题不复存在:

```
>>> s = Stock('ACME', shares=50, price=91.1)
>>> s.name
'ACME'
>>> s.shares
50
>>> s.price
91.1
>>> import inspect
>>> print(inspect.signature(Stock))
(name, shares, price)
```

但是上面的实现依然充满着重复性代码, 别着急, 正好可以用之前学到的类装饰器和元类来优化一下.

### 使用类装饰器优化

首先要创建一个带参数的类装饰器, 将类中的属性作为参数:

```python
def add_signature(*names):
    def decorate(cls):
        cls.__signature__ = make_signature(names)
        return cls
    return decorate
```

用起来看上去是这样的:

```python
@add_signature('name', 'shares', 'price')
class Stock(Structure):
    pass

@add_signature('x', 'y'):
class Point(Structure):
    pass
```

类的属性放到了装饰器的参数中, 这样看起来非常奇怪, 很别扭.

### 使用元类优化

使用元类重写, 思路是讲__signature__作为类的属性, 在类构造的时候加入到类属性字典中.

```python
class StructMeta(type):
    def __new__(cls, name, bases, clsdict):
        clsobj = super().__new__(cls, name, bases, clsdict)
        sig = make_signature(clsobj._fields)
        setattr(clsobj, '__signature__', sig)
        return clsobj

class Structure(metaclass=StructMeta):
    fields = []
    def __init__(self, *args, **kwargs):
        bound = self.__signature__.bind(*args, **kwargs)
        for name, val in bound.arguments.items():
            setattr(self, name, val)
```

在元类中, 不像类装饰器那样使用装饰器的参数来创建签名, 而是通过类中的_fields列表来创建签名, 用起来是这样的:

```python
class Stock(Structure):
    _fields = ['name', 'shares', 'price']

class Point(Structure):
    _fields = ['x', 'y']

class Host(Structrue):
    _fields = ['address', 'port']
```

看起来和最开始使用的方法一样, 但是却解决了属性签名的问题, 可以愉快地使用位置参数和键值参数了.

### 使用建议

* 当要修改一些无关的类时, 使用类装饰器
* 当需要使用继承的特性时, 使用元类
* 不要太快否定某种技术
* 所有这些工具都注定要一起工作

## 描述器

### 使用属性

设想上述`Stock`类:

```
>>> s = Stock('ACME', 50, 91.1)
>>> s.name = 42
>>> s.shares = 'a heck of a lot'
>>> s.price = (23.45 + 2j)
```

这很荒唐, 名字不应该是数字, 价格也不可能是复数. 

使用类属性增加检查:

```python
class Stock(Structure):
    _fileds = ['name', 'shares', 'price']

    @property
    def shares(self):
        return self._shares

    @shares.setter
    def shares(self, value):
        if not isinstance(value, int):
            raise TypeError('expected int')
        if value < 0:
            raise TypeError('Must be >= 0')
        self._shares = value    
```

现在shares不能再是字符串了:

```python
>>> s = Stock('ACME', 50, 91.1)
>>> s.shares = 'a lot'
Traceback (most recent call last):
  File "<stdin>", line 1, in <module>
  File "<stdin>", line 9, in shares
TypeError: expected int
```

看起来解决了问题, 但是我们必须对每一个属性都添加相应的检查, 这势必带来一大波重复性代码.

### 描述器协议

属性实际上是通过描述器来实现的, 一个描述器看起来像是这样的:

```python
class Descriptor:
    def __get__(self, instance, cls):
        pass

    def __set__(self, instance, cls):
        pass

    def __delete__(self, instance, cls):
        pass
```

通过自定义描述器, 就可以改变属性的实现过程.

基本的描述器实现:

```python
class Descriptor:
    def __init__(self, name):
        self.name = name

    def __get__(self, instance, cls):
        print('getting...')
        if instance is None:
            return self
        else:
            return instance.__dict__[self.name]

    def __set__(self, instance, value):
        instance.__dict__[self.name] = value

    def __delete__(self, instance):
        raise AttributeError("Can't delete")
```

其中当简单返回`__dict__`中的内容时, `__get__`方法可以省略.

使用描述器来定义属性:

```python
class Stock(Structure):
    _fields = ['name', 'shares', 'price']
    name = Descriptor('name')
    shares = Descriptor('shares')
    price = Descriptor('price')


>>> s = Stock('ACME', 50, 91.1)
>>> s.share
getting...
50
>>> del s.share
Traceback (most recent call last):
  File "<stdin>", line 1, in <module>
  File "<stdin>", line 13, in __delete__
AttributeError: Can't delete
```

### 添加类型检查

使用描述器, 就可以在`__set__`方法中添加类型检查:

```python
class Typed(Descriptor):
    ty = object
    def __set__(self, instance, value):
        if not isinstance(value, self.ty):
            raise TypeError('Expected {}'.format(self.ty))
        super().__set__(instance, value)

class Integer(Typed):
    ty = int

class Float(Typed):
    ty = float

class String(Typed):
    ty = str
```

使用描述器进行类型检查:

```python
class Stock(Structure):
    _fields = ['name', 'shares', 'price']
    name = String('name')
    shares = Integer('shares')
    price = Float('price')


>>> s = Stock('ACME', 50, 91.1)
>>> s.share = 'a lot'
Traceback (most recent call last):
  File "<stdin>", line 1, in <module>
  File "<stdin>", line 5, in __set__
TypeError: Expected <class 'int'>
```

### 添加值检查

当然一个价格除了必须是数字之外, 还必须是非负的.

类似的, 可以通过在描述器中添加值检查, 并且使用mixin类, 联合类型检查:

```python
class Positive(Descriptor):
    def __set__(self, instance, value):
        if value < 0:
            raise ValueError('Expected >= 0')
        super().__set__(instance, value)

class PosInteger(Integer, Positive):
    pass

class PosFloat(Float, Positive):
    pass
```

使用新的描述器:

```python
class Stock(Structure):
    _fields = ['name', 'shares', 'price']
    name = String('name')
    shares = PosInteger('share')
    price = PosFloat('price')


>>> s = Stock('ACME', 50, 91.1)
>>> s.shares = -10
Traceback (most recent call last):
  File "<stdin>", line 1, in <module>
  File "<stdin>", line 6, in __set__
  File "<stdin>", line 4, in __set__
ValueError: Expected >= 0
```

### 理解MRO(方法解析顺序)

上面使用到了mixin类, 当一个类继承自多个类时, 就有解析顺序的问题, 如上文中的`PosInteger`类, 其MRO为:

```python
>>> PosInteger.__mro__
(<class '__main__.PosInteger'>, <class '__main__.Integer'>, <class '__main__.Typed'>, 
<class '__main__.Positive'>, <class '__main__.Descriptor'>, <class 'object'>)
```

对于一般的类而言, 方法按照MRO列表的顺序查找, 找到第一个方法之后调用.

`super()`将会返回MRO列表之后的类, 并调用该类的对应方法(注意: 不是父类).

更多关于MRO的内容, 可以参考[C3方法](https://en.wikipedia.org/wiki/C3_linearization)的说明.

### 添加长度检查

如果不想让名字长度无限的话, 可以添加一个大小限制的描述器:

```python
class Sized(Descriptor):
    def __init__(self, *args, maxlen, **kwargs):
        self.maxlen = maxlen
        super().__init__(*args, **kwargs)

    def __set__(self, instance, value):
        if len(value) > self.maxlen:
            raise ValueError('Too big')
        super().__set__(instance, value)

class SizedString(String, Sized):
    pass
```

上面使用了一个key-word only的参数技巧, 在*args后面的具名参数, 只能通过键值对的方式传入.

```python
class Stock(Structure):
    _fields = ['name', 'shares', 'price']
    name = SizedString('name', maxlen=8)
    shares = PosInteger('shares')
    price = PosFloat('price')


>>> s = Stock('ACME', 50, 91.1)
>>> s.name = 'ABCDEFGHIJKLMNOPQ'
Traceback (most recent call last):
  File "<stdin>", line 1, in <module>
  File "<stdin>", line 6, in __set__
  File "<stdin>", line 8, in __set__
ValueError: Too big
```

### 添加模式检查

利用正则表达式, 可以对字符串进行更加深入的模式检查:

```python
import re
class Regex(Descriptor):
    def __init__(self, *args, pat, **kwargs):
        self.pat = re.compile(pat)
        super().__init__(*args, **kwargs)
    
    def __set__(self, instance, value):
        if not self.pat.match(value):
            raise ValueError('Invalid string')
        super().__set__(instance, value)

class SizedRegexString(String, Sized, Regex):
    pass
```

比如限制姓名中只使用大写字母:

```python
class Stock(Structure):
    _fields = ['name', 'shares', 'price']
    name = SizedRegexString('name', maxlen=8, pat='[A-Z]+$')
    shares = PosInteger('shares')
    price = PosFloat('price')


>>> s = Stock('ACME', 50, 91.1)
>>> s.name = 'WOW!'
Traceback (most recent call last):
  File "<stdin>", line 1, in <module>
  File "<stdin>", line 6, in __set__
  File "<stdin>", line 9, in __set__
  File "<stdin>", line 8, in __set__
ValueError: Invalid string
```

在这里看到了一件非常神奇的事情, SizedRegexString('name', maxlen=8, pat='[A-Z]+$')可以一次性传入三个参数, 并且这三个参数分别被三个MRO上的类的`__init__`正确调用, 仅仅是用了key-word only参数的技巧, 每个`__init__`会取出自己感兴趣的key-word参数, 然后把剩余的参数继续传给下一个`__init__`, 这就是Python3, 简单直接又如此有效.

### 新的元类

我们做了非常有效的工作, 将各种类型检查通过简单明确描述器来定义, 但是还是有很多重复性代码, 如:

1. _fields看起来非常多余, 因为里面的内容都会在接下来进行定义;
2. 描述器中的name参数也是多余的, 因为我们一直使用属性名称作为name参数;

这一切, 都可以通过重写之前的元类可以完成:

```python
from collections import OrderedDict

class StructMeta(type):

    @classmethod
    def __prepare__(cls, name, bases):
        return NoDupOrderedDict()

    def __new__(cls, name, bases, clsdict):
        fields = [key for key, val in clsdict.items()
                  if isinstance(val, Descriptor)]
        for name in fields:
            clsdict[name].name = name
        clsobj = super().__new__(cls, name, bases, dict(clsdict))
        sig = make_signature(fields)
        setattr(clsobj, '__signature__', sig)
        return clsobj
```

然后从基本的`Descriptor`中把`__init__`函数删除(这个工作在上述代码的`__new__`中完成了).

现在我们可以*干净*地使用描述器了:

```python
class Stock(Structure):
    name = SizedRegexString(maxlen=8, pat='[A-Z]+$')
    shares = PosInteger()
    price = PosFloat()
```

可以看到, 我们在`__prepare__`中使用了`collections.OrderedDict`替代一般的`dict`来保持属性定义的顺序(这样可以保证在实例化的时候传入正确的位置参数). 更加深入地, 我们可以使用一个不允许重复内容的字典, 来防止属性的重复定义.

```python
class NoDupOrderedDict(OrderedDict):
    def __setitem__(self, key, value):
        if key in self:
            raise NameError('{} already defined'.format(key))
        super().__setitem__(key, value)


class StructMeta(type):
    @classmethod
    def __prepare__(cls, name, bases):
        return NoDupOrderedDict()

    // ...
```

现在如果重复定义属性, 则会提示错误:

```python
>>> class Stock(Structure):
        price = PosFloat()
        price = PosFloat()

Traceback (most recent call last):
  File "<stdin>", line 1, in <module>
  File "<stdin>", line 5, in Stock
  File "<stdin>", line 4, in __setitem__
NameError: price already defined
```

代码清单: [typely.py](https://gist.github.com/lozybean/431a8829e09bf5050c02a436360e6567#file-typely-py)

## 代码生成

### 性能瓶颈

性能就像是程序时间的货币, 用这些货币可以购买许多令人兴奋的东西, 如: 开发效率, 可维护性, 甚至仅仅是美观. 

我们买到了什么: 

1. 我们不需要再频繁在`__init__`中逐一传入参数了;
2. 同时我们还保留了参数签名, 可以正常使用两类参数(附带消耗);
3. 优雅地添加了各种参数的各种检查;
4. 消除了所有的重复代码, 现在使用起来令人愉悦;

看看我们的改变:

```python
// Simple
class Stock:
    def __init__(self, name, shares, price):
        self.name = name
        self.shares = shares
        self.price = price

// Meta
class Stock(Structure):
    name = SizedRegexString(maxlen=8, pat='[A-Z]+$')
    shares = PosInteger()
    price = PosFloat()
```

在我Python 3.5.0的环境下, 使用line_profiler进行性能检查, 得到结果如下:

Simple:

```
Wrote profile results to simple.py.lprof
Timer unit: 1e-06 s

Total time: 1.8e-05 s
File: simple.py
Function: test at line 10

Line #      Hits         Time  Per Hit   % Time  Line Contents
==============================================================
    10                                           @profile
    11                                           def test():
    12         1           12     12.0     66.7      s = Stock('ACME', 50, 91.1)
    13         1            3      3.0     16.7      s.price
    14         1            2      2.0     11.1      s.price = 10.0
    15         1            1      1.0      5.6      s.name = 'ACME'
```

Meta:

```
Wrote profile results to typely.py.lprof
Timer unit: 1e-06 s

Total time: 0.000238 s
File: typely.py
Function: test at line 117

Line #      Hits         Time  Per Hit   % Time  Line Contents
==============================================================
   117                                           @profile
   118                                           def test():
   119         1          200    200.0     84.0      s = Stock('ACME', 50, 91.1)
   120         1            3      3.0      1.3      s.price
   121         1           13     13.0      5.5      s.price = 10.0
   122         1           22     22.0      9.2      s.name = 'ACME'
```

性能看来消耗地有点多, 实例化耗时将近17倍(如果是python3.3的话, 这个效率还要低, 大概耗时400个单位, 是python3.5的两倍), 属性设置也在6-22倍(取决于验证的复杂性). 而至于s.price消耗时间相同这个"bright spot"也只是因为我们的描述器中没有对`__get__`进行重写.

除了类型检查等功能性增加所消耗的性能有价值之外, 附带的消耗成为了性能的瓶颈:

1. 使用了签名;
2. 描述器中复杂的继承关系;

### 代码生成器

为了避免以上的瓶颈, 我们首先要做一些看起来不怎么好的动作:

```python
def _make_init(fields):
    code = 'def __init__(self, {}):\n'.format(', '.join(fields))
    for name in fields:
        code += '    self.{name} = {name}\n'.format(name=name)
    return code


>>> code = _make_init(['name', 'shares', 'price'])
>>> print(code)
def __init__(self, name, shares, price):
    self.name = name
    self.shares = shares
    self.price = price
```

我们使用一个函数来生成代码字符串, 然后只要在元类中执行这段代码, 就可以避免签名的问题:

```python
class StructMeta(type):

    @classmethod
    def __prepare__(cls, name, bases):
        return NoDupOrderedDict()

    def __new__(cls, name, bases, clsdict):
        fields = [key for key, val in clsdict.items()
                  if isinstance(val, Descriptor)]
        for name in fields:
            clsdict[name].name = name
        if fields:
            exec(_make_init(fields), globals(), clsdict)
        clsobj = super().__new__(cls, name, bases, dict(clsdict))
        return clsobj
```

使用新的方式的性能结果:

```
Wrote profile results to execly_init.py.lprof
Timer unit: 1e-06 s

Total time: 0.000108 s
File: execly_init.py
Function: test at line 117

Line #      Hits         Time  Per Hit   % Time  Line Contents
==============================================================
   117                                           @profile
   118                                           def test():
   119         1           72     72.0     66.7      s = Stock('ACME', 50, 91.1)
   120         1            3      3.0      2.8      s.price
   121         1           13     13.0     12.0      s.price = 10.0
   122         1           20     20.0     18.5      s.name = 'ACME'
```

可以看到, 实例化效率是之前的将近3倍. 虽然这个做法似乎不怎么漂亮, 但是带来的性能提升是实实在在的, 通过代码生成器直接形成逻辑, 避免了使用签名带来的性能瓶颈.

### 进一步改良

既然如此, 能否用代码生成器来解决描述器的继承关系带来的性能瓶颈呢? 到目前为止, 描述器的`__set__`方法都先有一段检查代码, 然后都使用`super()`来调用MRO列表中的下一个类的`__set__`方法.

```python
class Descriptor:
    ...
    def __set__(self, instance, value):
        instance.__dict__[self.name] = value

class Typed(Descriptor):
    ty = object
    def __set__(self, instance, value):
        if not isinstance(value, self.ty):
            raise TypeError('Expected {}'.format(self.ty))
        super().__set__(instance, value)

class Positive(Descriptor):
    def __set__(self, instance, value):
        if value < 0:
            raise ValueError('Expected >= 0')
        super().__set__(instance, value)
```

首先实现`__set__`方法中的实现代码:

```python
class Descriptor:
    ...
    @staticmethod
    def set_code():
        return [
            'instance.__dict__[self.name] = value'
        ]

class Typed(Descriptor):
    ty = object

    @staticmethod
    def set_code():
        return [
            'print("this is Typed.__set__")',
            'if not isinstance(value, self.ty):',
            '    raise TypeError("Expected {}".format(self.ty))',
        ]

class Positive(Descriptor):
    @staticmethod
    def set_code():
        return [
        'print("this is Positive.__set__")',
        'if value < 0:',
        '    raise ValueError("Expected >= 0")',
        ]
```

所有的描述器都通过上述示例来修改后, 我们就可以利用MRO来生成最终`__set__`方法:

```python
def _make_setter(dcls):
    code = 'def __set__(self, instance, value):\n'
    for d in dcls.__mro__:
        if 'set_code' in d.__dict__:
            for line in d.set_code():
                code += '    ' + line + '\n'
    return code
```

遍历一个类的MRO, 然后其中类的`__set__`方法拼接起来, 作为最终的`__set__`方法, 我们最后要做的就只是把这个过程加入到元类中:

```python
class DescriptorMeta(type):
    def __init__(self, clsname, bases, clsdict):
        if '__set__' not in clsdict:
            code = _make_setter(self)
            exec(code, globals(), clsdict)
            setattr(self, '__set__',
                    clsdict['__set__'])
        else:
            raise TypeError('Define set_code()')

class Descriptor(metaclass=DescriptorMeta):
    ...
```

该元类中首先按照之前的逻辑生成最终`__set__`方法代码, 然后使用`exec`执行, 并且将执行后的结果设置为`self.__set__`方法.

看起来和python干的工作是一样的, 再使用line_profiler进行性能检查:

```
Wrote profile results to execly.py.lprof
Timer unit: 1e-06 s

Total time: 7.3e-05 s
File: execly.py
Function: test at line 154

Line #      Hits         Time  Per Hit   % Time  Line Contents
==============================================================
   154                                           @profile
   155                                           def test():
   156         1           49     49.0     67.1      s = Stock('ACME', 50, 91.1)
   157         1            3      3.0      4.1      s.price
   158         1            8      8.0     11.0      s.price = 10.0
   159         1           13     13.0     17.8      s.name = 'ACME'
```

属性设置的效率是之前的2倍, 实例化也更快了(提升30%左右), 代码生成器确实对效率有很大改善.

代码清单: [execly.py](https://gist.github.com/lozybean/431a8829e09bf5050c02a436360e6567#file-execly-py)

## 接下来的问题

### 使用XML管理代码

目前定义类的代码为:

```python
class Stock(Structure):
    name = SizedRegexString(maxlen=8, pat='[A-Z]+$')
    shares = PosInteger()
    price = PosFloat()

class Point(Structure):
    x = Integer()
    y = Integer()

class Address(Structure):
    hostname = String()
    port = Integer()
```

干得漂亮, 看上去很干净. 但是如果想隐藏代码细节, 就可以使用XML来管理这些代码:

```xml
<structures>
  <structure name="Stock">
    <field type="SizedRegexString" maxlen="8" pat="'[A-Z]+$'">name</field>
    <field type="PosInteger">shares</field>
    <field type="PosFloat">price</field>
  </structure>

  <structure name="Point">
    <field type="Integer">x</field>
    <field type="Integer">y</field>
  </structure>

 <structure name="Address">
   <field type="String">hostname</field>
   <field type="Integer">port</field>
 </structure>
</structures>
```

使用下面的方法来解析XML文档, 将之前的元类和描述器保存为typestruct模块, XML文档使用datadefs.xml保存:

```python
from xml.etree.ElementTree import parse

def _xml_to_code(filename):
    doc = parse(filename)
    code = 'import typestruct as _ts\n'
    for st in doc.findall('structure'):
        code += _xml_struct_code(st)
    return code

def _xml_struct_code(st):
    stname = st.get('name')
    code = 'class %s(_ts.Structure):\n' % stname
    for field in st.findall('field'):
        name = field.text.strip()
        dtype = '_ts.' + field.get('type')
        kwargs = ', '.join('%s=%s' % (key, val)
                           for key, val in field.items()
                           if key != 'type')
        code += '    %s = %s(%s)\n' % (name, dtype, kwargs)
    return code
```

使用以上解析代码, 可以对XML文档正确解析:

```python
>>> code = _xml_to_code('datadefs.xml')
>>> print(code)
import typestruct as _ts
class Stock(_ts.Structure):
    name = _ts.SizedRegexString(maxlen=8, pat='[A-Z]+$')
    shares = _ts.PosInteger()
    price = _ts.PosFloat()
class Point(_ts.Structure):
    x = _ts.Integer()
    y = _ts.Integer()
class Address(_ts.Structure):
    hostname = _ts.String()
    port = _ts.Integer()

```

### 导入器

python在带入模块的时候, 其实是在`sys.meta_path`这个导入器列表中, 使用其中的导入器来导入模块:

```python
>>> import sys
>>> sys.meta_path
[<class '_frozen_importlib.BuiltinImporter'>, <class '_frozen_importlib.FrozenImporter'>, <class '_frozen_importlib_external.PathFinder'>]
```

其中的导入器主要实现了`find_module`方法, 通过实现该方法可以自定义导入器:

```python
import os

class StructImporter:
    def __init__(self, path):
        self._path = path

    def find_module(self, fullname, path=None):
        name = fullname.rpartition('.')[-1]
        if path is None:
            path = self._path
        for dn in path:
            filename = os.path.join(dn, name+'.xml')
            if os.path.exists(filename):
                return StructXMLLoader(filename)
        return None
```

导入器中会遍历path, 直到找到以`.xml`结尾的文件, 并返回该文件的模块载入器, 模块载入器是一个实现了`load_module`方法的类:

```python
import imp
class StructXMLLoader:
    def __init__(self, filename):
        self._filename = filename
        
    def load_module(self, fullname):
        mod = sys.modules.setdefault(fullname,
                                     imp.new_module(fullname))
        mod.__file__ = self._filename
        mod.__loader__ = self
        code = _xml_to_code(self._filename)
        exec(code, mod.__dict__, mod.__dict__)
        return mod
```

在载入器中, 使用了前文的解析方法, 将XML文档解析为python代码, 并且在创建的mod对象环境中执行该代码.

安装导入器的方法是在`sys.meta_path`中添加该导入器:

```python
import sys

def install_importer(path=sys.path):
    sys.meta_path.append(StructImporter(path))

install_importer()
```

至此, 模块就可以被正常使用了:

```
>>> import datadefs
>>> datadefs
<module 'datadefs' (<__main__.StructXMLLoader object at 0x7f84e763e390>)>
>>> s = datadefs.Stock('ACME', 50, 91.1)
>>> s.price
91.1
>>> import inspect
>>> print(inspect.getsource(datadefs))
<structures>
  <structure name="Stock">
    <field type="SizedRegexString" maxlen="8" pat="'[A-Z]+$'">name</field>
    <field type="PosInteger">shares</field>
    <field type="PosFloat">price</field>
  </structure>

  <structure name="Point">
    <field type="Integer">x</field>
    <field type="Integer">y</field>
  </structure>

 <structure name="Address">
   <field type="String">hostname</field>
   <field type="Integer">port</field>
 </structure>
</structures>

```

代码清单: [importly.py](https://gist.github.com/lozybean/431a8829e09bf5050c02a436360e6567#file-importly-py)

## 总结

### 我们做了什么

1. 像建筑积木一样使用描述器;
2. 隐藏讨厌的细节(签名等);
3. 动态代码生成;
4. 自定义导入;

而这些工作都没有做任何的'hack', 所有的一切都在python3的设计之中, 使用python3可以更加优雅地实现很多事情.

### 这只是开始

python3还有很多好用的特性:

* 函数注释:

```python
def add(x:int, y:int) -> int:
    return x + y
```

* 非本地变量:

```python
def outer():
    x = 0
    def inner():
        nonlocal x
        x = newvalue
    ...
```

* 上下文管理器:

```python
with m:
    ...
```

* Frame-hacks

```
import sys
f = sys._getfrmae(1)
```

* 代码修改器:

```python
import ast
```

### 什么时候使用元编程

元编程并不是正常的编程, 他常常在框架或者库中使用, 在日常编程工作中, 保持简单并不是一个不好的方案.

## 引用

本文根据[David Beazley 13年pycon的演讲](https://www.youtube.com/watch?time_continue=1&v=sPiWg5jSoZI)以及[对应的ppt](http://www.dabeaz.com/py3meta/Py3Meta.pdf)整理.