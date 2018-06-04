---
title: select禁止更改值
date: 2017-11-30 10:31:59
tags: css
---
### 一、效果介绍
select框选中了一个值后禁止更改select

### 二、方法

#### 1.css
利用` disabled="true" `来禁止选中
`缺点`：这种方法虽然简单，但是它禁止了select框获得焦点，不能看到select的内容
`优点`：简单

#### 2.js
``` javascript
<select  name="selectname" id="selectid" onfocus="this.defaultIndex=this.selectedIndex;" onchange="this.selectedIndex=this.defaultIndex;">
	<option value="1">dd</option>
	<option value="2">mm</option>
	<option value="3" selected="selected">cc</option>
	<option value="4">ff</option>
</select>
```
1.获取焦点时，将当前的值 `selectedIndex` 赋值给一个自定义的属性 `defaultIndex` 
2.change时，将当前的值设置为 `defaultIndex`

`优点`：select框依然可以选择，被下拉

