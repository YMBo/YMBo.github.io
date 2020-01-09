---
title: javascript的数据属性和访问器属性
date: 2018-02-28 11:25:55
tags: javascript
---
## 一、数据属性
### 1.基本认识
#### 1.1概念

数据属性包含一个数据值的位置，在这个位置可以读取和写入值

#### 数据属性有4个描述其行为的特性
1. [[Configurable]] : 表示能否通过 delte 删除属性从而重新定义属性，能修改属性的特性，或者能把属性修改为访问器属性。默认true,（**说的磨磨唧唧，其实就是能否通过delete删除属性、能否再修改Enumerable、Writable（特殊，把Configurable设置为false后，这个属性也能被修改）、Value（更特殊，看下面例子）、Configurable属性这四个属性**）
2. [[Enumerable]] : 表示能否通过 for-in 循环返回属性。 默认 true
3. [[Writable]] : 表示能否修改属性的值 。 默认 true
4. [[Value]] : 包含这个属性的数据值。 默认 undefined

> tip：为了表示特性是内部值，该规范把他们放在了两对括号中

### 2.对数据属性的操作
必须使用 ECMAScript 5 的 ` Object.defineProperty() ` 方法
三个参数
1. 属性所在的对象
2. 属性的名字
3. 描述符对象

####  栗子
``` javascript
var person = {
	name:'y'
}
//禁止修改属性值
Object.defineProperty(person,"name",{
	writable:false
})
console.log(person.name) 		// y
//尝试修改属性
person.name='b';
console.log(person.name) 		//y

//可以看到禁止修改属性的值后，修改属性值无效

```
####  2.1注意！！！
这四个数据属性中，` configurable ` 是比较特殊的，一旦把属性定义为不可配置的，就不能再把它变回可配置了，此时再调用 ` Object.defineProperty ` 方法修改 `除writable之外的特性` 都会导致失败。
``` javascript
var b={
	age:0
}
//定义为不可配置
Object.defineProperty(b,'age',{
	configurable:false
})
//尝试删除配置
delete b.age
console.log(b.age)
// 成功打印0

//尝试修改enumerable
Object.defineProperty(b,"age",{
	enumerable:false
})
/* 报错：Uncaught TypeError: Cannot redefine property: age
    at Function.defineProperty (<anonymous>)
    at <anonymous>:3:8 */


//   看下面这俩顺序👇👇👇👇👇👇👇👇👇👇👇👇👇
//尝试修改writable
Object.defineProperty(b,"age",{
	writable:false
})
/*修改成功，不会报错*/

//尝试修改value
Object.defineProperty(b,"age",{
	value:1
})
/*报错：
VM17512:1 Uncaught TypeError: Cannot redefine property: age
    at Function.defineProperty (<anonymous>)
    at <anonymous>:1:8
*/
//  👆👆👆👆👆👆👆👆👆👆👆👆👆
```
> **tip**:上面的操作只针对于 b对象的age 属性，所以此时我再添加一个属性也是成功的，比如
``` javascript
b.name='m';
console.log(b)
// 输出 {age: 0, name: "m"}
```
> **tip2**:
看上面的代码 22-37行，这俩属性很特殊，
1. 就算`configurable:false`了，**writable属性依然可以被修改**
2. **注意注意！！**  看到代码里的顺序了么？writable的修改是在value的修改的上面的，这时修改value报错，但我发现如果`writable`和`configurable`同时为`false`（且的关系，不是或），这个value属性才会报错（也就是修改不了）,否则也是可以修改的

> **tip3**:
ES5有三个操作会忽略enumerable为false的属性。

> 1. for...in循环：只遍历对象自身的和继承的可枚举的属性
> 2. Object.keys()：返回对象自身的所有可枚举的属性的键名
> 3. JSON.stringify()：只串行化对象自身的可枚举的属性

## 二、访问器属性
### 1.基本认识
#### 概念
访问器属性不包含数据值。它包含一对getter和setter函数。当读取访问器属性时，会调用getter函数并返回有效值；
当写入访问器属性时，会调用setter函数并传入新值，setter函数负责处理数据。

