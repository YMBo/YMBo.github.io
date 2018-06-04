---
title: react中的context
date: 2017-08-21 11:29:02
tags: react
---
## 一、为什么用context？
现在我们有一颗组件树：
![组件树](/images/react中的context/1.png)
假如这个组件树代表的应用是用户可以自定义主题的，每个子组件都会根据主题的不同来调整自己的样式，所以‘主题色’这个东西就应该是组件之间共享的一个状态，所以应该放到Index组件中。
但是在此之前能想到的办法只能是`this.props.主题色`
![主题色的传递](/images/react中的context/2.png)
这种形式，这种形式实在是麻烦，如果组件嵌套过深，就不得不一层层传递到最底层，所以就出现了简单的办法——通过context传递
context这种方法是全局都能共享的状态，我们需要的时候就去取这个状态，不需要手动传递
![context共享状态](/images/react中的context/3.png)

## 二、实践，看看代码怎么写
### 先创建一个整体结构
``` javascript
class Index extends Component {
  render () {
    return (
      <div>
        <Header />
        <Main />
      </div>
    )
  }
}

class Header extends Component {
  render () {
    return (
    <div>
      <h2>This is header</h2>
      <Title />
    </div>
    )
  }
}

class Main extends Component {
  render () {
    return (
    <div>
      <h2>This is main</h2>
      <Content />
    </div>
    )
  }
}

class Title extends Component {
  render () {
    return (
      <h1>React.js 小书标题</h1>
    )
  }
}

class Content extends Component {
  render () {
    return (
    <div>
      <h2>React.js 小书内容</h2>
    </div>
    )
  }
}

ReactDOM.render(
  <Index />,
  document.getElementById('root')
)
```
### 修改Index组件
修改Index组件，让他往自己的context中放一个themeColor：
``` javascript
class Index extends Component {
  static childContextTypes = {
    themeColor: PropTypes.string
  }

  constructor () {
    super()
    this.state = { themeColor: 'red' }
  }

  getChildContext () {
    return { themeColor: this.state.themeColor }
  }

  render () {
    return (
      <div>
        <Header />
        <Main />
      </div>
    )
  }
}
```
### 代码说明
1.`state`初始化一个`themeColor`状态，方便以后的修改
2.`getChildContext`方法设置`context`，返回一个的对象就是`context`，所有子组件动能访问到，且用`this.state.themeColor`来设置`context`里面的`themeColor`
3.注意，还需要加上一段参数的验证 `childContextTypes`,需要使用`prop-types`这个包，验证的是`getChildContext`返回的对象
以上这些都是`必须的`,这里要提及一下为啥要验证`context`，这么多步骤多麻烦，据说`context`是一个危险的属性(`context 里面的数据能被随意接触就能被随意修改，每个组件都能够改 context 里面的内容会导致程序的运行不可预料`),所以按照react.js团队的想法就是，把危险的事情搞复杂一些，提高使用门栏人们就不去用了

### 子组件的设置
``` javascript
class Title extends Component {
  static contextTypes = {
    themeColor: PropTypes.string
  }

  render () {
    return (
      <h1 style={{ color: this.context.themeColor }}>React.js 小书标题</h1>
    )
  }
}
``` 
1.利用`contextTypes`进行验证，必写的，不写就无法获取`context`的状态，
2.使用的话是通过`this.context.themeColor`来获取值的

## 修改context
在Index里面，我们已经初始化一个`state`状态了，叫：`this.state.themeColor`,所以使用setState就可以了

## 四、我的例子:
![点击变色按钮前](/images/react中的context/4.png)
![点击变色按钮后](/images/react中的context/5.png)