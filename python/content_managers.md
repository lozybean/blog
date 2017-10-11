Title: Python的上下文管理器
Date: 2015-10-10
Category: Learning
Tags: python

## 首先想到的

看到上下文管理器的时候，立马闪进我脑海的是《*Ruby元编程*》中使用ruby实现*C#*中的using关键字的那段：

```csharp
RemoteConnection conn = new RemoteConnection("my_server");
String stuff = conn.readStuff();
conn.dispose();
```

上面这段代码在连接的过程中如果发生异常，则连接不会被正确的释放，*C#*通过`using`关键字来保证正确释放：

```csharp
RemoteConnection conn = new RemoteConnection("some_remote_server");
using(conn){
  conn.readSomeData();
  doSomeMoreStuff();
}
```

在`using`关键字会保证在大括号中的代码执行后，`conn`对象不管有没有异常都会调用`dispose()`方法；

*Ruby*实现为：

```ruby
module Kernel
  def using(resource)
    begin
      yield
    ensure
      resource.dispose
    end
  end
end
```

*ps: ruby 基本忘记了，强烈想复习一回*

## 什么是上下文

`using`关键字的作用类似于`finally`，将`finally`中相同的执行内容作为关键字，比反复使用`try-except`要优雅得多，而`using`作用的对象是代码块（*block*），而所谓的上下文，就是代码块在执行前后进行的操作，前面的例子中，`using`代码块执行完毕后调用`dispose`方法，就是*下文*了。

一个普通的python实现：

```python
conn = RemoteConnection("some_remote_server")
try:
	conn.readSomeData()
	doSomeMoreStuff()
finally:
	conn.dispose()
```

是不是很像*Ruby*的实现过程，可是要知道*Ruby*修改的是内核模块，所以想象空间其实更多，而对应*Ruby*的实现过程，可以看到：

`try`到`finally`之间的内容即需要执行的代码块，而`finally`之后的内容即代码块的*下文*，这样的写法很直观，是个普通青年，虽然知道每次`conn`结束的动作都是`dispose`，但是还是不厌其烦地亲自执行，并且确保不会有哪天突然忘记。

Python当然不会局限于此，而是拥有非常易用的上下文管理，以及更加易用的上下文管理库：`contextlib`。

## Python的上下文管理器

Python中使用两个*魔术方法*来实现上下文管理：`__enter__`和`__exit__`，拥有这两个方法的对象，会自动拥有上下文管理的功能，而实现该功能需要用到`with`关键字，说起来有点复杂，看一个例子：

另外一种比较文艺的Python实现：

```python
class MyRemoteConnection(RemoteConnection):
	def __enter__(self):
		return self

	def __exit__(self, exception_type, exception_val, trace):
		self.dispose()

with MyRemoteConnection("some_remote_server") as conn:
	conn.readSomeData()
	doSomeMoreStuff()
```

似乎代码更多了，但是有很多是对类进行一次性的修改。

首先要使用上下文，就必须有`__enter__`和`__exit__`方法，分别表示上下文的内容，当`with`语句开始前，上文会被自动调用，由于代码块使用了该类的对象，所以需要返回`self`，而当`with`语句结束时，不论是正常结束还是异常结束，都会调用`__exit__`方法中的下文。

上面的例子代码块用到了具有上下文类的对象，所以`__enter__`中需要返回`self`，但是代码块可以和对象没有任何关系，比如用上下文实现的定时器，其上下文是代码块执行和结束时的时间记录。

同时，`__exit__`接受三个参数，由于该方法是自动调用的，所以参数也是自动传入，通过参数名可以看到，`__exit__`会自动接受异常信息，所以在其中也可以做异常的处理操作。

这种上下文管理的方式是对`using`关键字的提升，因为不再局限于某一种上下文(如dispose)，而是将上下文管理定义到类中，由类来决定上下文的内容，更加灵活并且强大。

Python的上下文在文件操作时尤其常见，文件对象都有一个下文：`close()`，所以使用`with`关键字打开操作文件是一种好习惯，保证了句柄的正常释放；同时，`with`关键字在Python2.7之后，支持同时管理多个上下文，如同时读写两个文件：

```python
with open('file_to_read','r') as reader\
		open('file_to_write','w') as writer:
	writer.write(reader.read())
```

## 更加方便的上下文管理

什么，听了那么多，还是在怀念`using`关键字？好吧，那就用Python来实现`using`。

还有一种更加高(zhuang)级(bi)的实现方式：

```python
from contextlib import contextmanager
@contextmanager
def using(resource):
	yield
	resource.dispose()
```

使用起来是这样的：

```python
conn = RemoteConnection("some_remote_server")
with using(conn):
	conn.readSomeData()
	doSomeMoreStuff()
```

虽然没有变成关键字，并且需要借助`with`，但是总算是看起来极其相像了吧？这里就用到了Python提供的更加方面的上下文管理库：**contextlib**，有了它，就可以避免创建一个新的具有上下文管理的新类，而是在外部对其进行管理。

这里虽然用了*更加方便*作为标题，但是并没有绝对的哪种方式更加好。当使用一个其他已有的类时，后面的方式避免创建继承的子类就完成了相应工作，更加直接，但是没有异常处理等功能；而当编写一个新的类时，如果需要用到上下文管理，则在类定义中直接添加，提供最基本的支持，就是更加好的方案。



