---
title: http-proxy-middleware配合gulp使用时的一些坑
date: 2018-01-09 09:23:58
tags: ['gulp','ajax','跨域','代理']
---
### 一、介绍
在我们用gulp、webpack等方式开发项目的时候，由于会启动个本地服务器，所以如果访问后台提供的接口就是跨域了，这种情况该怎么办呢？这就涉及到了 `代理服务` 的配置，react项目的webpack的代理服务配置我之前已经写过了，[传送门~~](https://ymbo.github.io/2017/09/27/react%E4%B8%AD%E8%B7%A8%E5%9F%9F%E8%AF%B7%E6%B1%82%E6%95%B0%E6%8D%AE/)。
因为使用 `http-proxy-middleware` 也遇到了多多少少的问题，网上关于遇到的问题解决办法很少或是没有提及（难道问题太弱智？），所以写下这篇文章记录一下（干货满满呦），`如果错误烦请指出~~~ `

### 二、环境
先说明我的环境：gulp + 静态html 。
gulp主要是处理 less 文件和启动服务器方便程序调试

### 三、目的
本地的服务器为：lcoalhost:8080，
想要请求的地址为：https://c.y.qq.com/soso/fcgi-bin/client_search_cp?ct=24&qqmusic_ver=1298&new_json=1&remoteplace=txt.yqq.center&searchid=36602231022813110&t=0&aggr=1&cr=1&catZhida=1&lossless=0&flag_qc=0&p=1&n=20&w=%E7%AB%A5%E5%B9%B4&g_tk=1134636089&jsonpCallback=MusicJsonCallback12090870182687685&loginUin=619697451&hostUin=0&format=jsonp&inCharset=utf8&outCharset=utf-8&notice=0&platform=yqq&needNewCode=0

（`  请求地址是我从QQ音乐找的，正因为这样我又遇到了另外一个 ajax的坑，最后一节有说明，所以这个地址也具有一定的教学意义 `）

### 四、配置
#### 1.gulpfile.js

``` javascript
var gulp = require('gulp'),
	connect = require('gulp-connect'),
	proxy = require('http-proxy-middleware'),
	path = require('path'),
gulp.task('connect', function() {
	connect.server({
		/*根路径*/
		root: './dist',
		/*开启浏览器自动刷新*/
		livereload: true,
		/*端口号*/
		port: 8080,
		/*使用代理服务*/
		middleware: function(connect, opt) {
         			return [
	               		proxy('/api/',  {
	              			target: 'https://c.y.qq.com/',
	                  			changeOrigin:true,
	                		})
            			]
       		 }
	});
});
```
` 说明 `  proxy配置项：
1. 其中return 返回是一个数组，所以通过配置多个proxy返回值可以实现多重代理
2. `changeOrigin ` 这个意思是，如果我们代理的目标地址是一个虚拟托管站点，比如 http://www.baidu.com 这种域名的形式的target项，则需要配置此项。如果target项为ip值，比如http://10.10.16.5/ 则不用配置此项

#### 2.index.html

``` javascript
$.ajax({
  type: 'GET',
	url: "/api/soso/fcgi-bin/client_search_cp?ct=24&qqmusic_ver=1298&new_json=1&remoteplace=txt.yqq.center&searchid=36602231022813110&t=0&aggr=1&cr=1&catZhida=1&lossless=0&flag_qc=0&p=1&n=20&w=%E7%AB%A5%E5%B9%B4&g_tk=1134636089&jsonpCallback=MusicJsonCallback12090870182687685&loginUin=619697451&hostUin=0&format=jsonp&inCharset=utf8&outCharset=utf-8&notice=0&platform=yqq&needNewCode=0",
	success: function(result){
		console.log(result);
	},
	error:function(err){
		console.log(err+'失败')
	}
});
```


#### 3.结果
可以看到，请求失败
![/(ㄒoㄒ)/~~](/images/http-proxy-middleware配合gulp使用时的一些坑/1.png)

#### 4.解决办法
就是在这个问题上我卡了一下午，为了能游刃有余的使用这个东西，有些基本的参数还是要非常明白的
``` javascript
	pathRewrite: {
   	 	'^/api/' : '',     // rewrite path 
	 },
```

很多的技术文章的配置都有写这个东西，但是很少有介绍这个参数的意思。

下面我们来看看不配置此项真正请求的地址：

不配置这个参数请求代理的地址：https://c.y.qq.com/ `api` /soso/fcgi-bin/client_search_cp?ct=24&qqmusic_ver=1298&new_json=1&remoteplace=txt.yqq.center&searchid=36602231022813110&t=0&aggr=1&cr=1&catZhida=1&lossless=0&flag_qc=0&p=1&n=20&w=%E7%AB%A5%E5%B9%B4&g_tk=1134636089&jsonpCallback=MusicJsonCallback12090870182687685&loginUin=619697451&hostUin=0&format=jsonp&inCharset=utf8&outCharset=utf-8&notice=0&platform=yqq&needNewCode=0 
看我标记粉色的地方，再回过头对比我们要请求的地址，是不是多个了  /api/ ?

所以  `pathRewrite` 这个配置项的意思是，当有 /api/ 字段请求的时候，指定 重写 /api/ 这个字符串，这里为空。

比如：
`目标地址`：http://www.baidu.com/aaa/bbb

`proxy配置`：
``` javascript
proxy('/api/',  {
	target: 'http://www.baidu.com/',
	changeOrigin:true,
	
})
```

`本地服务器`：http://localhost:8888

`ajax`：
``` javascript
$.ajax({
	type: 'GET',
	url: "/api/aaa/bbb",
	success: function(result){
		console.log(result);
	},
	error:function(err){
		console.log(err+'失败')
	}
});
```

`但是实际上请求的是 http://www.baidu.com/api/aaa/bbb 这个地址 ！！！！！`。

所以第一种解决办法是配置重写路径
``` javascript
proxy('/api/',  {
	target: 'http://www.baidu.com/',
	changeOrigin:true,
	pathRewrite: {
   	 	'^/api/' : '',     // rewrite path 
	 },
})
```
第二种解决办法就是利用已有路径

``` javascript
proxy('/aaa/',  {
	target: 'http://www.baidu.com/',
	changeOrigin:true,
})
```

关于 `http-proxy-middleware` 我遇到的问题，已经说完了。下面来说 ajax的一个坑

### 五、ajax

用jq的 ajax请求一个地址，如果返回的数据格式与ajax里预期格式的配置不一样，那么就会在 error 函数里面返回后台提供的数据 ，具体例子就不写了，很简单，可以试一试