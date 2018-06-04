---
title: sublime中使用EditorConfig
date: 2018-01-19 10:22:50
tags: 'EditorConfig'
---
## 一、作用
EditorConfig是统一代码格式的解决方案，它可以帮助开发者在不同的编辑器和IDE之间定义和维护一致的代码风格，比如多人合作的时候统一代码风格，避免一些潜在的问题，增加代码可读性

## 二、使用方法
### 1.编辑器插件
有些编辑器已经集成了这个插件，所以不用再安装，而有些编辑器没有集成这个插件，需要安装

下面这些编辑器不需要安装：
![已经集成的编辑器](/images/sublime中使用editconfig/1.png)
这些编辑器需要安装：
![需要单独安装插件的编辑器](/images/sublime中使用editconfig/2.png)

sublime里直接搜索 `EditorConfig` 安装即可
![安装](/images/sublime中使用editconfig/3.png)

### 2.配置文件说明

在需要的地方配置 `.editorconfig`文件
 >` 注意1： ` 当打开一个文件时，EditorConfig插件会在打开文件的目录和其每一级父目录查找.editorconfig文件，直到有一个配置文件root=true。
 >什么意思呢？ 
 >比如我的文件结构是这样的：
 >![文件目录](/images/sublime中使用editconfig/4.png) 
 >在 ` b.js ` 里的代码，会先从同级目录 ——> 父级目录 这个路径进行查找（冒泡 ），如果遇到了 `.editorconfig` 里 `root=true` 则停止，因为 `b.js` 同级目录的 `.editorconfig` 里 `root=true` 所以 b.js 用的就是b.js上面这个文件配置的规则

>----------------------
>` 注意2 `：EditorConfig配置文件从上往下读取，并且路径最近的文件最后被读取。匹配的配置属性按照属性应用在代码上，所以最接近代码文件的属性优先级最高。
>什么意思呢？
>![.editconfig](/images/sublime中使用editconfig/5.png) 
>意思就是 我的 js文件会读取 注释2 (第十行)下面的配置，而python文件会读取上面 (注释1)的配置 

### 3.文件格式详情
EditorConfig文件使用INI格式，斜杠(/)作为路径分隔符，#或者;作为注释。注释应该单独占一行。

### 4.支持的属性 
注意：不是每种插件都支持所有的属性，具体可见[Wiki](https://github.com/editorconfig/editorconfig/wiki/EditorConfig-Properties)。

indent_style：tab为hard-tabs，space为soft-tabs。
indent_size：设置整数表示规定每级缩进的列数和soft-tabs的宽度（译注：空格数）。如果设定为tab，则会使用tab_width的值（如果已指定）。
tab_width：设置整数用于指定替代tab的列数。默认值就是indent_size的值，一般无需指定。
end_of_line：定义换行符，支持lf、cr和crlf。
charset：编码格式，支持latin1、utf-8、utf-8-bom、utf-16be和utf-16le，不建议使用uft-8-bom。
trim_trailing_whitespace：设为true表示会除去换行行首的任意空白字符，false反之。
insert_final_newline：设为true表明使文件以一个空白行结尾，false反之。
root：表明是最顶层的配置文件，发现设为true时，才会停止查找.editorconfig文件。

## 三、我遇到的问题

看我的配置文件：
![.editconfig](/images/sublime中使用editconfig/6.jpg) 
我设置了缩进方式为 `tab`,每级缩进 8 列，但是我设置好后，打开a.js文件后每级的缩进并没有变化，为此我还找了好久~╮(╯▽╰)╭，最后我发现，只要把a.js关闭，再打开就生效了~

## 四、总结
EditorConfig 可以说很好用了，很适合多人合作情景，但这个只是编辑器级别的格式统一，我会在后面说到代码级别的检测---ESLint，它可以通过我们配置很详细的配置文件，来规范我们的代码风格。