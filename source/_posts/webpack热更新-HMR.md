---
title: webpack热更新(HMR)
date: 2018-01-02 15:40:44
tags: [webpack,react,HMR]
---
### 一、HMR介绍
在我们开发react应用的时候，在配置了webpack-dev-server的前提下每一次的组件内容修改都需要手动的刷新浏览器，为了解决这个问题，所以有了热更新这个概念，网上的文章弄得我一头雾水，在我配置成功后，自己来记录一下热更新的配置

时隔三个月今天发现了配置更简单的热替换的方法，[点我查看](https://ymbo.github.io/2018/04/12/webpack4-0%E4%B8%AD%E7%9A%84mode%E5%92%8C-babelrc%E7%9A%84env-development%E9%85%8D%E7%BD%AE%E9%A1%B9%E7%9B%AE/)

### 二、配置
#### 1.从零开始--项目初始化
首先创建一个基本的react的项目，然后命令行运行 `npm init` 生成 package.json 文件。

新建 `build`,`client` 文件夹，分别用来存放 webpack 配置（webpack.config.js文件）和 react 组件。
当前目录结构：
![目录结构](/images/webpack热更新-HMR/4.png)

#### 2.基本的webpack配置
为了自动产出html文件，安装 `html-webpack-plugin` 

为了解析 `react` 组件和对`es6`的解析，安装 `babel-loader babel-core babel-preset-es2015  babel-preset-es2015-loose babel-preset-react` 模块，并在根目录新建文件 `.babelrc` 文件 `babel` 的配置文件
![.babelrc](/images/webpack热更新-HMR/1.png)

此时 `package.json` 文件的包
![package.json](/images/webpack热更新-HMR/2.png)

添加 `webpack-devserver` 配置，此时 `webpack-config.js` 配置如下

``` javascript 
const path=require('path');
const HtmlWebpackPlugin=require('html-webpack-plugin');

module.exports={
	entry: {
		app:path.join(__dirname,'../client/app.js')
	},
	output:{
		filename:'[name].[hash].js',
		path:path.join(__dirname,'../dist'),
		publicPath:'/public/'
	},
	module:{
		rules:[
			{
				test:/\.(jsx|js)$/,
				loader:'babel-loader',
				exclude:path.join(__dirname,'../node_modules')
			}
		]
	},
	plugins:[
		new HtmlWebpackPlugin()
	],
	devServer:{
		/*代表本机 也可以通过ip或者localhost这种方式，但是用后面的两种方式的话，局域网内是访问不到本机的，所以用了0.0.0.0*/
		host:'0.0.0.0',
		port:'8888',
		contentBase:path.join(__dirname,'../dist'),
		/*显示错误信息*/
		overlay:{
			errors:true
		},
		/*因为上面的publicPath:'/public/'，所以访问的所有路径都要加上public*/
		publicPath:'/public/',
		historyApiFallback:{
			/*如果页面404则返回下面配置的页面*/
			index:'/public/index.html'
		}
	}
}
```
此时启动webpack-dev-server后，运行成功
![package.json](/images/webpack热更新-HMR/3.png)

----------------------------

`友情提示：`：
1. 如果在执行命令的过程中报错 'cannot find ...' 这类的错误，首先检查是否少装了包，如果没有少装，则删除 node_module文件夹，重新安装下。
2. 如果运行 webpack-dev-server 启动服务器的时候，网页的 js 文件显示404，首先检查是否本地已经有了一个编译好的 dist 文件夹，因为webpack-dev-server会优先读取本地文件，配置的时候我们添加了 /pubilc/ 所以是读取不道德，这时，删除本地 dist文件夹即可。（这个相当相当的坑）
3. 在入口文件中，我这样写 `document.body` 是不可取的，正确的做法是 获取Id的形式。

----------------------------

`知识点！( 敲黑板 )`：path.join 和 path.resolve 区别
>path.join：拼接地址（会正确使用当前系统的路径分隔符，Unix系统是/，Windows系统是\）
>比如：path.join（‘m’,’/b’） 或者 path.join（‘m’,’b’）  返回m/b这个路径
>path.resolve：将参数转换为绝对路径
>比如 path.resolve(‘m’) ;如果当前命令窗口是在c盘打开的，那么返回C:\m(总是返回一个绝对路径)
>对比：两种方法都可以获得当前目录的绝对路径（通过__dirname），因为path.join可以适应unix和windows，所以join可能好一些

### 三、重头戏-配置HMR
这里说是简单，但我觉得对于初次使用还是比较繁琐，整理一下，分为下面几个步骤(与顺序无关)
1. 安装 `react-hot-loader`
2. 配置 `babelrc` 文件
3. 配置 `入口文件app.js` 
4. 配置 `webpack.config.js`

#### 2.配置 .babelrc 文件
![.babelrc](/images/webpack热更新-HMR/5.png)

#### 3. 配置 `入口文件app.js` 

![入口文件app.js](/images/webpack热更新-HMR/6.png)

#### 4. 配置 `webpack.config.js`

改动部分标记为 ` //add`

``` javascript 
const path=require('path');
const HtmlWebpackPlugin=require('html-webpack-plugin');
const webpack=require('webpack');		//add

module.exports={
	entry: {
		app:[						
			'react-hot-loader/patch',			//add
			path.join(__dirname,'../client/app.js')
		]
	},
	output:{
		filename:'[name].[hash].js',
		path:path.join(__dirname,'../dist'),
		publicPath:'/public/'
	},
	module:{
		rules:[
			{
				test:/\.(jsx|js)$/,
				loader:'babel-loader',
				exclude:path.join(__dirname,'../node_modules')
			}
		]
	},
	plugins:[
		new HtmlWebpackPlugin(),
		new webpack.HotModuleReplacementPlugin() 	//add
	],
	devServer:{
		/*代表本机 也可以通过ip或者localhost这种方式，但是用后面的两种方式的话，局域网内是访问不到本机的，所以用了0.0.0.0*/
		host:'0.0.0.0',
		port:'8888',
		contentBase:path.join(__dirname,'../dist'),
		/*热加载*/	//add
		hot:true,	//add
		/*显示错误信息*/
		overlay:{
			errors:true
		},
		/*因为上面的publicPath:'/public/'，所以访问的所有路径都要加上public*/
		publicPath:'/public/',
		historyApiFallback:{
			/*如果页面404则返回下面配置的页面*/
			index:'/public/index.html'
		}
	}
}
```
[react-hot-loader参考](https://www.npmjs.com/package/react-hot-loader)

好了，该做的做完了，打开浏览器测试，发现更改App.jsx文件后浏览器竟然特么没有变化,这就很气人，我也按照你官网上写的做了，现在出不来？好吧，看下面的解决办法

### 四、解决上面的问题
app.js这个入口文件中，更改为：

![更改为](/images/webpack热更新-HMR/7.png)

如果有了更改，那么 重新用 require 的方式获取一下这个组件，然后进行渲染

![成功啦！](/images/webpack热更新-HMR/8.gif)

是不是有了疑问，为啥用 `require` 的方式引入呢？因为这种形式的文件引入import的方式引入不了

### 五、总结
本来一个挺好的功能，分别写在了 webpack官网和 react-hot-loader 官网，这还不算啥，最后还运行不了，多坑，好了就记录到这里了,如果有问题或者补充欢迎回复