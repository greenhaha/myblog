---
toc: true
title: React 16 新特性 error Boundary
date: 2019-06-18 15:49:15
tags:
- JavaScript
- React
category: 
- 学习笔记
---
# 什么是边界错误
Error boundary：错误边界。它是react16.0版本发布的时候，携带的一个新的特性。在它的中文官方文档中有这样一段描述：

过去，组件内的 JavaScript 错误会导致 React 的内部状态被破坏，并且在下一次渲染时 [产生](https://github.com/facebook/react/issues/4026) [可能无法追踪的](https://github.com/facebook/react/issues/6895) [错误](https://github.com/facebook/react/issues/8579)。这些错误基本上是由较早的其他代码（非 React 组件代码）错误引起的，但 React 并没有提供一种在组件中优雅处理这些错误的方式，也无法从错误中恢复。

根据官方文档的描述，它诞生的目的就是为了在部分UI出现JavaScript错误的时候。不应该导致整个应用的崩溃。

# 特性说明
错误边界本质上时react的一种组件。官方文档是这样定义：

错误边界是一种 React 组件，这种组件 **可以捕获并打印发生在其子组件树任何位置的 JavaScript 错误，并且，它会渲染出备用 UI** ，而不是渲染那些崩溃了的子组件树。错误边界在渲染期间、生命周期方法和整个组件树的构造函数中捕获错误。

然而错误边界并不能监控所有错误

以下错误是错误边界无法捕获的

- 事件处理（[了解更多](https://react.docschina.org/docs/error-boundaries.html#how-about-event-handlers)）
- 异步代码（例如 setTimeout 或 requestAnimationFrame 回调函数）
- 服务端渲染
- 它自身抛出来的错误（并非它的子组件）

如果在class组件中定义了 [static getDerivedStateFromError()](https://react.docschina.org/docs/react-component.html#static-getderivedstatefromerror) 或 [componentDidCatch()](https://react.docschina.org/docs/react-component.html#componentdidcatch) 这两个生命周期方法中的任意一个（或两个）时，那么它就变成一个错误边界。当抛出错误后，可以利用 [static getDerivedStateFromError()](https://react.docschina.org/docs/react-component.html#static-getderivedstatefromerror) 渲染备用UI  同时可以利用 [componentDidCatch()](https://react.docschina.org/docs/react-component.html#componentdidcatch)来打印错误信息。

错误组件：

class ErrorBoundary extends React.Component {

  constructor(props) {

    super(props);

    this.state = { hasError: false };

  }

  static getDerivedStateFromError(error) {

    // Update state so the next render will show the fallback UI.

    return { hasError: true };

  }

  componentDidCatch(error, info) {

    // You can also log the error to an error reporting service

    logErrorToMyService(error, info);

  }

  render() {

    if (this.state.hasError) {

      // You can render any custom fallback UI

      return \&lt;h1\&gt;Something went wrong.\&lt;/h1\&gt;;

    }

    return this.props.children;

  }

}

Error boundary是一个react组件。所以我们可以在常规组件中引入错误组件：

\&lt;ErrorBoundary\&gt;

  \&lt;MyWidget /\&gt;

\&lt;/ErrorBoundary\&gt;



错误边界只能捕获子组件发生的错误。如果错误无法被错误边界捕获，错误会冒泡至最近的上层组件。
# 工程demo地址
[工程demo](https://github.com/greenhaha/reactDemo1)；