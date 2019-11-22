---
title: http首部字段
date: 2019-11-19 14:33:29
tags: http
---
### 前言
#### 代理
##### 缓存代理
代理转发响应时，缓存代理会与湘江资源的副本保存在代理服务器上
当代理再次受到对相同资源的请求时，就可以不从源服务器哪里获取资源，而是将之前缓存的资源作为相应返回

##### 透明代理
转发请求或相应时，不对报文做任何加工的代理类型被称为透明代理

#### 客户端的缓存
浏览器的缓存称为零食网络文件  temporary internet file
当判定缓存期过期后，会向源服务器确认资源的有效性，（强缓存和协商缓存）

### http首部字段
http首部字段类型根据实际用途被分为以下4种类型：

1. 通用首部字段（General Header Fields）
请求报文和响应报文两方都会使用的首部。

2. 请求首部报文（Request Headers Fields）
从客户端向服务端发送请求报文时使用的首部。补充了请求的附加内容，客户端信息，响应内容相关优先级等信息。

3. 响应首部字段（Response Header Fields）
从服务器端向客户端返回响应报文时使用的首部。补充了响应的附加内容，也会要求客户端附加额外的内容信息。

4. 实体首部字段（Entity Header Fields）
针对请求报文和响应报文的实体部分使用的首部。补充了资源内容更新时间等与实体相关的信息。

###  一、通用首部字段（General Header Fields）
#### 1. catch-control
客户端缓存请求指令：
- no-cache
    可以在本地缓存，可以子啊代理服务器缓存，但是这个缓存要服务器验证才可以使用
- no-store
    彻底禁用缓存，本地和代理服务器都不缓存，每次都从服务端获取
- max-age
    如果缓存资源的缓存时间比这个数值小，那么客户端就使用缓存的资源，否则就要请求源服务器
- min-fresh
    要求缓存服务器返回值至少还未经过指定时间的缓存资源
    期望在指定时间内响应有效，比如min-fresh=60,那么60s后就要重新请求服务器而不是使用缓存
- max-stale:
    缓存资源即使过期也照常接收，如果没有参数，无论经过多多久客户端都会接收，如果有参数，只要处于这个时间内，就会被接收
- on-if-cached:
    缓存服务器只要对目标有缓存资源的情况下返回，不重新加载相应

服务端缓存相应指令：
客户端  -----   代理服务器   -------源服务器

- public
    可以向任意方提供响应的缓存
- private
    只能针对个人用户，而不能被代理服务器缓存
- no-cache
    强制客户端直接向服务器发送请求,也就是说每次请求都必须向服务器发送。服务器接收到请求，然后判断		资源是否变更，是则返回新内容，否则返回304，未变更
- max-age
    缓存服务器不对资源的有效性进行确认，max-age数值代表资源保存为缓存的最长时间

#### 2. connection
- 控制不再转发给代理的首部字段
> upgrade:http/1.1
	connection:upgrade  
    代理服务器：删除upgrate字段======>源服务器

- 管理持久连接（持久连接：只要任意一方不明确说明那么就不断开链接(长连接)）
> connection:close  
	http 1.1以前的都是非持久连接，如果要在旧版本保持持久连接，则指定connection为Keep-Alive

#### 3. Date
> 表示创建http报文的日期和时间
#### 4. Tralier
事先说明在报文主体后记录了哪些首部字段（在报文最后写了很重要的东西，记得仔细阅读哦）
Trailer:Expires
(报文主体)
expirs:一个日期
![过程](/images/http首部字段/Trailer.png)

#### 5. Transfer-Encoding：
传输报文主体时采用的编码方式
#### 6. Upgrade
用指定的协议进行通信 
`注意` :这个字段仅限于客户端和临界服务器，就是中间不能有别的代理服务器啥的，所以通常还要额外指定connection:upgrade字段
![过程](/images/http首部字段/Upgrade.png)
#### 7. Via
追踪客户端与服务器之间的请求和响应报文的传输路径
![过程](/images/http首部字段/Via.png)

#### 8.warning
警告与缓存相关的警告
![过程](/images/http首部字段/warning.png)


###  二、请求首部字段

#### 1.Accept
告知服务器用户能处理的媒体类型和优先级，用q=来表示优先级 （；） 进行分割，范围0-1

#### 2.Accept-Charset
通知服务器用户代理支持的字符集和顺序
Accept-Charset:iso-8859-5,unicode-1-1;q=0.8

#### 3.Accept-Encoding
通知服务器客户端支持的内容编码和优先级
就4种：gzip、compress、deflate、identity

#### 4.Accept-language
客户端能处理的自然语言集
Accept-Language:zh-cn,zh;q=0.7,en-us,en;q=0.3

#### 5.Authorization
认证信息，权限认证

#### 6.Expect
期待服务器出现某种特定行为,如果服务端无法理解客户端的期望发生错误时，会返回 417状态码
![过程](/images/http首部字段/Expect.png)

#### 7.From
用户的电子邮箱地址

#### 8.Host
host地址

#### 9.IF-Match
**if-xxx的都可以称为条件请求**
只有当if-match字段值和etag值匹配一致时，服务器才会接受请求否则返回412 precondition failed

