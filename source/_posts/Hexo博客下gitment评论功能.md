---
title: gitment评论插件的配置
tag: 'hexo'
---
今天我在第N次搭建博客的时候想加一个评论功能，我用的主题是indigo，在根目录_config.yml文件里经过测试发现最简洁好用的就是gitment，所以这里记录我gitment配置过程：
## 第一步
先注册一个 [oAuth Application](https://github.com/settings/applications/new)    
注册成功会得到一个`client id`和 `client secret`两个参数，它们将被用于后面的用户登录
![oAuth Application](/images/gitment评论插件的配置/1.png)
![_config.yml](/images/gitment评论插件的配置/2.png)
其中githubID需要https://api.github.com/users/github账户名  访问这个链接来获取

## 第二步
如果你想要此功能的页面存在这一部分：
![gitment页面配置](/images/gitment评论插件的配置/3.png)
这是一个ejs页面，里面是模板，通过使用主题的_config.yml文件配置
那么你可以跳过这一步，否则需要在使用评论功能的页面添加这写代码：
``` javascript
<link rel="stylesheet" href="https://imsun.github.io/gitment/style/default.css">
<script src="https://imsun.github.io/gitment/dist/gitment.browser.js"></script>
<script>
var gitment = new Gitment({
	id: '页面 ID', // 可选。默认为 location.href
	owner: '你的 GitHub ID', // 可以是你的GitHub用户名，也可以是github id
	repo: '存储评论的 github repo',
	oauth: {
		client_id: '你的 client id',
		client_secret: '你的 client secret',
	},
})
gitment.render('container')
</script>
```
## 第三步
到这里已经默认你添加了上面的代码，现在注意了，我用的主题本身就带上面的代码的，只不过需要在 主题的 _comfig.yml文件中配置对应项，我在配置文件中配置好后，启动本地服务器，发现回复模块报错，控制台中发现他会请求一个链接，链接上带有刚刚配置好的参数，这里参数全部为空，我以为我配置文件有问题，折腾了好久，仍然不行，后来发现我要添加这个功能的页面里的模板并没有获取到我在_config.yml配置，最后通过修改要添加回复功能页面的模板文件才成功：
![修改为](/images/gitment评论插件的配置/4.png)
直接赋值，而不是使用模板，不知道为啥用模板获取不到值，现在已经配置成功
如果配置失败，那么一定是上面的参数写错了，或者像我这种情况模板页面获取不到值，只需要直接赋值即可