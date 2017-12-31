# Tiny_Web_Server
A tiny web server based on the  book “Computer Systems: A Programmer's Perspective(2nd)”
笔者参考《深入理解计算机系统（第二版）》中11.6节的内容实现了这个TINY Web服务器程序。
源代码主要来自该书提供的参考代码，笔者针对实际调试中的问题进行了少量修改。代码经过测试，可以实现相应的功能。笔者的测试环境如下：

> 操作系统：CentOS 7；
> 编译器：g++ (GCC) 4.8.5 20150623 (Red Hat 4.8.5-11)

TINY是一个迭代服务器，执行典型的无限循环服务器循环，不断监听在命令行中传递来的端口上的连接请求，处理一个HTTP事务，并关闭连接它的那一端。

注意，TINY只支持HTTP的GET方法。如果客户端请求其他方法（比如POST），TINY会给它发送一个错误信息，并返回到主程序，主程序随后关闭连接并等待下一个连接请求。否则，读并且（像我们将要看到的那样）忽略任何请求报头。

下面展示程序的使用方法和实际效果：
首先，复制源代码到linux系统中，进入tiny目录，编译源程序：

> g++ -o tiny tiny.c csapp.c -w -lpthread

添加-w（小写）选项是为了避免显示警告（主要来自《深入理解计算机系统》一书提供的源代码），-lpthread是因为该书提供的csapp.c文件中包含关于POSIX线程库函数的使用。

然后，运行编译生成的tiny程序，并指定服务器绑定的端口号（如8888）：
> ./tiny 8888

在本机再打开一个命令行终端，使用telnet命令连接到服务器：

> telnet 127.0.0.1 8888

终端上显示如下信息，说明连接成功：

> Trying 127.0.0.1...
Connected to 127.0.0.1.
Escape character is '^]'.

 1. 请求静态内容

在终端输入HTTP请求：

> GET / HTTP

上面的命令为一个请求行，其一般形式为：
`<method><uri><version>`

输入完成回车换行后，再回车换行输入一个空行，表示终止报头列表。

此时，服务器会返回如下HTTP响应，包括一个响应行（HTTP/1.0 200 OK），后面跟随零个或多个响应报头，再跟随一个终止报头的空行，最后跟随一个响应主体，这里即tiny文件夹中的home.html文件中的内容，最后服务器主动关闭连接：

```
HTTP/1.0 200 OK
Server: Tiny Web Server
Content-length: 248
Content-type: text/html

<!DOCTYPE HTML PUBLIC "-//IETF//DTD HTML 2.0//EN">
<html><head>
<title>HomePage</title>
</head><body>
<h1>HomePage</h1>
<p>Welcome to our HomePage.<br />
</p>
<hr>
<address>Apache Server at ip-127-0-0-1 Port 2000</address>
</body></html>
Connection closed by foreign host.
```
 
注:也可以在本机打开浏览器,在地址栏输入127.0.0.1:8888 查看服务器返回的静态内容。
 
 2.请求动态内容
首先需要将服务器文件夹cgi-add中的程序adder.c进行编译：

> g++ adder.c -o adder

adder属于一个CGI（Common Gateway Interface，通用网关接口）程序，当客户端请求执行相应的可执行程序（如adder）时，通过URI（Uniform Resource Identifier，统一资源标识符）向程序传入参数。可执行文件运行产生的输出即为动态内容。服务器程序通过创建子进程运行CGI程序，并将其输出返回给客户端。

对于请求动态内容，客户端的请求如下：
> GET /cgi-bin/adder?1500&213 HTTP

同样，在命令之后要在输入一个空行终止报头列表。该命令表示请求/cgi-bin/adder程序，并传入两个参数1500、213。

服务器返回的信息如下：
```
HTTP/1.0 200 OK
Server: Tiny Web Server
Content-length: 113
Content-type: text/html

Welcome to add.com: THE Internet addition portal.
<p>The answer is: 1500 + 213 = 1713
<p>Thanks for visiting!
Connection closed by foreign host.
```
可以看出服务器调用了adder，计算并返回了两个参数的和，完成了加法计算。

注:也可以在本机打开浏览器,在地址栏输入127.0.0.1:8888/cgi-bin/adder?1500&213 查看服务器返回的动态内容。
