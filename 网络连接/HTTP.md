# HTTP
### HTTP到底是什么
两种最直观的印象

    1.浏览器地址栏输入地址，打开网页；
    2.Android 中发送网络请求，返回对应内容
    3.HyperText Transfer Protocol 超文本传输协议

- 超文本：在电脑中显示的，含有可以指向其他文本的链接的文本（HTML、Markdown）

### HTTP的工作方式
浏览器发送请求给服务器，服务器处理后响应给浏览器

    1.DNS域名解析；
    2.建立TCP连接；
    3.浏览器发送HTTP请求；
    4.服务器响应HTTP请求；
    5.关闭TCP连接；
    6.浏览器解析HTML代码，渲染页面

URL结构 协议类型://服务器地址/路径path

报文格式：Request
请求行  —— method  path  HTTP Version
|—— Host: api.github.com
Headers |—— Content-Type: text/plain
|—— Content-Length: 243
Body
报文格式：Response
状态行 —— HTTP Version  status code  status message
Headers
Body
请求方法
GET：获取资源，没有Body
POST：增加或修改资源，有Body
PUT：修改资源，有Body
DELETE：删除资源，没有Body
HEAD：使用方法和get一样，没返回体。使用场景：下载前发送head获取资源信息
post不是幂等的，put是幂等的，get和delete也是幂等的
状态码
1XX：临时性消息 100:需要继续发(Expect:100-continue)  101:切换协议
2XX：成功
3XX：重定向 301:永久重定向  302:临时重定向
4XX：客户端错误 404:资源不存在  401:未授权  403:无权限
5XX：服务器错误
Header
作用：HTTP消息的元数据（metadata）
Host：服务器主机地址（仅用于找到目标主机后确认主机域名和端口）
Content-Length：内容的长度（字节）
Content-Type：内容的类型
text/html：HTML文本，用于浏览器页面响应
application/x-www-form-urlencodeed：普通表单（@FormUrlEncoded  @Field）
multipart/form-data;boundary：多部分形式，一般用于传输包含二进制内容的多项											内容，boundary用于分界（@Multipart @Part）
application/json：json形式，用于Web API的响应或POST/PUT请求
image/jpeg、application/zip：单文件，用于Web API的响应或POST/PUT请求
Transfer-Encoding:chunked
表示Body长度无法确定，Content-Length不能使用
Body格式：
<length1>
<data1>
<length2>
<data2>
0（最后传输0表示内容结束）
Location：重定向的目标URL
User-Agent：用户代理
Range/Accept-Range：指定Body的内容范围（用于分段加载）
Cookie/Set-Cookie：发送Cookie/设置Cookie
Authorization：授权信息
Accept：客户端能接受的数据类型。如text/html
Accept-Charset：客户端接受的字符集。如utf-8
Accept-Encoding：客户端接受的压缩编码类型。如gzip
Content-Encoding：压缩类型。如gzip
Cache-Control：no-cache、no-store、max-age
Last-Modified：
Etag
Cache：private/public
RESTful HTTP
REST其实是一种创建Web服务的架构范式。
1.使用HTTPS协议
2.API部署在专用域名之下
3.API版本号放入URL
4.路径命名不能用动词，只能用名词，名词往往与数据库的表格名对应
5.正确使用HTTP请求方法
6.数据量太大API应该提供参数，过滤返回结果
7.正确使用状态码
8.如果状态码是4xx，就应该向用户返回出错信息
9.针对不同操作，服务器向用户返回的结果应该符合规范
10.Hypermedia API
11.API的身份认证应该使用OAuth 2.0框架
12.服务器返回的数据格式，应该尽量使用JSON，避免使用XML。


现代密码学
对称加密：使用密钥和加密算法对数据进行转换，得到的无意义数据即为密文；使用密	钥和解密算法对密文进行逆向转换，得到原数据
经典算法：DES、AES
非对称加密：使用公钥对数据进行加密得到密文；使用私钥对数据进行解密得到原数据	（公钥可以解私钥）
经典算法：RSA、DSA  延伸用途：数字签名（防止伪造）、签名与验证
工作过程：
1.乙方生成一对密钥（公钥和私钥）并将公钥向其它方公开。
2.得到该公钥的甲方使用该密钥对机密信息进行加密后再发送给乙方。
3.乙方再用自己保存的另一把专用密钥（私钥）对加密后的信息进行解密。乙方只能用		其专用密钥（私钥）解密由对应的公钥加密后的信息。
在传输过程中，即使攻击者截获了传输的密文，并得到了乙的公钥，也无法破解密文，	因为只有乙的私钥才能解密密文。
同样，如果乙要回复加密信息给甲，那么需要甲先公布甲的公钥给乙用于加密，
甲自己保存甲的私钥用于解密。
Base64
将二进制数据转换成由64个字符组成的字符串的编码算法
用途：让原数据其具有字符串所具有的特性，如可以放在URL中传输，可以保存到文	本文件，可以通过普通的聊天软件进行文本传输
变种：Base58，比特币存地址的，
不使用数字"0"，字母大写"O"，字母大写"I"，和字母小写"l"，以及"+"和"/"符号
URL encoding
将URL中的保留字符使用“%”进行编码
目的：消除歧义，避免解析错误


