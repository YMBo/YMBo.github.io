---
title: 箭头函数的this指向
date: 2017-08-25 15:14:31
tags: ES6
---
### 前言 
 es6的箭头函数非常简洁，而且还可以解决很多问题
1.解决以前通过var that=this方式传值的问题
``` javascript
document.body.addEventListener('click',function(){
	setTimeout(function(){
		console.log(this)
		/*window*/
		/*对body节点操作代码...*/
	})
})
```
比如点击body，过一段时间对body这个元素进行对应的js操作，如上代码，很明显是不对的，因为`setTimeout`里的`this`是全局`window`所以是不能通过它操作body元素，

一般这种情况以前都是通过在外层通过一个变量将this传递进来，但是有了箭头函数就不用这么做了

es6:
``` javascript
document.body.addEventListener('click',function(){
	setTimeout(()=>{
		console.log(this)
		/*body(当前节点)*/
		/*对body节点操作代码...*/
	})
})
```
这类的问题还有很多，但是为什么`箭头函数`能轻易地解决这些问题呢？
所以要清楚能这么写的原因就要清楚`箭头函数中this`的指向

### 一、写箭头函数的小提示
#### 1.当使用箭头函数创建普通对象时，你总是需要将对象包裹在小括号里。
``` javascript
/*错误*/
()=>{}
/*正确*/
()=>({})
```
因为JavaScript引擎会将{x}理解成一个代码块，空对象和块在这里都是`{}`这样表示，所以如果返回的是一个对象，需要加一个小括号

### 箭头函数的`this`
`箭头函数没有自己的this!!!!!!!!!`,它内部的this值集成自外围作用域
``` javascript
function foo() {
  return () => {
    return () => {
      return () => {
        console.log('id:', this.id);
      };
    };
  };
}

var f = foo.call({id: 1});

var t1 = f.call({id: 2})()(); // id: 1
var t2 = f().call({id: 3})(); // id: 1
var t3 = f()().call({id: 4}); // id: 1
```
t1、t2、t3都输出1
因为箭头函数没有自己的this所以`this.id`会顺着作用域链查找一直找到foo()函数

### 箭头函数的的特点

它没有`arguments`变量
``` javascript
function foo() {
  setTimeout(() => {
    console.log('args:', arguments);
  }, 100);
}

foo(2, 4, 6, 8)
// args: [2, 4, 6, 8]
```
上面代码中，箭头函数内部的变量arguments，其实是函数foo的arguments变量。


如果有了新发现，会回来补充