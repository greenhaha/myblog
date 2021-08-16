---
title: vue-router剖析
date: 2021-6-4 22:12:22
tags:  
- Node 
- JavaScript
- vue
- JavaScript
---

# vue-router剖析
## 什么是路由
路由最开始出现在后端。以前用模版开发时或者老系统开发或者做php开发时经常会看到这种url
```
http://wwww.aaaa.edu.cn/bbs/forum.php
```
有时还会有带`.asp`或`.html`的路径，这就是所谓的SSR(Server Side Render)，通过服务端渲染，直接返回页面。
> 服务端渲染（SSR）
SSR是Server Side Render简称；页面上的内容是通过服务端渲染生成的，浏览器直接显示服务端返回的html就可以了。

其响应过程为：
1.浏览器发出请求

2.服务器监听到80端口（或443）有请求过来，并解析url路径

3.根据服务器的路由配置，返回相应信息（可以是 html 字串，也可以是 json 数据，图片等）

4.浏览器根据数据包的`Content-Type`来决定如何解析数据
路由就是用来跟后端服务器进行交互的一种方式，通过不同的路径，来请求不同的资源，请求不同的页面是路由的其中一种功能。

前端路由的诞生：
前端路由的出现要从ajax开始。

Ajax，全称 Asynchronous JavaScript And XML，是浏览器用来实现异步加载的一种技术方案。在 90s 年代初，大多数的网页都是通过直接返回 HTML 的，用户的每次更新操作都需要重新刷新页面。及其影响交互体验，随着网络的发展，迫切需要一种方案来改善这种情况。
1996，微软首先提出 iframe 标签，iframe 带来了异步加载和请求元素的概念，随后在 1998 年，微软的 Outloook Web App 团队提出 Ajax 的基本概念（XMLHttpRequest的前身），并在 IE5 通过 ActiveX 来实现了这项技术。在微软实现这个概念后，其他浏览器比如 Mozilia，Safari，Opera 相继以 XMLHttpRequest 来实现 Ajax。（  兼容问题从此出现，话说微软命名真喜欢用X，MFC源码一大堆。。）不过在 IE7 发布时，微软选择了妥协，兼容了 XMLHttpRequest 的实现。
有了 Ajax 后，用户交互就不用每次都刷新页面，体验带来了极大的提升。
但真正让这项技术发扬光大的，(｡･∀･)ﾉﾞ还是后来的 Google Map，它的出现向人们展现了 Ajax 的真正魅力，释放了众多开发人员的想象力，让其不仅仅局限于简单的数据和页面交互，为后来异步交互体验方式的繁荣发展带来了根基。
而异步交互体验的更高级版本就是 SPA（那么问个问题，异步交互最高级的体验是什么？会在文末揭晓）—— 单页应用。
单页应用不仅仅是在页面交互是无刷新的，连页面跳转都是无刷新的，为了实现单页应用，所以就有了前端路由。单页应用的概念是伴随着 MVVM 出现的。最早由微软提出，然后他们在浏览器端用 `Knockoutjs` 实现。但这项技术的强大之处并未当时的开发者体会到，可能是因为 `Knockoutjs` 实现过于复杂，导致没有大面积的扩散。
同样，这次接力的选手依然是 Google。Google 通过 Angularjs 将 MVVM 及单页应用发扬光大，让前端开发者能够开发出更加大型的应用，职能变得更大了。（不得不感慨，微软 跟 Google 都是伟大的公司）。随后都是大家都知道的故事，前端圈开始得到了爆发式的发展，陆续出现了很多优秀的框架。

### 前端路由原理
对于vue-router和react-router等前端路由来看，其实原理很简单
本质上就是检测url 的变化，截获url地址，然后解析来匹配路由规则。但是这时就会有一些疑问：
* url每次变化都会刷新页面吗？
* 页面刷新，javascript怎么检测和获取url？

对于这2个问题，在很早之前，开发者就想到解决办法：通过hash来实现路由
url中hash就类似与下列
```
https://segmentfault.com/a/1190000011956628#articleHeader2
```

#### hash模式

这种 `#`。后面 hash 值的变化，并不会导致浏览器向服务器发出请求，浏览器不发出请求，也就不会刷新页面。另外每次 hash 值的变化，还会触发 `hashchange` 这个事件，通过这个事件我们就可以知道 hash 值发生了哪些变化.

