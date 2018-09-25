---
title: vue自定义父子组件组件v-model的通信
date: 2018-09-25 10:38:15
tags: [Vue]
---

### 一、前言
前些天写一个checkbox的vue组件的时候想用v-model来进行状态的判断，但是想到这了，却不知道v-model怎么在组件里面实现，这篇文章记录v-model的实现以及v-model的扩展。具体效果可以参考`iview`的 `Checkbox组件和CheckboxGroup组件`

### 二、分析 (功能描述)
我想做两个组件 第一个叫 ** ButtonSelect ** 、** ButtonSelectGroup **
1. `ButtonSelect` :  其实这个组件就是一个checkbox复选框，只不过样式上进行的修改，我想在这上面绑定v-model，通过v-model来告诉父级组件当前的复选框状态(true/false)，`ButtonSelect` 的效果同iview的 `Checkbox组件`
![CheckBox组件效果](/images/vue自定义父子组件组件v-model的通信/checkbox.gif)

2. `ButtonSelectGroup` : 当有多个 **ButtonSelect** 的时候，可以用 `ButtonSelectGroup`包裹`ButtonSelect`，然后通过 `ButtonSelectGroup` 上绑定 v-model ，来获得所有的选中项，格式是数组，效果同iview的 `CheckboxGroup`
![CheckboxGroup组件效果](/images/vue自定义父子组件组件v-model的通信/CheckboxGroup.gif)

