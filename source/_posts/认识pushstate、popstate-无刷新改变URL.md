---
title: '认识pushstate、popstate,无刷新改变URL'
date: 2017-09-22 14:39:04
tags: HTML5
---

# 一、回顾 `window.history`
`history`对象包含用户（在浏览器窗口中）访问过的url
``` javascript
//回退
history.back();
//前进
history.forward();
//跳转
history.go();
//历史记录条数（当前网页的，不是浏览器的）
history.length
//状态
history.state
```
# 二、认识新特性
HTML5扩展了`history`,使历史记录更加灵活，可以在历史记录中存储指定记录点、替换当前历史记录点，监听历史记录点

## 2.1.存储历史记录点
``` javascript
window.history.pushState('新添加的','','?page=3');
// 第一个参数：设置state
// 第二个参数：页面标题，但是所有浏览器都忽略了，传空字符串就行
// 第三个删除：想要添加的链接

//`注意，这个api会改变当前的网址，会添加一条历史记录，不是push到后面，这样会添加一条历史`
```
执行后，页面的URL为当前url？page=3
![执行前](/images/认识pushstate、popstate-无刷新改变URL/1.png)
![执行后](/images/认识pushstate、popstate-无刷新改变URL/2.png)
此时的`history.length`也会+1，新增了历史记录点

## 2.2.替换历史记录点
``` javascript
window.history.replaceState('新添加的','','?page=3');

//`注意，这个api会改变当前的网址(不跳转)，不会添加一条历史记录，注意与上面pushState区别`

> 补充：location.replace()也是替换当前网址，但是页面会跳转，而且不会添加历史记录
```
区别：
1.history.length不会变
2.替换了当前页的历史记录点

## 2.3.监听历史记录点
``` javascript
window.onpopstate=function(){}
```
浏览器前进后退都会触发这个事件
比如可以这样：
``` javascript
window.onpopstate=function(event){
    console.log(event.state)
}
```
后退操作时，就会打印出后退到的历史记录点的state信息。

# 四、用途
在我写`react`饿了么的项目时，点击首页的定位信息，会从右滑动出一个新的页面，此时再通过浏览器的前进后退按钮也可以实现页面的滑动
![浏览器控制动画](/images/认识pushstate、popstate-无刷新改变URL/3.gif)
具体制作过程看我的github：[ele项目](https://github.com/YMBo/react-ele) 

# 五、补充
在用 `document.referrer` 做返回按钮的时候，是只有 `a标签` 才能添加一条来源的。
比如我通过点击页面的一个 `链接` 调到了对应的页面，那么此时 `document.referrer` 是有值的，通过 `history.pushState` 添加的历史记录跳转的页面,不会添加一条 `document.referrer` 来源 