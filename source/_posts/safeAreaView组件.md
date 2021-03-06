---
toc: true
title: safeAreaView组件
date: 2019-09-04 21:56:12
tags:
- ReactNative
category: 
- 文档教程
---

#SafeAreaView 组件

`SafeAreaView` 的目的是在一个“安全”的可视区域内渲染内容。具体来说就是因为目前有 iPhone X 这样的带有“刘海”的全面屏设备，所以需要避免内容渲染到不可见的“刘海”范围内。本组件目前仅支持 iOS 设备以及 iOS 11 或更高版本。

`SafeAreaView` 会自动根据系统的各种导航栏、工具栏等预留出空间来渲染内部内容。更重要的是，它还会考虑到设备屏幕的局限，比如屏幕四周的圆角或是顶部中间不可显示的“刘海”区域。

##### 例子

只需简单地把你原有的视图用`SafeAreaView`包起来，同时设置一个`flex: 1`的样式。当然可能还需要一些和你的设计相匹配的背景色。

```
<SafeAreaView style={{flex: 1, backgroundColor: '#fff'}}>
  <View style={{ flex: 1 }}>
    <Text>Hello World!</Text>
  </View>
</SafeAreaView>
```

### 属性

#### emulateUnlessSupported

TYPE|REQUIRED|DEFAULT|
--|:--:|:--:|
bool|No|true|

