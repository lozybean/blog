Title: urllib初探
Date: 2015-10-28
Category: Learning
Tags: python, urllib, http

`urllib`是一个Python的内置库，用来完成url相关的相关处理，最主要的功能就是使用通过url来完成请求。常用但不限于HTTP协议。

## Python2.x和Python3.x的区别

在Python2.x中，有两个不同的模块：

* urllib : 只能通过一个url来发出请求，提供urlencode函数生成GET的查询字符串；
* urllib2 : 可以通过Request对象来发出请求，支持添加各种头部，意味着可以伪装成各种浏览器。没有urlencode函数；

而在Python3.x中，这两个模块以及urlparse、robotparser一共四个模块合并为一个urllib模块，并且按照不同的功能分成5个子模块：

* urllib.error : 定义一些异常类；
* urllib.parse : 完成url的解析工作，将一个url分解成不同部分；
* urllib.request : 通过Request对象发出请求，另外一个请求方式被废除；
* urllib.response : 针对获取到的响应设计的文件类接口，可以用处理文件的形式处理响应；
* urllib.robotparser : 解析robotstxt，文件相关定义参考[官方文档](http://www.robotstxt.org/orig.html)；

## 使用urllib发送HTTP请求

使用Python3.x的urllib模块来发送简单的HTTP请求，Python2.x对应之前的各个模块，以`GET`和`POST`为例；

## 1. 发送GET请求获取weibo主页

先看代码:

```python
#!/usr/bin/env python
# -*- coding: utf-8 -*- #

from urllib import request

req = request.Request('http://weibo.com', method='GET')
req.add_header(
    'Content-Type', 'application/x-www-form-urlencoded;charset=utf-8')
req.add_header(
    'User-Agent',
    'Mozilla/5.0 (Macintosh; Intel Mac OS X 10_11_0) AppleWebKit/601.1.56 (KHTML, like Gecko) Version/9.0 Safari/601.1.56')

with request.urlopen(req) as f, open('sina_index.html', 'wb') as out:
    data = f.read()
    head_line = 'HTTP/%s %s %s\r\n' % (f.version / 10, f.status, f.reason)
    real_url = f.url
    headers = str(f.info())
    print(head_line + headers)
    out.write(data)
```

其中进行的一些主要步骤为：

1. 新建`Request()`对象，指定url以及需要使用的方法，可以通过header参数来传递HTTP首部；
2. 使用`add_header()`方法，逐一添加需要的HTTP首部，主要是`User-Agent`必须指定一个浏览器类型，否则容易被网站屏蔽；
3. 使用`request.urlopen()`方法，发送请求，并将响应作为文件形式进行操作；
4. 打印响应的首部，并将响应实体写入到文件中；

使用`Safari`的用户代理工具可以轻松获取浏览器类型代码：

1. 选择一个浏览器类型；
2. 再选择其他，Safari会将当前浏览器的类型代码显示出来；

最后打印的响应首部为：

```http
HTTP/1.1 200 OK
Server: nginx
Date: Tue, 27 Oct 2015 17:30:08 GMT
Content-Type: text/html
Transfer-Encoding: chunked
Connection: close
Vary: Accept-Encoding
Cache-Control: no-cache, must-revalidate
Expires: Sat, 26 Jul 1997 05:00:00 GMT
Pragma: no-cache
Pragma: no-cache
DPOOL_HEADER: dryad39
SINA-LB: aGEuOTEuZzEucXhnLmxiLnNpbmFub2RlLmNvbQ==
SINA-TS: ZDZjYTk0Y2UgMCAwIDAgNiAxMQo=


```

## 2. 使用Cookie登陆

由于HTTP是无状态协议，网页服务如果需要管理一个`Session`(会话)，则势必要使用`Cookie`，以弥补HTTP的无状态缺点，利用这个特点，只需要在借助浏览器先完成一次登陆，然后获取Cookie，并使用Cookie来完成登陆。

获取Cookie的方式非常简单，通过`Chrome`浏览器自带的开发者工具就可以完成。

首先成功登陆微博，等待一会，完成本次连接的全部内容，然后调出开发者工具，并且选择Network选项卡：

![步骤1](http://ww3.sinaimg.cn/large/95202659gw1exh7w0ayjzj21ha0rcjwe.jpg)

接下来刷新页面，此时，浏览器发送Cookie来完成验证：

![步骤2](http://ww4.sinaimg.cn/large/95202659gw1exh7w1ark9j21hc0u0gwd.jpg)

在Network选项卡中，会把本次浏览器的连接过程按照时间先后排序后列出，第一个时间点但是是重新登陆验证了，所以右击这次连接，可以看到复制Request头以及Response头等各种选项，获取Cookie需要Request头，复制并且粘贴：

```http
GET /lyon0804/ HTTP/1.1
Host: weibo.com
Connection: keep-alive
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/webp,*/*;q=0.8
Upgrade-Insecure-Requests: 1
User-Agent: Mozilla/5.0 (Macintosh; Intel Mac OS X 10_11_0) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/46.0.2490.80 Safari/537.36
Accept-Encoding: gzip, deflate, sdch
Accept-Language: zh-CN,zh;q=0.8,en;q=0.6,zh-TW;q=0.4
Cookie: *****
```

由于Cookie包含会话的认证信息，所以不要向他人公开Cookie，将上述请求头写到程序中，再次发送请求。

```python
#!/usr/bin/env python
# -*- coding: utf-8 -*- #

from urllib import request

header = {
    'Host': 'weibo.com',
    'Connection': 'keep-alive',
    'Accept':
    'text/html,application/xhtml+xml,application/xml;q=0.9,image/webp,*/*;q=0.8',
    'Upgrade-Insecure-Requests': 1,
    'User-Agent':
    'Mozilla/5.0 (Macintosh; Intel Mac OS X 10_11_0) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/46.0.2490.80 Safari/537.36',
    'Accept-Encoding': 'gzip, deflate, sdch',
    'Accept-Language': 'zh-CN,zh;q=0.8,en;q=0.6,zh-TW;q=0.4',
    'Cookie':'*****'
}
req = request.Request('http://weibo.com', method='GET',headers=header)

with request.urlopen(req) as f, open('sina_home.html.gz', 'wb') as out:
    out.write(f.read())
```

注意本次连接时，为了减轻IO压力，传输了压缩的内容，下载后，需要通过`gzip -cd sina_home.html.gz >sina_home.html`命令解压该文件，登陆成功，但是注意下载的只是个人首页的网页框架，并不是全部内容，新鲜事这类的，需要通过解析其他的内容才能完成。

## 3. 使用POST命令模拟微博表单登陆

pc端的微博登陆页面有些复杂，先看相对简单的[移动端微博登陆页面](https://passport.weibo.cn/signin/login)，参考[廖雪峰的Python教程](http://www.liaoxuefeng.com/wiki/0014316089557264a6b348958f449949df42a6d3a2e542c000/001432688314740a0aed473a39f47b09c8c7274c9ab6aee000)：

```python
from urllib import request, parse

print('Login to weibo.cn...')
email = input('Email: ')
passwd = input('Password: ')
login_data = parse.urlencode([
    ('username', email),
    ('password', passwd),
    ('entry', 'mweibo'),
    ('client_id', ''),
    ('savestate', '1'),
    ('ec', ''),
    ('pagerefer', 'https://passport.weibo.cn/signin/welcome?entry=mweibo&r=http%3A%2F%2Fm.weibo.cn%2F')
])

req = request.Request('https://passport.weibo.cn/sso/login')
req.add_header('Origin', 'https://passport.weibo.cn')
req.add_header('User-Agent', 'Mozilla/6.0 (iPhone; CPU iPhone OS 8_0 like Mac OS X) AppleWebKit/536.26 (KHTML, like Gecko) Version/8.0 Mobile/10A5376e Safari/8536.25')
req.add_header('Referer', 'https://passport.weibo.cn/signin/login?entry=mweibo&res=wel&wm=3349&r=http%3A%2F%2Fm.weibo.cn%2F')

with request.urlopen(req, data=login_data.encode('utf-8')) as f:
    print('Status:', f.status, f.reason)
    for k, v in f.getheaders():
        print('%s: %s' % (k, v))
    print('Data:', f.read().decode('utf-8'))
```

这一个过程没有弄很明白，先记录一下，等待之后解决：

1. 如何获取表单所需要的信息；
2. 该表单中密码使用了明文的方式，是否存在安全问题；
3. 使用了Referer头部来指定原始URI，是否存在安全问题，和上一条一样，是否是由`HTTPS`来保证安全出传输；

## 问题解决(持续更新)

首先解决之前留下的三个问题：

1. 如何获取表单内容，使用Firefox浏览器，并安装HTTPfox插件，即可跟踪消息记录，获取到POST的内容，其中就有表单信息；
2. 明文密码的安全问题，还没有搞清楚，但是可以肯定的是建立在SSL或者TSL的安全通道基础上，防止被第三方截获；
3. Referer首部，使用POST方法时，该首部不含有账号密码的信息，更加重要的是，网站为了验证是否由站点自身发起，加入首部可以伪装成站点自身；
