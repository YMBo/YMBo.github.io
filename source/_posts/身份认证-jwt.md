---
title: 身份认证-jwt
date: 2018-09-27 20:39:51
tags:
---
### 一、jwt简介
关于jwt的概念网上有很多非常好的博客讲解，在这里就不解释了，看下面内容之前最好对jwt有个大致的了解。这里讲的是前后台怎么利用jwt进行身份验证。（react+axios+node）

### 二、基本逻辑
#### 1. 后台逻辑
前台 `/login` 路由进入后，在账号密码正确的前提下创建一个 token(附带上过期时间等)，返回给前台，后续前台的每次请求，都需要经过后台的一个中间件来判断 token是否过期或者有误，如果有误，返回错误信息

#### 2. 前台逻辑
前台收到返回信息后，将token存储在localStorage或者cookie里，然后后续的每次的请求带上这个token

#### 3.附加
其实除了每次的请求要附带token检查有效之外，还需要在路由跳转的时候进行token检查，下面会说到三种方法


### 三、 后台代码
所使用的包：
* express-jwt                  //用来验证token
* jsonwebtoken              //用来生成token给客户端

当/login进入的时候，生成token返回
``` javascript
//jwt.sign(payload, secretOrPrivateKey, [options, callback])
// 登录
exports.login = function(req, res) {
    const { user, pwd } = req.body;
    UserModel.findOne({ user: user, pwd: utility.md5(md5Pwd(pwd)) }, { pwd: 0 }, (err, doc) => {
        if (err) {
            console.log(err)
            return res.json({
                code: 1,
                msg: '未知错误'
            })
        }
        if (!doc) {
            return res.json({
                code: 1,
                msg: '用户不存在或密码错误'
            })
        }
        // 生成token
        // secretOrPrivateKey：用"YMBo's club"字符串加密
        let token = jwt.sign({
            name: 'job',
        }, "YMBo's club", {
            expiresIn: 1 * 60       //过期时间1分钟
        })
        return res.json({ code: 0, data: doc, token })
    })
}
```
![/login后返回信息](/images/身份认证-jwt/1.jpeg)

每次有路由请求的时候对token判断是否过期
添加一个中间件：
``` javascript
const expressJWT = require('express-jwt')
const secret = "ymb's club"

app.set(secret, secret)

// jwt验证
app.use(expressJWT({
    secret: secret
}, {
    expiresIn: 60
}).unless({ path: ['/user/login'] }))

// jwt验证,如果有错误（token不对，过期等错误）
app.use(function(err, req, res, next) {
    if (err.name === "UnauthorizedError") {
        res.status(401).send({
            code: 1,
            msg: '请登录'
        });
    }
});

app.use(bodyParser.json())
app.use('/user', userRouter)
```

> 这里有一个关键点，当使用`express-jwt`进行token验证的时候，前台发过来的token必须是带 **Bearer**前缀的，比如token是
>aaa.bbb.ccc，那么应该格式化为 **Bearer aaa.bbb.ccc** 这种形式


### 四、 前台代码
``` javascript
// setLocalStorage是自定义的方法，存储token
setLocalStorage('Authorization', `Bearer ${ res.data.token }`)
// 后台返回正确信息后 将token 存储在localStorage里
```
在项目的总入口或者次级入口加上这个，意思是后续的每一次axios请求都将携带`Authorization`信息
``` javascript
axios.defaults.headers.common['Authorization'] = getLocalStorage('Authorization');
```

### 五、验证token

#### 1. 第一种方法
上面说了后台添加中间件 `express-jwt` 来验证，网上的好多教程说的是通过一个单独的路由 /info，每次请求的时候先请求这个路由是否过期等，但是我觉得这种方法太浪费了，何必每次请求都要发送这个 /info这个验证请求呢

#### 2. 第二种方法
``` javascript
jwt.decode（token [，options]） 
//解密，注意了之前存储到localStorage里的token是带 Bearer 要把它去掉进行解密。
//返回解码没有验证签名是否有效的payload。警告：这不会验证签名是否有效。它只是返回后端设置的payload
// 特点：不用secretOrPrivateKey进行解密
```
前台的顶层路由（就是进别的路由都要经过的路由）进行判断，怎么判断呢？
用 **jsonwebtoken** 这个包(上面用到过) 进行解密，然后取出设定的 `expiresIn` 过期时间，然后取到本地时间戳，与这个进行判断，看是否过期。

#### 3. 第三种方法
``` javascript
jwt.verify（token，secretOrPublicKey，[options，callback]）
//验证token合法性

// 去掉了Bearer 
let token = getLocalStorage('Authorization').replace(/Bearer\s/, '');
jwt.verify(token, "ymb's club", (err, decoded) => {
    if (err) {
        alert('过期了快去登录')
        this.props.history.push('/login')
    }
})
//如果token没用了 直接返回err
//用到了secretOrPrivateKey：用"YMBo's club"字符串解密，注意这个字段一定要和后台那个加密字段一致，否则它一致err
```

### 六、效果查看
过期时间我设置的1分钟，为了方便调试

**/login 返回token成功**
![返回token成功](/images/身份认证-jwt/show.gif)

**后台express-jwt验证token成功**
![后台express-jwt验证token成功](/images/身份认证-jwt/show2.gif)

**路由跳转前验证token成功**
![路由跳转前验证token成功](/images/身份认证-jwt/show3.gif)