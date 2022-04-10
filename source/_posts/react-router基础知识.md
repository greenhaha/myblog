---
toc: true
title: react-router基础知识
date: 2021-6-30 20:34:00
tags:  
- 浏览器 
- react
- react-router
- JavaScript
category: 
- 知识剖析
---

# react-router
![router history 核心流程](https://www.zoo.team/images/upload/upload_fc4d71102131b3c8094572ecd4641111.jpg)
## router和route
react-router主要是用于Url跳转，其核心概念即为Router和Route。

Router相当于一个容器，用于包裹Route，一个Route即为一个Url。Route里面会包裹一个组件。当在浏览器里输入Url时，就会跳转到相应的Route并显示相关组件。<br>
一个Router里面可以包含多个Route的伪代码为：
```
<Router>
  <div>
      <Route />
      <Route />
      <Route />
  </div
<Router>
```
> Router要求子组件只能有一个。

Router和History有三个种类型：
* HashHistory和HashRouter
* BrowerHistory和BrowerRouter
* createMemoryHistory和MemoryRouter

BrowerHistory和HashHistory主要区别在于Url格式：

使用hashHistory,浏览器的url是这样的：/#/user/liuna?_k=adseis
使用browserHistory,浏览器的url是这样的：/user/liuna

这样看起来当然是browerHistory更好一些，但是它需要server端支持。

使用hashHistory时，因为有 # 的存在，浏览器不会发送request，react-router 自己根据 url 去 render 相应的模块。



* hashroter
一个`<Router>`使用该URL（即哈希部分window.location.hash），以确保URL与页面UI保持同步。
并且官网推荐使用`<BrowserHistory>`

> IMPORTANT NOTE: Hash history does not support location.key or location.state. In previous versions we attempted to shim the behavior but there were edge-cases we couldn’t solve. Any code or plugin that needs this behavior won’t work. As this technique is only intended to support legacy browsers, we encourage you to configure your server to work with <BrowserHistory> instead.

eg:
```js
<HashRouter
  basename={optionalString}
  getUserConfirmation={optionalFunc}
  hashType={optionalString}
>
  <App />
</HashRouter>

```

### 为什么react-router的官方文档推荐使用browserHistory？
首先 browserHistory 其实使用的是 HTML5 的 History API，浏览器提供相应的接口来修改浏览器的历史记录；而 hashHistory 是通过改变地址后面的 hash 来改变浏览器的历史记录；

History API 提供了 pushState() 和 replaceState() 方法来增加或替换历史记录。而 hash 没有相应的方法，hash 仅仅是通过使用hashchange方法来监听hash的变化，所以并没有替换历史记录的功能。

另一个原因是 hash 部分并不会被浏览器发送到服务端，也就是说不管是请求 http://domain.com/index.html#foo 还是 http://domain.com/index.html#bar ，服务只知道请求了 index.html 并不知道 hash 部分的细节。而 History API 需要服务端支持，这样服务端能获取请求细节。

*BrowserRouter
基于H5 History接口的路由
eg
``` jsx
<BrowserRouter
  basename={optionalString}    
  forceRefresh={optionalBool}   
  getUserConfirmation={optionalFunc}
  keyLength={optionalNumber}  
>
  <App />
</BrowserRouter>
```

* Link
声明式路由组件
``` jsx
<Link to="/about">About</Link>
```
* to 导向哪个路由，可以为字符串，也可以为一个对象或者函数
``` jsx
<Link to="/courses?sort=name" />

<Link
   to={{
     pathname: "/courses",    // 路径
     search: "?sort=name",  // 查询参数
     hash: "#the-hash",   // hash值
     state: { fromDashboard: true }  // 持久化到location的状态数据
   }}
 />
```
一个函数，当前位置作为参数传递给它，并且应该以字符串或对象的形式返回位置表示
```jsx
<Link to={location => ({ ...location, pathname: "/courses" })} />

<Link to={location => `${location.pathname}?sort=name`} />
```
其他可用属性

* replace: 当为true时，单击该链接将替换历史堆栈中的当前条目，而不是添加一个新条目。
* innerRef 值为函数
```jsx
<Link
  to="/"
  innerRef={node => {
    // node指向挂载的dom元素, 卸载时候为null
  }}
/>
```
值为Ref对象
```jsx
let anchorRef = React.createRef()

<Link to="/" innerRef={anchorRef} />
```
* component 定制化自己的导航组件
```jsx
const FancyLink = React.forwardRef((props, ref) => (
  <a ref={ref} {...props}>  {props.children}</a>
))

<Link to="/" component={FancyLink} />

```

* NavLink
是 Link 的一个特殊版本，当呈现的元素与当前URL匹配时，它将向该元素添加样式属性。

activeClassName 当元素处于active状态时，类将提供该class。默认的给定class是active。这将与className样式叠加

activeStyle 内嵌方式声明active状态样式

exact 布尔类型， 为true是路径完全匹配才会添加active class

strict 路径匹配是否严格， 为true的话结尾的斜杠会被考虑

isActive函数， 可以自定义active class添加逻辑

```jsx
<NavLink
  to="/events/123"
  isActive={(match, location) => {
    if (!match) {
      return false;
    }

    // only consider an event active if its event id is an odd number
    const eventID = parseInt(match.params.eventID);
    return !isNaN(eventID) && eventID % 2 === 1;
  }}
>
  Event 123
</NavLink>
```

* Redirect
重定向
```jsx
<Route exact path="/">
  {loggedIn ? <Redirect to="/dashboard" /> : <PublicHomePage />}
</Route>
```
to也可以为对象
```jsx
<Redirect
  to={{
    pathname: "/login",
    search: "?utm=your+face",
    state: { referrer: currentLocation }
  }}
/>
```
push属性: 当为真时，重定向将把一个新的条目推送到历史中，而不是取代当前的条目。 from属性: 要重定向的路径名。路径-regexp@^1.7.0能够理解的任何有效URL路径。在to中为模式提供了所有匹配的URL参数。必须包含to中使用的所有参数。不被to使用的其他参数将被忽略。
```jsx
<Switch>
  <Redirect from="/old-path" to="/new-path" />
  <Route path="/new-path">
    <Place />
  </Route>
</Switch>

// Redirect with matched parameters
<Switch>
  <Redirect from="/users/:id" to="/users/profile/:id" />
  <Route path="/users/profile/:id">
    <Profile />
  </Route>
</Switch>
```
exact属性， 路径是否完全匹配
strict属性： 路径匹配是否严格，区分斜杠
sensitive属性: 路径匹配是否大小写敏感

* route
path（string）: 路由匹配路径。（没有path属性的Route 总是会 匹配）；
exact（bool）：为true时，则要求路径与location.pathname必须完全匹配；
strict（bool）：true的时候，有结尾斜线的路径只能匹配有斜线的location.pathname；

``` js
<Router>
  <div>
    <Route exact path="/" component={Home}/>
    <Route path="/news" component={NewsFeed}/>
  </div>
</Router>

// 如果应用的地址是/,那么相应的UI会类似这个样子：
<div>
  <Home/>
</div>

// 如果应用的地址是/news,那么相应的UI就会成为这个样子：
<div>
  <NewsFeed/>
</div>
```

* Switch
* 使用场景: 只要有一个 `path`匹配上了对应的组件, 后续就不会再进行匹配了
* 我们来看下面的路由规则：
  * 当我们匹配到某一个路径时，我们会发现有一些问题
  * 比如 /about 路径匹配到的同时，/:userid也被匹配到了，并且最后的一个NoMatch组件总是被匹配到
![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/741d953942e14e3787c7353fbfa79f70~tplv-k3u1fbpfcp-zoom-1.image)

* 原因是什么呢？默认情况下，react-router中只要是路径被匹配到的Route对应的组件「都会被渲染」

  * 但是实际开发中, 我们希望有一种排他的思想
  * 只要匹配到了第一个, 后面就不应该继续匹配了
  * 这个时候我们可以使用Switch来将所有Route组件进行包裹
![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/5ed2ff8df0a4438c9de616e3e34f1126~tplv-k3u1fbpfcp-zoom-1.image)