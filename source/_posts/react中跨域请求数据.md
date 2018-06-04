---
title: react中跨域请求数据
date: 2017-09-27 14:03:37
tags: [webpack proxy,react跨域请求]
---
## 一、介绍
在我写 [react ele](https://github.com/YMBo/react-ele) 项目的时候，里面的所有数据都要从ele获取，所以我之前的想法是先用死的数据模拟，然后用 node 写几个接口请求ele数据并返回给我的react。
知道今天早上，我在纳闷 `react`开发过程中是用 `webpack` 起服务器的，那跟后台就是跨域了啊，这样ajax是没有办法请求的，所以吃完一个鸡蛋后查找了一番，果然让我找到了，看下面

[传送门 -- gulp里服务端代理配置 ](https://ymbo.github.io/2018/01/09/http-proxy-middleware%E9%85%8D%E5%90%88gulp%E4%BD%BF%E7%94%A8%E6%97%B6%E7%9A%84%E4%B8%80%E4%BA%9B%E5%9D%91/)


## 二、配置
webpack中 `proxy` 是设置代理的 
``` javascript 
proxy: {
  "/api": {
      target: "http://localhost:3000",
      secure: false,
      changeOrigin : true
    }
},
```
现在比如说我 `/api/users` 这么请求，现在将代理请求http://localhost:3000/api/users 

` 注意 ` ：
这里的 `secure` 参数表示能请求 `https` 的服务器
`changeOrigin` 表示是否支持跨域请求，默认 `false` 

## 实例
好了 现在我来试验一下
``` javascript 
"/base": {
    target: "http://c.y.qq.com/",
    secure: false,
    changeOrigin : true
}
```
这个是我从qq音乐找了一个借口，在react中通过 `fetch` 请求
``` javascript
var result = fetch('/base/fcgi-bin/fcg_wxdownload_config.fcg', { credentials: 'include', headers: { 'Accept': 'application/json, text/plain, */*' } })
result.then(res =>{return res.text();}).then(text => { console.log(text) })
```
看看返回结果
![控制台输出](/images/react中跨域请求数据/1.png)
是不是很方便~