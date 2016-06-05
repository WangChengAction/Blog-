layout: pages
title: Python专栏之二：Urllib库
date: 2016-05-31 15:37:28
tags: [python,scraping,urllib]
categories: python
toc: True
copyright: true
---

Urllib是python的标准库，包含了一些处理Url的模块：
- urllib.request 用于打开和读取urls
- urllib.error 包含一些urllib.request内抛出的异常
- urllib.parse 主要用于对Url进行分析
- urllib.robotparser 用于解析robots.txt文件

以上是官方文档的原文，我翻译了一下，其中robot.txt是一个文本文件，他其实是一个协议而非命令，搜索引擎访问网站的时候要看的第一个文件就是robot.txt，他可以告诉爬虫程序在服务器上什么文件是可以被查看的。
<!--more-->
>  比如：
    当一个搜索爬虫访问一个站点，它首先会检查该站点目录下是否存在robots.txt，如果存在，爬虫就会按照该文件中的内容来确定访问的范围；如果不存在，所有爬虫则均能访问网站上所有没有被口令保护的页面。

好，接下来我们分别看这四个模块。

# 1 urlllib.request模块 

urllib.request定义了一些类和函数，来帮助我们打开和读取一些urls。
它定义了如下函数：
## 1.1 urlopen()
<!--0-->

    urllib.request.urlopen(url,data=None,[timeout,]*,cafile=None,capath=None,cadefault=False，context=None)
    
打开一个url，即可以是一段字符串，也可以是一个Request对象。
data必须是一个byte对象，它指定了想要传给服务器的额外数据。若没有额外数据（即Get方法），就可以用默认值None。data也可以是可迭代对象，此时Content-length值就需要在header中指明。目前，http请求是唯一使用data的请求，如果提供了data参数，那么默认的请求类型就不是Get而是Post了。
timeout是第三个参数，用来设置等待多久超时，为了解决一些网站实现响应过慢而造成的影响。
另外几个参数，context是用来描述SSL选择的，其余的没有使用过。
该函数总是返回一个对象，用于Context Manager，并且有以下几个方法：
1. geturl() -- 返回检索到的Url，通常用于判断是否出现重新导向。
2. info() -- 返回页面的元信息，诸如headers等。
3. getcode() -- 返回response的http状态码。
4. read() -- 返回http页面的内容 

可以这样使用：
<!--0-->
    
    html = urllib.request.urlopen("www.baidu.com")
    print(html.read())

另外比较常用的函数还有：
## 1.2 Request()
<!--0-->
    
    urllib.request.Request(url,data=None,headers={},origin_req_host=None,univerifiable=False,method=None)

这个类是对url请求的抽象，实例化方法为：
<!--0-->
    
    req = urllib.request.Request("www.baidu.com",data,{},None,False,None)

其中：
url，data与之前urlopen中的用法相同。
headers必须是一个字典。他通常被用于“模仿”用户代理的头，用于通过浏览器来标识自己--一些Http服务只允许普通的浏览器的请求而抵制一些脚本。例如：Mozilla Firefox通过这样来标识它自己`“Mozilla/5.0 (X11; U; Linux i686) Gecko/20071127 Firefox/2.0.0.11”`,而urllib默认的用户代理是`“Python-urllib/2.6”`。
比如我们可以构建下面的headers：
`headers = { 'User-Agent' : 'Mozilla/4.0 (compatible; MSIE 5.5; Windows NT)','Referer':'http://www.zhihu.com/articles' }`
这样，在传送请求时把headers传入Request参数里，这样就能应付“防盗机制”了。
另外headers有一些属性需要注意下：
- User-Agent：有些服务器或Proxy会通过该值来判断是否是浏览器发出的请求
- Content-type：在使用Rest接口时，服务器回来检查该值，用来确定HTTP Body中的内容该如何进行解析(在使用服务器提供的RESTful或SOAP服务时，Content-Type设置错误会导致服务器拒绝服务)
- application/xml：在XML RPC，如RESTful/SOAP调用时使用
- application/json：在JSON RPC调用时使用
- application/x-www-form-urlencoded：浏览器提交web表单时使用

origin_req_host和unverifable只是用于对第三方Http Cookie的处理。
origin_req_host应该是最初的transaction的请求主机，通过**RF2965**进行定义。它默认为`“http.cookiejar.request_host(self)”`
method是指定http请求方式，这个必须是string类型，如‘GET’。若提供了这个参数，他会储存在method属性内，可以通过get_method()方法使用。
## 1.3 response()
<!--0-->

    urllib.response()

