---
toc: true
title: React浅比较
date: 2018-11-21 22:10:35
tags:
- React
- JavaScript
category: 
- 知识剖析
---

# 什么是浅比较

在react 15.3中更新了一个新的顶层api： [React.PureComponent](https://zh-hans.reactjs.org/docs/react-api.html#reactpurecomponent)(浅比较)。根据官网的介绍，React.PureComponent 与 React.Component 很相似。然而两者之间的区别在于React.Component并未实现shouldComponentUpdate()。而React.PureComponent中则以浅层对比prop和state的方式实现了该函数。在渲染含有相同的props和state的react组件，在某些情况下使用PureComponent可提高性能。在官网还给开发者进行了提醒：

React.PureComponent 中的 shouldComponentUpdate() 仅作对象的浅层比较。如果对象中包含复杂的数据结构，则有可能因为无法检查深层的差别，产生错误的比对结果。仅在你的 props 和 state 较为简单时，才使用 React.PureComponent，或者在深层数据结构发生变化时调用 [forceUpdate()](https://zh-hans.reactjs.org/docs/react-component.html#forceupdate) 来确保组件被正确地更新。你也可以考虑使用 [immutable 对象](https://facebook.github.io/immutable-js/)加速嵌套数据的比较。

此外，React.PureComponent 中的 shouldComponentUpdate() 将跳过所有子组件树的 prop 更新。因此，请确保所有子组件也都是'纯'的组件。

# 为什么用PureComponent?

Purecomponent 是优化react程序最重要方法之一。相对于component，Purecomponent可以减少不必要的render操作次数，从而提高性能，减少shouldComponentUpdate 函数。并且只需要把继承了从component 替换成purecomponent 即可。

# 浅比较的原理

当组件更新时，如果组件的 props 和state 都没发生改变， render 方法就不会触发，省去 Virtual DOM 的生成和比对过程，达到提升性能的目的。

具体就是由于PureComponent的shouldeComponentUpdate里，实际是对props/state进行了一个  **_浅对比，所以对于嵌套的对象不适用，没办法比较出来。_**

浅比较实际通过shallowEqual来完成

function isPureComponent(Component) {

  return !!(Component.prototype &amp;&amp; Component.prototype.isPureReactComponent);

}

当识别到使用purecomponent 时，就会调用shallowEqual  来实现状态的更新。而shallowEqual则是shouldComponentUpdate()核心函数。通过shallowEqual来实现状态的比较和更新。

shallowEqual函数完成的功能：

1. 通过is函数对两个参数进行比较，判断是否相同，相同直接返回true：基本数据类型值相同，同一个引用对象都表示相同
2. 如果两个参数不相同，判断两个参数是否至少有一个不是引用类型，存在即返回false，如果两个都是引用类型对象，则继续下面的比较；
3. 判断两个不同引用类型对象是否相同
  1. 1先通过keys获取到两个对象的所有属性，具有相同属性，且每个属性值相同即两个对相同（相同也通过is函数完成）        其中is函数是自己实现的一个[is](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Object/is)的功能，排除了===两种不符合预期的情况：

         +0 === -0  // true

         NaN === NaN // false

/\*\*

 \* Performs equality by iterating through keys on an object and returning false

 \* when any key has values which are not strictly equal between the arguments.

 \* Returns true when the values of all keys are strictly equal.

 \*/

function shallowEqual(objA: mixed, objB: mixed): boolean {

  if (is(objA, objB)) {

    return true;

  }

  if (

    typeof objA !== &#39;object&#39; ||

    objA === null ||

    typeof objB !== &#39;object&#39; ||

    objB === null

  ) {

    return false;

  }

  const keysA = Object.keys(objA);

  const keysB = Object.keys(objB);

  if (keysA.length !== keysB.length) {

    return false;

  }

  // Test for A&#39;s keys different from B.

  for (let i = 0; i \&lt; keysA.length; i++) {

    if (

      !hasOwnProperty.call(objB, keysA[i]) ||

      !is(objA[keysA[i]], objB[keysA[i]])

    ) {

      return false;

    }

  }

  return true;

}

# Purecomponent 使用时机

当对比的类型为Object的时候并且key的长度相等的时候，浅比较也仅仅是用Object.is()对Object的value做了一个基本数据类型的比较，所以如果key里面是对象的话，有可能出现比较不符合预期的情况，所以浅比较是不适用于嵌套类型的比较的。如果所比较的状态是有深层次嵌套，那么purecomponent就不适合你。

注：参考了几篇大佬文章，在根据开发中遇到的问题总结出这篇文章
