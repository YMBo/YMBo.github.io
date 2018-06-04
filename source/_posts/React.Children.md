---
title: react中的this.props.children
date: 2017-09-28 17:34:55
tags: react
---
## 一、介绍    
`React.Children` 是顶层API之一，为处理 `this.props.children` 提供了几个方法    
`this.props.children` 表示组件所有子节点    
    
## 二、所有方法    
### 1.React.Children.map    
``` javascript
//必须有返回值
React.Children.map(this.props.children,function(child){
	return <li>{child}</li>
})
//没有返回值
React.Children.map(this.props.children,function(child){
	/*这里进行处理*/
})
```
这里的 `child` 表示组件的每一个子元素，也可以用箭头函数来写，这样方便this的传递    

### 2.this.props.children
这个是获取当前组件的所有子节点
`注意` ：
1.如果没有子节点，返回 `undefined`
2.如果一个子节点，返回 `object`
3.如果多个子节点，返回 `array`
但是用 `React.Children.map` 来遍历的话不会有问题

### 3.React.Children.count
返回组件的所有子元素个数

### 4.React.Children.only
``` javascript
console.log(React.Children.only(this.props.children[0])); 
//输出对象this.props.children[0]
```
单独 `this.props.children[0]` 输出不出来

### 5.child.key
在用 `React.Children.map` 的时候可能想获取传过来的属性值，例如
``` javascript
<Tabs_li data={data}>
		<div key={0}>1</div>
		<div key={1}>2</div>
		<div key={2}>3</div>
</Tabs_li>

/*Tabs组件*/
{React.Children.map(this.props.children,(child)=>{
	console.log(child.key) 
	// 分别打印 0 ， 1 ，2
	return ...
})}
```