urllib.response模块定义了操作响应的一些函数和对象。它典型的响应对象是一个addinfourl实例，通过定义一个info()方法，来返回headers和定义geturl()方法，来返回url。通常这些函数都是通过urllib.request内部调用的。
我们在调用urllib.request.urlopen()时就会返回一个response对象

常用的函数就这些，接下来我们看urllib.error()
# 2 urllib.error模块

urllib.error模块定义了一些异常类，作用与由urllib.request引发的异常上，其基本的异常类是URLError。
接下来是urllib.error内包含的一些异常：
## 2.1 URLError()
<!--0-->
    
    urllib.error.URLError

handler发生异常的时候回抛这个错误，它有一个reason属性，是一个用来描述错误的字符串。
当网络连接出现问题（本地端、服务器端）时，就有可能出现这种异常，我们可以用try-except来捕获这种异常。
## 2.2 HTTPError()
<!--0-->

    urllib.error.HTTPError

HTTPError用来处理外部HTTP异常的时候很有用,比如验证请求。它是URLError的子类，有三个属性：code（Http的状态码），reason（对error的描述，它通常是一个字符串），headers（造成错误的Http请求的响应header）。当我们使用urlopen发出一个请求时，服务器端会有一个应答，而这个应答中包含http的状态码（如404，403等），而其他不能处理的，urlopen就会产生一个HTTPError，对应相应的状态码。HTTP的状态码由RFC 2616定义，表示网页服务器Http响应状态的3位数字代码。
我们可以看一下常用的一些状态码，所有状态码的第一个数字代表了响应的五种状态之一。
- 1xx消息：代表请求已被接受，需要继续处理。这类响应是临时响应，只包含状态行和某些可选的响应头信息，并以空行结束。如
    - 100 Continue，客户端应继续发送请求
    - 101 Switching Protocols，服务器端已经理解了客户端请求，并将通过Upgrade消息头通知客户端采用不同的协议来完成这个请求。
    - 102 Processing，代表处理将被继续执行
- 2xx成功：代表请求已成功被服务器接收，理解并接受。如
    - **200 OK，请求已成功，请求所希望的响应头或数据体随此响应返回。**
    - 201 Created，请求已经被实现，而且有一个新的资源已经依据请求的需要而创建，且其URI已经随Location头信息返回。
    - 202 Accepted，服务器已接受请求，但尚未处理。
    - 203 Non-Authoritative Information，服务器已成功处理了请求，但返回的实体头部元信息不是在原始服务器上有效的确定集合，而是来自本地或者第三方的拷贝。
    - 204 No Content，服务器成功处理了请求，但不需要返回任何实体内容，并且希望返回更新了的元信息。
    - 205 Reset Content，服务器成功处理了请求，且没有返回任何内容。但是与204响应不同，返回此状态码的响应要求请求者重置文档视图。
    - 206 Partial Content，服务器已经成功处理了部分GET请求。
    - 207 Multi-Status，代表之后的消息体将是一个XML消息，并且可能依照之前子请求数量的不同，包含一系列独立的响应代码。
- 3xx重定向：代表需要客户端采取进一步的操作才能完成请求。通常这些状态码用来重定向，后续的请求地址（重定向目标）在本次响应的Location域中指明。如
    - 300 Multiple Choices，被请求的资源有一系列可供选择的回馈信息，每个都有自己特定的地址和浏览器驱动的商议信息。用户或浏览器能够自行选择一个首选的地址进行重定向。
    - **301 Moved Permanently，被请求的资源已永久移动到新位置，并且将来任何对此资源的引用都应该使用本响应返回的若干个Url之一。**
    - 302 Found，请求的资源现在临时从不同的Url响应请求。
    - 303 See Other，对应当前请求的响应可以在另一个Url上被找到，且客户端应当才去GET的方式访问那个资源。303响应禁止被缓存。
    - 304 Not Modified，如果客户端发送了一个带条件的GET请求且该请求已被允许，而文档的内容（自上次访问以来或者根据请求的条件）并没有改变，则服务器应当返回这个状态码。304响应禁止包含消息体。
    - 305 Use Proxy，被请求的资源必须通过指定的代理才能被访问。
    - 307 Temporary Redirect，请求的资源现在临时从不同的URL响应请求。