压缩与解压缩
压缩：把数据换一种方式来存储，以减少存储空间
解压缩：把压缩后的数据还原成原先的形式，以便使用
压缩与解压缩也属于编码
常见压缩算法：DEFLATE、JPEG、MP3
媒体数据的编解码
图片的编码：把图像数据写成JPG、PNG等文件的编码格式
图片的解码：把JPG、PNG等文件中的数据解析成标准的图像数据
音频、视频的编解码
序列化
一个对象要实现序列化操作，该类就必须实现了Serializable接口或者Parcelable接口
序列化：把数据对象转换成字节序列的过程
反序列化：把字节序列重新转换成内存中的对象
Hash
把任意数据转换成指定大小范围（通常很小）的数据
作用：摘要、数字指纹
经典算法：MD5、SHA1、SHA256等
实际用途：数据完整性验证、快速查找：hasCode()和HashMap、隐私保护
Hash不是编码，Hash也不是加密
字符集
一个由整数向现实世界中的文字符号的Map
ASCII：128个字符、1字节
ISO-8859-1：对ASCII进行扩充，1字节
Unicode：13万个字符，多字节
UTF-8：Unicode的编码分支
UTF-16：Unicode的编码分支
GBK/GB2312/GB18030：中国自研标准，多字节，字符集+编码
Cookie
工作机制：1.服务器需要客户端保存的内容,放在Set-Cookie headers⾥返回,客户端会	⾃动保存。2.客户端保存的Cookies,会在之后的所有请求⾥都携带进Cookie header⾥	发回给服务器。3.客户端保存Cookie是按照服务器域名来分类的,例如a.cn发回的	Cookie保存下来以后,在之后向b.cn的请求中并不会携带。4.客户端保存的Cookie在	超时后会被删除，没有设置超时时间的Cookie(称作Session Cookie)在浏览器关闭后就	会⾃动删除；另外,服务器也可以主动删除还未过期的客户端Cookies。
作用：会话管理：登录状态(session)、购物车等
个性化：用户偏好、主题
Tracking：分析用户行为
XSS(Cross-site scripting):跨站脚本攻击.即使⽤JavaScript拿到浏览器的Cookie之后,	发送到⾃⼰的⽹站,以这种⽅式来盗取⽤户Cookie.应对⽅式:Server在发送Cookie时,	敏感的Cookie加上HttpOnly.应对⽅式:HttpOnly——这个Cookie只能⽤于HTTP请求,	不能被JavaScript调⽤.它可以防⽌本地代码滥⽤Cookie.
XSRF(Cross-site requst forgery):跨站请求伪造.即在⽤户不知情的情况下访问已经保存	了Cookie的⽹站,以此来越权操作⽤户账户(例如盗取⽤户资⾦).应对⽅式:Referer校验.
Authorization
Authorization: Basic<username:password(Base64ed)>
Authorization: Bearer<bearer token>
bearer token 的获取⽅式：通过 OAuth2 的授权流程








