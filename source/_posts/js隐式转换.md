---
toc: true
title: JavaScript隐式类型转换
date: 2021-08-26 15:47:56
pic: https://greenhaha.oss-cn-beijing.aliyuncs.com/frontend/assets/img/web-design-2906159_960_720.jpg
tags:
- JavaScript
category: 
- 知识剖析
---

![JavaScript隐式类型转换](https://greenhaha.oss-cn-beijing.aliyuncs.com/frontend/assets/img/how-it-fucking-work.png)
javascript 在发生隐式类型转换的时候会有各种个样的问题,即使有很多经验的开发者有时候也不一定能理清楚具体规律是什么。
{% blur 其实写这篇文章的主要原因是面试时候问到这个，而我却满脸❓ %}。因此我想要好好整理这一块

## 数学运算符中的类型转换
因为 JS 并没有类型声明，所以任意两个变量或字面量，都可以做加减乘除。

### 减、乘、除
非`Number`类型运用数学运算符`(- * /)`时，会先将非`Number`类型转换为`Number`类型。
``` javascript
1 - true // 0， 首先把 true 转换为数字 1， 然后执行 1 - 1
1 - null // 1,  首先把 null 转换为数字 0， 然后执行 1 - 0
1 * undefined //  NaN, undefined 转换为数字是 NaN
2 * ['5'] //  10， ['5']首先会变成 '5', 然后再变成数字 5
```
### 加法的特殊性
JS里 `+`还可以用来拼接字符串。谨记以下3条：
* 当一侧为`String`类型，被识别为字符串拼接，并会优先将另一侧转换为字符串类型。
* 当一侧为`Number`类型，另一侧为原始类型，则将原始类型转换为`Number`类型。
* 当一侧为`Number`类型，另一侧为引用类型，将引用类型和`Number`类型转换成字符串后拼接。

优先级从高到低，即 `3+'abc'` 会应用规则 1，而 `3+true`会应用规则2。

``` javascript
123 + '123' // 123123   （规则1）
123 + null  // 123    （规则2）
123 + true // 124    （规则2）
123 + {}  // 123[object Object]    （规则3）
```
## 逻辑语句中的类型转换
使用 `if while for` 语句时，我们期望表达式是一个`Boolean`，所以一定伴随着隐式类型转换。而这里面又分为两种情况:

### 单个变量
只有单个变量，会先将变量转换为`Boolean`值。

我们可以参考附录的转换表来判断各种类型转变为`Boolean`后的值。

不过这里有个小技巧：

只有 `null undefined '' NaN 0 false` 这几个是 `false`，其他的情况都是 `true`，比如 `{}` , `[]`。

### 使用 == 比较中的5条规则
可以严格使用 ===，不过了解==的习性还是很有必要的
根据 == 两侧的数据类型，我们总结出 5 条规则：
* 规则 1：NaN和其他任何类型比较永远返回false（包括和他自己）
``` javascript
NaN == NaN // false
```
* 规则 2：Boolean 和其他任何类型比较，Boolean 首先被转换为 Number 类型。
``` javascript
true == 1  // true 
true == '2'  // false, 先把 true 变成 1，而不是把 '2' 变成 true
true == ['1']  // true, 先把 true 变成 1， ['1']拆箱成 '1', 再参考规则3
true == ['2']  // false, 同上
undefined == false // false ，首先 false 变成 0，然后参考规则4
null == false // false，同上
```
* 规则 3：`String`和`Number`比较，先将`String`转换为`Number`类型

``` javascript
123 == '123' // true, '123' 会先变成 123
'' == 0 // true, '' 会首先变成 0
```
* 规则 4：null == undefined比较结果是true，除此之外，null、undefined和其他任何结果的比较值都为false。
``` javascript
null == undefined // true
null == '' // false
null == 0 // false
null == false // false
undefined == '' // false
undefined == 0 // false
undefined == false // false
```
* 规则 5：原始类型和引用类型做比较时，引用类型会依照`ToPrimitive`规则转换为原始类型。
> `ToPrimitive`规则，是引用类型向原始类型转变的规则，它遵循先`valueOf`后`toString`的模式期望得到一个原始类型。
如果还是没法得到一个原始类型，就会抛出 `TypeError`。
``` javascript 
'[object Object]' == {} 
// true, 对象和字符串比较，对象通过 toString 得到一个基本类型值
'1,2,3' == [1, 2, 3] 
// true, 同上  [1, 2, 3]通过 toString 得到一个基本类型值
```

附录：
![附录](https://greenhaha.oss-cn-beijing.aliyuncs.com/frontend/assets/img/convert-table.png)