#### 10.If-Modified-Since
**if-xxx的都可以称为条件请求**
表示本地文件最后修改日期，浏览器会在request header加上If-Modified-Since（上次返回的Last-Modified的值），询问服务器在该日期后资源是否有更新，有更新的话就会将新的资源发送回来，否则304状态码，从本地资源缓存读取

#### 11.If-None-Match
**if-xxx的都可以称为条件请求**
只有在这个字段的值与ETag不一致时，可处理该请求，与if-Match相反

#### 12.if-range
这个涉及到断点续传,如果这个字段的值满足条件，range头字段才会起作用这个字段可以用ETag验证，也可以用**last-modified** 验证
如果验证失败则返回全部资源 200状态码

#### 13.if-unmodified-Since
与if-modified-since相反，告知服务器只有在这个时间点后未发生了更新才处理请求，
如果发生了更新，则412(precondition failed)状态码返回

#### 14.Max-Forwards
指定可经过的服务器最大数目，过一个服务器就减1，直到为0时返回
![过程](/images/http首部字段/Max-Forwards.png)

#### 15.Referer
请求的URI是从哪个web页面发起的
![过程](/images/http首部字段/Referer.png)

#### 16.TE
告知服务器客户端能够处理响应的传输编码方式和优先级 
TE:gzip,deflate;q=5

#### 17.User-Agent
客户端浏览器信息


###  三、响应首部字段

#### 1.Accept-Ranges
当不能处理范围请求时，Accept-Ranges:none(就是不支持断点续传204)
Accept-Ranges:bytes 支持

#### 2.Age:
如果是源服务器：告知客户端多久前创建了响应，单位秒
缓存服务器：指缓存后的响应再次发起认证到认证完成的时间

#### 3.ETag:
资源标识，每个资源都会有，资源更新时，ETag也会更新。
这个相对last-modified和Expires优先级最高
##### 强ETag值：
不论实体发生多么细微的变化都会改变其值
##### 弱ETag值：
只有资源发生了根本的改变，产生差异时才会改变，这时会在字段开始处附加W/
>ETag:W/"asdf"

#### 4.Location:
重定向（302&301）如果是这俩状态码，就会读取这个location来确定要跳转的页面,
注意别的状态码浏览器不会跳转

#### 5.Proxy-Authenticate
注意：这是在代理服务器和客户端之间进行认证的
这个字段会把代理服务器所要求的的认证信息发送给客户端

#### 6.Retry-After
告知客户端多久之后再次发送请求，可以是秒数也可以是具体日期，配合503和3xx使用，试验了一下啥用没有，有的
浏览器还不支持

#### 7.server
告诉客户端服务器安装的http服务器应用程序和信息
Server:Apache/2.2.6 (unix) PHP/5.2.5
Server: nginx
![过程](/images/http首部字段/server.png)

#### 8.Vary（参数是首部字段）
关系				客户端------ 代理服务器-----------源服务器
代理服务器接收到源服务器包含Vary的响应后，如果客户端发送包含相同Vary指定的首部字段的请求，那么使用缓存，即便相同的资源如果没有Vary也要重新获取资源
比如：
![过程](/images/http首部字段/vary2.png)
![实例](/images/http首部字段/vary3.png)
    
#### 9.WWW-Authenticate
书上说 401 Unauthorized 状态码响应中，肯定会带有这个首部字段，我试验了一下，
发现返回401的时候，并没有带这个首部

###  四、实体首部字段（在请求和响应两房的报文中都包含有与实体相关的首部）
#### 1.Allow
通知客户端能支持的所有HTTP方法：
比如：Allow：GET,HEAD,如果遇到不支持的方法，那么返回405 Method Not Allowed

#### 2.Content-Encoding
Content-Encoding:gzip
告知客户端，服务器对实体的主题部分选用的内容编码方式
内容编码是指在不丢失实体信息的前提下所进行的压缩
有四种编码方式：
    gzip，compress，deflate，identity

#### 3.Content-Language
		实体主题使用的自然语言（中文、英文...）

#### 4.Content-Length  (字节)
		表明实体主题部分的大小，对实体主体进行编码传输时，不能再使用content-length 

#### 5.Content-Location（也没遇见过）
给出与报文主体对应的URI

#### 6.Content-MD5 （从来没遇见过）
客户端对接收的报文主体执行相同的MD5算法，然后与首部字段的值比较，目的是检查报文主体是否传输的完整
Content-MD5:SALDFJKHIJKNB UIJKNDSHIO234DF==

#### 7.Content-Range
针对范围请求，返回响应时的首部字段，告知客户端返回的是那部分的范围，断点续传
[断点续传](/2019/11/15/http状态码/#206-partial-content)

#### 8.Contnet-Type
实体主体的媒体类型
![类型](/images/http首部字段/Type.png)

#### 9.Expires:
将资源失效的日期告诉客户端，缓存服务器接收到含有expires的响应后，会使用缓存
max-age优先级大于这个

#### 10.last-modified
资源最终修改的时间

最后修改只能精确到秒级，如果某些文件在1秒钟以内，被修改多次的话，它将不能准确标注文件的修改时间

而且如果源服务器有资源删除后重新生成，内容不变，这时这个字段也会变化
