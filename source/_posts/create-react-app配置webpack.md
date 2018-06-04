---
title: 'create-react-app配置webpack'
date: 2017-09-06 16:42:15
tags: [react,webpack,create-react-app,px2rem,less]
---
在学习react过程中，每次都要配置webpack，非常的麻烦，我是要写react的，不是配置这东西哒！！
好在有了'create-react-app'，通过npm安装后，创建项目变得非常简单
但是它里面没有做`less`等你实际项目需要的`loader`或者`plugins`，所以这里记录的是怎么定制`create-react-app创建的项目的webpack`

## 一、先配置less
### 1.创建一个项目
我创建的叫`test`
![test测试项目](/images/create-react-app配置webpack/1.png)
创建完毕后，进入创建的项目，运行`npm start`启动此项目
![启动](/images/create-react-app配置webpack/2.png)

### 2.找到`webpack.config.dev.js`和`webpack.config.prod.js`
运行`npm run eject`
安装`less-loader`和`less`
进入`config`文件夹，这里会有两个文件`webpack.config.dev.js`和`webpack.config.prod.js`应该一个是开发环境一个生产环境的配置文件，两个文件都要修改
打开`webpack.config.dev.js`
找到`module rules`部分，也就是配置loader的部分，找到配置`css`文件的test（`/\.(css)$/`）修改为` test:/\.(css|less)$/`,添加一个loader：`{ loader: require.resolve('less-loader') },`放在最下面
``` javascript
{
	test: /\.css$/,
	use: [require.resolve('style-loader'), {
		loader: require.resolve('css-loader'),
		options: {
			importLoaders: 1,
		},
	},
	{
		loader: require.resolve('postcss-loader'),
		options: {
			// Necessary for external CSS imports to work
			// https://github.com/facebookincubator/create-react-app/issues/2677
			ident: 'postcss',
			plugins: () = >[require('postcss-flexbugs-fixes'), autoprefixer({
				browsers: ['>1%', 'last 4 versions', 'Firefox ESR', 'not ie < 9', // React doesn't support IE8 anyway
				],
				flexbox: 'no-2009',
			}), ],
		},
	},
	{
		loader: require.resolve('less-loader')
	},
	],
},
```

### 3.测试
现在我们做一下测试
可以随便在`css`文件中加入`less`语法
我在`App.css`中设置了整体的背景颜色：
```css
@base: pink;
.App-header {
  background-color: @base;
  height: 150px;
  padding: 20px;
  color: white;
}
```
因为我们修改了webpack的配置项，所以需要重新启动服务器：
![添加成功](/images/create-react-app配置webpack/3.png)
可以看到，配置已经成功

## 二、配置px2rem
### 1.解释
`px2rem`是做移动端页面开发的时候，自动将`px`单位转换为`rem`，非常的方便
这里要用支持`webpack`的对应`loader`，我用的是`postcss-px2rem`
配置这个功能相对来说有些复杂

### 2.安装
`npm install postcss-px2rem postcss-loader  --save`执行这个命令安装

### 3.配置
还是`webpack.config.dev.js`，引入
```javascript
const px2rem = require('postcss-px2rem')
```
添加在的`autoprefixer`下面添加，好像loaders里只能有一个`postcss`，多个的话不会报错，对应的功能会不管用
```javascript
{
  loader: require.resolve('postcss-loader'),
  options: {
    // Necessary for external CSS imports to work
    // https://github.com/facebookincubator/create-react-app/issues/2677
    ident: 'postcss',
    plugins: () => [
      require('postcss-flexbugs-fixes'),
      autoprefixer({
        browsers: [
          '>1%',
          'last 4 versions',
          'Firefox ESR',
          'not ie < 9', // React doesn't support IE8 anyway
        ],
        flexbox: 'no-2009',
      }),
      //这个位置
        px2rem({remUnit: 75})
    ],
  },
},
```
这里面的数值就是1rem对应的px

### 4.测试
现在启动服务器，因为原有的都是以px为单位，所以现在页面上应该全部转换为了rem
![添加成功](/images/create-react-app配置webpack/4.png)

不要忘了配置`webpack.config.prod.js`这个文件，
最后只需要在你的页面上对html的font-size做变换就可以了，这里略过

### 5.总结
webpack的文档和有些npm的文档写的走点心好么，搞得本来挺简单的东西查了好久！