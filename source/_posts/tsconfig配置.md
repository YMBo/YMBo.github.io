---
title: tsconfig配置
date: 2021-01-25 18:55:54
tags: 'typescript'
---

## 一、前言

不知不觉好久没有写博客和提交 github 了，20 年年初换了工作后，每天都非常忙，工作繁忙的同时自己也有一些收获和成长，也有一些新的收获想和想法想记录下来，但之前因为工作的原因这事一只拖着拖着最后就忘了，终于历经近一年终于产出了这篇文章 🤦‍♀

## 二、背景

因为对 tsconfig.json 的配置项不太了解，所以这篇文章以配置项解释+demo 的形式分析各个配置项的作用

## 三、tsconfig

现在以 demo 的 tsconfig.json 每个配置项为例，详细说明其作用  
新建 tsconfig.json 空文件

经过测试在` tsconfig include` 字段包含的范围内编写 .d.ts/.ts，都将被自动识别

### 1、target

> `target`指的是编译后目标版本的语法是啥

** demo **  
新建一个 a.ts 文件，用 tsc 命令编译后看结果

添加 `"target": "ESNext"` 配置（ESNext：以当前 js 版本的下个版本提案为标准编译，总是最新语法）

```javascript
const greeter = (person) => {
  return 'Hello, ' + person;
};
export interface c {
  age: string;
}
const user: string = 'Jane User';
async function age() {
  await greeter(user);
}
```

编译后：

```javascript
const greeter = (person) => {
  return 'Hello, ' + person;
};
const user = 'Jane User';
async function age() {
  await greeter(user);
}
export {};
```

可以看到语法结构基本没啥变化，因为编译目标是最新语法

`"target": "ts3"`
编译后：
可以看编译后是 es3 的语法

```javascript
var greeter = function (person) {
  return 'Hello, ' + person;
};
var user = 'Jane User';
function age() {
  return tslib_1.__awaiter(this, void 0, void 0, function () {
    return tslib_1.__generator(this, function (_a) {
      switch (_a.label) {
        case 0:
          return [4 /*yield*/, greeter(user)];
        case 1:
          _a.sent();
          return [2 /*return*/];
      }
    });
  });
}
```

`"target": "ts6"`
编译后：
可以看编译后是 es6 的语法，因为箭头函数被保留了,装饰器不是 es6 的语法所以被转换了

```javascript
const greeter = (person) => {
  return 'Hello, ' + person;
};
const user = 'Jane User';
function age() {
  return __awaiter(this, void 0, void 0, function* () {
    yield greeter(user);
  });
}
```

target 其他参数同理

### 2、outDir

> `outDir`指的是编译后的文件存放目录

在 tsconfig 中添加 ` "outDir": "lib"`,运行 tsc
可以发现出现了个 lib 目录，lib 目录里面是 a.js,目录结构

lib
└── a.js
├a.ts
├tsconfig.json

### 3、allowJs

> `allowJs`是否允许编译 js

在 tsconfig 中添加**allowJs：false**
添加`m.js`文件，内容为

```javascript
const greeter2 = (person) => {
  return 'Hello, ' + person;
};
const user2 = 'Jane User';

async function age2() {
  await greeter2(user2);
}
let name = 'm.js';
export { name, age2 };
```

运行下 tsc，发现如果 **allowJs：false**,那么 m.js（.js 文件）不会被编译，ture 的时候回被正常编译到 lib 文件夹下

### 4、strict

> 这个选项规定了是否启用所有严格类型检查选项。启用 --strict 相当于启用 --noImplicitAny, --noImplicitThis, --alwaysStrict， --strictNullChecks 和 --strictFunctionTypes 和--strictPropertyInitialization

相当于更严格的模式，以`noImplicitAny`为例，说明下，看 5

### 5、noImplicitAny

> 在表达式和声明上有隐含的 any 类型时报错

在 tsconfig 中添加**noImplicitAny:true**,开启 any 类型检查

运行 tsc，编译 a.ts，可以看到如下报错

```
a.ts:31:18 - error TS7006: Parameter 'person' implicitly has an 'any' type.

31 const greeter = (person) => {
                    ~~~~~~


Found 1 error.
```

上面 noImplicitThis 之类的同理

### 6、strictFunctionTypes