- 4xx客户端错误：这类的状态码代表了客户端看起来可能发生了错误，妨碍了服务器的处理。除非响应的是一个Head请求，否则服务器就应该返回一个解释当前错误状况的实体，以及这是临时的还是永久的状况。如
    - 400 Bad Request，由于包含语法错误，当前请求无法被服务器理解。
    - 401 Unauthorized，当前请求需要用户验证。该响应必须包含一个适用于被请求资源的WWW-Authenticate信息头用以询问用户信息。
    - 402 Payment Required，为了将来可能的需求而预留的。
    - 403 Forbidden，服务器已经理解请求，但是拒绝执行它。
    - **404 Not Found，请求失败，请求所希望得到的资源未被在服务器上发现。没有信息能够告诉用户这个状况到底是暂时的还是永久的。**
    - 405 Method Not Allowed，请求行中指定的请求方法不能被用于请求相应的资源。该响应必须返回一个Allow头信息用以表示处当前资源能够接收的请求方法列表。
    - 406 Not Acceptable，请求的资源的内容特性无法满足请求头中的条件，因而无法生成响应实体。
    - 407 Proxy Authentication Required，与401响应类似，只不过客户端必须在代理服务器上进行身份验证。
    - 408 Request Timeout，请求超时。客户端没有在服务器预备等待的时间内完成一个请求的发送。
    - 409 Conflict，由于和被请求的资源的当前状态之间存在冲突，请求无法完成。
    - 410 Gone，被请求的资源在服务器上已经不再可用，而且没有任何已知的转发地址。
    - 411 Length Required，服务器拒绝在没有定义Content-Length头的情况下接受请求。
    - 412 Precondition Failed，服务器在验证在请求的头字段中给出先决条件时，没能满足其中的一个或多个。
    - 413 Request Entity Too Large，服务器拒绝处理当前请求，因为该请求提交的实体数据大小超过了服务器愿意或者能够处理的范围。
    - 414 Request-URL Too Long，请求的URI长度超过了服务器能够解释的长度，因此服务器拒绝对该请求提供服务。
    - 415 Unsupported Media Type，对于当前请求的方法和所请求的资源，请求中提交的实体并不是服务器中所支持的格式，因此请求被拒绝。
- 5xx服务器错误：代表服务器在处理请求的过程中有错误或异常状态发生，也有可能是服务器意识到以当前的软硬件资源无法完成对请求的处理。如
    - 500 Internal Server Error，服务器遇到了一个未曾预料的状况，导致了它无法完成对请求的处理。
    - 501 Not Implemented，服务器不支持当前请求所需要的某个功能。
    - 502 Bad Gateway，作为网关或者代理工作的服务器尝试执行请求时，从上游服务器接收到无效的响应。
    - 503 Service Unavailable，由于临时的服务器维护或者过载，服务器当前无法处理请求。
    - 504 Gateway Timeout，作为网关或者代理工作的服务器尝试执行请求时，未能及时从上游服务器（URI标识出的服务器，例如HTTP、FTP、LDAP）或者辅助服务器（例如DNS）收到响应。
    - 505 HTTP Version Not Supported，服务器不支持，或者拒绝支持在请求中使用的HTTP版本。
    - 506 Variant Also Negotiates，代表服务器存在内部配置错误：被请求的协商变元资源被配置为在透明内容协商中使用自己，因此在一个协商处理中不是一个合适的重点。

第三种异常类型
## 2.3 ContentTooShortError()
<!--0-->
    
    urllib.error.ContentTooShortError(msg,content)

这个异常是当函数urlretrieve()查询到的下载数据的数量少于期望的数量时抛出的。

# 3 urllib.parse模块 

