# http

#### uri vs url

都是统一资源定位方式，url是一种特殊的uri，uri是url的更高层次的抽象。

#### http post携带数据的方式

Content-Type:multipart/form-data

> 首先生成了一个 boundary 用于分割不同的字段，为了避免与正文内容重复，boundary 很长很复杂。然后 Content-Type 里指明了数据是以 multipart/form-data 来编码，本次请求的 boundary 是什么内容。消息主体里按照字段个数又分为多个结构类似的部分，每部分都是以 –boundary 开始，紧接着是内容描述信息，然后是回车，最后是字段具体内容（文本或二进制）。如果传输的是文件，还要包含文件名和文件类型信息。消息主体最后以 –boundary– 标示结束。

Content-Type:application/x-www-from-urlencoded

> 这应该是最常见的 POST 提交数据的方式了。浏览器的原生 \\\<form\\> 表单，如果不设置 enctype 属性，那么最终就会以 application/x-www-form-urlencoded 方式提交数据。Content-Type 被指定为 application/x-www-form-urlencoded；其次，提交的数据按照 key1=val1\&key2=val2 的方式进行编码，key 和 val 都进行了 URL 转码。大部分服务端语言都对这种方式有很好的支持。如果上传的数据有层次结构，不适用这种方式

Content-Type:application/octet-stream

> 二进制数据

RAW:直接发送数据，可以上传任意格式的文本，可以上传text、json、xml、html等Content-Type: text/xmlContent-Type:application/json

> 可以方便的提交复杂的结构化数据，特别适合 RESTful 的接口。

#### Http Entity 的种类

HttpClient区分三种实体，根据其内容来源：streamed（流）：内容从流中接收的。流实体是不可重复的。self-contained（自包含）：内容在内存中或者从连接或其它实体自主获得的。自包含的实体通常都是可重复的。这种类型的实体通常用于entity enclosing request。wrapping（包装）：内容从其它实体获得。UrlEncodedFormEntity只能添加键值对，final List \<? extends NameValuePair> parameters,所以application/json 和 text/\* 这些类型的信息都不适用这个EntityStringEntity可以放任何类型的string类型[四种常见的 POST 提交数据方式](https://imququ.com/post/four-ways-to-post-data-in-http.html)

#### http 转发 与 重定向

转发是服务器行为，重定向是客户端行为。为什么这样说呢，这就要看两个动作的工作流程：转发过程：客户浏览器发送http请求—-》web服务器接受此请求–》调用内部的一个方法在容器内部完成请求处理和转发动作—-》将目标资源发送给客户；在这里，转发的路径必须是同一个web容器下的url，其不能转向到其他的web路径上去，中间传递的是自己的容器内的request。在客户浏览器路径栏显示的仍然是其第一次访问的路径，也就是说客户是感觉不到服务器做了转发的。转发行为是浏览器只做了一次访问请求。重定向过程：客户浏览器发送http请求—-》web服务器接受后发送302状态码响应及对应新的location给客户浏览器–》客户浏览器发现是302响应，则自动再发送一个新的http请求，请求url是新的location地址—-》服务器根据此请求寻找资源并发送给客户。在这里location可以重定向到任意URL，既然是浏览器重新发出了请求，则就没有什么request传递的概念了。在客户浏览器路径栏显示的是其重定向的路径，客户可以观察到地址的变化的。重定向行为是浏览器做了至少两次的访问请求的。

#### http重定向3XX

301状态码在HTTP 1.0和HTTP 1.1规范中均代表永久重定向在http 1.0规范中，302表示临时重定向，location中的地址不应该被认为是资源路径，在后续的请求中应该继续使用原地址。在HTTP 1.1中，实际上302是不再推荐使用的，只是为了兼容而作保留。规范中再次重申只有当原请求是GET or HEAD方式的时候才能自动的重定向，为了消除HTTP 1.0中302的二义性，在HTTP 1.1中引入了303和307来细化HTTP 1.0中302的语义。303继承了HTTP 1.0中302的实现（即原请求是post，也允许自动进行重定向，结果是无论原请求是get还是post，都可以自动进行重定向），而307则继承了HTTP 1.0中302的规范（即如果原请求是post，则不允许进行自动重定向，结果是post不重定向，get可以自动重定向）[http重定向301/302/303/307](https://blog.csdn.net/reliveIT/article/details/50776984)

#### 携带cookie, session

session与session实现session是会话信息session实现可以借助cookie或URL重写。核心是要带上sessionid，自己想实现也是可以的。URL重写

1. 保证WebApplication在进行结构调整，移动页面位置时，用户收藏的URL不会因此而成为死链。
2. SEO优化。[URL重写-wikipedia](https://zh.wikipedia.org/wiki/URL%E9%87%8D%E5%AF%AB)

一般来说，URL重写是支持会话的非常健壮的方法。在不能确定浏览器是否支持Cookie的情况下应该使用这种方法。然而，使用URL重写应该注意下面几点：

1. 如果使用URL重写，应该在应用程序的所有页面中，对所有的URL编码，包括所有的超链接和表单的action属性值。
2. 应用程序的所有的页面都应该是动态的。因为不同的用户具有不同的会话ID，因此在静态HTML页面中无法在URL上附加会话ID。
3. 所有静态的HTML页面必须通过Servlet运行，在它将页面发送给客户时会重写URL。[URL重写-百度百科](https://baike.baidu.com/item/url%E9%87%8D%E5%86%99)

cookie

#### http 携带登录信息

#### https连接

#### http代理

使用http代理之后的请求头部变化：1、连接的不是目标服务器的IP地址和端口而是代理服务器IP地址和端口2、提交的不是相对的地址而是绝对的HTTP地址3、Connection: Keep-Alive 和 Proxy-Connection: Keep-Alive

#### 反向代理

#### CDN

#### http2.0

[http2.0的时代真的来了…](https://www.jianshu.com/p/712eb3a65d33)
