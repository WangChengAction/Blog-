layout: pages
title: Python专栏之一：初识爬虫
categories: python
date: 2016-05-28 23:15:14
tags: [python,scraping]
---

学习爬虫，我们首先要对网页的构成有一定的了解，比如HTML的文本格式、CSS的样式和Javascript的格式等。然后我们还需要对网络连接有一些了解，这样说起来显着比较困难，但其实做起来很简单。

好，让我们一步一步来。

环境：Python3.5  | MacOs X EICaption

### 浏览器信息的获取过程:

假设我有一台主机，然后准备连接一个服务器：

1. 首先我的主机会发送一连串的比特值，这些比特值构成的信息包括两个部分，一是请求头，二是消息体；其中，请求头中包括我的本地路由器的Mac地址以及我将要连接的服务器的IP地址，然后消息体中包含了我发出的请求；

2. 当我的路由器收到这串比特值时，会将自己的Ip地址加进去，然后发出；
3. 这段数据会经过许多中介服务器，然后最终会到达我所希望访问的服务器的Ip地址所在地；
4. 服务器在它的Ip地址收到这段数据后，服务器会读取请求头中的目标端口，然后将其传递到对应的网络服务器应用上；
5. 网络服务器收到服务器处理器传过来的数据：这是一个GET请求；请求文件index.html；
6. 网络服务器应用找到对应的html文件，然后打包成一个新的数据包发回到我的路由器中，然后传到我的主机，到这儿，浏览器的信息即获取成功。

<!--more-->
<!--0-->
    from urllib.request import urlopen
	
	html = urlopen("http://www.baidu.com")
	
	print(html.read())


上面的这段代码会输出"wwww.baidu.com"这个页面的全部Html代码，
<!--0-->

    from urllib.request import urlopen

这句话是查找Python的urllib库内的request模块，然后导入这个模块内的一个函数urlopen，这个函数的作用是用来打开并读取一个从网络获取的远程的对象。
而urllib是python的标准库，包含了从网络请求数据，处理cookie，改变请求头，用户代理等一系列的函数。
具体内容可以参考urllib的文档：[https://docs.python.org/3.5/library/urllib.html]
