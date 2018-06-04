---
title: react生命周期详解
date: 2018-03-27 17:11:54
tags: react
---

### 一、react生命周期
* componentWillMount
* componentDidMount
* componentWillReceiveProps
* shouldComponentUpdate
* componentWillUpdate
* componentDidUpdate
* componentUnmount
每个生命周期概念不是本文讲解重点，重点是生命周期在组件渲染时候容易造成的混淆

### 二、componentWillReceiveProps 和 componentWillUpdate ？？

`componentWillReceiveProps` ： 组件接收到一个新的props会被调用，在初始化render时不会调用
`componentWillUpdate` : 组件接收到系的props`或state`时被调用，初始化不会被调用

看起来好像一样啊，有什么区别呢？来看下面

#### 1.来做测试
先定义两个组件，App和Son，我们的目的就是看看上面提到的两个生命周期的不同，所以这一部分暂不讨论每个生命周期执行顺序，留在下一节讨论。
``` javascript
class App extends Component{
	constructor(){
		super()
		this.state={
			word:'儿子'
		}
	}
	componentWillMount(){
		console.log('他爹生命周期：componentWillMount')
	}
	componentDidMount(){
		console.log('他爹生命周期：componentDidMount')
	}
	componentWillReceiveProps(nextprops){
		console.log('他爹生命周期：componentWillReceiveProps')
	}
	shouldComponentUpdate(){
		console.log('他爹生命周期：shouldComponentUpdate')
		return true;
	}
	componentWillUpdate(nextprops,nextstates){
		console.log('他爹生命周期：componentWillUpdate')
	}
	componentDidUpdate(){
		console.log('他爹生命周期：componentDidUpdate')
	}
	componentWillUnmount(){
		console.log('他爹生命周期：componentWillUnmount')
	}
	handleClick(word){
		this.setState({
			word:this.state.word ==='儿子' ? '孙子':'儿子'
		})
	}
	render(){
		console.log('他爹render')
		return(
			<div>
				<button onClick={this.handleClick.bind(this)}>App点我</button>
				<Son word={this.state.word} />
			</div>
		)
	}
}
class Son extends Component{
	constructor(){
		super()
		this.state={
			word:''
		}
	}
	componentWillMount(){
		console.log('儿子生命周期：componentWillMount')
		this.setState({
			word:this.props.word
		})
	}
	componentDidMount(){
		console.log('儿子生命周期：componentDidMount')
	}
	componentWillReceiveProps(nextprops){
		console.log('儿子生命周期：componentWillReceiveProps')
	}
	shouldComponentUpdate(){
		console.log('儿子生命周期：shouldComponentUpdate')
		return true;
	}
	componentWillUpdate(nextprops,nextstates){
		console.log('儿子生命周期：componentWillUpdate')
	}
	componentDidUpdate(){
		console.log('儿子生命周期：componentDidUpdate')
	}
	componentWillUnmount(){
		console.log('儿子生命周期：componentWillUnmount')
	}
	handleClick(){
		this.setState({
			word:this.state.word ==='儿子' ? '孙子':'儿子'
		})
	}
	render(){
		console.log('儿子：render')
		return(
			<div>
			{this.state.word}
			<button onClick={this.handleClick.bind(this)}>儿子按钮</button>
			</div>
		)
	}
}
```
逻辑很简单，父组件和子组件都可以通过各自的按钮改变子组件文字的内容

先点击 `儿子按钮` 按钮，也就是子组件Son里的按钮
![执行结果](/images/react生命周期详解/1.png)
可以看到 *componentWillReceiveProps*  没有执行

然后点击 `APP点我` 按钮：
点之前：
![点之前](/images/react生命周期详解/2.png)

点之后：
![点之后](/images/react生命周期详解/3.png)

可以看到子组件的文字并没有发生变化，执行了上图中的这些生命周期方法。

所以可以得到结论：componentWillReceiveProps触发条件是props更新，而componentWillUpdate触发条件是props或者states更新，上面说的时候也着重标记了

为了让父组件的按钮能更改子组件的内容，我需要在*componentWillReceiveProps*或*componentWillUpdate*里写上
``` javascript
this.setState({
	word:nextprops.word
})
```
官网上说不能写在*componentWillUpdate*里，建议写在*componentWillReceiveProps*，我就不明白了，我非得写在*componentWillUpdate*里，看看执行结果
![报错](/images/react生命周期详解/4.png)

看 死循环了，因为上面提到*componentWillUpdate*触发条件是 states 变化 和 props变化，写在了这里就不停的setState  states的值，所以就不停的触发*componentWillUpdate*，因此像这种父组件更改子组件的内容应该写在 *componentWillReceiveProps* 里，来试试：
![ojbk](/images/react生命周期详解/5.png)
好，一切正常！

** 总结：**

名称 | componentWillReceiveProps | componentWillUpdate 
- | :-: | :-:
触发条件 | props变化 |  props或states变化
注意事项 | setState的更新要写在componentWillReceiveProps里

		
	
### 三、生命周期执行顺序
还是上面的例子

#### 1.初始化时父子组件的声明周期执行顺序
![结果](/images/react生命周期详解/6.png)

#### 2.点击父组件的按钮时生命周期执行顺序
![结果](/images/react生命周期详解/7.png)

#### 3.点击子组件执行顺序
![结果](/images/react生命周期详解/8.png)

#### 4.还有一种情况，点击子组件触发父组件内容的更改
``` javascript
class Son extends Component{
	ponentWillMount(){
		console.log('儿子生命周期：componentWillMount')
	}
	componentDidMount(){
		console.log('儿子生命周期：componentDidMount')
	}
	componentWillReceiveProps(nextprops){
		console.log('儿子生命周期：componentWillReceiveProps')
	}
	shouldComponentUpdate(){
		console.log('儿子生命周期：shouldComponentUpdate')
		return true;
	}
	componentWillUpdate(nextprops,nextstates){
		console.log('儿子生命周期：componentWillUpdate')
	}
	componentDidUpdate(){
		console.log('儿子生命周期：componentDidUpdate')
	}
	componentWillUnmount(){
		console.log('儿子生命周期：componentWillUnmount')
	}
	handleClick(){
		var word=this.props.word === '儿子' ? '孙子':'儿子'
		this.props.click(word)
	}
	render(){
		console.log('儿子：render')
		return(
			<div>
			<button onClick={this.handleClick.bind(this)}>儿子按钮</button>
			</div>
		)
	}
}
class App extends Component{
	constructor(){
		super()
		this.state={
			word:'儿子'
		}
	}
	componentWillMount(){
		console.log('他爹生命周期：componentWillMount')
	}
	componentDidMount(){
		console.log('他爹生命周期：componentDidMount')
	}
	componentWillReceiveProps(nextprops){
		console.log('他爹生命周期：componentWillReceiveProps')
	}
	shouldComponentUpdate(){
		console.log('他爹生命周期：shouldComponentUpdate')
		return true;
	}
	componentWillUpdate(nextprops,nextstates){
		console.log('他爹生命周期：componentWillUpdate')
	}
	componentDidUpdate(){
		console.log('他爹生命周期：componentDidUpdate')
	}
	componentWillUnmount(){
		console.log('他爹生命周期：componentWillUnmount')
	}
	handleClick(word){
		this.setState({
			word:word
		})
	}
	render(){
		console.log('他爹render')
		return(
			<div>
				{this.state.word}
				<Son word={this.state.word} click={this.handleClick.bind(this)} />
			</div>
		)
	}
}
```

![结果](/images/react生命周期详解/9.png)

**上述的几种执行结果也很简单，看一看就会执行顺序就会变得清晰了**