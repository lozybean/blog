Title: Python2.x中的编码
Date: 2015-10-24
Category: Learning
Tags: python, 编码

## unicode和UTF

Python2.x中的字符串编码经常会引起一些困扰，根本原因往往是搞混了UTF-8和unicode的关系。

`unicode`(*Universal Multiple-Octet Coded Character Set*)是一种国际标准的字符集，是一张字符和编码的一张表格，用来表示世界上所有的语言。该字符集有两张表：`UCS-2`和`UCS-4`。这两张表的区别是使用多少个字节来表示一个字符，前者使用两个字节，后者使用四个字节，字节数越多说明可以表示的总字符量越大，下文中，将使用**对照码**来表示这两张字符集中的码，用以区分实际存储的**字节码**。

如果编译过Python的话，就会知道有一个编译选项：

```bash
--enable-unicode[=ucs[24] Enable Unicode strings (default is ucs2)
```

python默认使用`UCS-2`这张表来对unicode进行解读，不是十分必要下，不应该修改这个配置。

`UTF`(*UCS Transformation Format*)则是具体的一种编码存储方案，unicode是一张字符对应的表，而UTF则是这张表在计算机内部存储时的编码，说人话就是UTF是UCS的实现方式，同样也有两种方式：`UTF-16`和`UTF-8`。

`UTF-16`是最直观的方式，就是直接用UCS的对照码来存储，如`汉`字的unicode码字为：`\u6c49`，该码字是`汉`这个字在`UCS-2`的**对照码**，`UTF-16`有两种编码方式：

* 一种是`UTF-16-BE`，这种编码完全按照对照码顺序作为字节码，表示为：`\x6c\x49`；
* 另外一种`UTF-16-LE`，则是将对照码顺序颠倒作为字节码，表示为`\x49\x6c`。

这种方式比较直观，但是有一个问题是，由于直接使用了对照码作为字节码，原本只需要一个字节的英文字符，也统一为使用两个字节，而且和`ASCII`码无法兼容，又有两种编码的顺序问题，所以使用比较少，并且应该避免使用。

`UTF-8`则是为了解决上述问题的另外一种编码，根据UCS的范围使用不同的编码方式，利用这种分段编码的方式，兼容`ASCII`，对应关系如下：

|UCS-2对照码范围|UTF8字节编码|
|-------------------|----------:|
|0000 0000-0000 007F | 0xxxxxxx|
|0000 0080-0000 07FF | 110xxxxx 10xxxxxx|
|0000 0800-0000 FFFF | 1110xxxx 10xxxxxx 10xxxxxx|
|0001 0000-0010 FFFF | 11110xxx 10xxxxxx 10xxxxxx 10xxxxxx|

可以看到，虽然可以做到兼容ASCII，但是比较大的对照码则需要更多的字节来表示，如中文需要三个字节来存储。观察UTF8的编码规则，可以看到，当首子为`0`时，会被立即解码，当头部有若干个`1`时，则会被识别本次解码需要多少字节。

## Python2.x中的编码

### 1. python源代码的编码

Python2.x中，默认的编码方式是`ASCII`，也就是说，Python的整个脚本文件都是以ASCII码来编码的，这就意味着除了ASCII码以外的字符，都不能出现在脚本文件中，包括*注释中也不能出现诸如中文等字符*，否则都会得到类似的错误：

```bash
SyntaxError: Non-ASCII character '\xe6' in file coder.py on line 8, but no encoding declared; see http://python.org/dev/peps/pep-0263/ for details
```

解决这个问题，就要改变Python的脚本的编码格式，方法是在Python脚本的头部加入一个`magic comment`：

```python
# -*- coding: utf-8 -*- #
```