对于hash模式实现路由流程如下：
![router-hash](https://greenhaha.oss-cn-beijing.aliyuncs.com/frontend/assets/img/routerhash.jpeg)

hash实现方式不需要服务器的支持

而对于vue-router来说
``` JavaScript
// this is delayed until the app mounts
// to avoid the hashchange listener being fired too early
/**
* 添加 url hash 变化的监听器
*/
setupListeners () {
    if (this.listeners.length > 0) {
      return
    }

    const router = this.router
    const expectScroll = router.options.scrollBehavior
    const supportsScroll = supportsPushState && expectScroll

    if (supportsScroll) {
      this.listeners.push(setupScroll())
    }

    const handleRoutingEvent = () => {
      const current = this.current
      if (!ensureSlash()) {
        return
      }
      /**
        * transitionTo: 
        * 匹配路由
        * 并通过路由配置，把新的页面 render 到 ui-view 的节点
        */
      this.transitionTo(getHash(), route => {
        if (supportsScroll) {
          handleScroll(this.router, route, current, true)
        }
        if (!supportsPushState) {
          replaceHash(route.fullPath)
        }
      })
    }
    const eventType = supportsPushState ? 'popstate' : 'hashchange'
    /**
    * 每当 hash 变化时就解析路径
    * 匹配路由
    */

    window.addEventListener(
      eventType,
      handleRoutingEvent
    )
    this.listeners.push(() => {
      window.removeEventListener(eventType, handleRoutingEvent)
    })
  }
```

检测到hash发生变化后通过替换dom形式来实现页面变换

#### history模式

当html5发布后针对与url对象，新增了`pushState，` 和 `replaceState`
通过这两个 API 可以改变 url 地址且不会发送请求。同时还有 `onpopstate` 事件。通过这些就能用另一种方式来实现前端路由了，但原理都是跟 hash 实现相同的。用了 HTML5 的实现，单页路由的 url 就不会多出一个`#`，变得更加美观。但因为没有 `#` 号，所以当用户刷新页面之类的操作时，浏览器还是会给服务器发送请求。为了避免出现这种情况，所以这个实现需要服务器的支持，需要把所有路由都重定向到根页面。具体可以见：[HTML5 histroy 模式]([HTML5 History 模式](https://router.vuejs.org/zh/guide/essentials/history-mode.html))
![router-history](https://greenhaha.oss-cn-beijing.aliyuncs.com/frontend/assets/img/routerhistory.jpeg)

``` javascript
/* @flow */

export class HTML5History extends History {
  _startLocation: string

  constructor (router: Router, base: ?string) {
    super(router, base)

    this._startLocation = getLocation(this.base)
  }

  setupListeners () {
    if (this.listeners.length > 0) {
      return
    }

    const router = this.router
    const expectScroll = router.options.scrollBehavior
    const supportsScroll = supportsPushState && expectScroll

    if (supportsScroll) {
      this.listeners.push(setupScroll())
    }

    const handleRoutingEvent = () => {
      const current = this.current

      // Avoiding first `popstate` event dispatched in some browsers but first
      // history route not updated since async guard at the same time.
      const location = getLocation(this.base)
      if (this.current === START && location === this._startLocation) {
        return
      }

      this.transitionTo(location, route => {
        if (supportsScroll) {
          handleScroll(router, route, current, true)
        }
      })
    }
    /**
    * 原理还是跟 hash 实现一样
    * 通过监听 popstate 事件
    * 匹配路由，然后更新页面 DOM
    */
    window.addEventListener('popstate', handleRoutingEvent)
    this.listeners.push(() => {
      window.removeEventListener('popstate', handleRoutingEvent)
    })
  }

  go (n: number) {
    window.history.go(n)
  }

  push (location: RawLocation, onComplete?: Function, onAbort?: Function) {
    const { current: fromRoute } = this
    this.transitionTo(location, route => {
      pushState(cleanPath(this.base + route.fullPath))
      handleScroll(this.router, route, fromRoute, false)
      onComplete && onComplete(route)
    }, onAbort)
  }

  replace (location: RawLocation, onComplete?: Function, onAbort?: Function) {
    const { current: fromRoute } = this
    this.transitionTo(location, route => {
    // replaceState 跟 pushState 的区别在于，不会记录到历史栈
      replaceState(cleanPath(this.base + route.fullPath))
      handleScroll(this.router, route, fromRoute, false)
      onComplete && onComplete(route)
    }, onAbort)
  }

  ensureURL (push?: boolean) {
    if (getLocation(this.base) !== this.current.fullPath) {
      const current = cleanPath(this.base + this.current.fullPath)
      push ? pushState(current) : replaceState(current)
    }
  }

  getCurrentLocation (): string {
    return getLocation(this.base)
  }
}

export function getLocation (base: string): string {
  let path = window.location.pathname
  const pathLowerCase = path.toLowerCase()
  const baseLowerCase = base.toLowerCase()
  // base="/a" shouldn't turn path="/app" into "/a/pp"
  // https://github.com/vuejs/vue-router/issues/3555
  // so we ensure the trailing slash in the base
  if (base && ((pathLowerCase === baseLowerCase) ||
    (pathLowerCase.indexOf(cleanPath(baseLowerCase + '/')) === 0))) {
    path = path.slice(base.length)
  }
  return (path || '/') + window.location.search + window.location.hash
}

```

对于vue-router来说上面的两种模式其实是依赖与浏览器环境。而对于实际开发过程中，往往出现另外一种情况：
就是针对于服务端渲染情况下，不存在浏览器api环境中，如果需要路由该怎么办？

#### abstract模式

abstract模式是使用一个不依赖于浏览器的浏览历史虚拟管理后端。支持所有 JavaScript 运行环境，如 Node.js 服务器端。

对于abstract根据平台差异可以看出，在 Weex 环境中只支持使用 abstract 模式。 不过，vue-router 自身会对环境做校验，如果发现没有浏览器的 API，vue-router 会自动强制进入 abstract 模式，所以 在使用 vue-router 时只要不写 mode 配置即可，默认会在浏览器环境中使用 hash 模式，在移动端原生环境中使用 abstract 模式。 （当然，你也可以明确指定在所有情况下都使用 abstract 模式）

而对于vue-router中`abstract.js` 中`AbstractHistory`是个比较特别的实现，因为它不是浏览器环境，所以它没有`window.history`，它不需要再去考虑`popstate ` `hashchange`事件，不需要去管浏览器的地址栏，但是它需要自己管理路由历史记录。所在`AbstractHistory`里面，设置了stack数组来存储访问的路由对象，设置了一个index实例属性，来表示当前访问的路由位置。

``` javascript
/* @flow */

import type Router from '../index'
import { History } from './base'
import { NavigationFailureType, isNavigationFailure } from '../util/errors'

export class AbstractHistory extends History {
  index: number
  stack: Array<Route>

  constructor (router: Router, base: ?string) {
    super(router, base)
    this.stack = []
    this.index = -1
  }

  push (location: RawLocation, onComplete?: Function, onAbort?: Function) {
    this.transitionTo(
      location,
      route => {
        this.stack = this.stack.slice(0, this.index + 1).concat(route)
        this.index++
        onComplete && onComplete(route)
      },
      onAbort
    )
  }

  replace (location: RawLocation, onComplete?: Function, onAbort?: Function) {
    this.transitionTo(
      location,
      route => {
        this.stack = this.stack.slice(0, this.index).concat(route)
        onComplete && onComplete(route)
      },
      onAbort
    )
  }

  go (n: number) {
    const targetIndex = this.index + n
    if (targetIndex < 0 || targetIndex >= this.stack.length) {
      return
    }
    const route = this.stack[targetIndex]
    this.confirmTransition(
      route,
      () => {
        const prev = this.current
        this.index = targetIndex
        this.updateRoute(route)
        this.router.afterHooks.forEach(hook => {
          hook && hook(route, prev)
        })
      },
      err => {
        if (isNavigationFailure(err, NavigationFailureType.duplicated)) {
          this.index = targetIndex
        }
      }
    )
  }

  getCurrentLocation () {
    const current = this.stack[this.stack.length - 1]
    return current ? current.fullPath : '/'
  }

  ensureURL () {
    // noop
  }
}

```

###### go的处理
不同于Html5History HashHistory可以直接使用window.history.go来实现go这个方法，AbstractHistory必须自己来实现go方法。不过看go的代码，不太明白一点：为什么不直接用transitionTo这个统一的路由跳转入口，而是要使用更加底层的confirmTranstion方法。

这行代码:
``` javascript
if (isExtendedError(NavigationDuplicated, err)) {
  this.index = targetIndex
}
```
找到这行代码的添加的原因了： [bug fix](https://github.com/vuejs/vue-router/pull/2771)、[issue](https://github.com/vuejs/vue-router/issues/2607)

> This PR provides a fix to the issue #2607.
Reproduction link
https://jsfiddle.net/Rocka/bhu5vkn8/3/
In abstract mode, if you try to navigate through your history with the same routes appearing consecutively in your stack history, the history index remains unchanged.
In History or Hash mode, this issue does not appear because history navigation is handled by the browser and not by vue-router. In all modes, we do not want to re-render a page if the current route and the next route are identical. However, in the case of abstract mode, we still need to update the history index whenever the current and next routes are the same since AbstractHistory handles its own history stack.

go里面的处理有一点是对的，就是不改变stack数组的内容，只调整访问的指针index。这是与浏览器历史记录管理的特点相符的.

###### replace处理
replace方法看起来对于历史记录的管理，跟浏览器的管理方式不同，就是它会删除掉当前记录之后的记录，而不是仅仅替换当前记录，这一点在注释中有说明。