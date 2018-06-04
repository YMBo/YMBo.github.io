---
title: javascript时间处理小技巧
date: 2017-08-18 15:41:37
tags: javascript
---

# 获取当前时间
``` javascript
new Date()
```

# 获取当前时间的毫秒数
``` javascript
new Date().getTime()
/*返回1970年1月1日至今的毫秒数*/

/*下面是作用相同，写法不同*/
Date.now()

/*这个也返回值相同*/
+new Date()
```

# +”操作符
``` javascript
/*将元素转换成number类型*/
+'123'
/*返回123*/

+'www'
/*返回NaN*/

/*同理*/
+Date.now()
```

# toLocaleDateString()  和 toLocaleTimeString() 区别
``` javascript
new Date().toLocaleDateString()
/*2017/8/18  获取的年月日*/

new Date().toLocaleTimeString()
/*下午3:56:16 获取的时分秒*/

/*注意必须要用时间对象调用才行
   比如 Date.now().toLocaleDateString()会报错，因为Date.now()返回的是毫秒数
*/

```

# 时间格式处理技巧
如果想格式处理 当前时间与之前某个特定时间的时差 可以用这样的方式
``` javascript
/*duration为差值*/
duration>60 
	?`${Math.round(duration/60)}分钟前`
	: `${Math.round(Math.max(duration , 1))}秒前`
/*
1.利用了ES6的模板字符串，结构更清晰
2.利用三目运算符(三元运算符)进行判断，避免各种if
3.Math.max 如果差值小于1s，则按1s计算，避免又一次的if判断
*/
```