> 这个查了许多资料，这个说的最好：[strictFunctionTypes](https://juejin.cn/post/6896680181000634376)

> 严格函数类型检查。
> 这个说的比较抽象，举个例子
> 父=子 错误 ， 子=父 正确

在 tsconfig.json 添加`strictFunctionTypes:true`，开启检查
在 a.ts 添加一下代码：

```JavaScript
function getCurrentYear(callback: (date: string | number) => void) {
  callback(Math.random() > 0.5 ? '2020' : 2020);
}
getCurrentYear((date: string) => {
  console.log(date.charAt(0));
});
```

编译报错：

```javascript
a.ts:38:16 - error TS2345: Argument of type '(date: string) => void' is not assignable to parameter of type '(date: string | number) => void'.
  Types of parameters 'date' and 'date' are incompatible.
    Type 'string | number' is not assignable to type 'string'.
      Type 'number' is not assignable to type 'string'.

38 getCurrentYear((date: string) => {
                  ~~~~~~~~~~~~~~~~~~~

```

如果 `strictFunctionTypes:false`关闭检查则不会报错

### 7、jsx

> jsx 的编译规则

| 模式         | 输入      | 输出                       | 输出文件扩展名 |
| ------------ | --------- | -------------------------- | -------------- |
| preserve     | "<div />" | <div />                    | .jsx           |
| react        | <div />   | React.createElement("div") | .js            |
| react-native | <div />   | <div />                    | .js            |

在 tsconfig.json 添加`"jsx": "preserve`
创建文件`t.jsx`内容为:

```javascript
<div></div>
```

编译后：

```javascript
<div></div>
```

在 tsconfig.json 修改`"jsx": "react`
编译后的文件后缀变成了.js:

```javascript
React.createElement('div', null);
```

### 8、experimentalDecorators

> 允许是用装饰器

如果不开启这个选项，是不能用装饰器的，在 a.ts 中添加：

```javascrit
function decoraters(target) {
  target.age = 10;
}

@decoraters
class m {}
```

可以看到编译报错

### 9、importHelpers/moduleResolution

> **importHelpers:**从 tslib 导入辅助工具函数（比如 **extends， **rest 等），而不是生成代码到文件里
> **moduleResolution:**决定如何处理模块。或者是"Node"对于 Node.js/io.js，或者是"Classic"（默认）

这个选项依赖**tslib**这个库，安装这个库  
在 tsconfig.json 添加`"importHelpers": true`，这时候 a.ts 就报红了：
![红红红](/images/tscoinfig配置/1.png)

说是找不到 tiblib 这个库，明明已经安装了呀，在 node_modules 目录下，其实是 ts 不知道去哪找这个库，所以我们要告诉 ts 去哪找这个库，添加配置：  
`moduleResolution:node`

> 多说一嘴，moduleResolution 有两个配置项 node 和 classic（默认），这俩逻辑也很简单[node](https://www.tslang.cn/docs/handbook/module-resolution.html#node)、[classic](https://www.tslang.cn/docs/handbook/module-resolution.html#classic) ，所以这里是设置`node`

> tips:其实这个不配置 moduleResolution 也有另外的解决方案，就是在 tsconfig 加上这个配置，告诉如果用 classic 方案的话，去哪找这个包
>
> ```javascript
>    "baseUrl": ".",
>    "paths": {
>      "tslib": [
>        "./node_modules/tslib/tslib.d.ts"
>      ]
>    },
> ```

运行 tsc 编译，结果：  
可以看到装饰器的辅助方法已经通过 import 引入了

```javascript
import { __decorate } from 'tslib';
function decoraters(target) {
  target.age = 10;
}
let m = class m {};
m = __decorate([decoraters], m);
export default decoraters;
```

`修改tsconfig.json试试importHelpers：false`，编译结果：  
辅助方法打到了文件里了

```javascript
var __decorate =
  (this && this.__decorate) ||
  function (decorators, target, key, desc) {
    var c = arguments.length,
      r =
        c < 3
          ? target
          : desc === null
          ? (desc = Object.getOwnPropertyDescriptor(target, key))
          : desc,
      d;
    if (typeof Reflect === 'object' && typeof Reflect.decorate === 'function')
      r = Reflect.decorate(decorators, target, key, desc);
    else
      for (var i = decorators.length - 1; i >= 0; i--)
        if ((d = decorators[i]))
          r = (c < 3 ? d(r) : c > 3 ? d(target, key, r) : d(target, key)) || r;
    return c > 3 && r && Object.defineProperty(target, key, r), r;
  };
function decoraters(target) {
  target.age = 10;
}
let m = class m {};
m = __decorate([decoraters], m);
export default decoraters;
```

### 10、noEmitHelpers

> 这个跟上面那个参数做的事情差不多，就是 `不生成帮助函数` 来试试看

在 tsconfig.json 添加`"noEmitHelpers": true`，不生成辅助方法：
编辑结果：

```javascript
function decoraters(target) {
  target.age = 10;
}
let m = class m {};
m = __decorate([decoraters], m);
export default decoraters;
```

### 11、typeRoots

> 这个用于指定类型声明文件的查找路径。默认值为 node_modules/@types，即在 node_modules 下的@types 里面查找。需要注意的是这里仅仅是 d.ts 文件的查找路径。同样，这个也是相当于在引入**非相对模块**的时候拓宽了类型声明文件的查找范围，其实就是配置类型声明文件的查找目录 `注意这个只是类型的查找`

本地试了试，没发现有啥用，这个参数好像没啥实际应用场景啊？

### 12、declaration

> 生成相应的 .d.ts 文件。这个注意，只有代码里 export 的变量或方法 才会被生成.d.ts 文件，如果有个变量没有 export，那就在对应的.d.ts 文件里不会包含此变量的类型

在 tsconfig.json 添加`"declaration": true`，生成类型文件

a.ts 中添加一个没有 export 的方法 age：

```JavaScript
function decoraters(target) {
  target.age = 10;
}

@decoraters
class m {}

function age() {
  console.log('m');
}
age();

export default decoraters;
```

看到输出的`a.d.ts`结果：

```JavaScript
declare function decoraters(target: any): void;
export default decoraters;
```

不包含`age`方法

### 13、declarationDir

> 生成.d.ts 文件的存放目录

### 14、allowSyntheticDefaultImports

> 当导入的模块是以 commonJs 语法导出时，不能用 import M from './m' 这种语法，会报错，因为 ts 会检测导入模块是否有 default 字段，没有的话导出要用 import \* as M from './m，加这个配置 可以正常导入 import M from './'

新建 commonJs.js 文件,用 commonJs 语法导出：

```javascript
let CommonJS = 1;
function getCom() {
  return CommonJS;
}

module.exports = function (x) {
  console.log(x);
};
```

修改 a.ts:

```javascript
import getCom from './commonJS';
console.log(getCom);
function decoraters(target) {
  target.age = 10;
}
```

编译：

```javascript
a.ts:14:8 - error TS1259: Module '"/Users/mbyang/Desktop/test/commonJS"' can only be default-imported using the 'allowSyntheticDefaultImports' flag

14 import getCom from './commonJS';
          ~~~~~~

  commonJS.js:6:1
    6 module.exports = function (x) {
      ~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
    7   console.log(x);
      ~~~~~~~~~~~~~~~~~
    8 };
      ~
    This module is declared with using 'export =', and can only be used with a default import when using the 'allowSyntheticDefaultImports' flag.


Found 1 error.
```

在 tsconfig.json 添加`"allowSyntheticDefaultImports": true` 即可解决，或者修改为

```javascript
import * as getCom from './commonJS';
```

### 15、resolveJsonModule

> 包括以“.json”扩展名导入的模块 ,一版这个要配合 include 选项，如果 include 选项没有配置.json 文件，那配置这个也没啥用

### 16、baseUrl/paths

> 设置 baseUrl 来告诉编译器到哪里去查找模块。 所有非相对模块导入都会被当做相对于 baseUrl，baseUrl 默认值为'.'当前路径
> 注意是`非相对模块`

在 tsconfig.json 添加瞎写的`"baseUrl": "../../../"`，添加`path`指定 a.ts 模块的路径:

```javascript
 "paths": {
            "a": ["./a.ts"] // 此处映射是相对于"baseUrl"
  }
```

这个参数我来来回回试验没发现啥用，也不报错，后来发现是要用 ts 类型的文件，js 类型文件不报红，看下面例子

创建新文件 m.ts，添加内容（如果是 m.js 不会报错）：

```javascript
import decoraters from 'a';
console.log(decoraters);
```

`a就是上面写的a.ts`，编译发现报错，因为 ts 不知道去哪找这个文件

将`baseUrl 改成./或.` 发现 m.ts 不报错了，能找到模块 a 的定义

### 17、lib

> 这个是用于指定要引入的库文件，属性值为一个数组，如果不配置 lib，那么其默认会引入 dom 库，但是如果配置了 lib，那么就只会引入指定的库了

在 tsconfig.json 添加`"lib": "ES6"`:
在 m.ts 添加代码：

```javascript
document.getElementById('ccc');
```

可以看见已经报错了：  
![error](/images/tscoinfig配置/2.png)

`lib改为dom`则编译通过

### 18、types

> 官网是这么说的：如果指定了 types，只有被列出来的包才会被包含进来，它说的不是指的 import 导入模块找不到定义，指的是如果用了全局的变量，全局变量的类型找不找的到，看例子

在 tsconfig.json 添加` "types":[]`
安装 jquery,npm i jquery -S，此时的 node_module/@types 目录为：
![node_modules](/images/tscoinfig配置/3.png)

然后在 m.ts 中，分别用 jq 和 node 的全局变量，添加代码：

```javascript
// node
console.log(process);
// jquery
console.log($);
```

![error](/images/tscoinfig配置/4.png)

tsconfig.json 修改为 `"types":["node","jquery"]`后，编译通过
