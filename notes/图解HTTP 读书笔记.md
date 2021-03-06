## 图解HTTP 读书笔记

### 第 1 章	了解 Web 及网络基础

<div align="center"> <img src="https://raw.githubusercontent.com/BufferedStream/cs-learning-notes/master/notes/images/%E5%9B%BE%E8%A7%A3HTTP%20-%20%E5%9B%BE1.jpg"/> </div><br>
URI 是 Uniform Resource Identifier 的缩写，表示统一资源标识符。URI 用字符串标识某一互联网资源，而 URL 表示资源的地点（互联网上所处的位置）。可见 URL 是 URI 的子集。





### 第 2 章	简单的 HTTP 协议

HTTP 协议用于客户端和服务器之间的通信。

请求访问文本或图像等资源的一端称为客户端，而提供资源响应的一端称为服务器端。



转载知乎：

[HTTP为什么要设计成无状态的，这样比有状态有什么好处，有哪些协议是典型的有状态，好处又在哪里？](https://www.zhihu.com/question/265610863)

首先，要定义一下什么叫【无状态】。 假设用户A向服务B发了一个请求1，再次发送一个请求2。 服务端本身完全不知道两个请求来自同一个用户，这在协议层次就是【无状态】的。

【无状态】设计不是因为xx所说的历史原因，而是故意为之。无状态就意味着服务端可以根据需要将请求分发到集群的任何一个节点，对缓存、负载均衡有明显的好处，这一点很容易找到相关文献。

很多人对【无状态】感到不理解，大部分情况是误解了【无状态】的含义。http【无状态】仅仅是在***协议层\***，当业务需要状态的时候，可以通过request中数据携带所需状态的id来实现。例如，为了让服务器知道是同一个用户的请求，请求1和请求2中必须携带一个相同的id，让服务端可以根据这个id，最终找到用户数据（【状态】）。

实现1：这个状态如果放在处理请求的服务器进程中（例如session），那服务器进程就是有状态的，该用户下一个请求如果没分发到这个进程，就会拿不到上一次请求留下的状态，这样会影响负载均衡和缓存的实现。

实现2：这个状态如果放在处理请求的服务器进程之外的集中式存储，那服务器进程仍然是无状态的，可以集群、负载均衡。无状态服务一般都用这种方案。

最后我的观点是：无状态和有状态服务适合不同的场景，并没有绝对的优劣。



#### 告知服务器意图的 HTTP 方法

**GET**：	获取资源

GET 方法用来请求访问已被 URI 识别的资源。指定的资源经服务器端解析后返回响应内容。也就是说，如果请求的资源是文本，那就保持原样返回：如果是像 CGI（Common Gateway Interface，通用网关接口）那样的程序，则返回经过执行后的输出结果。



**POST**：	传输实体主体

POST 方法用来传输实体的主体。

虽然用 GET 方法也可以传输实体的主体，但一般不用 GET 方法进行传输，而是用 POST 方法。虽说 POST 的功能与 GET 很相似，但 POST 的主要目的并不是获取响应的主题内容。



**PUT**：	传输文件

PUT 方法用来传输文件。就像 FTP 协议的文件上传一样，要求在请求报文的主体中包含文件内容，然后保存到请求 URI 指定的位置。

但是，鉴于 HTTP/1.1 的 PUT 方法自身不带验证机制，任何人都可以上传文件，存在安全性问题，因此一般的 Web 网站不使用该方法。若配置 Web 应用程序的验证机制，或架构设计采用 RESR（REpresentational State Transfer，表征状态转移）标准的同类 Web 网站，就可能会开放使用 PUT 方法。



**HEAD**：	获得报文首部

HEAD 方法和 GET 方法一样，只是不返回报文主体部分。用于确认 URI 的有效性及资源更新的日期时间等。



**DELETE**：	删除文件

DELETE 方法用来删除文件，是与 PUT 相反的方法。DELETE 方法按请求 URI 删除指定的资源。

但是，HTTP/1.1 的 DELETE 方法本身和 PUT 方法一样不带验证机制，所以一般的 Web 网站也不使用 DELETE 方法。当配合 Web 应用程序的验证机制，或遵守 REST 标准时还是有可能会开放使用的。



**OPTIONS**：	询问支持的方法

OPTIONS 方法用来查询针对请求 URI 指定的资源支持的方法。



**TRACE**：	追踪路径

TRACE 方法是让 Web 服务器端将之前的请求通信环回给客户端的方法。

发送请求时，在 Max-Forwards 首部字段中填入数值，每经过一个服务器端就将改数字减 1，当数值刚好减到 0 时，就停止继续传输，最后接收到请求的服务器端则返回状态码 200 OK 的响应。

客户端通过 TRACE 方法可以查询发送出去的请求是怎样被加工修改/篡改的。这是因为，请求想要连接到源目标服务器可能会通过代理中转的，TRACE 方法就是用来确认连接过程中发生的一系列操作。

但是，TRACE 方法本来就不怎么常用，再加上它很容易引发 XST（Corss-Site Tracing，跨站追踪）攻击，通常就更不会用到了。



**CONNECT**：	要求用隧道协议连接代理

CONNECT 方法要求在与代理服务器通信时建立隧道，实现用隧道协议进行 TCP 通信。主要使用 SSL（Secure Sockets Layer，安全套接层）和 TLS（Transport Layer Security，传输层安全）协议把通信内容加密后经网络隧道传输。



转载知乎：

[post 相比get 有很多优点，为什么现在的HTTP通信中大多数请求还是使用get？](https://www.zhihu.com/question/31640769)

[GET 和 POST 到底有什么区别？](https://www.zhihu.com/question/28586791)



这个问题虽然看上去很初级，但实际上却涉及到方方面面，这也就是为啥面试里老爱问这个的原因之一。

HTTP 最早被用来做浏览器与服务器之间交互 HTML 和表单的通讯协议；后来又被被广泛的扩充到接口格式的定义上。所以在讨论 GET 和 POST 区别的时候，需要先确定下到底是浏览器使用的 GET/POST 还是用 HTTP 作为接口传输协议的场景。



**浏览器的GET和POST**

这里特指浏览器中**非** Ajax 的 HTTP 请求，即从 HTML 和浏览器诞生就一直使用的 HTTP 协议中的 GET/POST。浏览器用 GET 请求来获取一个 html 页面/图片/css/js 等资源；用 POST 来提交一个 form 表单，并得到一个结果的网页。

浏览器将 GET 和 POST 定义为：



**GET**

“读取” 一个资源。比如 Get 到一个 html 文件。反复读取不应该对访问的数据有副作用。比如 “GET 一下，用户就下单了，返回订单已受理。”，这是不可接受的。没有副作用被称为 “幂等”（Idempotent）。

因为 GET 是读取，就可以对 GET 请求的数据做缓存。这个缓存可以做到浏览器本身上（彻底避免浏览器发请求），也可以做到代理上（如 nginx），或者做到 server 端（用 Etag，至少可以减少带宽消耗）。

GET 的 URL 会被放在浏览器历史和 WEB 服务器日志里面。



**POST**

在页面里 form 标签会定义一个表单。单击其中的 submit 元素会发出一个 POST 请求让服务器做一件事。这件事往往是有副作用的，不幂等的。

不幂等也就意味着不能随意多次执行。因此也就不能缓存。比如通过 POST 下一个单，服务器创建了新的订单，然后返回订单成功的页面。这个页面不能被缓存。试想一下，如果 POST 请求被浏览器缓存了，那么下单请求就可以不向服务器发请求，而直接返回本地缓存的 “下单成功界面”，却又没有真的在服务器下单。那是一件多么滑稽的事情。

因为 POST 可能有副作用，所以浏览器实现为不能把 POST 请求保存为书签。想想，如果点一下书签就下一个单，是不是很恐怖？

此外如果尝试重新执行 POST 请求，浏览器也会弹一个框提示下这个刷新可能会有副作用，询问要不要继续。

当然，服务器的开发者完全可以把 GET 实现为有副作用；把 POST 实现为没有副作用。只不过这和浏览器的预期不符。**把 GET 实现为有副作用是个很可怕的事情**。我依稀记得很久之前百度贴吧因为有一个因为使用 GET 请求可以修改管理员的权限而造成的安全漏洞。反过来，把没有副作用的请求用 POST 实现，浏览器该弹框还是会弹框，对用户体验感改善不大。

但是后边可以看到，将 HTTP POST 作为接口的形式使用时，就没有这种弹框了。于是把一个 POST 请求实现为幂等就有实际的意义。POST 幂等能让很多业务的前后端交互更顺畅，以及避免一些因为前端 bug，触控失误等带来的重复提交。将一个有副作用的操作实现为幂等必须得从业务上能定义出怎么算是 “重复”。如提交数据中增加一个 dedupKey 在一个交易会话中有效，或者利用提交的数据里可以天然当 dedupKey 的字段。这样万一用户强行重复提交，服务器端可以做一次防护。

GET 和 POST 携带数据的格式也有区别。当浏览器发出一个 GET 请求时，就意味着要么是用户自己在浏览器的地址栏输入，要不就是点击了 html 里 a 标签的 href 中的 url。所以其实并不是 GET 携带数据只能用 url，而是浏览器直接发出的 GET 只能由一个 url 触发。所以没办法，GET 上要在 url 之外带一些参数就只能依靠 url 上附带 querystring。但是 HTTP 协议本身并没有这个限制。

浏览器的 POST 请求都来自表单提交。每次提交，表单的数据被浏览器用编码加到 HTTP 请求的 body 里。浏览器发出的 POST 请求的 body 主要有两种格式，一种是 application/x-www-form-urlencoded 用来传输简单的数据，大概就是 “key1=value1&key2=value2” 这样的格式。另外一种是传文件，会采用 multipart/form-data 格式。采用后者是因为 application/x-ww-from-urlencoded 的编码方式对于文件这种二进制的数据非常低效。

浏览器在 POST 一个表单时，url 上也可以带参数，只要  <form action="url"> 里的 url 带 querystring 就行。只不过表单里面的那些用 input 等标签经过用户操作产生的数据都会在 body 里。

因此我们一般会泛泛的说 “GET 请求没有 body，只有 url，请求数据放在 url 的 querystring 中；POST 请求的数据在 body 中”。但这种情况仅限于浏览器发请求的场景。



**接口中的 GET 和 POST**

这里是指通过浏览器的 Ajax api，或者 IOS/Android 的 App 的 http client，java 的 commons-httpcliet/okhttp 或者是 curl，postman 之类的工具发出来的 GET 和 POST 请求。此时 GET/POS 不光能用在前端和后端的交互中，还能用在后端各个子服务的调用中（即当一种 RPC 协议使用）。尽管 RPC 有很多协议，比如 thrift，grpc，但是 http 本身已经有大量的现成的支持工具可以使用，并且对人类很友好，容易 debug。HTTP 协议在微服务中的使用是相对普遍的。

当用 HTTP 实现接口发送请求时，就没有浏览器中那么多限制了，只要是符合 HTTP 格式的就可以发。HTTP 请求的格式，大概是这样的一个字符串（为了美观，我在\r\n后都换行一下）：

```http
<METHOD> <URL> HTTP/1.1\r\n
<Header1>: <HeaderValue1>\r\n
<Header2>: <HeaderValue2>\r\n
...
<HeaderN>: <HeaderValueN>\r\n
\r\n
<Body Data....>
```

其中的 ”<METHOD>“ 可以是 GET 也可以是 POST，或者其他的 HTTP Method，如 PUT、DELETE、OPTION......。从协议本身看，并没有限制说 GET 一定不能没有 body，POST 就一定不能把参数放到 <URL> 的 querystring 上。因此其实可以更加自由的去利用格式。比如 Elastic Search 的 _search api 就用了带 body 的 GET；也可以自己开发接口让 POST 一半的参数放在 url 的 querystring 里，另外一半放 body 里；你甚至还可以让所有的参数都放在 Header 里——可以做各种各样的定制，只要请求的客户端和服务器端都能够约定好。

当然，太自由也带来了另一种麻烦，开发人员不得不每次讨论确定参数是放 url 的 path 里，querystring 里，body 里，header 里这种问题，太低效了。于是就有了一些列接口规范/风格。其中名气最大的当属 REST。REST 充分使用 GET、POST、PUT 和 DELETE，约定了这 4 个接口分别获取、创建、替换和删除 ”资源“，REST 最佳实践还推荐在请求体使用 json 格式。这样仅仅通过看 HTTP 的 method 就可以明白接口是什么意思，并且解析格式也得到了统一。

json 相对于 x-www-from-urlencoded 的优势在于 1）可以有嵌套结构；以及 2）可以支持更丰富的数据类型。通过一些框架，json 可以直接被服务器代码映射为业务实体。用起来十分方便。但是如果是写一个接口支持上传文件，那么还是 multipart/form-data 格式更合适。

REST 中 GET 和 POST 不是随便用的。在 REST 中，【GET】 + 【资源定位符】被专用于获取资源或者资源列表，比如：

```http
GET http://foo.com/books          获取书籍列表
GET http://foo.com/books/:bookId  根据bookId获取一本具体的书
```

与浏览器的场景类似，REST GET 也不应该有副作用，于是可以被反复无脑调用。浏览器（包括浏览器的 Ajax 请求）对于这种 GET 也可以实现缓存（如果服务器端提示了明确需要 Caching）；但是如果用非浏览器，有没有缓存完全看客户端的实现了。当然，也可以从整个 App 角度，也可以完全绕开非浏览器的缓存机制，实现一套业务定制的缓存框架。

REST 【POST】 + 【资源定位符】则用于 ”创建一个资源“，比如：

```text
POST http://foo.com/books
{
  "title": "大宽宽的碎碎念",
  "author": "大宽宽",
  ...
}
```

这里你就能留意到浏览器中用来实现表单提交的 POST，和 REST 实现创建资源的 POST 语义上的不同。

顺便讲下 REST POST 和 REST PUT 的区别。有些 api 是使用 PUT 作为创建资源的 Method。PUT 与 POST 的区别在于，PUT 的实际语义是 ”replace“ replace。REST 规范里提到 PUT 的请求体应该是完整的资源，包括 id 在内。比如上面的创建一本书的 api 也可以定义为：

```text
PUT http://foo.com/books
{
  "id": "BOOK:affe001bbe0556a",
  "title": "大宽宽的碎碎念",
  "author": "大宽宽",
  ...
}
```



服务器应该先根据请求提供的 id 解析查找，如果存在一个对应 id 的元素，就用请求中的数据整体替换已经存在的资源；如果没有，就用 ”把这个 id 对应的资源从 【空】 替换为 【请求数据】“。直观看起来就是 ”创建“ 了。

与 PUT 相比，POST 更像是一个 “factory”，通过一组必要的数据创建出完整的资源。
至于到底用 PUT 还是 POST 创建资源，完全要看是不是提前可以知道资源所有的数据（尤其是 id），以及是不是完整替换。比如对于 AWS S3 这样的对象存储服务，当想上传一个新资源时，其 id就是 “ObjectName” 可以提前知道；同时这个 api 也总是完整的 replace 整个资源。这时的 api 用 PUT 的语义更合适；而对于那些 id 是服务器端自动生成的场景，POST 更合适一些。

有点跑题，就此打住。

回到接口这个主题，上面仅仅粗略介绍了 REST 的情况。但是现实中总是有 REST 的变体，也可能用非 REST 的协议（比如 JSON-RPC、SOAP 等），每种情况中的 GET 和 POST 又会有所不同。



**关于安全性**

我们常常听到 GET 不如 POST安全，因为 POST 用 body 传输数据，而 GET 用 url 传输，更加容易看到。但是从攻击的角度，无论是 GET 还是 POST 都不够安全，因为 HTTP 本身是**明文协议**。**每个 HTTP 请求和返回的每个 byte 都会在网络上明文传播，不管是 url，header 还是 body**。这完全不是一个 ”是否容易在浏览器地址栏上看到“ 的问题。

为了避免传输中数据被窃取，必须做从客户端到服务器的端端加密。业界的通行做法就是 https——即用 SSL 协议协商出的密钥加密明文的 http 数据。这个加密的协议和 HTTP 协议本身相互独立。如果是利用 HTTP 开发公网的站点/App，要保证安全，https 是最最基本的要求。

当然，端端加密并不一定非得用 https。比如国内金融领域都会用私有网络，也有 GB 的加密协议 SM 系列。但除了军队，金融等特殊机构之外，似乎并没有必要自己发明一套类似于 ssl 的协议。

回到 HTTP 本身，的确 GET 请求的参数更倾向于放在 url 上，因此有更多机会被泄露。比如携带私密信息的 url 会展示在地址栏上，还可以分享给第三方，就非常不安全了。此外，从客户端到服务器端，有大量的中间节点，包括网关，代理等。他们的 access log 通常会输出完整的 url，比如 nginx 的默认 acces log 就是如此。如果 url 上携带敏感数据，就会被记录下来。但请注意，**就算私密数据在 body 里，也是可以被记录下来的**，因此如果请求要经过不信任的公网，避免泄密的**唯一手段就是 https**。这里说的 ”避免 acces log 泄露“ 仅仅是指避免可信区域中的 http 代理的默认行为带来的安全隐患。比如你是不太希望让自己公司的运维同学从公司主网关的 log 里看到用户的密码吧。

另外，上面讲过，如果是用作接口，GET 实际上也可以带 body，POST 也可以在 url 上携带数据。所以实际上到底怎么传输私密数据，要看具体场景具体分析。当然，绝大多数场景，用 POST + body 里写私密数据是合理的选择。一个典型的例子就是 “登录”：

```text
POST http://foo.com/user/login
{
  "username": "dakuankuan",
  "passowrd": "12345678"
}
```

安全是一个巨大的主题，有由很多细节组成的一个完备体系，比如返回私密数据的mask，XSS，CSRF，跨域安全，前端加密，钓鱼，salt，…… POST和GET在安全这件事上仅仅是个小角色。因此单独讨论POST和GET本身哪个更安全意义并不是太大。只要记得一般情况下，私密数据传输用POST + body就好。

**关于编码**

常见的说法有，比如 GET 的参数只能支持 ASCII，而 POST 能支持任意 binary，包括中文。但其实从从上面可以看到，GET 和 POST 实际上都能用 url 和 body。因此所谓编码确切地说应该是 http 中 url 有什么编码，body 用什么编码。

先说下 url。url 只能支持 ASCII 地说法源自于 RFC1738

实际上这里规定的仅仅是一个 ASCII 的子集 [a-zA-Z0-9$-_.+!*'(),]。它们是可以“不经编码”在 url 中使用。比如尽管空格也是 ASCII 字符，但是不能直接用在 url 里。

那这个 “编码” 是什么呢？如果有了特殊符号和中文怎么办呢？一种叫做 percent encoding 的编码方法就是干这个用的。

百分号编码（Percent-Encoding）也被称为 URL 编码，是一种编码机制。该机制主要应用于 URI 编码中，URI 包含 URL 和 URN，所以它们也同样适用。除此之外，也用于 MIME 类型为 "application/x-www-form-urlencoded" 的内容。

百分号编码会对 URI 中不允许出现的字符或者其他特殊情况的允许的字符进行编码，对于被编码的字符，最终会转为以百分号 "%“ 开头，后面跟着两位 16 进制数值的形式。举个例子，空格符（SP）是不允许的字符，在 ASCII 码对应的二进制值是 "00100000”，最终转为 "%20"。这也就是为啥我们偶尔看到url里有一坨%和16位数字组成的序列。

使用 Percent Encoding，即使是 **binary data，也是可以通过编码后放在 URL 上**的。

但要特别注意，这个**编码方式只管把字符转换成 URL 可用字符，但是却不管字符集编码**（比如中文到底是用 UTF8 还是 GBK）这块早期一直都相当乱，也没有什么统一规范。比如有时跟网页编码一样，有的是操作系统的编码一样。最要命的是浏览器的地址栏是不受开发者控制的。这样，对于同样一个带中文的 url，如果有的浏览器一定要用 GBK（比如老的 IE8），有的一定要用 UTF8（比如 chrome）。后端就可能认不出来。对此常用的办法是避免让用户输入这种带中文的 url。如果有这种形式的请求，都改成用户界面上输入，然后通过 Ajax 发出的办法。Ajax 发出的编码形式开发者是可以 100% 控制的。

不过目前基本上 utf8 已经大一统了。现在的开发者除非是被国家规定要求一定要用 GB 系列编码的场景，基本上不会再遇到这类问题了。 

顺便说一句，尽管在浏览器地址栏可以看到中文。但这种 url 在发送请求过程中，浏览器会把中文用字符编码 + Percent Encode 翻译为真正的 url，再发给服务器。浏览器地址栏里的中文只是想让用户体验好些。

再讨论下 Body。HTTP Body 相对好些，因为有个 Content-Type 来比较明确的定义。比如：

```http
POST xxxxxx HTTP/1.1
...
Content-Type: application/x-www-form-urlencoded ; charset=UTF-8
```

这里 Content-Type 会同时定义请求 body 的格式（application/x-www-form-urlencoded）和字符编码（UTF-8）。

所以 body 和 url 都可以提交中文数据给后端，但是 POST 的规范好一些，相对不容易出错，容易让开发者安心。对于 GET+url 的情况，只要不涉及到在老旧浏览器的地址栏输入 url，也不会有什么太大的问题。

回到 POST，浏览器直接发出的 POST 请求就是表单提交，而表单提交只有 application/x-www-form-urlencoded 针对简单的 key-value 场景；和 multipart/form-data，针对只有文件提交，或者同时有文件和 key-value 的混合提交表单的场景。

如果是 Ajax 或者其他 HTTP Client 发出去的 POST 请求，其 body 格式就非常自由了，常用的有json，xml，文本，csv…… 甚至是你自己发明的格式。只要前后端能约定好即可。



**浏览器的 POST 需要发两个请求吗？**

上文中的 “HTTP 格式” 清楚的显示了 HTTP 请求可以被大致分为 “请求头”和“请求体”两个部分。使用 HTTP 时大家会有一个约定，即所有的 “控制类” 信息应该放在请求头中，具体的数据放在请求体里。于是服务器端再解析时，总是会先完全解析全部的请求头部。这样，服务器端总是希望能够了解请求的控制信息后，就能决定这个请求怎么进一步处理，是拒绝，还是根据 content-type 去调用相应的解析器处理数据，或者直接用 zero copy 转发。

比如在用 Java 写服务时，请求处理代码总是能从 HttpServletRequest 里 getParameter/Header/url。这些信息都是请求头里的，框架直接就解析了。而对于请求体，只提供了一个 inputstram，如果开发人员觉得应该进一步处理，就自己去读取和解析请求体。这就能体现出服务器端对请求头和请求体的不同处理方式。

举个实际的例子，比如写一个上传文件的服务，请求url中包含了文件名称，请求体中是个尺寸为几百兆的压缩二进制流。服务器端接收到请求后，就可以先拿到请求头部，查看用户是不是有权限上传，文件名是不是符合规范等。如果不符合，就不再处理请求体的数据了，直接丢弃。而不用等到整个请求都处理完了再拒绝。

为了进一步优化，客户端可以利用 HTTP 的 Continued 协议来这样做：客户端总是先发送所有请求头给服务器，让服务器校验。如果通过了，服务器回复 “100 - Continue”，客户端再把剩下的数据发给服务器。如果请求被拒了，服务器就回复个 400 之类的错误，这个交互就终止了。这样，就可以避免浪费带宽传请求体。但是代价就是会多一次 Round Trip。如果刚好请求体的数据也不多，那么一次性全部发给服务器可能反而更好。

基于此，客户端就能做一些优化，比如内部设定一次 POST 的数据超过 1KB 就先只发 “请求头”，否则就一次性全发。客户端甚至还可以做一些 Adaptive 的策略，统计发送成功率，如果成功率很高，就总是全部发等等。不同浏览器，不同的客户端（curl，postman）可以有各自的不同的方案。不管怎样做，优化目的总是在提高数据吞吐和降低带宽浪费上做一个折衷。

因此到底是发一次还是发 N 次，客户端可以很灵活的决定。因为不管怎么发都是符合 HTTP 协议的，因此我们应该视为这种优化是一种实现细节，而不用扯到 GET 和 POST 本身的区别上。更不要当个什么世纪大发现。



**到底什么算请求体**

看完了上面的内容后，读者也许会对 “什么是请求体” 感到困惑不已，比如 x-www-form-endocded 编码的 body 算不算 “请求体” 呢？

从 HTTP 协议的角度，“请求头” 就是 Method + URL（含 querystring）+ Headers；再后边的都是请求体。

但是从业务角度，如果你把一次请求记为一个调用的话。比如上面的

```text
POST http://foo.com/books
{
  "title": "大宽宽的碎碎念",
  "author": "大宽宽",
  ...
}
```

用Java写大概等价于

```text
createBook("大宽宽的碎碎念", "大宽宽");
```

那么这一行函数名和两个参数都可以看作是一个**请求，不区分头和体**。即便用 HTTP 协议实现，title 和 author 编码到了 HTTP 请求体中。Java 的 HttpServletRequest 支持用 getParameter 方法获取 x-www-url-form-encoded 中的数据，表达的意思就是 “请求“ 的 ”参数“。

对于 HTTP，**需要区分【头】和【体】**，Http Request 和 Http Response 都这么区分。Http 这么干主要用作**：**

对于 HTTP 代理

- 支持转发规则，比如 nginx 先要解析请求头，拿到 URL 和 Header 才能决定怎么做（转发 proxy_pass，重定向 redirect，rewrite 后重新判断……）
- 需要用请求头的信息记录 log。尽管请求体里的数据也可以记录，但一般只记录请求头的部分数据。
- 如果代理规则不涉及到请求体，那么请求体就可以不用从内核态的 page cache 复制一份到用户态了，可以直接 zero copy 转发。这对于上传文件的场景极为有效。
- ……



对于 HTTP 服务器

- 可以通过请求头进行 ACL 控制，比如看看 Athorization 头里的数据是否能让认证通过
- 可以做一些拦截，比如看到 Content-Length 里的数太大，或者 Content-Type 自己不支持，或者 Accept 要求的格式自己无法处理，就直接返回失败了。
- 如果 body 的数据很大，利用 Stream API，可以方便支持一块一块的处理数据，而不是一次性全部读取出来再操作，以至于占用大量内存。
- ……



但从高一级的业务角度，我们在意的其实是【请求】和【返回】。当我们在说 “请求头” 这三个字时，也许实际的意思是【请求】。而用 HTTP 实现【请求】时，可能仅仅用到【HTTP 的请求头】（比如大部分 GET 请求），也可能是【HTTP 请求头】+【HTTP 请求体】（比如用 POST 实现一次下单）。

总之，这里有两层，不要混哦。



**关于 URL 的长度**

因为上面提到了不论是GET和POST都可以使用URL传递数据，所以我们常说的“GET数据有长度限制“其实是指”URL的长度限制“。

HTTP协议本身对URL长度并没有做任何规定。实际的限制是由客户端/浏览器以及服务器端决定的。

先说浏览器。不同浏览器不太一样。比如我们常说的 2048 个字符的限制，其实是IE8的限制。并且原始文档的说的其实是“URL 的最大长度是 2083 个字符，path 的部分最长是 2048 个字符“。

IE8 之后的 IE URL 限制我没有查到明确的文档，但有些资料称 IE 11 的地址栏只能输入法 2047 个字符，但是允许用户点击 html 里的超长 URL。我没实验，哪位有兴趣可以试试。

Chrome 的 URL 限制是2MB。

Safari，Firefox 等浏览器也有自己的限制，但都比 IE 大的多，这里就不挨个列出了。

然而新的 IE 已经开始使用 Chrome 的内核了，也就意味着“浏览器端 URL 的长度限制为 2048 字符”这种说法会慢慢成为历史。

其他的客户端，比如 Java 的，js 的 http client 大多数也并没有限制 URL 最大有多长。

除了浏览器，服务器这边也有限制，比如 apache 的 LimieRequestLine 指令。

apache 实际上限制的是 HTTP 请求第一行 “Request Line“ 的长度，即 <METHOD><URL> <VERSION> 那一行。

再比如 nginx 用 `large_client_header_buffers` 指令来分配请求头中的很长数据的 buffer。这个 buffer 可以用来处理 url，header value 等。

Tomcat 的限制是 web.xml 里 maxHttpHeaderSize 来设置的，控制的是整个 “请求头” 的总长度。

为啥要限制呢？如果写过解析一段字符串的代码就能明白，解析的时候要分配内存。对于一个字节流的解析，必须分配 buffer 来保存所有要存储的数据。而 URL 这种东西必须当作一个整体看待，无法一块一块处理，于是就处理一个请求时必须分配一整块足够大的内存。如果 URL 太长，而并发又很高，就容易挤爆服务器的内存；同时，超长 URL 的好处并不多，我也只有处理老系统的 URL 时因为不敢碰原来的逻辑，又得追加更多数据，才会使用超长 URL。

对于开发者来说，使用超长的 URL 完全是给自己埋坑，需要同时要考虑前后端，以及中间代理每一个环节的配置。此外，超长 URL 会影响搜索引擎的爬虫，有些爬虫甚至无法处理超过 2000 个字节的 URL。这也就意味着这些 URL 无法被搜到，坑爹啊。

其实并没有太大必要弄清楚精确的 URL 最大长度限制。我个人的经验是，只要某个要开发的资源 /api 的 URL 长度**有可能达到 2000 个 bytes 以上，就必须使用 body 来传输数据，除非有特殊情况**。至于到底是 GET + body 还是 POST + body 可以看情况决定。

留意，1 个汉字字符经过 UTF8 编码 + percent encoding 后会变成 9 个字节，别算错哦。



**总结**

上面讲了一大堆，是希望读者不要死记硬背 GET 和 POST 的区别，而是能从更广的层面去看待和思考这个问题。

最后，**协议都是人定的**。只要客户端和服务器能彼此认同，就能工作。在常规的情况下，用符合规范的方式去实现系统可以减少很多工作量——大家都约定好了，就不要折腾了。但是，总会有一些情况用常规规范不合适，不满足需求。这时思路也不能被规范限制死，更不要死抠 RFC。这些规范也许不能处理你遇到的特殊问题。比如：

- Elastic Search 的 _search 接口使用 GET，却用 body 来表达查询，因为查询很复杂，用 querystring 很麻烦，必须用 json 格式才舒服，在请求体用 json 编码更加容易，不用折腾 percent encoding。
- 用 POST 写一个接口下单时可能也要考虑幂等，因为前端可能实现 “下单按键” 有 bug，造成用户一次点击发出 N 个请求。你不能说因为 POST by design 应该是不幂等就不管了。



协议是死的，人是活的。遇到实际的问题时灵活的运用手上的工具满足需求就好。





#### 持久连接节省通信量

HTTP 协议的初始版本中，每进行一次 HTTP 通信就要断开一次 TCP 连接。

以当年的通信情况来说，因为都是些容量很小的文本传输，所以即使这样也没有多大问题。可随着 HTTP 的普及，文档中包含大量图片的情况多了起来。

比如，使用浏览器浏览一个包含多张图片的 HTMP 页面时，在发送请求访问 HTML 页面资源的同时，也会请求该 HTML 页面里包含的其他资源。因此，每次的请求都会造成无谓的 TCP 连接建立和断开，增加通信量的开销。



##### 持久连接

为解决上述 TCP 连接的问题，HTTP/1.1 和一部分的 HTTP/1.0 想出了持久连接（HTTP Persistent Connections，也称为 HTTP keep-alive 或 HTTP connection reuse）的方法。持久连接的特点是，只要任意一端没有明确提出断开连接，则保持 TCP 连接状态。

持久连接的好处在于减少了 TCP 连接的重复建立和断开所造成的额外开销，减轻了服务器端的负载。另外，减少开销的那部分时间，使 HTTP 请求和相应能够更早地结束，这样 Web 页面的显示速度也就相应提高了。

在 HTTP/1.1 中，所有的连接默认都是持久连接，但在 HTTP/1.0 内并未标准化。虽然有一部分服务器通过非标准的手段实现了持久连接，但服务器端不一定能够支持持久连接。毫无疑问，除了服务器端，客户端也需要支持持久连接。



##### 管线化

持久连接使得多数请求以管线化（pipelining）方式发送成为可能。从前发送请求后需等待并收到响应，才能发送下一个请求。管线化技术出现后，不用等待响应亦可直接发送下一个请求。

这样就能够做到同时并行发送多个请求，而不需要一个接一个地等待相应了。

比如，当请求一个包含 10 张图片的 HTML Web 页面，与挨个连接相比，用持久连接可以让请求更快结束。而管线化技术则比持久连接还要快。请求数越多，时间差就越明显。



##### 使用 Cookie 的状态管理

HTTP 是无状态协议，它不对之前发生过的请求和响应的状态进行管理。也就是说，无法根据之前的状态进行本次的请求处理。

假设要求登录认证的 Web 页面本身无法进行状态的管理（不记录已登录的状态），那么每次跳转新页面不是要再次登录，就是要在每次请求报文中附加参数来管理登录状态。

不可否认，无状态协议当然也有它的优点。由于不必保存状态，自然可减少服务器的 CPU 及内存资源的消耗。从另一侧面来说，也正是因为 HTTP 协议本身是非常简单的，所以才会被应用在各种场景里。

保留无状态协议这个特征的同时又要解决类似的矛盾问题，于是引入了 Cookie 技术。Cookie 技术通过在请求和响应报文中写入 Cookie 信息来控制客户端的状态。

Cookie 会根据从服务器端发送的响应报文内的一个叫做 Set-Cookie 的首部字段信息，通知客户端保存 Cookie。当下次客户端再往该服务器发送请求时，客户端会自动在请求报文中加入 Cookie 值后发送出去。

服务器端发现客户端发送过来的 Cookie 后，会去检查究竟是从哪一个客户端发来的连接请求，然后对比服务器上的记录，最后得到之前的状态信息。





### HTTP 报文内的 HTTP信息



#### HTTP 报文

用于 HTTP 协议交互的信息被称为 HTTP 报文。请求端（客户端）的HTTP 报文叫做请求报文，响应端（服务器端）的叫做响应报文。HTTP 报文本身是由多行（用 CR+LF 作换行符）数据构成的字符串文本。
HTTP 报文大致可分为报文首部和报文主体两块。两者由最初出现的空行（CR+LF）来划分。通常，并不一定要有报文主体。

【CR + LF】

CR（Carriage Return，回车符：16 进制 0x0d）和 LF（Line Feed，换行符：16 进制 0x0a）



#### 编码提升传输速率

HTTP 在传输数据时可以按照数据原貌直接传输，但也可以在传输过程中通过编码提升传输速率。通过在传输时编码，能有效地处理大量的访问请求。但是，编码的操作需要计算机来完成，因此会消耗过多的 CPU 等资源。



##### 报文主体和实体主体的差异

报文（message）

是 HTTP 通信中的基本单位，由 8 位字节流（octet sequence，其中 octet 为 8 个比特）组成，通过 HTTP 通信传输。

实体（entity）

作为请求或响应的有效载荷数据（补充项）被传输，其内容由实体首部和实体主体组成。



HTTP 报文的主体用于传输请求或响应的实体主体。

通常，报文主体等于实体主体。只有当传输中进行编码操作时，实体主体的内容发生变化，才导致它和报文主体产生差异。



**压缩传输的内容编码**

向带发送邮件内增加附件时，为了使邮件容量变小，我们会先用 ZIP 压缩文件之后再添加附件发送。HTTP 协议中有一种被称为内容编码的功能也能进行类似的操作。

内容编码指明应用再实体内容上的编码格式，并保持实体信息原样压缩。内容编码后的实体由客户端接收并负责解码。

常见的内容编码有几下几种。

- **gzip**（GNU zip）
- **compress**（UNIX 系统的标准压缩）
- **deflate**（zlib）
- **identity**（不进行编码）



##### 分隔发送的分块传输编码

在 HTTP 通信过程中，请求的编码实体资源尚未全部传输完成之前，浏览器无法显示请求页面。在传输大容量数据时，通过把数据分割成多块，能够让浏览器逐步显示页面。

这种把实体主体分块的功能称为分块传输编码（Chunked Transfer Coding）。

分块传输编码会将实体主体分成多个部分（块）。每一块都会用十六进制来标记块的大小，而实体主体的最后一块会使用 “0(CR+LF)” 来标记。

使用分块传输编码的实体主体会由接收的客户端负责解码，恢复到编码前的实体主体。

HTTP/1.1 中存在一种称为传输编码（Transfer Coding）的机制，它可以在通信时按某种编码方式传输，但只定义作用于分块传输编码中。





#### 发送多种数据的多部分对象集合

发送邮件时，我们可以在邮件里写入文字并添加多份附件。这是因为采用了 MIME（Multipurpose Internet Mail Extensions，多用途因特网邮件扩展）机制，它允许邮件处理文本、图片、视频等多个不同类型的数据。例如，图片等二进制数据以 ASCII 码字符串编码的方式指明，就是利用 MIME 来描述标记数据类型。而在 MIME 扩展中会使用一种称为多部分对象集合（Multipart）的方法，来容纳多份不同类型的数据。

相应地，HTTP 协议中也采纳了多部分对象集合，发送的一份报文主体内可含有多类型实体。通常是在图片或文本文件等上传时使用。

多部分对象集合包含的对象如下。

- **multipart/form-data**
  在 Web 表单文件上传时使用。

- **multipart/byteranges**
  状态码 206（Partial Content，部分内容）响应报文包含了多个范围的内容时使用。

- **multipart/form-data**

  ```
  Content-Type: multipart/form-data; boundary=AaB03x
  　--AaB03x
  Content-Disposition: form-data; name="field1"
  　
  Joe Blow
  --AaB03x
  Content-Disposition: form-data; name="pics"; filename="file1.txt"
  Content-Type: text/plain
  　
  ...（file1.txt的数据）...
  --AaB03x--
  ```

- **multipart/byteranges**

  ```
  HTTP/1.1 206 Partial Content
  Date: Fri, 13 Jul 2012 02:45:26 GMT
  Last-Modified: Fri, 31 Aug 2007 02:02:20 GMT
  Content-Type: multipart/byteranges; boundary=THIS_STRING_SEPARATES
  --THIS_STRING_SEPARATES
  Content-Type: application/pdf
  Content-Range: bytes 500-999/8000
  ...（范围指定的数据）...
  --THIS_STRING_SEPARATES
  Content-Type: application/pdf
  Content-Range: bytes 7000-7999/8000
  ...（范围指定的数据）...
  --THIS_STRING_SEPARATES--
  ```





在 HTTP 报文中使用多部分对象集合时，需要在首部字段里加上 Content-type。有关这个首部字段，我们稍后讲解。

使用 boundary 字符串来划分多部分对象集合指明的各类实体。在 boundary 字符串指定的各个实体的起始行之前插入“--” 标记（例如：--AaB03x、--THIS_STRING_SEPARATES），而在多部分对象集合对应的字符串的最后插入“--” 标记（例如：--AaB03x--、--THIS_STRING_SEPARATES--）作为结束。

多部分对象集合的每个部分类型中，都可以含有首部字段。另外，可以在某个部分中嵌套使用多部分对象集合。



#### 获取部分内容的范围请求

以前，用户不能使用现在这种高速的带宽访问互联网，当时，下载一个尺寸稍大的图片或文件就已经很吃力了。如果下载过程中遇到网络中断的情况，那就必须重头开始。为了解决上述问题，需要一种可恢复的机制。所谓恢复是指能从之前下载中断处恢复下载。

要实现该功能需要指定下载的实体范围。像这样，指定范围发送的请求叫做范围请求（Range Request）。

对一份 10000 字节大小的资源，如果使用范围请求，可以只请求 5001~10000 字节内的资源。

执行范围请求时，会用到首部字段 Range 来指定资源的 byte 范围。

byte 范围的指定形式如下。

- 5001~10 000 字节
  Range: bytes=5001-10000
- 从 5001 字节之后全部的
  Range: bytes=5001-
- 从一开始到 3000 字节和 5000~7000 字节的多重范围
  Range: bytes=-3000, 5000-7000

针对范围请求，响应会返回状态码为 206 Partial Content 的响应报文。另外，对于多重范围的范围请求，响应会在首部字段 Content-Type 标明 multipart/byteranges 后返回响应报文。

如果服务器端无法响应范围请求，则会返回状态码 200 OK 和完整的实体内容。





#### 内容协商返回最合适的内容

同一个 Web 网站有可能存在着多份相同内容的页面。比如英语版和中文版的 Web 页面，它们内容上虽相同，但使用的语言却不同。当浏览器的默认语言为英语或中文，访问相同 URI 的 Web 页面时，
则会显示对应的英语版或中文版的 Web 页面。这样的机制称为内容协商（Content Negotiation）。

内容协商机制是指客户端和服务器端就响应的资源内容进行交涉，然后提供给客户端最为适合的资源。内容协商会以响应资源的语言、字符集、编码方式等作为判断的基准。

包含在请求报文中的某些首部字段（如下）就是判断的基准。这些首部字段的详细说明请参考下一章。

- **Accept**
- **Accept-Charset**
- **Accept-Encoding**
- **Accept-Language**
- **Content-Language**



内容协商技术有以下 3 种类型。
服务器驱动协商（**Server-driven Negotiation**）
由服务器端进行内容协商。以请求的首部字段为参考，在服务器端自动处理。但对用户来说，以浏览器发送的信息作为判定的依据，并不一定能筛选出最优内容。
客户端驱动协商（**Agent-driven Negotiation**）
由客户端进行内容协商的方式。用户从浏览器显示的可选项列表中手动选择。还可以利用 JavaScript 脚本在 Web 页面上自动进行上述选择。比如按 OS 的类型或浏览器类型，自行切换成 PC 版页面或手机版页面。
透明协商（**Transparent Negotiation**）
是服务器驱动和客户端驱动的结合体，是由服务器端和客户端各自进行内容协商的一种方法。





### 返回结果的 HTTP 状态码

HTTP 状态码负责表示客户端 HTTP 请求的返回结果、标记服务器端的处理是否正常、通知出现的错误等工作。



#### 状态码告知从服务器端返回的请求结果

状态码的职责是当客户端向服务器端发送请求时，描述返回的请求结果。借助状态码，用户可以知道服务器端是正常处理了请求，还是出现了错误。

状态码如 200 OK，以 3 位数字和原因短语组成。
数字中的第一位指定了响应类别，后两位无分类。响应类别有以下 5 种。

表 4-1：状态码的类别

|      | 类别                             | 原因短语                   |
| ---- | -------------------------------- | -------------------------- |
| 1XX  | Informational（信息性状态码）    | 接收的请求正在处理         |
| 2XX  | Success（成功状态码）            | 请求正常处理完毕           |
| 3XX  | Redirection（重定向状态码）      | 需要进行附加操作以完成请求 |
| 4XX  | Client Error（客户端错误状态码） | 服务器无法处理请求         |
| 5XX  | Server Error（服务器错误状态码） | 服务器处理请求出错         |

只要遵守状态码类别的定义，即使改变 RFC2616 中定义的状态码，或服务器端自行创建状态码都没问题。

仅记录在 RFC2616 上的 HTTP 状态码就达 40 种，若再加上 WebDAV（Web-based Distributed Authoring and Versioning，基于万维网的分布式创作和版本控制）（RFC4918、5842） 和附加 HTTP 状态码（RFC6585）等扩展，数量就达 60 余种。别看种类繁多，实际上经常使用的大概只有 14 种。接下来，我们就介绍一下这些具有代表性的 14 个状态码。





#### 2XX 成功

2XX 的响应结果表明请求被正常处理了。



##### 200 OK

表示从客户端发来的请求在服务器端被正常处理了。

在响应报文内，随状态码一起返回的信息会因方法的不同而发生改变。比如，使用 GET 方法时，对应请求资源的实体会作为响应返回；而使用 HEAD 方法时，对应请求资源的实体首部（不随报文主体）作为响应返回（即在响应中只返回首部，不会返回实体的主体部分）。



##### 204 No Content

该状态码代表服务器接收的请求已成功处理，但在返回的响应报文中不含实体的主体部分。另外，也不允许返回任何实体的主体。比如，当从浏览器发出请求处理后，返回 204 响应，那么浏览器显示的页面不发生更新。

一般在只需要从客户端往服务器发送信息，而对客户端不需要发送新信息内容的情况下使用。



##### 206 Partial Content

该状态码表示客户端进行了范围请求，而服务器成功执行了这部分的 GET 请求。响应报文中包含由 Content-Range 指定范围的实体内容。



#### 3XX 重定向

3XX 响应结果表明浏览器需要执行某些特殊的处理以正确处理请求。



##### 301 Moved Permanently

永久性重定向。该状态码表示请求的资源已被分配了新的 URI，以后应使用资源现在所指的 URI。也就是说，如果已经把资源对应的 URI 保存为书签了，这时应该按 Location 首部字段提示的 URI 重新保存。



##### 302 Found

临时重定向。该状态码表示请求的资源已被分配了新的 URI，希望用户（本次）能使用新的 URI 访问。

和 301 Moved Permanently 状态码相似，但 302 状态码代表的资源不是被永久移动，只是临时性质的。换句话说，已移动的资源对应的 URI 将来还有可能发生改变。比如，，用户把 URI 保存成书签，但不会像 301 状态码出现时那样去更新书签，而是仍旧保留返回 302 状态码的页面对应的 URI。



##### 304 See Other

该状态码表示由于请求对应的资源存在着另一个 URI，应使用 GET 方法定向获取请求的资源。
303 状态码和 302 Found 状态码有着相同的功能，但 303 状态码明确表示客户端应当采用 GET 方法获取资源，这点与 302 状态码有区别。

比如，当使用 POST 方法访问 CGI 程序，其执行后的处理结果是希望客户端能以 GET 方法重定向到另一个 URI 上去时，返回 303 状态码。虽然 302 Found 状态码也可以实现相同的功能，但这里使用 303 状态码是最理想的。

本书采用的是 HTTP/1.1，而许多 HTTP/1.1 版以前的浏览器不能正确理解 303 状态码。虽然 RFC 1945 和 RFC 2068 规范不允许客户端在重定向时改变请求的方法，但是很多现存的浏览器将 302 响应视为 303 响应，并且使用 GET 方式访问在 Location 中规定的 URI，而无视原先请求的方法。所以作者说这里使用 303 是最理想的。——译者注



##### 304 Not Modified

该状态码表示客户端发送附带条件的请求时，服务器端允许请求访问资源，但未满足条件的情况。304 状态码返回时，不包含任何响应的主体部分。304 虽然被划分在 3XX 类别中，但是和重定向没有关系。

附带条件的请求是指采用 GET 方法的请求报文中包含 If-Match，If-Modified-Since，If-None-Match，If-Range，If-Unmodified-Since 中任一首部。



##### 307 Temporary Redirect

临时重定向。该状态码与 302 Found 有着相同的含义。尽管 302 标准禁止 POST 变换成 GET，但实际使用时大家并不遵守。

307 会遵照浏览器标准，不会从 POST 变成 GET。但是，对于处理响应时的行为，每种浏览器有可能出现不同的情况。



#### 4XX 客户端错误

4XX 的响应结果表明客户端是发生错误的原因所在。



##### 400 Bad Request

该状态码表示请求报文中存在语法错误。当错误发生时，需修改请求的内容后再次发送请求。另外，浏览器会像 200 OK 一样对待该状态码。



##### 401 Unauthorized

该状态码表示发送的请求需要有通过 HTTP 认证（BASIC 认证、DIGEST 认证）的认证信息。另外若之前已进行过 1 次请求，则表示用户认证失败。

返回含有 401 的响应必须包含一个适用于被请求资源的 WWWAuthenticate 首部用以质询（challenge）用户信息。当浏览器初次接收到 401 响应，会弹出认证用的对话窗口。



##### 403 Forbidden

该状态码表明对请求资源的访问被服务器拒绝了。服务器端没有必要给出拒绝的详细理由，但如果想作说明的话，可以在实体的主体部分对原因进行描述，这样就能让用户看到了。未获得文件系统的访问授权，访问权限出现某些问题（从未授权的发送源 IP 地址试图访问）等列举的情况都可能是发生 403 的原因。



##### 404 Not Found

该状态码表明服务器上无法找到请求的资源。除此之外，也可以在服务器端拒绝请求且不想说明理由时使用。





#### 5XX 服务器错误

5XX 的响应结果表明服务器本身发生错误。



##### 500 Internal Server Error

该状态码表明服务器端在执行请求时发生了错误。也有可能是 Web 应用存在的 bug 或某些临时的故障。



##### 503 Service Unavailable

该状态码表明服务器暂时处于超负载或正在进行停机维护，现在无法处理请求。如果事先得知解除以上状况需要的时间，最好写入 RetryAfter 首部字段再返回给客户端。



状态码和状况的不一致
不少返回的状态码响应都是错误的，但是用户可能察觉不到这点。比如 Web 应用程序内部发生错误，状态码依然返回 200 OK，这种情况也经常遇到。





### 与 HTTP 协作的 Web 服务器

一台 Web 服务器可搭建多个独立域名的 Web 网站，也可作为通信路径上的中转服务器提升传输效率。



#### 用单台虚拟主机实现多个域名

HTTP/1.1 规范允许一台 HTTP 服务器搭建多个 Web 站点。比如，提供 Web 托管服务（Web Hosting Service）的供应商，可以用一台服务器为多位客户服务，也可以以每位客户持有的域名运行各自不同的网站。这是因为利用了虚拟主机（Virtual Host，又称虚拟服务器）的功能。

即使物理层面只有一台服务器，但只要使用虚拟主机的功能，则可以假想已具有多台服务器。

客户端使用 HTTP 协议访问服务器时，会经常采用类似 www.hackr.jp 这样的主机名和域名。

在互联网上，域名通过 DNS 服务映射到 IP 地址（域名解析）之后访问目标网站。可见，当请求发送到服务器时，已经是以 IP 地址形式访问了。

所以，如果一台服务器内托管了 www.tricorder.jp 和 www.hackr.jp 这两个域名，当收到请求时就需要弄清楚究竟要访问哪个域名。

在相同的 IP 地址下，由于虚拟主机可以寄存多个不同主机名和域名的 Web 网站，因此在发送 HTTP 请求时，必须在 Host 首部内完整指定主机名或域名的 URI。



#### 通信数据转发程序：代理、网关、隧道

HTTP 通信时，除客户端和服务器以外，还有一些用于通信数据转发的应用程序，例如代理、网关和隧道。它们可以配合服务器工作。

这些应用程序和服务器可以将请求转发给通信线路上的下一站服务器，并且能接收从那台服务器发送的响应再转发给客户端。



代理

代理是一种有转发功能的应用程序，它扮演了位于服务器和客户端 “中间人” 的角色，接收由客户端发送的请求并转发给服务器，同时也接收服务器返回的响应并转发给客户端。

网关

网关是转发其他服务器通信数据的服务器，接收从客户端发送来的请求时，它就像自己拥有资源的源服务器一样对请求进行处理。有时客户端可能都不会察觉，自己的通信目标是一个网关。

隧道
隧道是在相隔甚远的客户端和服务器两者之间进行中转，并保持双方通信连接的应用程序。



##### 代理

代理服务器的基本行为就是接收客户端发送的请求后转发给其他服务器。代理不改变请求 URI，会直接发送给前方持有资源的目标服务器。

持有资源实体的服务器被称为源服务器。从源服务器返回的响应经过代理服务器后再传给客户端。

在 HTTP 通信过程中，可级联多台代理服务器。请求和响应的转发会经过数台类似锁链一样连接起来的代理服务器。转发时，需要附加 Via 首部字段以标记出经过的主机信息。

使用代理服务器的理由有：利用缓存技术（稍后讲解）减少网络带宽的流量，组织内部针对特定网站的访问控制，以获取访问日志为主要目的，等等。代理有多种使用方法，按两种基准分类。一种是是否使用缓存，另一种是是否会修改报文。



缓存代理
代理转发响应时，缓存代理（Caching Proxy）会预先将资源的副本（缓存）保存在代理服务器上。当代理再次接收到对相同资源的请求时，就可以不从源服务器那里获取资源，而是将之前缓存的资源作为响应返回。



透明代理
转发请求或响应时，不对报文做任何加工的代理类型被称为透明代理（Transparent Proxy）。反之，对报文内容进行加工的代理被称为非透明代理。



##### 网关

网关的工作机制和代理十分相似。而网关能使通信线路上的服务器提供非 HTTP 协议服务。

利用网关能提高通信的安全性，因为可以在客户端与网关之间的通信线路上加密以确保连接的安全。比如，网关可以连接数据库，使用 SQL 语句查询数据。另外，在 Web 购物网站上进行信用卡结算时，网关可以和信用卡结算系统联动。



##### 隧道

隧道可按要求建立起一条与其他服务器的通信线路，届时使用 SSL 等加密手段进行通信。隧道的目的是确保客户端能与服务器进行安全的通信。

隧道本身不回去解析 HTTP 请求。也就是说，请求保持原样中转给之后的服务器。隧道会在通信双方断开连接时结束。

通过隧道的传输，可以和远距离的服务器安全通信。隧道本身是透明的，客户端不用在意隧道的存在





### 基于 HTTP 的功能追加协议

虽然 HTTP 协议既简单又简捷，但随着时代的发展，其功能使用上捉襟见肘的疲态已经凸显。本章我们将讲解基于 HTTP 新增的功能的协议。



#### 基于 HTTP 的协议

在建立 HTTP 标准规范时，制订者主要想把 HTTP 当作传输 HTML 文档的协议。随着时代的发展，Web 的用途更具多样性，比如演化成在线购物网站、SNS（Social Networking Service，社交网络服务）、企业或组织内部的各种管理工具，等等。

而这些网站所追求的功能可通过 Web 应用和脚本程序实现。即使这些功能已经满足需求，在性能上却未必最优，这是因为 HTTP 协议上的限制以及自身性能有限。

HTTP 功能上的不足可通过创建一套全新的协议来弥补。可是目前基于 HTTP 的 Web 浏览器的使用环境已遍布全球，因此无法完全抛弃 HTTP。有一些新协议的规则是基于 HTTP 的，并在此基础上添加了新的功能。



#### 消除 HTTP 瓶颈的 SPDY 

Google 在 2010 年发布了 SPDY（取自 SPeeDY，发音同 speedy），其开发目标旨在解决 HTTP 的性能瓶颈，缩短 Web 页面的加载时间（50%）。



##### HTTP 的瓶颈

在 Facebook 和 Twitter 等 SNS 网站上，几乎能够实时观察到海量用户公开发布的内容，这也是一种乐趣。当几百、几千万的用户发布内容时，Web 网站为了保存这些新增内容，在很短的时间内就会发生大量的内容更新。

为了尽可能实时地显示这些更新的内容，服务器上一有内容更新，就需要直接把那些内容反馈到客户端的界面上。虽然看起来挺简单的，但 HTTP 却无法妥善地处理好这项任务。

使用 HTTP 协议探知服务器上是否有内容更新，就必须频繁地从客户端到服务器端进行确认。如果服务器上没有内容更新，那么就会产生徒劳的通信。

若想在现有 Web 实现所需的功能，以下这些 HTTP 标准就会成为瓶颈。

- 一条连接上只可发送一个请求。

- 请求只能从客户端开始。客户端不可以接收除响应以外的指令。

- 请求/响应首部未经压缩就发送。首部信息越多延迟越大。

- 发送冗长的首部。每次互相发送相同的首部造成的浪费较多。

- 可任意选择数据压缩格式。非强制压缩发送。

  <div align="center"> <img src="https://raw.githubusercontent.com/BufferedStream/cs-learning-notes/master/notes/images/%E5%9B%BE%E8%A7%A3HTTP%20-%20%E5%9B%BE2.jpg"/> </div><br>





**Ajax** 的解决方法

Ajax（Asynchronous JavaScript and XML， 异 步 JavaScript 与 XML 技术）是一种有效利用 JavaScript 和 DOM（Document Object Model，文档对象模型）的操作，以达到局部 Web 页面替换加载的异步通信手段。和以前的同步通信相比，由于它只更新一部分页面，响应中传输的数据量会因此而减少，这一优点显而易见。

Ajax 的核心技术是名为 XMLHttpRequest 的 API，通过 JavaScript 脚本语言的调用就能和服务器进行 HTTP 通信。借由这种手段，就能从已加载完毕的 Web 页面上发起请求，只更新局部页面。

而利用 Ajax 实时地从服务器获取内容，有可能会导致大量请求产生。另外，Ajax 仍未解决 HTTP 协议本身存在的问题。

<div align="center"> <img src="https://raw.githubusercontent.com/BufferedStream/cs-learning-notes/master/notes/images/%E5%9B%BE%E8%A7%A3HTTP%20-%20%E5%9B%BE3.jpg "/> </div><br>

**Comet** 的解决方法

一旦服务器端有内容更新了，Comet 不会让请求等待，而是直接给客户端返回响应。这是一种通过延迟应答，模拟实现服务器端向客户端推送（Server Push）的功能。

通常，服务器端接收到请求，在处理完毕后就会立即返回响应，但为了实现推送功能，Comet 会先将响应置于挂起状态，当服务器端有内容更新时，再返回该响应。因此，服务器端一旦有更新，就可以立即反馈给客户端。

内容上虽然可以做到实时更新，但为了保留响应，一次连接的持续时间也变长了。期间，为了维持连接会消耗更多的资源。另外，Comet 也仍未解决 HTTP 协议本身存在的问题。

<div align="center"> <img src="https://raw.githubusercontent.com/BufferedStream/cs-learning-notes/master/notes/images/%E5%9B%BE%E8%A7%A3HTTP%20-%20%E5%9B%BE4.jpg "/> </div><br>

**SPDY** 的目标

陆续出现的 Ajax 和 Comet 等提高易用性的技术，一定程度上使 HTTP 得到了改善，但 HTTP 协议本身的限制也令人有些束手无策。为了进行根本性的改善，需要有一些协议层面上的改动。

处于持续开发状态中的 SPDY 协议，正是为了在协议级别消除 HTTP 所遭遇的瓶颈。





##### SPDY 的设计与功能

SPDY 没有完全改写 HTTP 协议，而是在 TCP/IP 的应用层与运输层之间通过新加会话层的形式运作。同时，考虑到安全性问题，SPDY 规定通信中使用 SSL。

SPDY 以会话层的形式加入，控制对数据的流动，但还是采用 HTTP 建立通信连接。因此，可照常使用 HTTP 的 GET 和 POST 等方 法、Cookie 以及 HTTP 报文等。

<div align="center"> <img src="https://raw.githubusercontent.com/BufferedStream/cs-learning-notes/master/notes/images/%E5%9B%BE%E8%A7%A3HTTP%20-%20%E5%9B%BE5.jpg"/> </div><br>

使用 SPDY 后，HTTP 协议额外获得以下功能。

多路复用流

通过单一的 TCP 连接，可以无限制处理多个 HTTP 请求。所有请求的处理都在一条 TCP 连接上完成，因此 TCP 的处理效率得到提高。



赋予请求优先级

SPDY 不仅可以无限制地并非处理请求，还可以给请求逐个分配优先级顺序。这样主要是为了在发送多个请求时，解决因带宽低而导致响应变慢的问题。



压缩 HTTP 首部
压缩 HTTP 请求和响应的首部。这样一来，通信产生的数据包数量和发送的字节数就更少了。



推送功能
支持服务器主动向客户端推送数据的功能。这样，服务器可直接发送数据，而不必等待客户端的请求。



服务器提示功能

服务器可以主动提示客户端请求所需的资源。由于在客户端发现资源之前就可以获知资源的存在，因此在资源已缓存等情况下，可以避免发送不必要的请求。



##### SPDY 消除 Web 瓶颈了吗

希望使用 SPDY 时，Web 的内容端不必做什么特别改动，而 Web 浏览器及 Web 服务器都要为对应 SPDY 做出一定程度上的改动。有好几家 Web 浏览器已经针对 SPDY 做出了相应的调整。另外，Web 服务器也进行了实验性质的应用，但把该技术导入实际的 Web 网站却进展不佳。
因为 SPDY 基本上只是将单个域名（ IP 地址）的通信多路复用，所以当一个 Web 网站上使用多个域名下的资源，改善效果就会受到限制。
SPDY 的确是一种可有效消除 HTTP 瓶颈的技术，但很多 Web 网站存在的问题并非仅仅是由 HTTP 瓶颈所导致。对 Web 本身的速度提升，还应该从其他可细致钻研的地方入手，比如改善 Web 内容的编
写方式等。





#### 使用浏览器进行全双工通信的 WebSocket

利用 Ajax 和 Comet 技术进行通信可以提升 Web 的浏览速度。但问题在于通信若使用 HTTP 协议，就无法彻底解决瓶颈问题。WebSocket 网络技术正是为解决这些问题而实现的一套新协议及 API。

当时筹划将 WebSocket 作为 HTML5 标准的一部分，而现在它却逐渐变成了独立的协议标准。WebSocket 通信协议在 2011 年 12 月 11 日，被 RFC 6455 - The WebSocket Protocol 定为标准。



##### WebSocket 的设计与功能

WebSocket，即 Web 浏览器与 Web 服务器之间全双工通信标准。其中，WebSocket 协议由 IETF 定为标准，WebSocket API 由 W3C 定为标准。仍在开发中的 WebSocket 技术主要是为了解决 Ajax 和 Comet 里 XMLHttpRequest 附带的缺陷所引起的问题。



##### WebSocket 协议

一旦 Web 服务器与客户端之间建立起 WebSocket 协议的通信连接，之后所有的通信都依靠这个专用协议进行。通信过程中可互相发送 JSON、XML、HTML 或图片等任意格式的数据。

由于是建立在 HTTP 基础上的协议，因此连接的发起方仍是客户端，而一旦确立 WebSocket 通信连接，不论服务器还是客户端，任意一方都可直接向对方发送报文。

下面我们列举一下 WebSocket 协议的主要特点。

推送功能

支持由服务器向客户端推送数据的推送功能。这样，服务器可直接发送数据，而不必等待客户端的请求。

减少通信量

只要建立起 WebSocket 的连接，就希望一直保持连接状态。和 HTTP 相比，不但每次连接时的总开销减少，而且由于 WebSocket 的首部信息很小，通信量也相应减少了。

为了实现 WebSocket 通信，在 HTTP 连接建立之后，需要完成一次 “握手”（Hanshaking）的步骤。

- 握手 请求

  为了实现 WebSocket 通信，需要用到 HTTP 的 Upgrade 首部字段，告知服务器通信协议发送改变，以达到握手的目的。

  ```http
  GET /chat HTTP/1.1
  Host: server.example.com
  Upgrade: websocket
  Connection: Upgrade
  Sec-WebSocket-Key: dGhlIHNhbXBsZSBub25jZQ==
  Origin: http://example.com
  Sec-WebSocket-Protocol: chat, superchat
  Sec-WebSocket-Version: 13
  ```

  

  Sec-WebSocket-Key 字段内记录着握手过程中必不可少的键值。Sec-WebSocket-Protocol 字段内记录使用的子协议。

  子协议按 WebSocket 协议标准在连接分开使用时，定义那些连接的名称。



- 握手 响应	

  对于之前的请求，返回状态码 101 Switching Protocols 的响应。

  ```http
  HTTP/1.1 101 Switching Protocols
  Upgrade: websocket
  Connection: Upgrade
  Sec-WebSocket-Accept: s3pPLMBiTxaQ9kYGzzhZRbK+xOo=
  Sec-WebSocket-Protocol: chat
  ```

  

  Sec-WebSocket-Accept 的字段值是由握手请求中的 Sec-WebSocket-Key 的字段值生成的。

  成功握手确立 WebSocket 连接之后，通信时不再使用 HTTP 的数据帧，而采用 WebSocket 独立的数据帧。

  <div align="center"> <img src="https://raw.githubusercontent.com/BufferedStream/cs-learning-notes/master/notes/images/%E5%9B%BE%E8%A7%A3HTTP%20-%20%E5%9B%BE6.jpg"/> </div><br>



- **WebSocket API**

  JavaScript 可调用“The WebSocket API”（http://www.w3.org/TR/websockets/，由 W3C 标准制定）内提供的 WebSocket 程序接口，以实现 WebSocket 协议下全双工通信。

  以下为调用 WebSocket API，每 50ms 发送一次数据的实例。

  ```js
  var socket = new WebSocket('ws://game.example.com:12010/updates');
  socket.onopen = function () {
  	setInterval(function() {
      	if (socket.bufferedAmount == 0)
      		socket.send(getUpdateData());
  	}, 50);
  };
  ```

  



期盼已久的 HTTP/2.0

目前主流的 HTTP/1.1 标准，自 1999 年发布的 RFC2616 之后再未进行过改订。SPDY 和 WebSocket 等技术纷纷出现，很难断言 HTTP/1.1 仍是适用于当下的 Web 的协议。 

负责互联网技术标准的 IETF（Internet Engineering Task Force，互联网工程任务组）创立 httpbis（Hypertext Transfer Protocol Bis，http://datatracker.ietf.org/wg/httpbis/）工作组，其目标是推进下一代 HTTP——HTTP/2.0 在 2014 年 11 月实现标准化。

HTTP/2.0 的特点

HTTP/2.0 的目标是改善用户在使用 Web 时的速度体验。由于基本上都会先通过 HTTP/1.1 与 TCP 连接，现在我们以下面的这些协议为基础，探讨一下它们的实现方法。

- **SPDY**
- **HTTP Speed ＋ Mobility**
- **Network-Friendly HTTP Upgrade**



HTTP Speed ＋ Mobility 由微软公司起草，是用于改善并提高移动端通信时的通信速度和性能的标准。它建立在 Google 公司提出的 SPDY 与 WebSocket 的基础之上。

Network-Friendly HTTP Upgrade 主要是在移动端通信时改善 HTTP 性能的标准。

HTTP/2.0 的 7 项技术及讨论

HTTP/2.0 围绕着主要的 7 项技术进行讨论，现阶段（2012 年 8 月 13 日），大都倾向于采用以下协议的技术。但是，讨论仍在持续，所以不能排除会发生重大改变的可能性。

<div align="center"> <img src="https://raw.githubusercontent.com/BufferedStream/cs-learning-notes/master/notes/images/%E5%9B%BE%E8%A7%A3HTTP%20-%20%E5%9B%BE7.jpg"/> </div><br>
