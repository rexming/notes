一、CGI
1、基础介绍
公共网关接口CGI(Common GatewayInterface) 是WWW技术中最重要的技术之一，有着不可替代的重要地位。CGI是外部应用程序（CGI程序）与Web服务器之间的接口标准，是在CGI程序和Web服务器之间传递信息的规程。CGI规范允许Web服务器执行外部程序，并将它们的输出发送给Web浏览器。
Common Gateway Interface，简称CGI。在物理上是一段程序，运行在服务器上，提供同客户端HTML页面的接口。这样说大概还不好理解。那么我们看一个实际例子：现在的个人主页上大部分都有一个留言本。留言本的工作是这样的：先由用户在客户端输入一些信息，如名字之类的东西。接着用户按一下“留言”（到目前为止工作都在客户端），浏览器把这些信息传送到服务器的CGI目录下特定的cgi程序中，于是cgi程序在服务器上按照预定的方法进行处理。在本例中就是把用户提交的信息存入指定的文件中。然后cgi程序给客户端发送一个信息，表示请求的任务已经结束。此时用户在浏览器里将看到“留言结束”的字样。整个过程结束。
绝大多数的CGI程序被用来解释处理来自表单的输入信息，并在服务器产生相应的处理，或将相应的信息反馈给浏览器。CGI程序使网页具有交互功能。
CGI是一种基于浏览器的输入、在Web服务器上运行的程序方法。CGI脚本使你的浏览器与用户能交互，为了在数据库中寻找一个名词，提供你写入的评论，或者从一个表单中选择几个条目并且能得到一个明确的回答。如果你曾经遇到过在web上填表或进行搜索，你就是用的CGI脚本，你那时也许没有意识到，因为大部分工作是在服务器上运行的，你看到的只是结果。
2、CGI 优点
CGI可以为我们提供许多HTML无法做到的功能。比如：
a. 一个记数器
b. 顾客信息表格的提交以及统计
c. 搜索程序
d. WEB数据库
用Html是没有办法记住客户的任何信息的，就算用户愿意让你知道。用Html也是无法把信息记录到某一个特定文件里的。要把客户端的信息记录在服务器的硬盘上，就要用到CGI。这是CGI最重要的作用，它补充了Html的不足。是的，仅仅是补充，不是替代。
使在网络服务器下运行外部分应用程序（或网关）成为可能。CGI-BIN 目录是存放CGI脚本的地方。这些脚本使WWW服务器和浏览器能运行外部程序，而无需启动另一个原因程序。
CGI是运行在Web服务器上的一个程序，并由来自于浏览者的输入触发。CGI是在HTTP服务器下运行外部程序（或网关）的一个接口，它能让网络用户访问远程系统上的使用类型程序，就好像他们在实际使用那些远程计算机一样。
CGI能够让浏览者与服务器进行交互，如果你曾经遇到过在网络上填表或者进行搜索，就很有可能就是用的CGI。
尽管CGI易于使用，但是当大批人同时使用一个CGI应用程序是会反应较慢，网络服务器速度也会受到很大影响。CGI应用程序的优点是可以独立运行。
CGI应用程序可以由大多数的编程语言编写，如Perl（Practical Extraction and Report Language)、C\C++、Java 和 Visual Basic 等。不过对于那些没有太多编程经验的网页制作人来说，实在是一个不小的难题。
3、CGI应用程序的工作原理
a. 浏览器通过HTML表单或超链接请求指上一个CGI应用程序的URL。
b. 服务器收发到请求。
c. 服务器执行指定所CGI应用程序。
d. CGI应用程序执行所需要的操作，通常是基于浏览者输人的内容。
e. CGI应用程序把结果格式化为网络服务器和浏览器能够理解的文档（通常是HTML网页）。
f. 网络服务器把结果返回到浏览器中。
4、一个简单的例子
Display Date处是个指向CGI脚本的连接，它的HTML是这样的：

```
    <ahref="http://www.popchina.com/cgi-bin/getdate"
    >DisplaytheDate</a>
```
说明是个CGI脚本是因为这里面有个cgi-bin的路径，在许多服务器cgi-bin是仅能够放置CGI脚本的目录。
当你选择这个连接时，你的浏览器将向 www.popchina.com 服务器提出请求，服务器接收这个请求计算出URL处的脚本文件名然后执行这个脚本。
这个getdate脚本，在UNIX系统中执行是这样的：
```
#!/bin/sh
echoContent-type:text/plain
echo
/bin/date
```
第一行是个特殊的命令，告诉UNIX系统这是个shell脚本；真实的情况是从这行开始的下一行。这个脚本做两件事：它输出行Content-type: text/plain，接着开始一个空行；第二，它调用UNIX系统时间date程序，这样输出日期和时间。
脚本执行后输出应该这样：

