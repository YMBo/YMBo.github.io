---
title: body-parser使用注意
date: 2018-03-08 15:53:13
tags: npm
---
## 一、前言
这几天在学习服务端渲染的时候用到了这个包 **body-parser** ,很方便的一个包，这个模块提供了四种解析器
1. JSON body parser
2. Raw body parser
3. Text body parser
4. URL-encoded form body parser

每一项详细的配置都能在网上找到，这篇记下的是我在使用中遇到的问题 `bodyParser.json(option)和bodyParser.urlencoded(option)的不同`，在网上搜这个问题，得到的答案还是让我一头雾水，终于试了几次并结合资料弄清楚两者的使用场景

## 二、区别
看官方的解释：
bodyParser.json(options)：中间件只会解析 json ，允许请求任意Unicode编码,支持 gzip 和 deflate 编码。

bodyParser.urlencoded(option)：中间件只解析urlencoded 请求体，并返回，只支持UTF-8编号文本，支持gzip deflate 压缩。(tip:一般这一项是接收表单提交form)

## 三、例子
看到这里我明白了一个接受form请求，一个接受json请求，好，我写了下面的例子：
``` javascript
//server端关键代码
var jsonParser  =bodyParser.json();
app.post('/login',jsonParser,function(req,res){
	console.log(req.body)
	res.send(req.body)
})

//客户端请求
<form method="post" action="/login">
	<div class="form-group">
		<label for="inputTitle" class="col-sm-3 control-label">电影名称</label>
		<input id="inputTitle" type="text" name="title" value="" class="form-control" />
	</div>
	<div class="form-group">
		<label for="inputLanguage" class="col-sm-3 control-label">海报</label>
		<input id="inputLanguage" type="text" name="poster" value="" class="form-control" />
	</div>
<button type="submit" class="btn btn-default">确定</button>
</form>

setTimeout(function(){
	$.ajax({
		type:'post', 
		url:'login',
		data:{
			url:'login',
			name:'123',
			password:123456
		},
		success:function(data){
			console.log(data)
		}
	})
},1000)
```

客户端有一个用于测试的表单和一个1s后发送post请求的ajax

先看一下表单请求的结果：

![表单请求](/images/body-parser使用注意/1.png)

![表单方式请求后服务端返回结果](/images/body-parser使用注意/2.png)
![ajax方式请求后服务端返回结果](/images/body-parser使用注意/2.png)

可以看到bodyParser.json()并没有解析到ajax和form表单请求，现在来换bodyParser.urlencoded(option)的解析方式
``` javascript
var urlencodedParser = bodyParser.urlencoded({ extended: false });
app.post('/login',urlencodedParser,function(req,res){
	console.log(req.body)
	res.send(req.body)
})
```

![表单方式请求后服务端返回结果](/images/body-parser使用注意/3.png)
![ajax方式请求后服务端返回结果](/images/body-parser使用注意/4.png)

可以看到这种解析方式成功将请求解析到了body里。
到了这里，我又有疑问了，知道了bodyParser.urlencoded期望的数据形式，那bodyParser.json适用于什么情况呢？

看下面例子，既然api说bodyParser.json适用于json的数据类型，所以我将ajax的content-Type设置为json

``` javascript
setTimeout(function(){
	$.ajax({
		type:'post', 
		url:'login',
		contentType:'application/json',
		data:{
			url:'login',
			name:'123',
			password:123456
		},
		success:function(data){
			console.log(data)
		}
	})
},1000)
```
请求结果：
![返回结果](/images/body-parser使用注意/5.png)

请求失败，那么疑问来了，我设置了json格式的请求，为什么bodyParser.json解析不了

**关键**：
> jq的ajax
1. 默认的ContentType的值为:application/x-www-form-urlencoded; charset=UTF-8 
此格式为表单提交格式，数据为 `key1=value1&key2=value2`的格式 
2. 虽然ajax的data属性值格式为:{key1:value1,key2:value2},但最后会转为key1=value1&key2=value2的格式提交到后台 
> 3. 如果想传json格式数据，content-type设置好后，ajax必须将date属性值转为json字符串，不能为json对象（js对象，会自动转为key=value形式）,这样传输的时候才是json格式

我们来试一下：
``` javascript
setTimeout(function(){
	$.ajax({
		type:'post', 
		url:'login',
		contentType:'application/json',
		data:JSON.stringify({
			url:'login',
			name:'123',
			password:123456
		}),
		success:function(data){
			console.log(data)
		}
	})
},1000)
```
![返回结果](/images/body-parser使用注意/6.png)

现在就可以请求成功啦