OAuth2的流程
0.第三⽅⽹站向授权⽅⽹站申请第三⽅授权合作，拿到client id和client secret
1.⽤户在使⽤第三⽅⽹站时，点击「通过 XX (如GitHub) 授权」按钮，第三⽅⽹站将⻚	⾯跳转到授权⽅⽹站，并传⼊client id作为⾃⼰的身份标识 
2.授权⽅⽹站根据client id，将第三⽅⽹站的信息和第三⽅⽹站需要的⽤户权限展示给	⽤户，并询问⽤户是否同意授权 
3.⽤户点击「同意授权」按钮后，授权⽅⽹站将⻚⾯跳转回第三⽅⽹站，并传⼊ 		Authorization code作为⽤户认可的凭证。 
4.第三⽅⽹站将Authorization code发送回⾃⼰的服务器 
5.服务器将Authorization code和⾃⼰的client secret⼀并发送给授权⽅的服务器，授	权⽅服务器在验证通过后，返回 access token。OAuth流程结束。 
6.在上⾯的过程结束之后，第三⽅⽹站的服务器（或者有时客户端也会）就可以使⽤ 	access token作为⽤户授权的令牌，向授权⽅⽹站发送请求来获取⽤户信息或操作⽤户	账户。但这已经在 OAuth 流程之外。
在⾃家 App 中使⽤ Bearer token 
有的App会在Api的设计中,将登录和授权设计成类似OAuth2的过程,但简化掉 	Authorization code概念.即登录接⼝请求成功时,会返回access token,然后客户端在之	后的请求中,就可以使⽤这个access token来当做bearer token进⾏⽤户操作了
Refresh token
⽤法：access token有失效时间，在它失效后，调⽤refresh token接⼝，传⼊ 
refresh_token来获取新的access token。 
⽬的：安全。当access token失窃，由于它有失效时间，因此坏⼈只有较短的 
时间来「做坏事」；同时，由于（在标准的 OAuth2 流程中）refresh token 永 
远只存在与第三⽅服务的服务器中，因此 refresh token ⼏乎没有失窃的⻛险。



TCP/IP 协议族
一系列协议所组成的一个网络分层模型
为什么要分层？由于网络的不稳定因素
Application Layer 应⽤层：HTTP、FTP、DNS 
Transport Layer 传输层：TCP、UDP 
Internet Layer ⽹络层：IP 
Link Layer 数据链路层：以太⽹、Wi-Fi
TCP连接
什么叫做连接？TCP是有状态的连接，发送消息时不需要携带身份信息，所以需要一个	互相认识的过程，这个过程就叫连接
Socket是对TCP端口的具象化，TCP互相认识是基于TCP端口来实现的
TCP连接的建立与关闭
三次握手：客户端发送syn包到服务器；服务器确认后应答发送ack包，同时发送一个	syn包给客户端；客户端确认后应答ACK包给服务器
四次挥手：客户端发送fin包到服务器；服务器确认后应答发送ack包；服务器也需要	关闭的时候，发送fin包到客户端；客户端确认后应答发送ack包
关闭的时候为什么需要四次握手？
因为在一端需要关闭时，另一端也许还在发送中，不能立即关闭
长连接
为什么要⻓连接？因为移动⽹络并不在Internet中，⽽是在运营商的内⽹，并不具有真	正的公⽹IP，因此当某个TCP连接在⼀段时间不通信之后，⽹关会出于⽹络性能考虑		⽽关闭这条TCP连接和公⽹的连接通道，导致这个TCP端⼝不再能收到外部通信消		息，即TCP连接被动关闭。
长连接的实现方式：心跳。即在⼀定间隔时间内，使⽤TCP连接发送超短⽆意义消息		来让⽹关不能将⾃⼰定义为「空闲连接」，从⽽防⽌⽹关将⾃⼰的连接关闭


HTTPS
定义：HTTP over SSL的简称，即⼯作在SSL(或TLS)上的HTTP。在HTTP之下增加	的一个安全层，用于保障HTTP的加密传输
工作原理：在客户端和服务器之间用非对称加密协商出一套对称加密密钥，每次发送消	息之前将内容加密，收到之后解密，达到内容的加密传输
为什么不直接用非对称加密？⾮对称加密由于使⽤了复杂了数学原理，因此计算相当复	杂，如果完全使⽤⾮对称加密来加密通信内容，会严重影响⽹络通信的性能
HTTPS连接
1.客户端发送Client Hello:1字节握手信息、可选的TLS版本、可选的加密套件：可选	大的对称加密算法、可选的非对称加密算法、可选的hash算法、客户端随机数
2.服务器发回Server Hello：1字节握手信息、TLS版本、加密套件、服务器随机数
3.服务器发送证书（[服务器公钥、服务器公钥的签名、服务器主机名、服务器地区]、	[证书签发机构的公钥、证书签发机构的名字、证书签发机构的地区、证书签发机构的公	钥的签名]、[根证书机构的公钥、根证书机构的名字、根证书机构的地区]）
4.客户端发送加密后Pre-master secret
Pre-master secret、客户端随机数、服务器随机数生成Master secret再生成客户端加	密密钥、服务器加密密钥、客户端MAC secret、服务器MAC secret
5.客户端通知将使用加密通信
6.客户端发送Finished
7.服务器通知将使用加密通信
8.服务器发送Finished
HTTPDNS
不走传统的DNS解析，而是自己搭建基于HTTP协议的DNS服务器集群，分布在多个	地点和多个运营商，当客户端需要DNS解析的时候，直接通过HTTP协议进行请求这	个服务器集群，获取就近的地址


