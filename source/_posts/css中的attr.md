---
title: css中的attr()
date: 2017-10-11 11:43:15
tags: [css,attr()]
---
# 1.介绍
`attr()` 用来获取一个html元素的属性值，可以用于伪元素，可以用于任何css属性

# 2.用法
这个使用起来非常方便且简单
``` css
content:attr(属性值);
```
`attr()`可以传三个值，分别是属性值、类型限制、默认值（必须符合类型限制），不过我试了半天也没调试成功，而且一般写一个参数就行，三个参数的情况基本用不到，所以就跳过了

# 3.例子
``` html
<hear>
	<style type="text/css">
		.box{
			width: 400px;
			height: 400px;
			border:1px solid #ccc;
		}
		.box:after{
			content:attr(data-foo);
			background-color: #ff461d;
			color: #fff;
			border-radius: .32rem;
			padding: .053333rem .133333rem;
		}
	</style>
</head>
<body>
	<div class="box" data-foo="hello"></div>
</body>
```
此时，在div中将会出现一个红色背景的`hello`

# 4.实例
那么这个东西在什么情况下能用到呢？
![购物车](/images/css中的attr/1.png)