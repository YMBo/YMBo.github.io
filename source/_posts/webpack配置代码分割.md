---
title: webpack配置代码分割
date: 2018-05-21 18:05:27
tags: webpack
---

## 一、前言
webpack 4.0 使用 `optimization.splitChunks.cacheGroups` 配置项来进行包的拆分，其实默认情况下，webpack是会自动帮我们分割的，但是有时候我们可能也需要自定义配置，下面来说下我对这个配置项的理解，如有错误，烦请指出

## 二、配置项说明
1. ** cacheGroups**                           
    * 用这个配置项来自定义生成的文件
2. ** test **                                              
    * 限制范围，可以是正则，匹配文件夹或文件
3. ** name**                                            
    * 生成的文件名
4. ** priority**                                         
    * 优先级，当有chunks满足多个分组条件的时候，优先选择优先级高的
5. ** minSize**                                        
    * 分出来的包最小尺寸必须大于此值，默认30000B
6. ** minChunks**                                   
    * 表示分离前被引用次数,默认为1
7. ** maxInitialRequests**                       
    * 最大初始化加载次数，一个入口文件可以并行加载的最大文件数量，默认1
8. ** maxAsyncRequests**                      
    * 最大按需加载次数，最大异步加载次数，默认1
9. ** enforce: true**                                
    * 优先处理，这一项好像和priority有些重叠了
10. ** reuseExistingChunk:true**            
    * 表示可以使用已经存在的块，即如果满足条件的块已经存在就是用己有的的，不再创建一个新的块
11. ** chunks 值为"initial", "async"（默认） 或 "all"** 
    * initial 入口chunk，对于异步导入的文件不处理
    * async 异步chunk，只对异步导入的文件处理
    * all 全部chunk
    
12.
``` javascript
runtimeChunk: {
    name: 'manifest'
        /* 管理被分出来的包，runtime 指的是 webpack 的运行环境(具体作用就是模块解析, 加载) 和 模块信息清单，
                    模块信息清单在每次有模块变更(hash 变更)时都会变更 */
},
cacheGroups:{//设置缓存chunks
    priority: 0,//缓存组优先级
    //当需要优先匹配缓存组的规则时，priority需要设置为正数，当需要优先匹配默认设置时，缓存组需设置为负数，0为两者分割线
    default:{//设置缓存组默认配置，可通过default:false禁用默认缓存组，
    //然后就可以自定义缓存组，将初始化加载时被重复引用的模块进行拆分
        minChunks:2,//引用两次
        priority:-20,//缓存组优先级为-20
        reuseExistingChunk:true,//表示可以使用已经存在的块，即如果满足条件的块已经存在就是用己有的的，不再创建一个新的块
    }，
    [key]:{//自定义缓存组，可以根据需求，自由创建
        chunks:'initial',
        test: /vue/,//正则规则验证，如符合就提取chunk放入当前缓存组，值可以是function、boolean、string、RegExp，默认为空
        enforce: true//优先处理
    }
}
```

## 三、疑难配置项
** maxInitialRequests** :
这个配置项我一直很不理解，网上的很多资源都是千篇一律，有用的基本没有，后来我发现了这篇文章  [关于webpack模块拆分规则](https://www.cnblogs.com/wmhuang/p/8967639.html)  让我有对这个配置项有了点浅显的了解

现在说说我的理解，举个小例子：
这是一个多页应用，index.js和shop.js是入口文件
我的目录结构：
![目录结构](/images/webpack配置代码分割/1.jpeg)；

这是webpack分割代码
``` javascript
runtimeChunk: {
    name: 'manifest'
},
splitChunks: {
    cacheGroups: {
        //项目公共组件
        common: {
            chunks: 'initial', 
            name: 'common',
            minChunks: 2, 
            maxInitialRequests: 3, //最大初始化加载次数，一个入口文件可以并行加载的最大文件数量，默认1
            minSize: 0 //表示在分离前的最小模块大小，默认为0，最小为30000
        },
        //第三方组件
        vendor: {
            test: /node_modules/,
            chunks: "initial",
            name: "vendor",
            priority: 10,
            enforce: true
        }
    }
}
```
现在先别急着看分包结果，先来分析一下：
其中除了React、ReactDOM是第三方包以外剩下的都是自定义组件

**index.js(入口文件)**：React、ReactDom、index.jsx(index目录下)
**shop.js(入口文件)**：React、ReactDom、index.jsx(shop目录下)
**index.jsx(shop目录下)**：React、Header.jsx、A.jsx、B.jsx、shop.css
**index.jsx(index目录下)**：React、A.jsx、index.css
**A.jsx**：React、Common.css
**B.jsx**：React、b.css、common.css
那么画出的依赖图就是这个样子：
![目录结构](/images/webpack配置代码分割/2.png)

这里来自 `node_modules` 的包肯定是被打包到 `vender` 里了，不用管这个，主要是看 `common` 这个包的信息

暂且忽略掉**maxInitialRequests**这个配置（当然代码里是不能忽略的因为有默认值，想象这一项不起作用就行了），如果按照使用次数超过2次，大小超过0的包，打包到一起，那么结果是这样的：
> Common（chunks）：A.jsx、Common.css
> vender（chunks）：第三方
> shop.js（chunks）：Header.jsx、B.jsx、b.css、shop.css
> index.js（chunks）：index.css

那么此时的maxInitialRequests（初始加载次数）是：
`shop页入口`：shop.js、vender、Common     **一共三次**

`index页入口`：index.js、vender(chunks)、Common   **一共三次**

-----------------------

如果此时我限制了 **maxInitialRequests：2** 为2的话，那么结果是这样的
> vender（chunks）：第三方
> shop.js（chunks）：Header.jsx、B.jsx、b.css、shop.css、A.jsx、Common.css
> index.js（chunks）：index.css、A.jsx、Common.css

`shop页入口`：shop.js、vender **一共两次**

`index页入口`：index.js、vender**一共两次**
看截图：
![maxInitialRequests：2的情况](/images/webpack配置代码分割/3.jpeg)

![maxInitialRequests：3或大于三的情况](/images/webpack配置代码分割/4.jpeg)


可以看到以上的分析是正确的

## 四、总结
不过话说会来，如果每次配置wepback的这一项都按这么分析，那得累死，可以这么分析，每次页面初始加载我想让它的并发请求不超过自己的预期就可以了