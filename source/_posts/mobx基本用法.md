---
title: mobx基本用法
date: 2018-03-06 09:59:46
tags: mobx
---

### 一、前言
本文是我初学mobx时对mobx一些基本的认识，如果新发现会继续更新

### 二、redux和mobx

相同点：都是用来管理JavaScript应用的状态，他们不一定要跟react结合使用，还可以与别的框架结合

不同点：


1. **redux**学习成本相对于mobx成本要高很多，有reducer、action、dispatch等概念，规则多，比如更新数据必须要用 dispatch，更新的逻辑必须要用 action，刚学的时候可能有点懵。
**mobx**比较自由，可以用obj.key 的方式更新
2. **redux**更新数据的时候，要将更新数据的整个对象替换为一个新的对象才可以触发更新(这点接触过redux的会有感受)，而mobx自始至终是一份引用，所以redux每次会触发很多的组件的重新渲染，为了优化会配合immutable。
**mobx**则是更新哪个属性，仅仅这个属性所在的位置会重新渲染（不是组件的重新渲染，不触发componentWillMount等方法，会触发componentWillUpdata）

以上为自己理解，如有错误烦请指出

### 三、mobx基本概念
> tip:以下提到的 `@` 是ES7里的修饰器，可以到网上找资源学习 es7的修饰器

#### 1. **@observable **
将属性转为可观察的，一旦发生变化，则变量所处的位置会立即发生变化

#### 2. **@computed**

这是一个有点不好理解的API，而且官网描述的也不是很清楚，下面是我的理解，如果有误，请指出
讲这个API之前先看一下下面的例子：
``` javascript
/************************************mobx********************************/

import { observable ,computed,autorun,action } from 'mobx'

class AppState {
	@observable count=0;
	@observable max = 5;
	@computed get msg(){
		console.log('msg的getter里执行...')
		return `msg结果===>${this.max>this.count}`
	}
	@action add(){
		this.count +=1;
	}
	@action changeName(c){
		this.max=c
	}
}
const appState = new AppState;
setInterval(()=>{
	appState.add()
},2000)
export default appState;

/************************************index.jsx********************************/

import React,{ Component } from 'react'
import {observer,inject} from 'mobx-react'
import PropTypes from 'prop-types'

@inject('appState')

@observer
export default class TopicList extends Component{
	constructor(){
		super()
		this.changeName=this.changeName.bind(this)
	}
	componentWillUpdate(){
		console.log('更新！')
	}
	changeName(event){
		this.props.appState.changeName(event.target.value)
	}
	render(){
		return (
			<div>
				<input type="text" onChange={this.changeName}/>
				<span>{this.props.appState.msg}</span>
			</div>
		)
	}
}

TopicList.propTypes={
	appState:PropTypes.object
}

```
> 逻辑是有两个变量 count和max，在msg的getter里判断max>count的情况,其中max为固定值，count每秒+1。

在第二节的redux和mobx异同之处提出了mobx数据变化的时候组件只会触发 ` componentWillUpdata ` （没什么来源，自己试的），下面来看看上面代码 (` mobx里msg部分加了 @computed `) 运行情况：


![加了 @computed](/images/mobx基本用法/1.gif)

** 可以看到只有当 `msg` 值发生变化的时候，才会触发 ` componentWillUpdata ` ，也就是前5s msg的结果一直为 true时，组件里是没有接收到新的 {this.props.appState.msg} 的（因为接受新的props值会触发 componentWillUpdata）**

下面再来看看 ` mobx里msg部分不加 @computed `  运行情况
``` javascript
get msg(){
	console.log('msg的getter里执行...')
	return `msg结果===>${this.max>this.count}`
}
```
![不加 @computed](/images/mobx基本用法/2.gif)

** 可以看到每一次的count值变化都会触发组件的componentWillUpdata **

看明白了上面的例子和运行时的不同，再看看下面的总结就会好理解了

** 总结：
如果使用了@computed （@computed msg getter()），那么msg的值将会被缓存，如果count的变化没有触发msg值的变话，那么msg的getter()值就不会改变，index.jsx组件也不会收到通知(componentWillUpdata)。（第一种情况）**

** 如果不使用@computed属性，直接msg getter()的话，那么一旦count改变，所有用到msg getter()的地方都将重新计算(第二种情况)。 
 ** 
 > @computed的意义在于它能够由MobX进行更智能的优化


 #### 3. ** autorun **
 定义的 @observable 变量如果发生变化，autorun会自动执行相应的方法,例如将上面的mobx修改，index.jsx不变

 ``` javascript 
 import { observable ,computed,autorun,action } from 'mobx'

 class AppState {
 	@observable count=0;
 	@observable max = 5;
 	@computed get msg(){
 		console.log('msg的getter里执行...')
 		return `msg结果===>${this.max>this.count}`
 	}
 	@action add(){
 		this.count +=1;
 	}
 	@action changeName(c){
 		this.max=c
 	}
 }

 const appState = new AppState;
// 添加了 autorun
 autorun(()=>{
 	console.log(`${appState.count} 运行运行~`)
 })
 setInterval(()=>{
 	appState.add()
 },1000)
 export default appState;
 ```
 因为autorun里的函数依赖了 count ，所以每一次setInterval的时候，都会执行autorun里的函数

 #### 4.** @action**
 如果要更新store里的内容，就要在相应的函数前加上 `@action` 例如上面的例子，标记为@action 。

 作用：在mobx-react高级渲染性能优化小节中，我们知道，使用transaction可以将多个应用状态(Observable)的更新视为一次操作，并只触发一次监听者(Reactions)的动作(UI更新、网络请求等)，从而更大程度地提升应用的性能，避免多余的UI渲染和网络请求。action中封装了transaction，对函数使用action修饰符后，无论函数中对@observable变量（应用状态）有多少次修改，都只会在函数执行完成后，触发一次对应的监听者。如下代码，reset函数只会触发一次UI更新。
 ``` javascript
 class TodoItemModel {
     id;
     @observable title;
     @observable completed;

     //使用action后，reset函数执行完成后，才会触发一次其监听者
     @action
     reset() {
         this.completed = false;
         this.title= '';
     }
 }
 ```


 #### 5.** @oberver**

 observer 函数/装饰器可以用来将 React 组件转变成响应式组件
 oberver是用来连接mobx与组件的一个API（告诉mobx本组件依赖于mobx的状态），由一个单独的包 mobx-react 提供，例子看上面的 index.jsx组件。使用了 observer 的react组件中用到的变量发生变化，组件才会更新

 ### 四、运行环境的配置

@(修饰器)是ES7的一个提案，Babel 转码器已经支持 Decorator
需要安装 babel-preset-stage-1 （stage-0也可以）和babel-plugin-transform-decorators，babel-plugin-transform-decorators一定要放在别的plugins前

![.babelrc配置](/images/mobx基本用法/3.png)

#### 6. **useStaticRendering**
我们知道可以通过使用@observer，将react组件转换成一个监听者(Reactions)，这样在被监听的应用状态变量(Observable)有更新时，react组件就会重新渲染。而对于服务端的React组件，我们只需要它被渲染一次，而不需要组件监听模型的状态。事实上，如果服务端React组件像客户端组件一样监听模型的状态变化，就会造成严重的内存泄漏问题。官方提供了useStaticRendering方法，用于避免mobx服务端渲染的内存泄漏问题; 该方法只需要在server启动时设置一次。

useStaticRendering(true);