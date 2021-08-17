---
toc: true
title: vue初级知识
date: 2021-6-20 20:26:22
tags: 
- HTML 
- Vue 
- JavaScript
---
# VUE 初级知识

## 什么是渐进式框架
![渐进式框架](https://greenhaha.oss-cn-beijing.aliyuncs.com/frontend/assets/img/%E6%B8%90%E8%BF%9B%E5%BC%8F%E6%A1%86%E6%9E%B6.png)<br>
“渐进式框架”也非常简单，就是用你x想用或者能用的功能特性，你不想用的部分功能可以先不用。
VUE不强求你一次性接受并使用它的全部功能特性。

## 什么是MVVM响应式编程模型
![MVVM模型](https://greenhaha.oss-cn-beijing.aliyuncs.com/frontend/assets/img/MVVM%E6%A8%A1%E5%9E%8B.png)

MVVM：页面输入改变数据，数据改变影响页面数据展示与渲染

M（model）：普通的javascript数据对象

V（view）：前端展示页面

VM（ViewModel）：用于双向绑定数据与页面，可以理解为vue的实例

“响应式”，是指当数据改变后，Vue 会通知到使用该数据的代码。例如，视图渲染中使用了数据，数据改变后，视图也会自动更新。

## 什么是vue
Vue是一套用于构建用户界面的渐进式框架。与其它大型框架不同的是，Vue 被设计为可以自底向上逐层应用。Vue 的核心库只关注视图层，不仅易于上手，还便于与第三方库或既有项目整合。另一方面，当与现代化的工具链以及各种[支持类库](https://github.com/vuejs/awesome-vue#libraries--plugins)结合使用时，Vue 也完全能够为复杂的单页应用提供驱动。

### 声明式渲染

Vue.js 的核心是一个允许采用简洁的模板语法来声明式地将数据渲染进 DOM 的系统：
``` HTML
<div id="app">
  {{ message }}
</div>
```

``` JS
var app = new Vue({
  el: '#app',
  data: {
    message: 'Hello Vue!'
  }
})
```
我们已经成功创建了第一个 Vue 应用！看起来这跟渲染一个字符串模板非常类似，但是 Vue 在背后做了大量工作。现在数据和 DOM 已经被建立了关联，所有东西都是响应式的。我们要怎么确认呢？打开你的浏览器的 JavaScript 控制台 (就在这个页面打开)，并修改 `app.message` 的值，你将看到上例相应地更新。

注意我们不再和 HTML 直接交互了。一个 Vue 应用会将其挂载到一个 DOM 元素上 (对于这个例子是 `#app`) 然后对其进行完全控制。那个 HTML 是我们的入口，但其余都会发生在新创建的 Vue 实例内部

### 组件(component)

组件(Component)是是一个含义很大的概念，一般是指软件系统的一部分，承担了特定的职责，可以独立于整个系统进行开发和测试，一个良好设计的组件应该可以在不同的软件系统中被使用(可复用)。
现在流行的react 和vue 都是组件框架。

* 基本的例子

这里有一个 Vue 组件的示例：
``` JS
// 定义一个名为 button-counter 的新组件
Vue.component('button-counter', {
  data: function () {
    return {
      count: 0
    }
  },
  template: '<button v-on:click="count++">You clicked me {{ count }} times.</button>'
})
```
组件是可复用的 Vue 实例，且带有一个名字：在这个例子中是 <button-counter>。我们可以在一个通过 new Vue 创建的 Vue 根实例中，把这个组件作为自定义元素来使用：
``` HTML
<div id="components-demo">
  <button-counter></button-counter>
</div>
```
``` JS
new Vue({ el: '#components-demo' })
```
因为组件是可复用的 Vue 实例，所以它们与 new Vue 接收相同的选项，例如 data、computed、watch、methods 以及生命周期钩子等。仅有的例外是像 el 这样根实例特有的选项。

### 模版语法

Vue.js 使用了基于 HTML 的模板语法，允许开发者声明式地将 DOM 绑定至底层 Vue 实例的数据。所有 Vue.js 的模板都是合法的 HTML，所以能被遵循规范的浏览器和 HTML 解析器解析。

在底层的实现上，Vue 将模板编译成虚拟 DOM 渲染函数。结合响应系统，Vue 能够智能地计算出最少需要重新渲染多少组件，并把 DOM 操作次数减到最少。

如果你熟悉虚拟 DOM 并且偏爱 JavaScript 的原始力量，你也可以不用模板，直接写渲染 (render) 函数，使用可选的 JSX 语法。

#### 文本
数据绑定最常见的形式就是使用[“Mustache”语法](https://github.com/janl/mustache.js) (双大括号) 的文本插值：
``` HTML
<span>Message: {{ msg }}</span>
```
Mustache 标签将会被替代为对应数据对象上 msg property 的值。无论何时，绑定的数据对象上 msg property 发生了改变，插值处的内容都会更新。

通过使用 v-once 指令，你也能执行一次性地插值，当数据改变时，插值处的内容不会更新。但请留心这会影响到该节点上的其它数据绑定：
``` HTML
<span v-once>这个将不会改变: {{ msg }}</span>
```
#### 原始HTML
双大括号会将数据解释为普通文本，而非 HTML 代码。为了输出真正的 HTML，你需要使用 v-html 指令：
``` HTML
<p>Using mustaches: {{ rawHtml }}</p>
<p>Using v-html directive: <span v-html="rawHtml"></span></p>
```

这个 `span` 的内容将会被替换成为 property 值 `rawHtml`，直接作为 HTML——会忽略解析 property 值中的数据绑定。注意，你不能使用 `v-html` 来复合局部模板，因为 Vue 不是基于字符串的模板引擎。反之，对于用户界面 (UI)，组件更适合作为可重用和可组合的基本单位。

#### Attribute
Mustache 语法不能作用在 HTML attribute 上，遇到这种情况应该使用 v-bind 指令：
``` HTML
<div v-bind:id="dynamicId"></div>
```
对于布尔 attribute (它们只要存在就意味着值为 true)，v-bind 工作起来略有不同，在这个例子中：
``` HTML
<button v-bind:disabled="isButtonDisabled">Button</button>
```
如果 `isButtonDisabled` 的值是 `null`、`undefined` 或 `false`，则 `disabled` `attribute` 甚至不会被包含在渲染出来的 `<button> `元素中。

#### 使用JavaScript表达式

在我们的模板中，我们一直都只绑定简单的 property 键值。但实际上，对于所有的数据绑定，Vue.js 都提供了完全的 JavaScript 表达式支持.
``` HTML
{{ number + 1 }}

{{ ok ? 'YES' : 'NO' }}

{{ message.split('').reverse().join('') }}

<div v-bind:id="'list-' + id"></div>
```
这些表达式会在所属 Vue 实例的数据作用域下作为 JavaScript 被解析。有个限制就是，每个绑定都只能包含单个表达式，所以下面的例子都不会生效。
``` HTML
<!-- 这是语句，不是表达式 -->
{{ var a = 1 }}

<!-- 流控制也不会生效，请使用三元表达式 -->
{{ if (ok) { return message } }}
```

### 指令

指令（Directives）是带有`v-`前缀的特殊attribute。指令attribute的值预期是单个JavaScript表达式（`v-for`是例外）。
指令的职责是，当表达式的值改变时，将其产生的连带影响，响应式地作用于 DOM。回顾我们在介绍中看到的例子：
``` HTML
<p v-if="seen">在你看到我了</p>
```
这里，v-if 指令将根据表达式 seen 的值(`true` or `false`)的真假来插入/移除 <p> 元素。
### `VUE`常用指令

* v-text
> v-text 等同于 innerText
``` HTML
<div id="app">
    <!-- 数据插值 : 把数据插入到指定位置 ,并且不会覆盖原有的内容 -->
    <p>{{name}}</p>
    <p>+++++n{{name}}+++++</p>
    <!-- 数据插值 : 不会解析HMTL标签 -->
    <p>{{span}}</p>
    <!-- v-text 直接渲染数据, 并且会覆盖原有的数据 -->
    <p v-text="name"></p>
    <p v-text="name">++++++++++</p>
    <!-- v-text 不会渲染 HTML 标签 -->
    <p v-text="span"></p>
</div>
<script>
    new Vue({
        el: "#app",
        data: ({
            name: "张三",
            span: "<span> 我是span标签 </span>"
        })
    })
</script>
```
* v-html
> v-html 等同于 innerHTML
``` HTML
<div id="app">
    <!-- 数据插值 : 把数据插入到指定位置 ,并且不会覆盖原有的内容 -->
    <p>{{name}}</p>
    <p>+++++n{{name}}+++++</p>
    <!-- 数据插值 : 不会解析HMTL标签 -->
    <p>{{span}}</p>
    <!-- v-html 会覆盖原有数据, 并且可以渲染数据和html标签 -->
    <p v-html="name">+++++++++</p>
    <p v-html="span">+++++++++</p>
</div>
<script>
    new Vue({
        el: "#app",
        data: ({
            name: "张三",
            span: "<span> 我是span标签 </span>"
        })
    })
</script>
```
* v-show

源码路径：src/platforms/web/runtime/directives/show.js
``` JS
/* @flow */
// 过度动画 在进入和离开的时候有一定过渡效果
import { enter, leave } from '../modules/transition'

// recursively search for possible transition defined inside the component root 定位元素即效果作用的目标元素
function locateNode (vnode: VNode): VNodeWithData {
  return vnode.componentInstance && (!vnode.data || !vnode.data.transition)
    ? locateNode(vnode.componentInstance._vnode)
    : vnode
}
// 对目标元素绑定方法，利用v-dom中element.js 中获取到的元素对象 添加隐藏与显示样式控制
export default {
  bind (el: any, { value }: VNodeDirective, vnode: VNodeWithData) {
    vnode = locateNode(vnode)
    const transition = vnode.data && vnode.data.transition
    const originalDisplay = el.__vOriginalDisplay =
      el.style.display === 'none' ? '' : el.style.display
    if (value && transition) {
      vnode.data.show = true
      enter(vnode, () => {
        el.style.display = originalDisplay
      })
    } else {
      el.style.display = value ? originalDisplay : 'none'
    }
  },

  update (el: any, { value, oldValue }: VNodeDirective, vnode: VNodeWithData) {
    /* istanbul ignore if */
    if (!value === !oldValue) return
    vnode = locateNode(vnode)
    const transition = vnode.data && vnode.data.transition
    if (transition) {
      vnode.data.show = true
      if (value) {
        enter(vnode, () => {
          el.style.display = el.__vOriginalDisplay
        })
      } else {
        leave(vnode, () => {
          el.style.display = 'none'
        })
      }
    } else {
      el.style.display = value ? el.__vOriginalDisplay : 'none'
    }
  },

  unbind (
    el: any,
    binding: VNodeDirective,
    vnode: VNodeWithData,
    oldVnode: VNodeWithData,
    isDestroy: boolean
  ) {
    if (!isDestroy) {
      el.style.display = el.__vOriginalDisplay
    }
  }
}
```
带有 v-show 的元素始终会被渲染并保留在 DOM 中。v-show 只是简单地切换元素的 CSS property display

* v-if

因为 v-if 是一个指令，所以必须将它添加到一个元素上。但是如果想切换多个元素呢？此时可以把一个 `<template>` 元素当做不可见的包裹元素，并在上面使用 v-if。最终的渲染结果将不包含 `<template>` 元素。<br>
原理：Vue进行了如下转化template ---> ast算法（抽象语法树） ---> render函数，最后根据生成的render函数来生成相应的DOM，这里就不拓展讲了。在生成ast和render函数的时候，Vue对v-if这一类指令进行了解析。
判断对应的templage中代码块是否显在render函数中进行渲染挂在到真实DOM中
详细介绍：v-if 实现原理篇

* v-if-else-if

使用 v-else 指令来表示 v-if 的“else 块”：
``` HTML
<div v-if="Math.random() > 0.5">
  Now you see me
</div>
<div v-else>
  Now you don't
</div>
```
v-else 元素必须紧跟在带 v-if 或者 v-else-if 的元素的后面，否则它将不会被识别。

v-else-if，顾名思义，充当 v-if 的“else-if 块”，可以连续使用：
``` HTML
<div v-if="type === 'A'">
  A
</div>
<div v-else-if="type === 'B'">
  B
</div>
<div v-else-if="type === 'C'">
  C
</div>
<div v-else>
  Not A/B/C
</div>
```
类似于 v-else，v-else-if 也必须紧跟在带 v-if 或者 v-else-if 的元素之后。

* v-for

v-for 指令基于一个数组来渲染一个列表。v-for 指令需要使用 item in items 形式的特殊语法，其中 items 是源数据数组，而 item 则是被迭代的数组元素的别名。
``` HTML
<ul id="example-1">
  <li v-for="item in items" :key="item.message">
    {{ item.message }}
  </li>
</ul>
```
``` JS
var example1 = new Vue({
  el: '#example-1',
  data: {
    items: [
      { message: 'Foo' },
      { message: 'Bar' }
    ]
  }
})
```

在 `v-for` 块中，我们可以访问所有父作用域的 property。`v-for` 还支持一个可选的第二个参数，即当前项的索引。
``` HTML
<ul id="example-2">
  <li v-for="(item, index) in items">
    {{ parentMessage }} - {{ index }} - {{ item.message }}
  </li>
</ul>
```
``` JS
var example2 = new Vue({
  el: '#example-2',
  data: {
    parentMessage: 'Parent',
    items: [
      { message: 'Foo' },
      { message: 'Bar' }
    ]
  }
})
```
更多详细内容：v-for 实现原理与介绍

* v-on

缩写：@

预期：Function | Inline Statement | Object

参数：event

修饰符：

- `.stop` - 调用 `event.stopPropagation()`。
- `.prevent` - 调用 `event.preventDefault()`。
- `.capture` - 添加事件侦听器时使用 capture 模式。
- `.self` - 只当事件是从侦听器绑定的元素本身触发时才触发回调。
- `.{keyCode | keyAlias}` - 只当事件是从特定键触发时才触发回调。
- `.native` - 监听组件根元素的原生事件。
- `.once` - 只触发一次回调。
- `.left` - (2.2.0) 只当点击鼠标左键时触发。
- `.right` - (2.2.0) 只当点击鼠标右键时触发。
- `.middle` - (2.2.0) 只当点击鼠标中键时触发。
- `.passive` - (2.3.0) 以 `{ passive: true }` 模式添加侦听器

* 用法：

绑定事件监听器。事件类型由参数指定。表达式可以是一个方法的名字或一个内联语句，如果没有修饰符也可以省略。

用在普通元素上时，只能监听原生 DOM 事件。用在自定义元素组件上时，也可以监听子组件触发的自定义事件。

在监听原生 DOM 事件时，方法以事件为唯一的参数。如果使用内联语句，语句可以访问一个 `$event` property：`v-on:click="handle('ok', $event)"`。

从 2.4.0 开始，v-on 同样支持不带参数绑定一个事件/监听器键值对的对象。注意当使用对象语法时，是不支持任何修饰器的。

* v-bind

缩写：:

预期：any (with argument) | Object (without argument)

参数：attrOrProp (optional)

修饰符：

- .prop - 作为一个 DOM property 绑定而不是作为 attribute 绑定。(差别在哪里？)
- .camel - (2.1.0+) 将 kebab-case attribute 名转换为 camelCase。(从 2.1.0 开始支持)
- .sync (2.3.0+) 语法糖，会扩展成一个更新父组件绑定值的 v-on 侦听器。

* 用法：

动态地绑定一个或多个 attribute，或一个组件 prop 到表达式。

在绑定 class 或 style attribute 时，支持其它类型的值，如数组或对象。可以通过下面的教程链接查看详情。

在绑定 prop 时，prop 必须在子组件中声明。可以用修饰符指定不同的绑定类型。

没有参数时，可以绑定到一个包含键值对的对象。注意此时 class 和 style 绑定不支持数组和对象。


* v-model

`v-model` 指令在表单 `<input>`、`<textarea>` 及 `<select>` 元素上创建双向数据绑定。它会根据控件类型自动选取正确的方法来更新元素。尽管有些神奇，但 v-model 本质上不过是语法糖。它负责监听用户的输入事件以更新数据，并对一些极端场景进行一些特殊处理。

> 想要组件 v-model生效 它必须:<br>
>「接收一个value属性」<br>
>「在value值改变时 触发input事件」

注意⚠️
>v-model即我们常说的双向绑定，但一定不能跟数据响应原理混为一谈，因为数据响应式是通过数据的改变去驱动视图渲染，而双向绑定除了可以数据驱动DOM渲染，DOM的变化反过来也可以影响数据，是一个双向的关系。

![数据响应式](https://greenhaha.oss-cn-beijing.aliyuncs.com/frontend/assets/img/MVVM%E6%A8%A1%E5%9E%8B.png)


* v-cloak

`vue`如果在网络不好的情况下,在渲染生命式模版的时候比如`{{data}}`。在浏览器上会显示字符串 {{data}}。会降低用户体验
``` HTML
<style>
    [v-cloak] {
    /* v-cloak 必须配合这个style 进行隐藏 */
        display: none;
    }
</style>

<div id="app">
    <p v-cloak>{{ name }} <span>如果网络差,我则不会显示</span></p>
    <p>{{ name }} <span>无论网络状态如何 ,我都会显示</span></p>
</div>

<script src="vue.js"></script>

<script>
    new Vue({
        el: "#app",
        data: ({
            name: "张三",
        })
    })
</script>

```
* v-once
> 默认情况下, `vue`事件是可以重复触发的,但是在特定的情况下, 我们只想要事件执行一次.所以就有了`v-once`