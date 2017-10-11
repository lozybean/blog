Title: HTTP初探及python实现
Date: 2015-10-26
Category: Learning
Tags: python, http

## HTTP协议

`HTTP`协议(*HyperText Transfer Protocol* 超文本转移协议，常又被称为超文本传输协议)是TCP/IP协议簇中的其中一种应用层协议，广泛应用于网页服务器和浏览器之间的通信。HTTP协议建立在`TCP`协议的基础上，所以在建立HTTP通信之前必须建立TCP连接，HTTP的请求和响应必须符合协议规范，是比TCP更高层次的具体规定。

HTTP的请求和响应都由几个部分组成：`请求行/状态行`、`请求首部/响应首部`、`请求实体/响应实体`，前两个部分统称`报文首部`，和`报文实体`通过`\r\n`分割（有一些服务器支持只由`\n`分割）。

一个HTTP请求实例：

```http
POST /uniform/resource/identifier HTTP/1.1
Host: www.sina.com
Content-Length: 1024
(空行\r\n)
//实体内容
```

上面的例子中，第一行由三个部分组成：`方法名`、`URI`、`协议版本`，第二行开始一直到`\r\n`位置，都是HTTP的首部字段，定义一系列关于本次请求的详细信息，`\r\n`开始到结束是请求的实体部分。通过这样的格式告诉服务器这次请求想干什么。

一个HTTP响应实例：

```http
HTTP/1.1 200 OK
Server: Apache
(空行\r\n)
//实体内容
```

服务器收到请求之后，返回类似于上面的响应，第一行一样由三个部分组成：`协议版本`、`状态码`、`原因短语`，以及第二行开始到空行的各种响应头，最后是响应的实体。

更多关于HTTP的内容，可以参考*《图解HTTP》*。

## Python发送HTTP请求

使用`httplib`以及`httplib2`等库，都可以发送HTTP请求，以`httplib2`为例：

```python
#!/usr/bin/env python
# -*- coding: utf-8 -*- #

import httplib2

http = httplib2.Http()

headers = {
    'Host' : '127.0.0.1'
}
uri = 'http://127.0.0.1:8080/index.html'
response, content = http.request(uri,'GET', headers=headers)
print response.__dict__
print response,content
```

上述过程实现从本机的`8080`端口的HTTP服务器上访问主页内容，这个过程是每次访问网页时，浏览器做的基本工作，但是浏览器还需要做的工作非常多。

上述实现中，使用了有关http的类库，将连接的过程封装了起来，这样做对于使用是非常有好处的，但是为了更好理解HTTP工作的过程，应该有更底层的考察：

```python
#!/usr/bin/env python
# -*- coding: utf-8 -*- #

import socket

s = socket.socket()
s.connect(('127.0.0.1', 8080))
head_line = 'GET /index.html HTTP/1.1\r\n'
headers = 'Host : 127.0.0.1\r\n'
s.send(head_line + headers + '\r\n' + '')
data = s.recv(1024)
print data
```

## Python搭建HTTP服务器

建立连接光有客户端还不够，还需要一个服务器，下面将用三种方法建立一个简单的HTTP服务器，都是基于Python。

最简单的方法，只需要一个命令即可：

```python
python -m SimpleHTTPServer 8080
```

更加详细地如下：

```python
#!/usr/bin/env python
# -*- coding: utf-8 -*- #

from BaseHTTPServer import HTTPServer
from SimpleHTTPServer import SimpleHTTPRequestHandler

server_class = HTTPServer
handler_class = SimpleHTTPRequestHandler
Protocol = 'HTTP/1.1'
Hostname = '127.0.0.1'
Port = 8080
server_address = (Hostname,Port)

handler_class.protocol_version = Protocol
httpd = server_class(server_address, handler_class)
while 1:
    httpd.handle_request()
```

上述两种方法都是一样的方式，都是通过Python内置的简单服务来搭建最简单的HTTP服务器，然而这个过程封装了一些东西，并不是特别直观，最直观的服务应该从TCP连接开始，从TCP到HTTP的转换过程可以更加利于理解：

```python
#!/usr/bin/env python
# -*- coding: utf-8 -*- #

import socket
import os
from threading import Thread, activeCount
CWD = os.getcwd()


def parse_head_line(head_line):
    pos_a = head_line.find(' ')
    pos_b = head_line.rfind(' ')
    method = head_line[:pos_a]
    protocol = head_line[pos_b + 1:]
    url = head_line[pos_a:pos_b].strip()
    url = CWD + url
    return method, url, protocol


def tcplink(sock, add):
    try:
        data = sock.recv(1024)
        space_line = data.find('\r\n')
        head = data[:space_line]
        data = data[space_line + 1:]
        head_line = head.split('\r\n').pop(0)
        method, url, protocol = parse_head_line(head_line)
        # do something about method
        print method, url, protocol
        request_content = ''
        if method == 'GET':
            with open(url) as fp:
                request_content = fp.read()
        head_line = 'HTTP/1.1 200 OK\n'
        head = 'Content-Type: text/html\n'
        request = head_line + head + '\r\n' + request_content
        sock.send(request)
    finally:
        sock.close()


if __name__ == '__main__':
    s = socket.socket()
    s.bind(('127.0.0.1', 8080))
    s.listen(5)
    while 1:
        sock, add = s.accept()
        t = Thread(target=tcplink, args=(sock, add))
        t.start()
        while 1:
            if activeCount() < 5:
                break
```

上面的过程反应了HTTP协议的内部细节，也是各种HTTP服务实现的基础，但是如果到实际应用上，并发、IO等各种问题需要解决，使用一个成熟的服务器软件是比较好的选择，例如nginx，搭建nginx服务器非常简单：

```bash
brew install nginx
nginx -h  # 查看帮助信息，里面有一个配置文件，修改配置文件可以改变监听端口、根目录等配置
nginx     # 使用默认配置文件启动
nginx -p $PWD  # 以当前路径为前缀路径启动（根目录根据设置不同，可能为prefix/html等）
nginx -c user.conf # 使用自定义配置启动
nginx -s stop # 发送以下四个命令控制服务：stop, quit, reopen, reload
```

## HTTP协议中的认证方式

HTTP协议内的认证方式有两种：`BASIC`认证、`DIGEST`认证，这两种方式都是通过客户端请求->服务器认证请求->客户端发送认证信息这样三步完成，不同的是DIGEST认证增加了质询码的加密方式，但是总体来说，这两种方式都不安全且不灵活。

`SSL`(Secure Socket Layer)客户端认证：该认证并不属于HTTP协议内容。通过值得信赖的第三方证书来完成认证过程，在SSL建立的安全线路基础上进行的HTTP通信，称为`HTTPS`，这种方式HTTP并不直接和TCP对接，而是和SSL协议对接，由于第三方介入以及双因素认证等更加安全的措施，是一种比较安全的认证方式。但是向第三方证书发行机构购买证书会产生客观的额外费用。

表单认证是通常网页采取的认证方式，通过表单将用户名和密码与服务器的信息做匹配来完成。表单验证实际上是服务器的一个应用，所以安全性取决于服务器应用的设计，但是不会产生多余费用，基本上的网页都会采取表单认证的方式。
