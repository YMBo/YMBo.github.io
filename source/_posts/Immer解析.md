---
title: Immer解析
date: 2021-11-30 17:45:08
tags: [javascript]
---

### 前言

**Immutable**思想

> 数据就是一旦创建，就不能更改的数据。每当对 Immutable 对象进行修改的时候，就会返回一个新的 Immutable 对象，以此来保证数据的不可变。

如果不用 **Immutable**，直接对源数据操作有什么不好？

1.因为通常修改一个引用类型的数据，为了降低对源数据的副作用，一般要进行深拷贝，数据量大的话对性能有影响,效率低

2.数据操作过于灵活，不好追踪在哪修改的数据

关于 Immutable 思想的实现有**Immutable.js**、**immer.js**等  
而**Immutable.js**用法相对复杂，**immer.js**用法简单效率高，所以这篇文章是对 immer.js 源码的**分析解读**

### 一、immer 思想及意义

> 1.不再需要数据复制的防御机制(因为源数据是不变的)
>
> 2.优化对数据变化的检测
>
> 3.有助于简化开发过程，因为开发者不再需要在代码中追踪数据，寻找数据变更的位置（因为修改后的数据不可变，只能在 produce 理修改）
>
> 4.降低修改数据过程中的复杂度（不用深拷贝的，采用原生代理，哪里修改代理哪里，实现数据共享）
>
> 5.性能优化，数据共享，只改动被修改的部分（同上）
>
> 6.提供草稿功能，保存了每次的修改，相当于提供了数据历史快照的功能

### 二、熟悉 Immer

来个 demo 先熟悉下 Immer 用法

```javascript
import produce, { enablePatches, applyPatches } from "./immer";
// 这个是immer的插件使用的方法
enablePatches();
let patches = [];
let inversePatches = [];
let demo = { name: { age: 333 }, fan: { zz: 333 } };
// 第三个参数可选
let end = produce(
  demo,
  (draft) => {
    draft.name.age = 1000;
  },
  (patches, inversePatches) => {
    patches = stash.concat(patches);
    inverseStash = inversePatches.concat(inversePatches);
  }
);
// 通过applyPatches+inverseStash，能将值恢复到未修改状态
// 通过applyPatches+patches，能将值变为修改后的状态
console.log("历史回溯", applyPatches(end, inverseStash));
```

**这篇文章讲了啥？**

- [x] **Immer 的调度流程**
- [x] **immer 的插件模式**
- [x] **文件组织结构**
- [x] **实现按需代理的原理**
- [x] **将 proxy 转为普通对象的原理**

### 三、Immer 目录结构

```javascript
| |____immer
| | |____types
| | | |____globals.d.ts
| | | |____types-internal.ts
| | | |____index.js.flow
| | | |____types-external.ts
| | |____core
| | | |____a.d.ts
| | | |____scope.ts
| | | |____current.ts
| | | |____finalize.ts
| | | |____proxy.ts
| | | |____immerClass.ts
| | |____plugins
| | | |____patches.ts
| | | |____mapset.ts
| | | |____es5.ts
| | | |____all.ts
| | |____utils
| | | |____errors.ts
| | | |____common.ts
| | | |____env.ts
| | | |____plugins.ts
| | |____immer.ts
| | |____test.ts
| | |____internal.ts
```

- 核心功能以文件的形式放在了`core`下
- 不同插件以文件的形式放在了`plugins`下
- 剩下一些工具放在了`utils`下
- internal.ts 作为一个入口将上面提到的功能同一暴露出去

### 四、Immer 功能拆分

组织方式：immer 将主要功能拆分为不同文件，并将这些文件集中放在`core`目录下管理

- `immerClass`
- `scope`
- `proxy`
- `processResult`
- `maybeFreeze`
- `patchListener`
- `pluign系统`

![Immer](/images/Immer解析/immer.png)

下面挨个对这些功能进行分析

