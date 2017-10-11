Title: [翻译]Python中的类、实例和属性
Date: 2017-03-07
Category: pages
Tags: python


本文已获得原作者同意翻译并转载, [原文链接](https://www.chrisbarra.me/posts/classes-instances-and-attributes-in-python.html?nsukey=8bjWaPynrRb%2B8YFaWVC4sr0RzDBpwDmxHhRk8Uoj%2BpV4KH31tPmapBv2VOXALNNUvycR%2FT6NeoPodERRAw77ClzhS15DK%2FPkR5pU0GqkF6JZy9mvKPCo%2FEdxZSah1E2jjKE9NcKDB4Xxyi2olcAHItpQkVUXbyoTyQbSmI%2BcNleEdf9XguyRQeuJoLzaSAtW)

## 回归基础
![baby_python](http://wx4.sinaimg.cn/mw690/95202659ly1fdemafx3mjj21vs0xwq60.jpg)

几天前我得到一个问题.

```python
class A():
    x = 1

a = A()
b = A()

b.x = 11
a.x # ?
```

我的回答是错误的, 但是这是一个很好的时机去选一些书和[参考文档](https://docs.python.org/3.5/reference/)回来, 并且花几个小时在背后的几个概念上: **类**、**实例**、**属性**还有**命名空间**.

所有你即将读到的东西都和Python3.x相关.

## 作用域简介

在Python中有两个非常重要的概念: **作用域**和**命名空间**. 两个概念都和名称相关, 但是一般来说, **作用域**和unqualified名称(如X)相关, 而**命名空间**和qualified属性名称相关(如object.X). *有关qualified names和unqualified names相关概念可以参考: [stackoverflow](http://stackoverflow.com/questions/17403941/what-is-a-qualified-unqualified-name-in-python)、[PEP3155](https://www.python.org/dev/peps/pep-3155/). 由于**在Python中所有东西都是对象**的事实, 这两者的差异很小, 但是一般来说我们可以这样假设.

是时候来点代码了

```python
X = 20 # global X
def f():
    print(X)

def f1():
    X = 1 # local X
    print(X)

f() # 20
f1() # 1
```

Python(一般来说)遵循**LEGB**的规则, **LEGB**表示: **Local** -> **Enclosed** -> **Global** -> **Built-in**.

**LEGB**规则的含义是: 当你呼叫**X**时, Python会按照以下顺序来查找:

1. Local Scope, 局部作用域
2. Enclosed Scope, 封闭作用域
3. Global, 全局空间
4. Built-in, 内置空间

如果Python没有找到任何东西的话, 就会报错:

```python
# X = 20
def f():
    print(X)

f() # 20
---------------------------------------------------------------------------
NameError                                 Traceback (most recent call last)
<ipython-input-2-0ec059b9bfe1> in <module>()
----> 1 f()

<ipython-input-1-4ce367479db4> in f()
      1 def f():
----> 2     print(X)
      3

NameError: name 'X' is not defined
```

如果你想了解更多有关**LEGB**的话, 你可以从[这里](http://stackoverflow.com/questions/291978/short-description-of-scoping-rules)开始.

让我们回到我们的类.

```python
class C():
    X = 10
    def f(self):
        print(X)
c = C()
```

我们定义了一个简单的类, 叫做**C**, **X**在**class C**内部定义, **c**就是我们所说的**C的对象**或者**C的实例**.

**"f"**是一个*函数*并且接受一个参数: **self**.

```python
print(C.f, c.f)
<function C.f at 0x1039aaae8> <bound method C.f of <__main__.C object at 0x103ea8e10>>
```

现在出现了第一个奇怪的部分, 我们呼叫了**f**, 一个在**C**内部定义的函数, 但是我们得到了**两个不同的东西**.

一个叫**C.f**的***函数***和一个叫**c.f**的***方法***.

这里的关键词是**bound**或者这是存在的主要不同点.

但是让我们来运行我们的函数(或者说方法):

```python
c.f()
---------------------------------------------------------------------------
NameError                                 Traceback (most recent call last)
<ipython-input-5-69c69864e152> in <module>()
----> 1 c.f()

<ipython-input-3-72549c815346> in f(self)
      2     X = 10
      3     def f(self):
----> 4         print(X)
      5 c = C()

NameError: name 'X' is not defined
```

mmmm... Python按照**LEGB**的规则应该会找到X才对, **LEGB**的规则仍然被采取吗?

让我们试试这个:

```python
X = 50
class C():
    X = 10
    def f(self):
        Y = 10
        print(X)
        def f1():
        	print(Y)
        f1()
c = C()
c.f()

# Output: 50
# Output: 10
```

我们有一个嵌套的函数(f1)并且我们在全局作用域中加上了 X = 50, now it works.

**但是class C内部的X呢?**

实际上**X**(C内部的)准确的说并不是一个*变量*, 它是一个**属性**并且在**LEGB**中和*变量*的有不同的表现.

```python
class C():
    X = 10
    def f(self):
        print(self.X)
c = C()
c.f() # 10
```

我们仅仅在print函数内部改变用**self.X**替换了X, now it works.

为什么?

好吧... 是时候来解释一下**self**和**命名空间**的概念了.

## Self

按我说的话, **self**只不过是一个参数.

```python
class C():
    X = 10
    def f(legion):
        print(legion.X)
c = C()
c.f() # 10
```

这段代码起到一样的作用, 我们使用**self**作为惯例, 它仅仅是**实例**的一个引用, 在这里就是**c**, 当我们运行这个函数时作为参数传入.

当我们输入**c.f()**时, Python会运行**C.f(c)**, 这里**C**是我们实例的类, **f**是我们的方法/函数, **c**是**f**需要的的第一个参数(self或者legion).

你还记得这个吗?

```python
c.f # <bound method C.f of <__main__.C object at 0x1039cf6d8>>
```

现在**bound**的含义更加清楚了, 它意味着当我们使用**c.f**来运行**f**时, 我们自动地传入了实例的一个引用.

所以从这一刻开始, 当我们谈到接收一个self参数的函数时, 我们就叫它**实例方法**.

是的, 你可以有**unbound method**, 这些方法和实例不关联, 如**类方法**或者**静态方法**, 我们以后再讨论它.

## 命名空间: Namespaces

命名空间是一个名称的集合.

一个**对象**的**引用**的集合, 如**name=object**.

为什么**命名空间**那么重要?

因为每一个类都有一个命名空间, 并且类的每一个实例都有命名空间.

他们是完全**独立**的, 但是*不知何故*也是相关的.

让我们看一下类命名空间和实例命名空间的内部, 我们用到一个内置属性:**\_\_dict__**.

```python
# 我移除了C.__dict__输出中的所有内置函数/方法
C.__dict__ # mappingproxy({'f': <function C.f at 0x1039c1378>, 'X': 10})
c.__dict__ # {}
```

正如你看到的, 类命名空间和实例命名空间是完全不同的.

类命名空间是一个**mappingproxy**, 实例命名空间是一个**dict**.

mappingproxy是一种**只读**的字典.

你可以在[**这里**](http://stackoverflow.com/questions/32720492/why-is-a-class-dict-a-mappingproxy)找到为什么要使用mappingproxy.

当我说mappingproxy是只读的时候, 我的意思是你不能像字典一样给mappingproxy赋值一个值.

```python
C.__dict__['X'] # 10
C.__dict__['X'] = 10 #
---------------------------------------------------------------------------
TypeError                                 Traceback (most recent call last)
<ipython-input-193-ec143c56e8cd> in <module>()
      1 C.__dict__['X']
----> 2 C.__dict__['X'] = 10

TypeError: 'mappingproxy' object does not support item assignment
```

如果你想在类命名空间中加入新的成员, 你必须使用其他的方式.

回到**C的命名空间**, 我们找到了**f**(我们的实例方法)和**X**.

如我们所说, **X**不是一个*变量*, 并且在**C.\_\_dict__**中定义.

我们怎么称呼它呢? 叫它**类属性**

类属性的意思是, **X**属于**C**(class), 因为我们可以看到**X**在**C的命名空间**中(C.\_\_dict__).

我们可以直接访问类属性吗?

是的, 使用**NameOfTheClass.AttributeName**的方式.

```python
C.X # 10
```

概括如下:

1. C的命名空间包含**f**(一个方法)和**X**(一个类属性)
2. c的命名空间是空的
3. 我们使用**C.X**来访问C的X

但是我们之前说到: **类命名空间**和**实例命名空间**是完全分离的.

```python
class C():
    X = 10
    def f(self):
        print(self.X)
c = C()
c.f() # 10
```

当我们输入**self.X**的时候, 我们告诉Python:"*在实例self的命名空间中找X*".

**我们从self.X(实例中)访问到属于C(类属性X)的东西呢?**

是时候开启下一个部分了.

## MRO

MRO表示***Method Resolution Order***, 是为什么以及如何使用**self.X**访问到C.\_\_dict__['X']的原因.

如我所说: 类命名空间和实例命名空间是完全独立的, 但是不知何故也是相关的.

**MRO**就是*不知何故*的原因.

当Python找一个属性的时候, 就像**self.X**, 它会按照下面的顺序进行:

1. 实例命名空间
2. 类命名空间
3. 父类命名空间
4. object

object是所有类的父类, 我们在这里不打算讨论它.

让我们看另外一个例子

```python
class Mother():
    M = 22
class Father():
    F = 34
class Son(Mother, Father):
    S = 10
    def f(self, x):
        print(getattr(self, x))
a = Son()
a.f("M") # Output: 22
a.f("F") # Output: 34
a.f("S") # Output: 10
print(a.M) # Output: 22
print(a.F) # Output: 34
print(a.S) # Output: 10
```

利用[getattr](https://docs.python.org/3/library/functions.html#getattr)	我们用我们的实例(self)和另外一个参数("M"、"F"或者"S")来得到我们的类属性;

我使用[getattr](https://docs.python.org/3/library/functions.html#getattr)是因为这样可以动态地指定属性的名称, 但和self.M是完全类似的(在我们的例子中是a.M).

如我所说, 实例和它的类(Son)以及类的父类(Mother和Father)相关联.

在实例和类以及类的父类之间给出一个顺序的过程称为**linearization**.

在Python的MRO背后, 使用了一个叫[C3](https://www.python.org/download/releases/2.3/mro/)的算法, 主要记住以下一点:

* 孩子优先于父母, 并且尊重在**\_\_bases__**中出现的顺序.

bases就是我们作为参数放到类后面的东西: class Name(bases).

* 实例**a**是类**A**的一个孩子
* 类**A**是类**Mother**和**Father**的一个孩子
* 类**Mother**和**Father**都是object的孩子

使用\_\_bases\_\_可以一个类的父母的元组, 使用\_\_class__可以得到实例的类:

```python
a.__class__ # __main__.A,
Son.__bases__ # (__main__.Mother, __main__.Father)
Mother.__bases__ # (object,)
Father.__bases__ # (object,)
```

我不会说一些新的东西, 我们可以从中得到MRO的搜索顺序吗?

是的, 使用**\_\_mro__**:

```python
Son.__mro__ # (__main__.Son, __main__.Mother, __main__.Father, object)
```

当我们输入下面的代码时:

```python
a.M # 22
```

Python按照下面的顺序搜索:

1. a.\_\_dict__
2. Son.\_\_dict__
3. Mother.\_\_dict__
4. Father.\_\_dict__
5. object.\_\_dict__

使用先到先得的规则.

```python
class Mother():
    M = 22
class Father():
    F = 34
class Son(Mother, Father):
    S = 10
    F = 50
    def f(self, x):
        print(getattr(self, x))
a = Son()
a.F # Output: 50
```

这就是为什么我们得到50而不是34的原因.

1. 首先Python看了看**a**(Son的实例)的的命名空间内部, 但是找不到任何叫做**"F"**的东西
2. 然后是时候看看**Son**(我们实例的类)的命名空间内部了, 并且找到了**"F"**

那在Father中定义的F呢?

如我所说, 先到先得, 在我们的**\_\_MRO__**中, Son的命名空间是在Father命名空间之前的.

如果我们找一个在\_\_mro__的命名空间中不存在的引用会发生什么呢?

```python
a.DXIUISD
---------------------------------------------------------------------------
AttributeError                            Traceback (most recent call last)
<ipython-input-242-bcb50197f1e4> in <module>()
----> 1 a.DXIUISD

AttributeError: 'A' object has no attribute 'DXIUISD'
```

你得到了一个AttributeError.

**但是棘手的部分在哪里?**

MRO以及上面解释的所有事情, 都只在你尝试**访问**一个属性或者方法时才会发生. 通常**访问**的意思是使用object.attribute或者object.method的方式.

当你尝试**赋值**一个属性或者方法(如object.attribute=10)时, 你会在**对象的命名空间中进行(实例或者类)**.

你可以使用一些高级的魔法来改变这个特性(元类、继承、描述器、属性等...), 但这是**一般**的工作方式.

所以当我们输入:

```python
Son.F
```

Python会在命名空间中, 根据\_\_mro__的顺序查找该属性.

```python
Son.F = 50
```

但是像上面这样就和\_\_mro__没什么关系, **Python会创建一个新的属性或者如果该属性存在的话改变该属性**.

```python
class Mother():
    F = 34
class Son(Mother):
    S = 10
    F = 50

Mother.F # 34
Son.F # 50 # operation 1
del(Son.F) # remove F from Son.__dict__ # operation 2
Son.F # 34 # operation 3
Son.F = 30 # operation 4
Son.F # 30
```

清楚它如何工作的了吗?

1. Son.F的结果是50是因为F在Son的命名空间中定义
2. 我们从Son的命名空间中删除F
3. 在Son的命名空间中没有"F", 接下来Python会在Mother的命名空间中查找, 并且找到F = 34
4. 这个赋值命令, Son.F = 30, 是在Son的命名空间中完成的, 现在我们在Son的命名空间中又发现了新的F, 所以Son.F是30

下面是Son命名空间的变化

```python
{'S': 10, 'F': 50} # After 1
{'S': 10,} # After 2
{'S': 10, 'F': 30} # After 4
```

## \_\_init__方法

init的方法是一个特殊的方法, 用来定制我们的实例, 它在我们创建一个实例的时候运行.

```python
class A():
    C = 10
    def __init__(self, x):
        self.x = x
a = A(10)
b = A(50)
```

翻译过来就是"*当你运行A(something)的时候创建一个实例, 并且赋值somthing给self.x*".

让我们看看命名空间.

```python
A.__dict__ # (mappingproxy({'__init__': <function A.__init__ at 0x103ecab70>, 'C': 10})
a.__dict__ # {'x': 10}
b.__dict__ #  {'x': 50}
```

**A**有它的属性, C以及\_\_init__, **a**和**b**拥有他们的x.

**a**和**b**的属性成为**实例属性**, 它们**属于实例**.

现在应该清楚下面的输出了:

```
A.C, a.C, b.C # (10, 10, 10)
```

**a**和**b**在他们的命名空间中都没有C属性, 所以Python在**A**的命名空间中查找并且找到了一些东西(通过MRO).

但是当我们做下面的事情的时候会发生什么呢?

```python
a.C = 50
a.__dict__ # {'C': 50, 'x': 10}
```

我们在**a**的命名空间中创建了一个新的引用.

```python
A.C, a.C, b.C # (10, 50, 10)
```

这就是为什么我们得到这个结果, 因为现在当我们在**a**里面找C的时候, 我们找到了.

让我们再看一下**A**, **a**, **b**的**\_\_dict__**

```python
A.__dict__ # (mappingproxy({'__init__': <function A.__init__ at 0x103ecab70>, 'C': 10})
a.__dict__ # {'C': 50, 'x': 10}
b.__dict__ #  {'x': 10}
```

那么我们改变A.C会怎么样呢?

```python
A.C = 20
A.C, a.C, b.C # (20, 50, 20)
```

a.C任然是50, 但是b.C仍然在**A**的命名空间中查找, 因为它自己的命名空间中没有任何叫做"C"的东西.

```python
b.C = 70
A.C, a.C, b.C
(20, 50, 70)
```

现在实例**a**, **b**以及类**A**都在它们的命名空间中有一个"C"了.

```python
A.__dict__ # (mappingproxy({'__init__': <function A.__init__ at 0x103ecab70>, 'C': 10})
a.__dict__ # {'C': 50, 'x': 10}
b.__dict__ #  {'C': 70, 'x': 50}
```

## 我能从一个实例获取到类的属性吗?

是的, 但是如果你在自己的命名空间中有一个同名的引用, 就需要显示调用.

```python
class A():
    C = 10
    def __init__(self, x):
        self.x = x
    def p(self):
        print(A.C, self.x)
a = A(10)
b = A(50)
a.p() # 10 10
b.p() # 10 50
```

所以我们在方法中使用了**A.C**的硬编码.

有什么更好的办法吗?

```python
class A():
    C = 10
    def __init__(self, x):
        self.x = x
    def p(self):
        print(type(self).C, self.x) # First
        print(self.__class__.C, self.x) # Second
a = A(10)
b = A(50)
a.p() # 10 10
b.p() # 10 50
```

个人更加喜欢第二种(对我来说更加清楚).

## 但是如果...?

我们有这样的代码:

```python
class A():
    C = []
    def __init__(self, x):
        self.x = x
    def p(self):
        print(self.__class__.C, self.x)
a = A(10)
b = A(50)
```

然后我们输入:

```python
A.C, a.C, b.C # ([], [], [])
a.C.append(50)
A.C, a.C, b.C # ([50], [50], [50])
b.C.append(10)
A.C, a.C, b.C # ([50, 10], [50, 10], [50, 10])
A.C.pop(0)
A.C, a.C, b.C # ([10], [10], [10])
```

类属性**A.C**这次看上去真的被共享了, 而不是像我们之前使用的类属性一样只是*初始共享*.

所以为什么我们还可以append或者pop元素, 而不发生任何意外呢?

**因为我们没有做任何赋值的操作.**

我们的实例和类在可变的对象上工作, **使用引用(C)来获得对象的指针然后直接在上面修改**.

```python
a.C = a.C * 2
A.C, a.C, b.C # ([10], [10, 10], [10])
```

在一次赋值之后, **a.C**在它的命名空间中有了一个新的引用.

但是棘手的部分在这里... 如果我们用下面的代码来替代最后一次输的代码:

```python
a.C *= 2
A.C, a.C, b.C # ?
```

输出会是什么?

```python
([10, 10], [10, 10], [10, 10])
```

以上结果中, 我们并没有做任何新的赋值, 仍然在引用的对象上直接修改.

为什么?

**这是因为自增赋值(+=, -=, \*=, /=, ...)的工作方式**

对于列表或者可变对象, 这个运算符号并没有做任何*赋值*, 而是"原地"完成操作, 我们只是直接在**引用的对象**上更新而已. *译者注: 该操作会将类属性的引用添加到实例属性中, 所以在实例的\_\_dict__中能够找到相关的引用, 但是和类中的引用共享内存空间(相同id), 所以在实例的可变类型中应用该操作会直接修改类中的引用.

所以回到最初的问题...

```python
class A():
    x = 1

a = A()
b = A()

b.x = 11
a.x # ?
```

答案是1因为b.x = 11在**b**的命名空间中创建了一个新的属性(实例属性).

**a**的命名空间仍然是空的, 所以**a.x**会在**A**的命名空间中查找并且找到一个等于1的**x**.

如果你想在Python的面向对象上走的更远的话, 我想没有比[Leonardo Giordani's training](https://speakerdeck.com/lgiordani/object-oriented-python-from-scratch)更好的了.