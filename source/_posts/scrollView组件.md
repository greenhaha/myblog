---
toc: true
title: scrollView组件
date: 2019-09-04 21:55:32
tags:
---

# scrollView 组件

scrollview为reactNative 中的滚动视图组件， 同时还集成了触摸锁定的“响应者”系统
scrollView必须有一个确定的高度才能工作。换句话说就是他的内部必须要有其他的视图容器进行支撑。它本身没有高度。通常scrollview设置`flex: 1`时期自动填充父容器的空余空间。如果所有的父容器本身没有设置flex或者指定固定高度，那么就会导致无法正常滚动。
scrollView 会简单粗暴把所有元素一次性渲染出来。同时也带来了性能上的问题。而FlatList组件会惰性渲染子元素。只在他们将要出现在屏幕中的时候开始渲染。所有这种惰性渲染复杂很多API在使用上也更加的繁琐。

## 属性

#### contentContainerStyle
这些样式会应用到一个内层的内容容器上，所有的子视图都会包裹在内容容器内。
例子：
```
return (
  <ScrollView contentContainerStyle={styles.contentContainer}>
  </ScrollView>
);
...
const styles = StyleSheet.create({
  contentContainer: {
    paddingVertical: 20
  }
});
```
类型|必填|
--|:--:|
StyleSheetPropType(View Style props)|否|

#### keyboardDismissMode
用户拖拽滚动视图的时候，是否要隐藏软键盘。
跨平台可用的值
- `'none'` （默认值），拖拽时不隐藏软键盘。
- `'on-drag'` 当拖拽开始的时候隐藏软键盘。

仅iOS可用值
- `'interactive'`，软键盘伴随拖拽操作同步地消失，并且如果往上滑动会恢复键盘。安卓设备上不支持这个选项，会表现的和`none`一样。

类型|必填|
--|:--:|
enum('none', 'on-drag', 'interactive')|否|

#### keyboardShouldPersistTaps

如果当前界面有软键盘，那么点击scrollview后是否收起键盘，取决于本属性的设置。（译注：很多人反应TextInput无法自动失去焦点/需要点击多次切换到其他组件等等问题，其关键都是需要将TextInput放到ScrollView中再设置本属性）

- `'never'`（默认值），点击TextInput以外的子组件会使当前的软键盘收起。此时子元素不会收到点击事件
- `'always'`键盘不会自动收起，ScrollView也不会捕捉点击事件，但子组件可以捕获。
- `'handled'`当点击事件被子组件捕获时，键盘不会自动收起。这样切换TextInput时键盘可以保持状态。多数带有TextInput的情况下你应该选择此项。
- `false` 已过期，请使用'never'代替。
- `true` 已过期，请使用'always'代替。

类型|必填|
--|:--:|
enum('always', 'never', 'handled', false, true)|否|

#### onContentSizeChange
此函数会在ScrollView内部可滚动内容的视图发生变化时调用。
调用参数为内容视图的宽和高:`(contentWidth, contentHeight)`.
此方法是通过绑定在内容容器上的onLayout来实现的。

类型|必填|
--|:--:|
function|否|

#### onMomentumScrollBegin
滚动动画开始时调用此函数。

类型|必填|
--|:--:|
function|否|

#### onMomentumScrollEnd
滚动动画结束时调用此函数。

类型|必填|
--|:--:|
function|否|

#### onScroll
在滚动的过程中，每帧最多调用一次此回调函数。调用的频率可以用scrollEventThrottle属性来控制。The event has the shape `{ nativeEvent: { contentInset: { bottom, left, right, top }, contentOffset: { x, y }, contentSize: { height, width }, layoutMeasurement: { height, width }, zoomScale } }` . All values are numbers.

类型|必填|
--|:--:|
function|否|

#### onScrollBeginDrag

当用户开始拖动此视图时调用此函数。

类型|必填|
--|:--:|
function|否|

#### refreshControl
指定RefreshControl组件，用于为ScrollView提供下拉刷新功能。只能用于垂直视图，即horizontal不能为 `true`.

类型|必填|
--|:--:|
element|否|

#### showsHorizontalScrollIndicator

当此属性为true的时候，显示一个水平方向的滚动条。

类型|必填|
--|:--:|
bool|否|

#### showsVerticalScrollIndicator

当此属性为true的时候，显示一个垂直方向的滚动条。

类型|必填|
--|:--:|
bool|否|

#### stickyHeaderIndices

一个子视图下标的数组，用于决定哪些成员会在滚动之后固定在屏幕顶端。举个例子，传递`stickyHeaderIndices={[0]}`会让第一个成员固定在滚动视图顶端。这个属性不能和`horizontal={true}`一起使用。

类型|必填|
--|:--:|
array of number|否|

#### horizontal

当此属性为true的时候，所有的子视图会在水平方向上排成一行，而不是默认的在垂直方向上排成一列。默认值为false。

类型|必填|
--|:--:|
bool|否|

注：还有其他相关使用属性。以上内容整理自官网
    官网链接：[https://reactnative.cn/docs/view.html](https://reactnative.cn/docs/view.html)