事实上，python的解释器将通过`coding[:=]\s*([-\w.]+)`这个正则表达式来获取脚本文件的编码内容，具体规则参考[PEP 0263](https://www.python.org/dev/peps/pep-0263/)。加上该头之后，Python2.x的脚本中就可以使用中文，但是要注意，在字符串中直接使用中文时，内容是字节码，根据UTF-8的规则，每个中文都是三个字节的码字。

### 2. python中字符串的编码

python中的字符串可以通过两个方法实现编码和解码：`encode()`、`decode()`，通过指定不同的编码格式，可以对字符串进行编码和解码。编码过程就是将字节码转换成实际的字符，而解码则是将实际的字符转换成存储时的字节码。看起来不会有任何问题，但是如果我们对一个字节码进行编码时，就会出现很大的问题：

```python
# -*- coding: utf-8 -*- #
s = '中文'   # 注意，s是一个用utf-8表示的字节码，一个6个字节，每个汉字一个字节
s.decode('utf-8')   # 将字节码按照utf-8解码成unicode字符，正确
s.encode('utf-8')   # 对字节码按照utf-8进行编码，错误！
```

其实是一个很简单的逻辑问题，字节码怎么可以再次被编码呢？在python中确实可以这样做，但是事实上，在Python的内部，是这样进行的：

```python
# -*- coding: utf-8 -*- #
s = '中文'
# s.encode('utf-8')  # 错误
s.decode('ascii').encode('utf-8')  # 真实发生的过程
s.decode('utf-8').encode('utf-8')  # 正确的处理过程
```

python中的str进行编码时，会先将其按照默认的编码方式：ascii码解码成相应的字符，然后再把该字符按照指定的格式进行编码。这样问题就来了，一个字节能表示的范围远远大于ascii码的范围，当这个字节超过ascii吗范围后，再用ascii码解码时，自然就无法进行解码。

需要通过`sys`模块中的两个函数可以获取默认编码方式：

```python
# -*- coding: utf-8 -*- #
import sys
sys.getdefaultencoding()  # 获取默认编码，python2.x中为ascii
reload(sys)  # reload之后才能调用setdefaultencoding()
sys.setdefaultencoding('utf-8')  # 将默认编码改为'utf-8'
```

如果不调用`encode()`方法，或者在调用之前按照正确的解码方式事先解码为字符串，上述更改没有必要进行，但是在python中，有一些方法并不提供另外的编码方法，只能调用系统默认编码方法时，就必须修改这个设置，如datetime.strptime。

## Python2.x中的str和unicode

Python2.x中，`basestring`有两个子类：`str`和`unicode`：

* 一般的字符串为`str`对象，表示的就是字节码，默认编码方式为`ascii`；
* 前面加`u`的对象为`unicode`对象，表示的是unicode字符，使用`magic comment`头部定义的方式解码；

unicode对象有两种实例化方式，一种是常见的，直接用`u'汉字'`来直接表示一个unicode字符串，这种方式一般不会有任何问题；另外一种方式是通过`unicode()`方法传入一个待转换的对象，并调用该对象的`__unicode__`方法，如果该对象没有这个方法，则会调用其`__str__`将其转变为字符串，如果该字符串是unicode字符串则直接返回，如果不是，则为了识别不同的编码字符串，将会按照默认的编码方式解码，然后再编码为unicode，这里就会出现上面说的编码问题。注意，unicode()接收的对象并不是字节码，而是字符串。

```python
# -*- coding: utf-8 -*- #
u = u'中文'  # 表示一个unicode字符串：u'\u4e2d\u6587'
class A(str):
    def __unicode__(self):
        return self.decode('utf-8')
s = '中文'  # 表示utf-8编码的字节串，'\xe4\xb8\xad\xe6\x96\x87'
unicode(s)  # 错误，相当于unicode(s,defaultencoding)，内部过程为 s.encode(defaultencoding).decode('utf-8')，发生上文提到的编码问题
unicode(s,'utf-8') # 相当于unicode(s,'utf-8')，正确
a = A(s)
unicode(a)    # 调用a对象的__unicode__方法，正确
```

## 常见问题以及推荐解决方案

1. 读写文件时，都应该对编码后的字节码操作，读入文件后应该解码，写入文件前应该编码；
2. 调用`datetime.strptime()`方法时，将会使用默认编码方式解码，如果格式中有中文，如`datetime.strptime(u'2015年10月25日', "%Y年%m月%d日")`，则只能修改系统默认编码方式为`utf-8`。
2. 正则表达式中，如果使用了unicode对象，则其pattern应该也应该使用unicode对象，如：`re.search(u'中',u'中文')`；
3. 其他涉及到encode和decode的方法时，都应该检查是否使用了正确的编码方法，而不是调用默认方法，或者修改系统的默认编码方法；
4. 为了减少编码带来的问题，应该在每一个脚本的头部添加`magic comment`；
5. 尽可能使用python3.x
