---
title: view组件
date: 2019-09-04 21:54:48
tags:
---

# React View 组件

View作为创建ui时最基础的组件，功能上类似于html中的div 标签。但是在reactNative中，View是一个支持flexbox布局、样式、一些触摸处理、和一些无障碍功能（为残疾人士提供的功能）的容器。并且它支持嵌套，可以和多个任意的类型子视图容器搭配使用。无论在什么平台中，view都会对应一个原生视图。无论是uiview、div还是android.viwe.View。

>在官方文档中，提示开发者：
>View的设计初衷是和styleSheet 搭配使用，这样可以使用代码更清晰并且获得更高的性能。尽管内联样式也同样可以使用。

## 相关属性

### 合成触摸事件

用于View相应属性（例如： omResponderMove），合成触摸事件采用一下的格式：

- nativeEvent
    - changedTouches - 从上一次事件以来的触摸事件数组
    - identifier - 触摸事件的id
    - locationX - 触摸事件相对元素位置的X坐标
    - locationY - 触摸事件相对元素位置的Y坐标
    - pageX - 触摸事件相对根元素位置的X坐标
    - pageY - 触摸事件相对根元素位置的Y坐标
    - target - 接收触摸事件的元素ID
    - timestamp - 触摸事件的时间标记
    - touches - 屏幕上所有当前触摸事件的数组

### 属性

#### hitSlop

定义触摸事件在距离视图多远以内时可以触发的。

> 触摸范围不会扩展到父视图之外，另外如果触摸到两个重叠的视图， Z-index高的元素会优先。

类型|必填|
--|:--:|
object: {top: number, left: number, bottom: number, right: number}|否|

#### onMagicTap

当 accessible 为 true 时，如果用户做了一个双指轻触(Magic tap)手势，系统会调用此函数。

类型|必填|
--|:--:|
function|否|

#### onMoveShouldSetResponderCapture

如果父视图想要阻止子视图响应 touch move 事件时，它就应该设置这个方法并返回 true View.props.onMoveShouldSetResponderCapture: (event) => [true | false], 其中 event 是一个合成触摸事件。

类型|必填|
--|:--:|
function|否|

#### onResponderMove

当用户正在屏幕上移动手指时调用这个函数。

View.props.onResponderMove: (event) => {}, 其中 event 是一个合成触摸事件。

类型|必填|
--|:--:|
function|否|

#### accessible

当此属性为 true 时，表示此视图是一个启用了无障碍功能的元素。默认情况下，所有可触摸操作的元素都是无障碍功能元素。

类型|必填|
--|:--:|
bool|否|

注：还有其他相关使用属性。以上内容整理自官网
    官网链接：[https://reactnative.cn/docs/view.html](https://reactnative.cn/docs/view.html)