#### 1、[immerClass](https://github.com/YMBo/codeLearn/blob/861c0e53b8fc59da64657d86d0c76734da34647d/Immer/src/immer/core/immerClass.ts)

用这个 immer 库的时候，主要用的是`produce`这个函数 ，这个函数作为一个 Immer 类的实例方法暴露出来

源码：[produce](https://github.com/YMBo/codeLearn/blob/861c0e53b8fc59da64657d86d0c76734da34647d/Immer/src/immer/core/immerClass.ts#L66)

```javascript
const immer = new Immer();
export const produce: IProduce = immer.produce;
export default produce;
```

其中`produce`有两种用法，一种是普通传两个参数的调用

```javascript
nextState = produce(state,draft=>{ ... })
```

一种是高阶函数

```javascript
producer = produce(draft=>{ ... })
nextState = producer(state)
```

produce 方法里统筹调度了整个数据流转的过程，如下

##### 1.1、主流程

produce 主要干了下面的几件事

- [isDraftable(base)](https://github.com/YMBo/codeLearn/blob/861c0e53b8fc59da64657d86d0c76734da34647d/Immer/src/immer/core/immerClass.ts#L91)

  > 对数据类型判断，只处理基本类型 对象、数组

- [const scope = enterScope(this);](https://github.com/YMBo/codeLearn/blob/861c0e53b8fc59da64657d86d0c76734da34647d/Immer/src/immer/core/immerClass.ts#L93)

  > 创建 scope，scope 相当是一个 immer 实例、proxy 的管理器，具体的下面有说

- [const proxy = createProxy(this, base, undefined)](https://github.com/YMBo/codeLearn/blob/861c0e53b8fc59da64657d86d0c76734da34647d/Immer/src/immer/core/immerClass.ts#L96)

  > 对 base（源数据）创建代理，并返回给 produce 的第二个函数，就是 draft

- [result = recipe(proxy)](https://github.com/YMBo/codeLearn/blob/861c0e53b8fc59da64657d86d0c76734da34647d/Immer/src/immer/core/immerClass.ts#L101)

  > 执行 produce 的第二个参数函数，这里的 draft 就是上面创建的 proxy

  ```javascript
  produce(state,draft=>{ ... })
  ```

- [const m = processResult(result, scope);](https://github.com/YMBo/codeLearn/blob/861c0e53b8fc59da64657d86d0c76734da34647d/Immer/src/immer/core/immerClass.ts#L115)

  > 解析结果，对于 proxy 的修改都反映到了的代理数据（下面有讲）的 copy\_字段，这里是将这些修改组装，反映到 result 上并返回。  
  > 主要依赖这俩函数[finalize](https://github.com/YMBo/codeLearn/blob/861c0e53b8fc59da64657d86d0c76734da34647d/Immer/src/immer/core/finalize.ts#L63) [finalizeProperty](https://github.com/YMBo/codeLearn/blob/861c0e53b8fc59da64657d86d0c76734da34647d/Immer/src/immer/core/finalize.ts#L134)，将 proxy 类型的值进行递归取值，赋值给 result

- [maybeFreeze 数据自动冻结](https://github.com/YMBo/codeLearn/blob/861c0e53b8fc59da64657d86d0c76734da34647d/Immer/src/immer/core/finalize.ts#L182)  
  经过 produce 返回的数据都是冻结状态，这个函数就是冻结数据用的

#### 2、scope [github](https://github.com/YMBo/codeLearn/blob/861c0e53b8fc59da64657d86d0c76734da34647d/Immer/src/immer/core/scope.ts)

> scope 这个概念在这里相当于一个 proxy 管理器，进行 proxy 历史操作的保存和 proxy 的 revoke

```javascript
interface ImmerScope {
  // 历史快照，保存的是操作的顺序
  patches_?: Patch[];
  // 历史快照反向，保存的是反解顺序，执行这个会恢复为初始状态
  inversePatches_?: Patch[];
  // 是否自动冻结 默认都是自动冻结
  canAutoFreeze_: boolean;
  // proxy 队列
  drafts_: any[];
  // 父级scope
  parent_?: ImmerScope;
  // 补丁监听器
  patchListener_?: PatchListener;
  // immer实例
  immer_: Immer;
  // 没有标记为true的proxy
  unfinalizedDrafts_: number;
}
```

- patches*、inversePatches*

  > 历史操作信息并不是在每次操作的时候 push 的，而是在 processResult 阶段通过调用[generatePatches\_](https://github.com/YMBo/codeLearn/blob/861c0e53b8fc59da64657d86d0c76734da34647d/Immer/src/immer/core/finalize.ts#L123) push 的

  > 当 recipe（produce 第二个参数）执行时，如果出现错误则直接调用 scope 的[revokeScope](https://github.com/YMBo/codeLearn/blob/861c0e53b8fc59da64657d86d0c76734da34647d/Immer/src/immer/core/immerClass.ts#L108) 方法，目的是将所有 proxy 撤销，终止流程

- canAutoFreeze\_默认就是 true，冻结结果
- drafts\_
  > 每次创建的 proxy 都会推入这里存储
- patchListener\_
  > 这个字段是个引用，指向的是 procude 第三个参数
- immer\_
  > immerClass 的实例
- unfinalizedDrafts\_
  > 这里存储的是未标记为 finallized 的 proxy 的数量，当流程结束时会调用[finalize](https://github.com/YMBo/codeLearn/blob/861c0e53b8fc59da64657d86d0c76734da34647d/Immer/src/immer/core/finalize.ts#L63) 将 proxy 标记为**finalized**，同时 unfinalizedDrafts\_减 1

如果执行`recipe`顺利则执行[leaveScope](https://github.com/YMBo/codeLearn/blob/861c0e53b8fc59da64657d86d0c76734da34647d/Immer/src/immer/core/scope.ts#L122)方法标记当前 scope 为完成状态  
否则执行[revokeScope](https://github.com/YMBo/codeLearn/blob/861c0e53b8fc59da64657d86d0c76734da34647d/Immer/src/immer/core/immerClass.ts#L108) 方法，将所有 proxy 撤销，终止流程

#### 3、proxy [github](https://github.com/YMBo/codeLearn/blob/861c0e53b8fc59da64657d86d0c76734da34647d/Immer/src/immer/core/proxy.ts)

这个功能就是 immer 的精髓了，对应这个图

![proxy](/images/Immer解析/immerProxy.png)

##### 3.1、注解 ❶ ❷

> ❶ ❷  
> 这个 state 数据结构

```javascript

// 对这个东西做了代理
const state: ProxyState = {
  // 代理的数据类型
  type_: isArray ? ProxyType.ProxyArray : (ProxyType.ProxyObject as any),

  // Track which produce call this is associated with.
  scope_: parent ? parent.scope_ : getCurrentScope()!,

  // True for both shallow and deep changes.
  modified_: false,

  // Used during finalization.
  finalized_: false,

  // Track which properties have been assigned (true) or deleted (false).
  // 表示已被分配（就是被修改过）
  assigned_: {},

  // The parent draft state.
  parent_: parent,

  // The base state.
  base_: base,

  // The base proxy.
  draft_: null as any, // set below

  // 会将每一层的base浅拷贝到copy上
  copy_: null,

  // Called by the `produce` function.
  revoke_: null as any,
  isManual_: false,
};
```

创建代理的时候是使用**Proxy.revocable**创建的，因为用这个创建可以有**revoke**方法销毁 proxy

关于这个代理后的结构举个例子

```javascript
let demo = { name: { age: 333 }, fan: { zz: 333 } };
let end = produce(demo, (draft) => {
  draft.name.age = 1000;
});
```

**提出问题：那这里的 draft 是啥结构呢？**

> 这个 draft 就是上面代理返回的 proxy，它保存的所有的修改信息，数据结构如下

```javascript
interface Ibase {
  [key: string]: any;
}

interface Icopy extends Ibase {
  [key: string]: Istate;
}

type Istate = ProxyHandler<{
  base_: Ibase,
  copy_: Icopy,
}>;
```

**那上面的`draft`因为修改了 name 下的 age，那这个 draft 就是这样的**

```javascript
let proxy = (base) =>
  Proxy.revocable(
    {
      base_: base,
      copy_: null,
    },
    {}
  ).proxy;

let draft: Istate = proxy({
  base_: { name: { age: 333 }, fan: { zz: 333 } },
  copy_: { name: proxy({ age: 333 }), fan: { zz: 333 } },
});
```

`draft.name.age = 1000`因为修改的深度是两层，而且只修改了`name`下的`age`字段,所以只代理的这两个值

可以看到`base_`只是存源数据，所有的修改都反映到了 copy\_字段上，这个代理有下面的特点

- 每次的代理只代理对象的一层
- 并不是对象所有值都会代理
- 修改只会反映到 copy\_字段上

按需代理  
![按需代理](/images/Immer解析/immerProxy2.png)

##### 3.2、注解 ❸ [源码](https://github.com/YMBo/codeLearn/blob/861c0e53b8fc59da64657d86d0c76734da34647d/Immer/src/immer/core/proxy.ts#L126)

```javascript
// peek直接取得是原始值，这里做的判断，如果相等那就proxy一下，否则不重复代理
console.log('ooooooo', value);
if (value === peek(state.base_, prop)) {
  console.log('代理get', state.copy_, prop);
  // 浅拷贝base到copy
  prepareCopy(state);
  // 针对具体的属性做代理
  state.copy_![prop as any] = createProxy(
    state.scope_.immer_,
    value,
    state,
  );
  console.log('zzzzz', state.copy_![prop as any]);
  return state.copy_![prop as any];
}
```

##### 3.3、注解 ❹ [github](https://github.com/YMBo/codeLearn/blob/301f0243993dadcd9adc3c895cb9fddf71360d6b/Immer/src/immer/core/proxy.ts#L188)

##### 3.4、注解 ❺ [github](https://github.com/YMBo/codeLearn/blob/301f0243993dadcd9adc3c895cb9fddf71360d6b/Immer/src/immer/core/proxy.ts#L198)

##### 3.5、注解 ❼ [github](https://github.com/YMBo/codeLearn/blob/301f0243993dadcd9adc3c895cb9fddf71360d6b/Immer/src/immer/core/proxy.ts#L188)

#### 4、processResult [github](https://github.com/YMBo/codeLearn/blob/301f0243993dadcd9adc3c895cb9fddf71360d6b/Immer/src/immer/core/finalize.ts#L23)

这个函数主要干了下面三件事

1、计算出最后结果  
2、冻结结果  
3、计算补丁`patches_`、`inversePatches_`后触发补丁函数 patchListener\_

这三个功能主要依赖的是  
[finalize](https://github.com/YMBo/codeLearn/blob/301f0243993dadcd9adc3c895cb9fddf71360d6b/Immer/src/immer/core/finalize.ts#L63)、[finalizeProperty](https://github.com/YMBo/codeLearn/blob/301f0243993dadcd9adc3c895cb9fddf71360d6b/Immer/src/immer/core/finalize.ts#L134) 计算结果 result  
[generatePatches\_](https://github.com/YMBo/codeLearn/blob/301f0243993dadcd9adc3c895cb9fddf71360d6b/Immer/src/immer/core/finalize.ts#L123) 计算 patches*、inversePatches*

```javascript
// 这个scope就是produce进来的时候enterScope创建的scope
// baseDraft取得是scope.drafts_的第一项
result = finalize(scope, baseDraft, []);
```

现在来简化一下 finalize 流程，看看 immer 是怎么解析出最后结果的

```javascript
function finalize(rootScope,value){
    // state.draft_指向的是当前state的proxy
    // 而 shallowCopy 主要是浅拷贝值到copy_字段上
    // 那目前的result就是一个不包含proxy的对象
    const result = state.copy_ = shallowCopy(state.draft_)
    // 这个patch第一次进来是个undefined
    each(result，(key,childValue)=>{
        finalizeProperty(
          rootScope,
          state,
          result,
          key,
          childValue,
          path,
        )
    })

    // 冻结结果
    maybeFreeze(rootScope, result, false);
    // 计算补丁值，计算出补丁数组并赋值到scope的patches_、inversePatches_字段
    if (path && rootScope.patches_) {
      getPlugin('Patches').generatePatches_(
        state,
        path,
        rootScope.patches_,
        rootScope.inversePatches_!,
      );
    }
}

// 这个函数的作用是查找已经变成proxy的值 并把这些值修改结果反映到result上
function finalizeProperty(rootScope: ImmerScope,
  parentState: undefined | ImmerState,
  targetObject: any,
  prop: string | number,
  childValue: any,
  rootPath?: PatchPath,){
    if (isDraft(childValue)) {
        // 查找修改路径,主要是根据assigned_字段
    const path =
      rootPath &&
      parentState &&
      parentState!.type_ !== ProxyType.Set && // Set objects are atomic since they have no keys.
      !has((parentState as Exclude<ImmerState, SetState>).assigned_!, prop) // Skip deep patches for assigned keys.
        ? rootPath!.concat(prop)
        : undefined;
        // 深度优先遍历子项，一个递归
        const res = finalize(rootScope, childValue, path);
        // targetObject[prop]=res
        set(targetObject, prop, res);
    }

}
```

#### 5、maybeFreeze

这个是依赖`Object.freeze()`实现

#### 6、patchListener

补丁监听函数，”补丁“指的是操作数据的历史，主要有三个类型`REPLACE`、`ADD`、`REMOVE`

patchListener 也是 produce 的第三个参数，它接收两个参数分别是`inversePatches`和`patches`

- patches 是操作数据的历史动作

- inversePatches 是 patches 反向，这个反向比如 patches 其中一向是 Add，那么在 inversePatches 就是 Remove,也就是说通过 inversePatches 可以反解数据

**produce 第三个参数**

```javascript
// 通过 patchListener 函数，暴露正向和反向的补丁数组
patchListener: (patches: Patch[], inversePatches: Patch[]) => void
```

**数组 patch 参数格式**

```javascript
interface Patch {
  op: "replace" | "remove" | "add" // 一次更改的动作类型
  path: (string | number)[] // 此属性指从树根到被更改树杈的路径
  value?: any // op为 replace、add 时，才有此属性，表示新的赋值
}
```

通过补丁功能实现数据回溯

##### 1、历史回溯 demo

```javascript
let stash = [];
let inverseStash = [];
console.log("执行前");
var b = produce(
  a,
  (draft) => {
    draft.name.gg = "1000";
    draft.name.age = 1000;
    delete draft.fan.zz;
  },
  (patches, inversePatches) => {
    stash = stash.concat(patches);
    inverseStash = inversePatches.concat(inversePatches);
    console.log("patches", patches);
    console.log("inversePatches", inversePatches);
  }
);
console.log("执行后", b, a);
console.log("stash", stash);
console.log("历史回溯", applyPatches(b, inverseStash));
```

#### 7、plugin 系统

为啥要说 immer 的插件呢，因为用 patchListener 功能的时候必须引入`enablePatches()`调用一下才能用，达到了按需使用的效果

immer 设计了一个 plugin 管理器存放在 plugins.ts 文件
主要功能有  
1、`plugins={}`作为插件表，存放这所有插件  
2、`loadPlugin`注册插件，其实就是往插件表上存  
3、`getPlugin`调用插件,去插件表检索返回

所有的插件以文件的形式保存在 plugins 目录下

```javascript
|____plugins
| |____patches.ts
| |____mapset.ts
| |____es5.ts
| |____all.ts
```

这种可插拔的插件模式在自己写功能的时候也可以借鉴，这样能让小功能通过插件的方式引入，即插即用，提高了代码的灵活性，也便于不同功能的维护管理
