Title: urllib.request模块中的三大组件
Date: 2015-10-31
Category: Learning
Tags: python, urllib, http

## Request类

`Request`类用于生成请求对象，描述一个的URL请求的相关信息。其构造方法为：

```python
Request(url, data=None, headers={}, origin_req_host=None, unverifiable=False, method=None)
```

各个参数依次为：

1. `url` : 包含一个有效URL，该URL不一定是真实的URL，urllib会自动处理重定向过程，并将真实url返回，可以被`Response`对象获取到，并通过`geturl()`方法返回真实URL；
2. `data` : 必须是字节码数据，如果是`unicode`，应该先`encode`成字节码；当该参数被设定时，请求方法将会变成POST，并且只能被当前Request对象使用；`data`应该作为标准`application/x-www-form-urlencoded`格式的缓存，可以通过`urllib.parse.urlencode()`函数生成该格式；使用`data`时，应该正确设置`Content-Type`首部，确保服务器正确识别；
3. `headers` : 是一个`dict`对象，该按照键值对的形式，添加到首部中，除了通过`headers`参数，还可以使用`add_header()`方法后续逐一添加首部；
4. `origin_req_host` : 该值代表原始主机地址，如网页图片的原始主机地址等；
5. `unverifiable` : 通过设定该值为`True`，可以获取一些无法被自动获取的资源；
6. `method` : 指定使用的HTTP方法，如果不指定该参数，则使用默认方法，当`data`为空时，该参数默认为`GET`，当`data`不为空时，该参数默认为`POST`；可以通过`get_method()`方法获取实际使用的方法；

`Request()`对象是一个基本对象，该对象包括`方法`、`数据`、`首部`、`代理`等各种URL基本信息，当使用该请求时，使用这些已经配置好的信息来完成请求。该对象的常规属性方法不再赘述，有几个比较特殊的方法：

* `add_unredirected_header(key,header)` : 和普通的首部不同，由该方法指定的首部将不会被添加到重定向的请求中；
* `set_proxy(host,type)` : 该方法设定请求的代理，发送请求前先连接到代理服务器；

## OpenerDirector类

通过`OpenerDirector`类实例化的对象(以下称`opener`)，是句柄(`handlers`)链和异常恢复的管理器，模块中的`urlopen()`函数就是一个`opener`的具体方法，通过模块中的`install_opener()`函数，可以改变默认的`urlopen()`，更加好的办法是，通过`build_opener()`函数来新建一个`opener`；

>如果说Request是演员的话，Opener是一个导演的角色负责统筹规划，而Handler则是场工包揽粗活重活，但是三者联合才能做出作品。

该对象包含以下方法：

1. `add_handler(handler)` : 添加一个句柄，该句柄是BaseHandler()对象，该方法将搜索该对象的以下方法: protocol_open()、http_error_type()、protocol_error()、protocal_request()、protocol_response()，并将其添加到句柄链中。
2. `open(url, data=None[, timeout])` : 打开一个url，更加常用的是一个`Request()`对象。该方法的返回值和异常值和`urlopen()`函数类似；可选参数`timeout`仅在HTTP、HTTPS、FTP协议中有效，指定连接超时的时间，默认由套接字(socket)的默认超时时间指定；
3. `error(proto, *args)` : 指定协议的错误，如果是HTTP协议，则可以使用错误代码来指定，如：`http_error_404`等；

一个opener打开一个URL，需要通过以下三个阶段：

1. 所有的handler执行`protocal_request()`方法，完成请求的初始化操作；
2. 所有的handler执行`protocal_open()`方法，如果有一个handler接收到`None`或者引发异常，都会终止该阶段；该算法首先尝试`default_open()`方法，当其返回空时，再循环调用`protocal_open()`方法(该方法名根据不同的handler会不同)，如果这些方法仍然返回空，则再循环调用`unknown_open()`方法；
3. 所有handler执行`protocal_response()`方法，处理响应结果；

## BaseHandler类

该类中定义基本的handler方法和属性，并被具体的Handler类继承，这些类定义了具体的请求的处理方式。

常用的HTTP子类有：

* `HTTPDefaultErrorHandler` : 定义HTTP错误码的默认处理；
* `HTTPRedirectHanlder` : 重定向句柄，该句柄处理返回码为`30*`的HTTP请求，并重定向到新的URL；
* `HTTPCookieProcessor(cookiejar=None)` : Cookie管理句柄，需要传入一个`cookiejar`对象，该对象可由`http.cookiejar.CookieJar()`类实例化；
* `ProxyHandler(proxies=None)` : 代理管理句柄，会自动读取系统环境变量中的代理；
* `HTTPHandler` : HTTP处理句柄
* `HTTPSHandler` : HTTPS处理句柄

通过以上这一些句柄定义具体的请求操作，并使用opener管理统筹整个过程的进行。

## 补充实验