```
Content-type:text/plain
TueOct::EDT
```
这个Content-type是什么东东？它是个特殊的编码，Web服务器用来告诉浏览器输出这个文本是什么类型的，这与HTML中Content-type含义是一样的。
5、环境变量列表
SERVER_NAME 运行CGI序为机器名或IP地址。
SERVER_INTERFACE WWW服务器的类型，如：CERN型或NCSA型。
SERVER_PROTOCOL 通信协议，应当是HTTP/1.0。
SERVER_PORT TCP端口，一般说来web端口是80。
HTTP_ACCEPT HTTP定义的浏览器能够接受的数据类型。
HTTP_REFERER 发送表单的文件URL。（并非所有的浏览器都传送这一变量）
HTTP_USER-AGENT 发送表单的浏览器的有关信息。
GETWAY_INTERFACE CGI程序的版本，在UNIX下为 CGI/1.1。
PATH_TRANSLATED PATH_INFO中包含的实际路径名。
PATH_INFO 浏览器用GET方式发送数据时的附加路径。
SCRIPT_NAME CGI程序的路径名。
QUERY_STRING 表单输入的数据，URL中问号后的内容。
REMOTE_HOST 发送程序的主机名，不能确定该值。
REMOTE_ADDR 发送程序的机器的IP地址。
REMOTE_USER 发送程序的人名。
CONTENT_TYPE POST发送，一般为application/xwww-form-urlencoded。
CONTENT_LENGTH POST方法输入的数据的字节数。
二、FastCGI
1、什么是FastCGI
FastCGI像是一个常驻(long-live)型的CGI，它可以一直执行着，只要激活后，不会每次都要花费时间去fork一次(这是CGI最为人诟病的fork-and-execute 模式)。它还支持分布式的运算，即 FastCGI程序可以在网站服务器以外的主机上执行并且接受来自其它网站服务器来的请求。
FastCGI是语言无关的、可伸缩架构的CGI开放扩展，其主要行为是将CGI解释器进程保持在内存中并因此获得较高的性能。众所周知，CGI解释器的反复加载是CGI性能低下的主要原因，如果CGI解释器保持在内存中并接受FastCGI进程管理器调度，则可以提供良好的性能、伸缩性、Fail-Over特性等等。
2、FastCGI的工作原理
a. Web Server启动时载入FastCGI进程管理器（IIS ISAPI或Apache Module)
b. FastCGI进程管理器自身初始化，启动多个CGI解释器进程(可见多个php-cgi)并等待来自Web Server的连接。
c. 当客户端请求到达Web Server时，FastCGI进程管理器选择并连接到一个CGI解释器。Web server将CGI环境变量和标准输入发送到FastCGI子进程php-cgi。
d. FastCGI子进程完成处理后将标准输出和错误信息从同一连接返回Web Server。当FastCGI子进程关闭连接时，请求便告处理完成。FastCGI子进程接着等待并处理来自FastCGI进程管理器(运行在Web Server中)的下一个连接。 在CGI模式中，php-cgi在此便退出了。
在上述情况中，你可以想象CGI通常有多慢。每一个Web请求PHP都必须重新解析php.ini、重新载入全部扩展并重初始化全部数据结构。使用FastCGI，所有这些都只在进程启动时发生一次。一个额外的好处是，持续数据库连接(Persistent database connection)可以工作。
3、FastCGI的不足
因为是多进程，所以比CGI多线程消耗更多的服务器内存，PHP-CGI解释器每进程消耗7至25兆内存，将这个数字乘以50或100就是很大的内存数。
Nginx 0.8.46+PHP 5.2.14(FastCGI)服务器在3万并发连接下，开启的10个Nginx进程消耗150M内存（15M*10=150M），开启的64个php-cgi进程消耗1280M内存（20M*64=1280M），加上系统自身消耗的内存，总共消耗不到2GB内存。如果服务器内存较小，完全可以只开启25个php-cgi进程，这样php-cgi消耗的总内存数才500M。
上面的数据摘自Nginx 0.8.x + PHP 5.2.13(FastCGI)搭建胜过Apache十倍的Web服务器(第6版)
三、FastCGI与CGI特点
1、如CGI、FastCGI也具有语言无关性。
2、如CGI、FastCGI在进程中的应用程序，独立于核心web服务器运行，提供了一个比API更安全的环境。(APIs把应用程序的代码与核心的web服务器链接在一起，这意味着在一个错误的API的应用程序可能会损坏其他应用程序或核心服务器；恶意的API的应用程序代码甚至可以窃取另一个应用程序或核心服务器的密钥。)
3、FastCGI技术目前支持语言有：C/C++、Java、Perl、Tcl、Python、SmallTalk、Ruby等。相关模块在Apache、ISS、Lighttpd等流行的服务器上也是可用的。
4、如CGI、FastCGI的不依赖于任何Web服务器的内部架构，因此即使服务器技术的变化，FastCGI依然稳定不变。