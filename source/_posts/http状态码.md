---
title: http状态码
date: 2019-11-15 11:42:16
tags: http
---
### 1XX
#### 101 Switching Protocols
客户端发送带有首部字段Upgrade的字段，告知服务器通信协议发生改变
服务端返回 **101 Switching Protocols**，之后的通信不再采用HTTP，而采用Upgrade指定的方式
### 2XX

#### 200 ok
<span style='color:#00860b;font-weight: bold;'>含义</span> ：成功处理

#### 204 no content
 <span style='color:#00860b;font-weight: bold;'>含义</span> ：服务器接受的请求已成功处理，但是在相应报文中不包含实体的主体部分（就是没有返回数据），浏览器不刷新页面也不导向新的页面

<span style='color:#00860b;font-weight: bold;'>举例</span>：
1. form表单等如果返回 ` 204，那么页面不刷新   `
2. a的href如果返回 ` 204，那么页面不跳转 `

#### 206 partial content
<span style='color:#00860b;font-weight: bold;'>含义</span> ：返回一部分内容（分片请求数据）

<span style='color:#00860b;font-weight: bold;'>请求头限定范围</span> ：Range: bytes=0-1024 。

<span style='color:#00860b;font-weight: bold;'>头部字段说明</span>
<span style='color:#00860b;font-weight: bold;'>响应头 </span>:
响应头Content-Range表示文件真实大小
``` javascript
    Content-Range: bytes 557056-6160883/6160884
    Content-Length: 5603828
```
<span style='color:#00860b;font-weight: bold;'>请求头</span>:
``` javascript
    Range: bytes=557056-
```

Range: bytes=startOffset-targetOffset/sum  [表示从startOffset读取，一直读取到targetOffset位置)
每次要续传时，先读取已下载文件的字节大小比如100000，然后 rang：100000-，就可以继续下载

Range: bytes=startOffset-targetOffset  [字节总数也可以去掉]

<span style='color:#00860b;font-weight: bold;'>应用</span>：断点续传，大文件下载，迅雷，百度



### 3XX
#### 301：永久重定向
<span style='color:#00860b;font-weight: bold;'>含义</span>：
	该状态码表示请求的资源已被分配了新的uri，也就是会做一个跳转
<span style='color:#00860b;font-weight: bold;'>后台 </span>
``` javascript
res.writeHead(302,{'Location': '/'});
res.end();
```
<span style='color:#00860b;font-weight: bold;'>记住</span>：url重定向是在`浏览器端完成的`，url的重定向与`状态码和location有关`，浏览器先判断状态码是否为301或302时，才会根据location响应头内容进行跳转，上面的代码里，如果返回的状态码是别的比如200啥的，那么浏览器不会跳转

####  302：临时重定向
临时跳转，不是永久性的

#### 区别
从 SEO 角度，302 跳转，搜索引擎仍然保留原来的地址，301 跳转，则会保留跳转后的地址

#### 303：see other

请求的资源存在另一个uri，应该使用get方法定向获取请求的资源

<span style='color:#00860b;font-weight: bold;'>理解</span>：这和302似乎很像，举个例子
比如我要用`post`请求创建一个用户admin，到了后台发现admin已经存在，那返回个303和location位置，然后浏览器用`get`去请求admin的位置（响应头location里返回）

<span style='color:#00860b;font-weight: bold;'>资源已存在</span>


####  304：not modified
资源找到了，但不符合条件，不返回任何主体

<span style='color:#00860b;font-weight: bold;'>理解</span>：当触发`协商缓存`时，就返回304
<span style='color:#00860b;font-weight: bold;'>一段悄悄地对话</span>
浏览器：我来找2019年后更新的A数据
服务器：A数据2019年后没更新过啊，304走你
浏览器：从（浏览器）缓存中读取

### 4XX：bad request 客户端错误
请求报文中存在语法错误，当错误发生时，需要修改请求内容后再次发送

#### 401 unauthorized
一般是客户端需要认证（登录状态失效等）

#### 412 precondition failed
服务器对比if-Match（请求头）和ETag失败时，返回

#### 403 forbidden
你介个用户没有权限访问指定资源的权限

#### 404 not found
没找到请求资源

#### 405 Method Not Allowed
服务端不支持这种HTTP方法

### 5XX 服务端错误

#### 500
服务端有bug或错误

#### 503 service unavailable
服务器停机或超载维护
