---
toc: true
title: TS 类型中的 any、void 和 never
date: 2021-08-25 10:11:51
pic: https://greenhaha.oss-cn-beijing.aliyuncs.com/frontend/assets/img/images.png
tags:
- 函数式编程
- ELM
- JavaScript
category: 
- 文档教程
---

在ts的类型中常见的类型`number`, `string`, `boolean`等之外, 也有其他不直观的类型表达`any` `void` `never`

## any 类型
在一些情况下，如果我们无法确定变量的类型时（或者无需确认类型时），我们可以将其指定为 any 类型。
``` javascript
function isNumber(value: any) {
    return typeof value === 'number';
}
```
实际上，TS中对于被标记为 `any` 类型的变量，是没有进行类型检查而直接通过编译阶段的检查。
> We want to opt-out of type checking and let the values pass through compile-time checks. To do so, we label these with the any type.

就个人来说，使用ts 的目的是为了提高并保证系统的健壮性，而`any`类型却没有进行类型的检查直接通过编译阶段对系统的稳定性来说有一定影响

### any 类型的特点
* 允许赋值为任意类型
``` javascript
let value: any = 'seven';
value = 7;
```
* 可以访问任意属性和方法
``` javascript
let value: any = 'hello';

// 可以访问任意属性
console.log(value.name);
console.log(value.name.firstName);

// 可以调用任意方法
value.setName('Jerry');
value.setName('Jerry').sayHello();
value.name.setFirstName('Cat');

```
当变量为任意值之后，对它的任何操作返回的内容的类型都是任意值。很容易让某一块代码变得难以维护，丧失了静态类型检查阶段发现错误的可能性

### 变量如果在声明的时候，未指定其类型，那么它会被识别为任意值类型
``` javascript
let value;
value = 'seven';
value = 7;
```
未声明类型的变量虽然一开始被识别为 any 类型，但是经过赋值后，TS 会根据赋值类型来标识变量的类型

``` javascript
let value;

value = 'seven';
const diff1 = value - 1;
// 类型检查错误： The left-hand side of an arithmetic operation must be of type 'any', 'number', 'bigint' or an enum type

value = 7;
cosnt diff2 = value - 1;

```

### void 类型
`void` 类型表示没有任何类型
> void is a little like the opposite of any: the absence of having any type at all

没有返回值的函数，其返回值类型为 `void`

``` javascript
function foo(): void {
    console.log("this is foo function")
}
```
声明为`void`类型的变量，只能赋予`undefined`和`null`
``` javascript
let unusable: void = undefined;
```

## never
`never` 类型表示永远不会有值的一种类型。(很抽象是不是)
> The never type represents the type of values that never occur.

* never类型是那些总是会抛出异常或根本就不会有返回值的函数表达式或箭头函数表达式的返回值类型
``` javascript
// 返回never的函数必须存在无法达到的终点

// 因为总是抛出异常，所以 error 将不会有返回值
function error(message: string): never {
    throw new Error(message);
}

// 因为存在死循环，所以 infiniteLoop 将不会有返回值
function infiniteLoop(): never {
    while (true) {
    }
}
```
* 永远不可能存在的情况：
```javascript
const foo = 123;
if (foo !== 123) {
    const bar = foo;    // bar: never
}
```
### never 和 void 的差异
`void` 表示没有任何类型，`never` 表示永远不存在的值的类型。
``` javascript
// 返回never的函数必须存在无法达到的终点
function infiniteLoop(): never {
    while (true) {
    }
}

// 这个函数不能申明其返回值类型是 never
function warnUser(): void {
    console.log("This is my warning message");
}
```