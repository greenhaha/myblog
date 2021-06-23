---
title: button组件
date: 2019-09-04 21:57:22
tags:
---

# Button组件

一个简单的跨平台按钮组件。可以进行一些简单定制。

![buttonImage](https://reactnative.cn/docs/assets/buttonExample.png)

在ReactNative中组件样式是固定的。如果有定制化外观需求的时候，可以使用button组件提供的接口来定制化样式

例子：
```
    import { Button } from 'react-native';
    <Button
    onPress={onPressLearnMore}
    title="Learn More"
    color="#841584"
    accessibilityLabel="Learn more about this purple button"
    />
```

## 属性

#### onPress

用户点击此按钮时所调用的处理函数（类似 div中 click方法）

类型|必填|
--|:--:|
function|否|

#### title

按钮内显示的文本

类型|必填|
--|:--:|
string|否|

#### color

文本的颜色(iOS)，或是按钮的背景色(Android)

类型|必填|
--|:--:|
color|否|

#### disabled

设置为 true 时此按钮将不可点击。

类型|必填|
--|:--:|
bool|否|

注：还有其他相关使用属性。以上内容整理自官网
    官网链接：[https://reactnative.cn/docs/view.html](https://reactnative.cn/docs/view.html)