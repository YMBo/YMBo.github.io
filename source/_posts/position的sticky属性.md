---
title: position的sticky属性
date: 2017-09-13 10:25:03
tags: css
---

![表头跟随](/images/position的sticky属性/1.gif)
图中展示的效果使我们在日常开发中常见的效果。
以前我都是用js判断是否滚动一定距离然后给这个元素设置`position:fixed`这种方式来做的，
今天就来记录一下用css怎么做

### 一、介绍
`css` 中的 `position` 属性的值常用的主要有下面几种：
1.absolute
2.relative
3.fixed
现在用到的值为 `sticky`：
设置了`sticky`的元素，不脱离文档流，在屏幕范围时（viewport），该元素位置不受到定位影响（top、left等值无效），当该元素的视图将要离开偏移范围时，定位会变成 `fixed` 的效果，并且根据设置的top 、left值进行定位

### 二、用法
上面的gif图里，我将搜索框的元素设置为
``` css
postion:sticky;
top:0;
left:0
```
所以当移出屏幕时就会触发fixed定位

### 三、兼容性
刚刚把它用在实际项目中去了，结果安卓自带的浏览器不支持，所以附加上解决办法：
``` javascript
//判断是否支持
if (CSS.supports("position", "sticky") || CSS.supports("position", "-webkit-sticky")) {
    // 支持 sticky,顶部吸附
	this.setState({
		support:true
	})
}else{
	document.addEventListener('scroll',this._scrollTop)
}
//不支持的主要处理
_scrollTop(){
	let offsetY=this.searchDOM.offsetTop;
	let css=null;
	if(window.scrollY>offsetY){
		css={
			position:'fixed',
			top:0,
			zIndex:999
		}
	}else{
		css={
			position:'static',
		}
	}
	this.setState({
		css
	})
}
```

### 四、总结
* 该元素不脱离文档流，仍保留元素原本在文档流中的位置
* 当元素在容器中被滚动超过指定偏移值时，元素在容器内固定在指定位置。比如你设置了 `top:50px` 那么sticky元素到达距离相对定位的元素顶部50px的位置时固定，不再向上移动
* 元素固定的相对偏移是相对于它最近的具有滚动框的祖先元素，如果祖先元素都不可以滚动，那么是相对于 `viewport` 来计算元素的偏移量