---
title: cocos/webgl截图
date: 2021-03-05 15:55:06
tags: [cocos-creater, canvas, webgl]
---

（待补充）

### 背景

本文是介绍 cocos-creator 截图遇到的问题、原理和解决方案

### 方案

#### 第一种方案

因为 cocos-creater 本质就是一个 canvas，所以直接用 canvas.toDataUrl()获取 base64 即可
**问题**
如果实际操作一下就会发现，通过 canvas.toDataUrl()获取的 base64 数据是透明的，没有任何内容
**问题分析及原理**
关于 canvas 和 webgl 的工作原理了，跟绘图缓冲区有关，还不太懂 留坑
canvas

#### 第二种方案

通过查找文档得知 cocos creator 有个这个方法 readPixels，
从 render texture 读取像素数据，数据类型为 RGBA 格式的 Uint8Array 数组。 默认每次调用此函数会生成一个大小为 （长 x 高 x 4） 的 Uint8Array。 你可以通过传入 data 来接收像素数据，也可以通过传参来指定需要读取的区域的像素。

其实为啥上面标签打上 webgl 呢，因为这个方法我看 cocos 的实现，是在 webgl 的基础上封装了一下，cocos 也是基于 webgl 的，所以它俩的截图处理应该也一致

现在能获取了图像的像素数据了！

#### 像素数据的处理

[知识回归](https://ymbo.github.io/2021/03/01/Blob/#2%E3%80%81%E4%BB%80%E4%B9%88%E6%A0%B7%E7%9A%84%E6%95%B0%E6%8D%AE-createObjectURL-%E5%90%8E%E6%89%8D%E8%83%BD%E8%B5%8B%E5%80%BC%E7%BB%99-img-src-%E5%B1%95%E7%A4%BA%E5%87%BA%E6%9D%A5%EF%BC%9F)
由之前写的这个得知，readPixels 得到的数据不能直接转 blob 然后 createrObjectUrl，因为是像素点数据

所以思路应该是 先将这些像素点在 canvas 绘制出来，然后再调用 toDataUrl()转成图片
