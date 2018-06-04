---
title: webpack4.0中的mode和.babelrc的env.development配置项目
date: 2018-04-12 14:29:57
tags: webpack
---

### 一、前言
之前写过怎么在webpack中使用react-hot-loader做热更新功能，不得不说是真的非常麻烦，今天发现了更好用的办法来记录一下
#### 1.这篇文章讲的什么？
1.更简单的配置 **热更新**的方法
2.webpack4.0的**mode配置项**有啥用？
3.babelrc中的**env.development**是干啥的？

### 二、热更新
`注意，本部分着重讲解的是热更新，至于上面提到的2和3放在下个部分`
webpack.config.js的代码：
``` javascript
const webpack=require('webpack')
const opn=require('opn')
const merge=require('webpack-merge')
const path=require('path')
const baseWebpackConfig=require('./webpack.base.config')
const webpackFile=require('./webpack.file.config')
const htmlWebpackPlugin=require('html-webpack-plugin')

let config=merge(baseWebpackConfig,{
	mode:'development',
	output:{
		path:path.join(webpackFile.devDirectory),
		filename:'js[name].[hash].js',
		publicPath:''
	},
	plugins:[
		/*热更新*/
		new webpack.HotModuleReplacementPlugin(),
		new htmlWebpackPlugin({
			template:path.join(__dirname,'../template/index.html')
		})
	],
	module:{
		rules:[
			{
				test:/\.(js|jsx)$/,
				use:'babel-loader',
				include:[
					path.join(__dirname,'../client')
				],
				exclude:path.join(__dirname,'../node_modules')
			}
		]
	},
	devServer:{
		host:'0.0.0.0',
		port:'8888',
		hot:true,
		overlay:{
			errors:true
		},
		contentBase:path.join(__dirname,webpackFile.devDirectory),
		historyApiFallback: true,
		proxy:[
			{
				context:['/api'],
				target:'localhost:8888',
				secure: false
			}
		],
		/*打开浏览器*/
		after(){
			opn('http://localhost:'+this.port)
		}
	}
})
module.exports=config;

```
*.babelrc*


``` javascript
{
	"presets":['react','env'],
	"env":{
		/*开发环境下执行*/
		"development":{
			"presets":["react-hmre"]
		}
	}	
}
```

package.json文件的启动命令
``` javascript
"dev": "webpack-dev-server --config build/webpack.dev.config.js"
```

现在配置热替换仅需三个步骤
1. webpack中`devServer.hot=true`
2. webpack中plugins添加`new webpack.HotModuleReplacementPlugin()`
3. '.babelrc'文件中使用 `react-hmre` （npm i babel-presets-react-hmre）预设



配置完成是不是很简单？看看执行结果

![hmr](/images/webpack4-0中的mode和-babelrc的env-development配置项目/1.gif)

### 三、webpack4.0的mode配置项有啥用？
mode配置项会告诉webpack使用相应的内置优化

webpack运行时还会根据mode设置一个全局变量process.env.NODE_ENV,这里的process.env.NODE_ENV不是node中的环境变量,而是webpack.DefinePlugin中定义的全局变量,允许你根据不同的环境执行不同的代码.

参数：

选项 | 描述
- | :-: |
development | Provides process.env.NODE_ENV with value development. Enables NamedChunksPlugin and NamedModulesPlugin.
production | Provides process.env.NODE_ENV with value production. Enables FlagDependencyUsagePlugin, FlagIncludedChunksPlugin, ModuleConcatenationPlugin, NoEmitOnErrorsPlugin, OccurrenceOrderPlugin, SideEffectsFlagPlugin and UglifyJsPlugin.

从网上找到了更详细的优化说明
列出了针对这两种情况做的对应优化
**development**
``` javascript
//调试
devtool:eval
//缓存模块, 避免在未更改时重建它们。
cache:true
//缓存已解决的依赖项, 避免重新解析它们。
module.unsafeCache:true
//在 bundle 中引入「所包含模块信息」的相关注释
output.pathinfo:true
//在可能的情况下确定每个模块的导出,被用于其他优化或代码生成。
optimization.providedExports:true
//找到chunk中共享的模块,取出来生成单独的chunk
optimization.splitChunks:true
//为 webpack 运行时代码创建单独的chunk
optimization.runtimeChunk:true
//编译错误时不写入到输出
optimization.noEmitOnErrors:true
//给模块有意义的名称代替ids
optimization.namedModules:true
//给模chunk有意义的名称代替ids
optimization.namedChunks:true
```
**production**
``` javascript
//性能相关配置
performance:{hints:"error"....}
//某些chunk的子chunk已一种方式被确定和标记,这些子chunks在加载更大的块时不必加载
optimization.flagIncludedChunks:true
//给经常使用的ids更短的值
optimization.occurrenceOrder:true
//确定每个模块下被使用的导出
optimization.usedExports:true
//识别package.json or rules sideEffects 标志
optimization.sideEffects:true
//尝试查找模块图中可以安全连接到单个模块中的段。- -
optimization.concatenateModules:true
//使用uglify-js压缩代码
optimization.minimize:true

```

例如:

``` javascript
if(process.env.NODE_ENV === 'development'){
    //开发环境 do something
}else{
    //生产环境 do something
}
```
最终将编译成

``` javascript
if(true){
   //开发环境 do something
}else{
   //生产环境 do something
}
```
生产环境下,uglify打包代码时会自动删除不可达代码,也就是说生产环境压缩后最终的代码为:
``` javascript
//生产环境 do something
```

### 四、babelrc中的env.development是干啥的？
env选项可以针对特定环境进行设置。此env值会从**process.env.BABEL_ENV**中获取；如果该值不存在，会使用**process.env.NODE_ENV**；二者都不存在，使用默认值”development”。

启动命令设置`process.env.BABEL_ENV`
``` javascript
"dev": "cross-env BABEL_ENV=development webpack-dev-server --config build/webpack.dev.config.js"
```
``` javascript
{
	"presets":['react','env'],
	"env":{
		/*开发环境下执行*/
		"development":{
			"presets":["react-hmre"]
		}
	}	
}
```
这样就会执行"development"下配置的内容