### 三、关于v-model
v-model 双向绑定是一个很好用的功能，对于不同的组件v-model返回值也不同,但是这里只说说 `checkbox复选框的v-model`
1. 单个复选框，绑定到布尔值
2. 多个复选框，绑定到同一个数组
[官网描述](https://cn.vuejs.org/v2/guide/forms.html)

> **这里一定要仔细观察，当多个复选框的时候它的v-model的值是一样的，只有这样才能返回数组，而数组每一项的值就是checkbox的value值**

#### 1.自定义v-model（v-model传递）
父组件===>子组件：默认名为value的prop
子组件===>父组件：默认名为input的事件
通过 **value** 和 **input** 来完成v-model的赋值和传递

但是有时候我们不想用value的input进行传递，name可以用**model**选项自定义：
``` javascript
model: {
    prop: 'checked',
    event: 'change'
},
```
这样就变成了 prop为change 和 checked事件来代替value和input了

### 四、ButtonSelect的实现
最顶级的组件ButtonSelectGroup、ButtonSelect的父组件，就叫他HelloWord组件
``` vue 
<template>
  <div>
    <ButtonSelectGroup v-model="select">
        <ButtonSelect label='第一个' ></ButtonSelect>
        <ButtonSelect label='第二个'></ButtonSelect>
        <ButtonSelect label='第三个'></ButtonSelect>
    </ButtonSelectGroup>
    {{select}}
  </div>
</template>

<script>
import  ButtonSelect from "./ButtonSelect.vue";
import ButtonSelectGroup from "./ButtonSelectGroup.vue";
export default {
  name: 'HelloWorld',
  components:{
    'ButtonSelect':ButtonSelect,
    'ButtonSelectGroup':ButtonSelectGroup
  },
  data(){
    return{
      select:[]
    }
  },

}
</script>
```
``` vue 
<style>
@keyframes ani {
    0% {
    background-color: #52c41a;
    -webkit-box-shadow: 0 0 5px #52c41a;
    box-shadow: 0 0 5px #52c41a;
    }
    50% {
        background-color: #73d13d;
        -webkit-box-shadow: 0 0 10px #73d13d;
        box-shadow: 0 0 10px #73d13d;
    }
    100% {
        background-color: #95de64;
        -webkit-box-shadow: 0 0 5px #95de64;
        box-shadow: 0 0 5px #95de64;
    }
}
.selectBut{
    display: flex;
    flex-wrap: nowrap;
    margin: 5px;
    border-radius: 5px ;
    line-height: 30px;
    height: 30px;
    position: relative;
}
.selectBox{
    display: inline-block;
    overflow: hidden;
}
.con{
    padding:0 5px;
    box-shadow: 0 0 8px 0px  #88d6f7;
    background: -webkit-linear-gradient(left,#5ec8ff,#4a9fd7);
    background: -o-linear-gradient(left,#63e77d,#3e8aa6);
    background: -moz-linear-gradient(left,#63e77d,#3e8aa6);
    background: linear-gradient(left,#63e77d,#3e8aa6);
}
.check{
    position: absolute;
    top: 0;
    bottom: 0;
    left: 0;
    right: 0;
    z-index: 1;
    cursor: pointer;
    opacity: 0;
    width: 100%;
    height: 100%;
    box-sizing: border-box;
    padding: 0;
    margin: 0;
}
input.check:checked+.light{
    /* background: #19be6b; */
    animation: ani 3s infinite alternate;
}
.num{
    padding: 0 5px;
    border-radius:0 5px 5px 0;
    color: #fff;
    font-weight: bold;
    text-shadow: 0 -1px 1px #40a9ff;
    /* box-shadow: 0 0 8px 0px  #88d6f7; */
    background: -webkit-linear-gradient(left,#4a9fd7,#2d60a2);
    background: -moz-linear-gradient(left,#3e8aa6,#2c5d9d);
    background: -o-linear-gradient(left,#3e8aa6,#2c5d9d);
    background: linear-gradient(left,#3e8aa6,#2c5d9d);
}
.lightbox{
    position: relative;
    width: 20px;
    height: 100%;
}
.light{
    transition:all .5;
    background: #ccc;
    height: 100%;
    cursor: pointer;
    border-radius: 5px 0 0 5px;
}
</style>
<template>
    <div class="selectBox">
            <div class="selectBut">
                <div class="lightbox">
                    <input 
                        type="checkbox" 
                        class="check" 
                        :checked="currentValue"
                        @change="change"/>
                    <div class="light"></div>
                </div>
                <div class="con">
                    <slot name="con">ssss</slot> 
                </div>
                <div class="num">
                   <slot name="num">aaa</slot> 
                </div>
            </div>
    </div>
</template>
<script>
export default {
    name:'ButtonSelect',
    data(){
        return{
            //根据v-model初始化当前组件状态
            currentValue:this.value
        }
    },
    props:{
        // v-model获取
        value:{
            type:[Array,Boolean],
            default:false
        }
    },
    methods:{
        change(event){
            var value=event.target.checked;
            // 赋值v-model
            this.$emit('input',value)
        }
    }
}
</script>

```
关键点：
>template的 input 
>script:props的value和data里currentValue（获得初始状态）
>script: change里this.$emit('input')

单个的复选框组件基本功能完成

五、ButtonSelectGroup组件
实际应用中`ButtonSelect`会有很多个，我需要获取每一个的选中状态或者值（参考上面的CheckBoxGroup组件）
#### 1. 分析
1. 让多个checkbox返回数组的原理是什么？ 上面提到了，是给每一个checkbox相同的model，所以就要在父组件（ButtonSelectGroup）里给所有子组件（ButtonSelect）相同的v-model
2. 当所有的子组件其中有一个改变的时候，应该给父组件返回变化后的数组，父组件（上面提到了，是给每一个checkbox相同的model，所以就要在父组件（ButtonSelectGroup）里给所有子组件（ButtonSelect）设置相同的v-model）再将该数组通过input事件返回给
ButtonSelectGroup的调用者(ButtonSelectGroup的父组件)，完成v-model的传递
3. 当ButtonSelectGroup的父组件给定一个初始v-model值的时候，需要把每一个ButtonSelect进行状态的变换

#### 2. 实现
``` vue
<style>

</style>
<template>
    <div>
        <slot></slot>
    </div>
</template>
<script>
// 寻找子组件
function findComponentsDownward (context, componentName) {
    return context.$children.reduce((components, child) => {
        if (child.$options.name === componentName) components.push(child);
        const foundChilds = findComponentsDownward(child, componentName);
        return components.concat(foundChilds);
    }, []);
}
export default {
    name:'ButtonSelectGroup',
    props:{
        value:{
            type:Array,
            default(){
                return []
            }
        }
    },
    data(){
        return{
            childrens:[],
        }
    },
    mounted(){
        this.updateModel()
    },
    methods:{
        updateModel(){
            this.childrens = findComponentsDownward(this, 'ButtonSelect');
            if(this.childrens){
                const { value }=this;
                // 给子组件设置相同的model
                this.childrens.forEach(child=>{
                    child.model=value;
                    child.group = true;
                })
            }
        },
        change(data){
            // 通知上级组件的v-model
            this.$emit('input',data);
        }
    },
    watch: {
        value () {
            this.updateModel();
        }
    }
}
</script>
```
** 注意注意注意 **
>这里有一个非常非常非常重要的点，就是这个watch里面这个value监听触发updateModel方法，你可以尝试去掉这个watch是什么结果。我就在这里卡了好久。来解释一下原因：
>来一起想一下，子组件(ButtonSelect)的`model`是他爹(ButtonSelectGruop)给的，每次我点击了 `ButtonSelect`会通过`this.parent.change(this.model)` 来告诉父组件去更新使用的HelloWord里的 `select`，好了此时此刻我已经更新完了HelloWord里的 `v-model(select)`,但是注意子组件（ButtonSelect）的v-model是通过 父组件 （ButtonSelectGroup）赋值过来了，并不能直接更改，所以ButtonSelectGruop如果不加 watch的value监听的话，子组件的`v-model`永远是[]，所以可以在ButtonSelect change的时候打印一下，每次数组一定是一个值，所以要用过watch监听，每一次HelloWord的v-model变动，都要重新给ButtonSelect赋值一次，这样才能达到预期目的

不加watch的样子：
![不加watch](/images/vue自定义父子组件组件v-model的通信/notok.gif)

加watch的样子：
![加watch](/images/vue自定义父子组件组件v-model的通信/ok.gif)

### 五、总结
上面什么input事件、默认value什么的都还好理解，最最最重要的点就是上面说的watch来监听value，听我这么说可能云里雾里，需要自己动手实践一下，才能明白其中的意思