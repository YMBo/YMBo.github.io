---
title: Blob
date: 2021-03-01 10:53:51
tags: [javascript, Blob]
---

### 前情提要

[arrayBuffer](https://ymbo.github.io/2021/02/24/js%E7%9A%84%E4%BA%8C%E8%BF%9B%E5%88%B6/)
![Blob](/images/Blob/3.jpeg)

### 一、概念

表示一个不可变、原始数据的类文件对象
文件对象 File 继承了所有 Blob 的属性

### 二、用途

![Blob](/images/Blob/1.png)

> tips:window.URL.createObjectUR 转换的 Url 相当于一个引用，只能在当前页面有效

> **提出问题：**什么样的数据 createObjectURL 后才能赋值给 img.src 展示出来？

### 三、例子

#### 1、文件下载

**文本内容下载**

```javascript
// <a href="" download="" id="dd">点击下载</a>
let blob = new Blob(['下载内容内容']);
let url = URL.createObjectURL(blob);
let a = document.getElementById('dd');
a.download = '下载标题';
a.href = url;
```

上面是文本类型文件下载，如果 Blob 的内容是图片、压缩资源的 buffer 那就意味着前端就可以做任何类型的文件下载

**图片资源**

随便找个百度图片（不能跨域）

```javascript
fetch(
  'https://gimg2.baidu.com/image_search/src=http%3A%2F%2Fimages4.fanpop.com%2Fimage%2Fphotos%2F21800000%2FPain-akatsuki-21871797-1280-720.jpg&refer=http%3A%2F%2Fimages4.fanpop.com&app=2002&size=f9999,10000&q=a80&n=0&g=0n&fmt=jpeg?sec=1617163410&t=b4da03a23ce7038703128e42919f5448'
).then((r) => {
  r.blob().then((data) => {
    console.log(data);
    let url = URL.createObjectURL(data);
    let a = document.getElementById('dd');
    a.download = '下载标题';
    a.href = url;
  });
});
```

#### 2、Blob 文件分片上传

通过 Blob.slice 将资源文件切片，分段上传资源，后端再将这些分片资源拼接起来

### 四、Blob 资源转换

因为 File 是继承 Blob，所以 Blob 的方法 File 也能用

1. 通过 **FileReader** 对象
   1. **FileReader.readAsText(Blob/File)**：将 Blob 转为文本字符串
   2. **FileReader.readAsArrayBuffer(Blob/File)**： 将 Blob 转为 ArrayBuffer 格式数据
   3. **FileReader.readAsDataURL(Blob/File)**: 将 Blob 转化为 Base64 格式的 Data URL
2. 通过 **Blob/File** 对象
   1. **Blob.text()** ：将 Blob 转化为文本字符串
   2. **Blob.arrayBuffer()** ：将 Blob 转为 ArrayBuffer 格式数据

#### 1、FileReader

> tips:
>
> 1. 调用 FileReader Api 解析的结果都是通过 onload 回调取到的
> 2. FileReader 上述 Api 的参数是 Blob 或 File 对象

```javascript
// 创建一个FileReader对象
const reader = new FileReader();
// 注册onload方法，解析成功的数据通过这个回调函数返回
reader.onload = function () {
  const content = reader.result;
  console.log('content', content);
};
// 触发解析api
reader.readAsArrayBuffer(file);
```

#### 2、Blob/File

> tips:
> Blob 上的转换方法都返回一个 promise

```javascript
document.getElementById('f').addEventListener(
  'change',
  function (e) {
    var file = this.files[0];
    file.arrayBuffer().then((text) => {
      console.log('111', text);
    });
    file.text().then((text) => {
      console.log('111', text);
    });
  },
  false
);
```

### 四、Blob 数据分析

#### 1. 当 blob 转换为 Arraybuffer 后，数据是按什么规则转换的？

```javascript
let a = ['a'];
let b = new Blob(a);
b.arrayBuffer().then((r) => {
  console.log(r);
});
```

![arrayBuffer](/images/Blob/2.png)

可以看到'a'的 arrayBuffer 是 97，所以转换规则应该是这样的：
`将字符串转成的 Unicode 编码`

```javascript
String.fromCharCode(97);
// "a"
'a'.charCodeAt();
// 97
```

#### 2、什么样的数据 createObjectURL 后才能赋值给 img.src 展示出来？

blob 存储的内容是一个数组，通过 input type='file'标签传的图片可以直接 createrObjectUrl 就能给 img.src 直接展示，
**这种情况下 blob 存储的是图片文件的 base64 内容**
而在一些情况下比如`readPixels()`返回的数据里，**存储的是图像的像素点 rgba**，所以不能直接用 createrObjectUrl 转换成图片显示
