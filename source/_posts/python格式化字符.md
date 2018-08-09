---
title: python格式化字符
date: 2018-08-09 11:38:07
tags: python
---

### 一、前言
本文是我初学python对格式化字符操作中的一些相对疑难问题的记录和理解

### 二、python中格式化方法format

#### 1.简单用法
对于format最简单的用法就是这样了
``` python
name='YMBo'
age=18
print '{0} is {1} years old'.format(name,age)
```
输出为
> YMBo is 18 years old

对于这种字符串拼接也可以用这种方式：
``` python
name='YMBo'
age=18
print name+' and '+str(age)
```
* 注意！这种方式的字符串拼接类型都要为 str，所以`age`这一项要转为str，而format这中方式就不用

#### 2.复杂用法
``` python
name='YMBo'
print('lalallala :{0:3}'.format(name))
# {0:3}
```
* 0：这里的 0 表示第几个参数，这里只有一个所以是 0
* 3：这个3表示输出宽度，如果输出宽度小于字符串宽度则最后输出宽度为字符串宽度
------------------------------------
``` python
name='YMBo'
print('lalallala :{0:^3}'.format(name))
# {0:^3}
```
* 这里多了个 `^`表示右对齐
------------------------------------
``` python
name='YMBo'
print('lalallala :{0:.3f}'.format(1.0/3))
# {0:.3f}
```
* 0表示第一个参数
* ：后面没有数字表示宽度则自动分配
* .3表示小数点后3位
* f表示按照浮点数输出


#### 总结
对比一下，python字符串格式化操作和JavaScript很相似的
都有两种方式
1.字符串和变量++++这种操作，但是python需要将不是str类型的变量转为str类型，而js不用
2.python中'{}'.format()，js中\`${}\`这种操作
