---
title: js的二进制
date: 2021-02-24 19:13:16
tags: [javascript, ArrayBuffer]
---

### 一、前言

[blob](https://ymbo.github.io/2021/02/24/js%E7%9A%84%E4%BA%8C%E8%BF%9B%E5%88%B6/)

本篇是关于 js 二进制从操作的一些介绍，其中**提出问题**相关是我在操作过程中的一些疑问，在多次尝试并搜索相关资料后得出的结论，重点关注

### 二、ArrayBuffer 概念

用来表示通用的、固定长度的原始二进制数据缓冲区
它是一个字节数组，每一位表示一个字节

### 三、特点

不能读写，只能通过 TypedArray 视图和 DataView 视图来读写

### 四、关于类型

```javascript
// 创建一个 32 字节的内存空间，默认值0
let buf = new ArrayBuffer(32);
```

![视图](/images/js的二进制/1.png)

可以看到以上几种类型视图，Int8Array、Int16Array、Int32Array、Uint8Array

> **提出问题**：为什么每种视图长度不一样？
> 1 字节=8 位
> 32 字节=8\*32 位
> 所以在`有符号8位整数类型`（一项表示一字节）中的长度是 32
> `有符号16位整数类型`（一项表示两个字节）中的长度是 16
> `有符号32位整数类型`（一项表示四个字节）中的长度是 8
> `无符号8位整数类型`（一项表示一字节）中的长度是 32

### 五、ArrayBuffer 读写

> tip:写操作是针对于内存的，所以通过视图修改字节后，原始 buffer 也同步变化

#### 1、通过 DataView 视图读写

```javascript
//  buf也同步改变
const buf = new ArrayBuffer(32);
const dataView = new DataView(buf);
// 设置第一个字节为整数1，第二个字节为整数2，并获取第二个字节
// 其他类型同理
dataView.setInt8(0, 1);
dataView.setInt8(1, 2);
dataView.getInt8(1);
console.log(buf);
```

![结果](/images/js的二进制/2.png)

> **提出问题**：为什么 Int8Array、Int16Array、Int32Array 前两项不一样
> Int8Array 的两项相当于 Int16Array 的一项
> 1 的二进制：00000001
> 2 的二进制：00000010
> 两项转一项 Int8Array>Int16Array
> 00000001,00000010 =>0000001000000001 =513

#### 2、通过 typesArray 视图读写

Uint8Array、Int8Array、Int16Array 的实例是**类数组对象**，不是数组但可以用数组的一些方法 map、every 等

```javascript
const buffer = new ArrayBuffer(12);
const x1 = new Int8Array(buffer);
x1[0] = 1;
const x2 = new Uint8Array(buffer);
```

### 六、关于溢出

转换规则：
**正向溢出（overflow）**：当输入值大于当前数据类型的最大值，结果等于当前数据类型的最小值加上余值，再减去 1。
**负向溢出（underflow）**：当输入值小于当前数据类型的最小值，结果等于当前数据类型的最大值减去余值的绝对值，再加上 1。

> 一个有符号 8 位整数的范围是[-128,127],那如果赋值`130`会变成啥数呢？
> 130%127=3
> -128+3-1=-126
> 所以最后的值是-126
> ![-120](/images/js的二进制/3.png)

> 赋值`-130`同理:
> 127-(|-130%-128|)+1=126
> ![-120](/images/js的二进制/4.png)
> 其实这个东西相当于一个表盘，从-128 开始 127 结束
> ![表盘](/images/js的二进制/5.png)

### 七、arrayBuffer 转 base64

#### 1、第一种方法

`btoa`:用于创建一个 base-64 编码的字符串。参数是字符串

```javascript
const buffer = new ArrayBuffer(12);
const uint8Array = new Uint8Array(buffer);
let str = String.fromCharCode(...uint8Array);
// 拼上文件头
let base64 = `data:image/jpeg;base64,${window.btoa(str)}`;
```

`注意注意！！` 这种方式如果 buffer 太大就会转换失败，`fromCharCode参数太大就会这样`
`Uncaught (in promise) RangeError: Maximum call stack size exceeded`

#### 2、第二种方法

tips:这种方法大图片特别费时间

```javascript
var binary = '';
var bytes = new Uint8Array(buffer);
var len = bytes.byteLength;
for (var i = 0; i < len; i++) {
  binary += String.fromCharCode(bytes[i]);
}
window.btoa(binary);
```

#### 3、第三种方法

tips：这种方法不会出现转换失败和费时间

先将 buffer 转为 blob 对象，再把 blob 转为 base64

```javascript
const buffer = new ArrayBuffer(buffer数据);
let blob = new Blob([buffer]);
let reader = new FileReader();
reader.onload = function () {
  const base = reader.result;
  console.log(base);
};
reader.readAsDataURL(blob);
```