parse的源代码在这里：[https://hg.python.org/cpython/file/3.5/Lib/urllib/parse.py]
这个模块定义了许多处理URL的函数，它可以对URL进行解析，将其解析成不同的部分，同样也可以将不同的部分拼装成URL，甚至可以将相对URL转换成绝对URL。
这个模块已经被设计为在相对Url上匹配Internet RFC。它支持下列的Url模式：`file`, `ftp`, `gopher`, `hdl`, `http`, `https`, `imap`, `mailto`, `mms`, `news`, `nntp`, `prospero`, `rsync`, `rtsp`, `rtspu`, `sftp`, `shttp`, `sip`, `sips`, `snews`, `svn`, `svn+ssh`, `telnet`, `wais`.
Urllib.parser模块定义的这些函数有两大功能：网址解析和Url引用。下面的内容中我们将会详细进行解释：
**Url Parsing （URL解析）**
Url解析部分的函数主要是用于将Url字符串解析成不同的部分，或者将不同部分的Url部分合并成Url字符串。
## 3.1 urlparse()
<!--0-->
    
    urllib.parse.urlparse(urlstring,scheme=",allow_fragments=True)

这个函数是将URL解析成六个Components，返回一个六元组。它与URl的通用架构相符合，即`schema://netloc/path;parameters?query#fragment`。每个元组项是一个字符串或为空。组件不能被解析为更小的部分，%后面也不会被解析，分隔符号并不是解析结果的一部分，除非用斜线转义。代码示例如下：
<!--0-->
    
    >>> from urllib.parse import urlparse
    >>> o = urlparse('https://docs.python.org/3.5/library/urllib.parse.html#url-parsing')
    >>> o
    ParseResult(scheme='https', netloc='docs.python.org', path='/3.5/library/urllib.parse.html', params='', query='', fragment='url-parsing')
    >>> o.scheme
    'https'
    >>> o.geturl()
    'https://docs.python.org/3.5/library/urllib.parse.html#url-parsing'
    >>> urlparse('help/Python.html')
    ParseResult(scheme='', netloc='', path='help/Python.html', params='', query='', fragment='')
    >>> urlparse('www.cwi.nl/%7Eguido/Python.html')
    ParseResult(scheme='', netloc='', path='www.cwi.nl/%7Eguido/Python.html', params='', query='', fragment='')

Urlparse 只有当正确的添加‘//’时才能识别出netloc字段，如上例子，后两个网址netlic的值都为空。
这里面：
- scheme：Url scheme specifier
- netloc：Network location part
- path：Hierarchical path
- params：parameters for last path element
- query：Query component
- fragment：Fragment identifier
## 3.2 parse_qs()
<!--0-->
    
    urllib.parse.parse_qs(qs,keep_blank_values=False,strict_parsing=False,encoding='utf-8',errors='replace')

上述函数是解析给出字符串参数的查询字符串，返回一个字典类型，字典的Key是唯一的查询变量名，Value是每个名字值的列表。
## 3.3 parse_qsl()
<!--0-->

    urllib.parse.parse_qsl(qs, keep_blank_values=False, strict_parsing=False, encoding='utf-8', errors='replace')

上述函数同样是解析给出字符串参数的查询字符串，不同的是该函数返回的是一个名字列表。
## 3.4 urlunparse()
<!--0-->

    urllib.parse.urlunparse(parts)

这个函数将`urlparse()`返回的六元组构建成一个正确格式的URL。
## 3.5 urlsplit()
<!--0-->

    urllib.parse.urlsplit(urlstring,schema=",allow_fragments=True)

它与`urlparse()`相似，用法基本一致，但略有不同，split函数在分割的时候，path属性和params属性是在一起的。如：
<!--0-->
    
    >>> from urllib.parse import urlparse
    >>> from urllib.parse import urlsplit
    >>> o = urlparse('https://docs.python.org/3.5/library/urllib.parse.html#url-parsing')
    >>> p = urlsplit('https://docs.python.org/3.5/library/urllib.parse.html#url-parsing')
    >>> o
    ParseResult(scheme='https', netloc='docs.python.org', path='/3.5/library/urllib.parse.html', params='', query='', fragment='url-parsing')
    >>> p
    SplitResult(scheme='https', netloc='docs.python.org', path='/3.5/library/urllib.parse.html', query='', fragment='url-parsing')

我们可以看到split函数并没有params属性。它主要是用来分析Urlstring，返回的是一个五元组。
## 3.6 urlunsplit()
<!--0-->

    urllib.parse.urlunsplit(parts)

这个函数的作用与`urlunparse()`类似，但它合并的是`urlsplit()`函数返回的元组，将其合并成一个完整的Url字符串。
## 3.7 urljoin()
<!--0-->

    urllib.parse.urljoin(base,url,allow_fragments=True)

这个函数主要用来拼接Url，它以base为基地址，然后同url中的相对地址结合形成一个绝对地址。通俗的说，它使用基础Url的组成部分，尤其是解决方案、网络位置和部分路径，来提供相对Url所缺少的部分。例如：
<!--0-->

    >>> from urllib.parse import urljoin
    >>> urljoin('http://www.cwi.nl/%7Eguido/Python.html', 'FAQ.html')
    'http://www.cwi.nl/%7Eguido/FAQ.html'

注意：如果url是绝对url（即由`//`或`scheme://`开始的），那么这个url的主机名或scheme会出现在结果中。例如：
<!--0-->
    >>> urljoin('http://www.cwi.nl/%7Eguido/Python.html','//www.python.org/%7Eguido')
    'http://www.python.org/%7Eguido'

若去掉url前面的`//`，则不会出现这种情况。所以，如果你不希望出现这种情况，应该先用`urlsplit()`和`urlunsplit()`对url进行预处理,消除可能的scheme和netloc部分。