#### 访问器属性的四个特性
1. [[Configurable]]：默认为true。表示能否通过delete删除属性从而重新定义属性，能否修改属性特性，或者能否把属性修改为访问器属性；
2. [[Enumerable]]：默认为true。表示能否通过for-in循环返回属性；
3. [[Get]]：读取属性时调用的函数，默认为undefined；
4. [[Set]]：写入属性时调用的函数，默认为undefined；

### 2.对访问器属性的操作
> tip :
> 1.当使用了getter或setter方法，不允许使用 ` writable ` 和 ` value ` 这两个属性。
> 2.get和set 函数不是必须的
> 3.先看这种写法：
> ![特别要注意这种写法，容易写错！](/images/javascript的数据属性和访问器属性/1.png)
> 可以看到 ` 堆栈溢出 ` 了，` 原因是因为如果在get里return this.year ，这样如果我们在外面 book .year时，就会运行get函数，而get函数里面执行到this.year时又会运行get函数，造成死循环。正常写法是利用一个中间值来设置某一个属性，看下面代码
![正确写法](/images/javascript的数据属性和访问器属性/2.png)
_year 前面的下划线是一种常用的记号，用于表示只能通过对象方法访问的属性。
一般上面上面代码目的是 设置一个属性的值会导致其他属性发生变化，就不用把属性单拿出来计算了，很方便

## 三、总结
### 1.到底怎么判断是数据属性还是访问器属性

可以通过 Object.getOwnPropertyNames()方法获取到所有实例中的属性，包括不可枚举的属性。
然后，使用Object.getOwnPropertyDescriptor(）方法获取到每一个属性的描述符，如果描述符中有get/set方法，说明它是访问器方法，否则它就是数据属性。
其实由于两种属性各自的4种特征都是都不一样的，如果一个对象的属性描述符里含有一个单独的特征就可以判断是什么类型的属性，比如
``` javascript
var book={};
Object.defineProperty(book,'age',{
	value:5
})
```
` 描述符里我单独定义了value属性，而value属性仅仅输入数据属性，所以可以判断出 age属于数据属性 `

### 2.定义多个属性

Object.defineProperty()这种写法每次只能定义一个属性，那么可以通过 `Object.defineProperties` 来定义多个属性
定义多个属性：
`请注意下面图片内代码是有问题的，这是《js高级程序设计》 里的例子`
![定义多个属性](/images/javascript的数据属性和访问器属性/3.png)
这个方法定义多个属性没问题，但是也可以从上面的图看出来，定义的访问器属性year的set方法里面并没有生效，这俩属于数据属性，` 原因是因为 定义的_year属性和edition属性不能往里面写值了 `。

自己解决历程：我自己猜想原因可能是这样，因为_year和edition写不进去值了，所以设置 enumerable 为true。我尝试修改了一下，如下
![给两个数据属性添加了writable(可读写 )特征](/images/javascript的数据属性和访问器属性/4.png)
这样运行成功。为了验证猜想，做了如下例子：

![所有特性](/images/javascript的数据属性和访问器属性/5.png)；

可以看到只要是用 `Object.defineProperties()`  定义的对象的属性，那么定义过的每个属性描述符的4个默认属性（除了set和get外）都默认都是 false

总结：使用 `Object.defineProperties()` 方法来定义多个属性的时候，每个属性的描述符里的属性都是默认的false(除了set和get外)，set或者get没定义的话默认 undefined

## 四、实际应用

介绍了这个东西后，在实际中我该在什么情况用呢？

**数据双向绑定 （Module<==>View）**
数据双向绑定就是通过 `Object.defineProperty` 实现的，我们来手写一个数据绑定
``` html
<input class="inputText" type="text" />
<p class="text"></p>
```	
``` javascript
var inputText=document.querySelector('.inputText');
var text=document.querySelector('.text');
//module
var obj={_txt:null};
Object.defineProperty(obj,'txt',{
	get:function(){
		return this._txt;
	},
	set:function(value){
		this._txt=value;
		text.innerHTML=value;
		inputText.value=value
	}
})
//view
inputText.addEventListener('input',function(e){
	const text=e.target.value;
	obj.txt=text;
},false)
```	
此时在input输入值的时候p的内容和obj.txt内容会变，在浏览器console中更改obj.txt的值，input和p也会变
