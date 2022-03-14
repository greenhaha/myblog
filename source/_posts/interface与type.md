---
toc: true
title: Interface 与 Type的区别
date: 2022-03-14 11:48:43
pic: http://greenhaha.oss-cn-beijing.aliyuncs.com/frontend/assets/img/4_734.png
tags:  
- Node 
- TypeScript
category: 
- 知识剖析
---

## Interface 与Type

在使用typescript的时候经常会使用到interface与type。对于这两者区别在[官方文档](https://github.com/microsoft/TypeScript/blob/main/doc/spec-ARCHIVED.md)中说明了两者区别


> * An interface can be named in an extends or implements clause, but a type alias for an object type literal cannot.<br/>(接口可以在扩展或实现子句中命名，但对象类型文字的类型别名不能)
> * An interface can have multiple merged declarations, but a type alias for an object type literal cannot.(一个接口可以有多个合并的声明，但对象类型文字的类型别名不能。)


## 相同点

简单一句话： <strong>都可以描述一个对象或者函数</strong>

### interface

``` typescript 
interface person {
  name: string
  age: number
}

interface setperson {
  (name: string, age: number): void;
}
```
### type

``` typescript
type person = {
  name: string
  age: number
};

type setperson = (name: string, age: number)=> void;

```

### 都允许拓展（extends）与 交叉类型（Intersection Types）

interface 和 type 都可以拓展，并且两者并不是相互独立的，也就是说 interface 可以 extends type, type 也可以 extends interface 。interface 可以 extends， 但 type 是不允许 extends 和 implement 的，但是 type 却可以通过交叉类型 实现 interface 的 extend 行为，并且两者并不是相互独立的，也就是说 interface 可以 extends type, type 也可以 与 interface 类型 交叉 。 <strong>虽然效果差不多，但是两者语法不同。</strong>


### interface extends interface

interface 继承使用关键字 extends

``` typescript
interface Name { 
  name: string; 
}
interface User extends Name { 
  age: number; 
}
```

### type 与 type 交叉

type 继承使用符号&

``` typescript
type Name = { 
  name: string; 
}
type User = Name & { age: number  };
```

### interface extends type

``` typescript
type Name = { 
  name: string; 
}
interface User extends Name { 
  age: number; 
}
```
### type 与 interface 交叉

``` typescript
interface Name { 
  name: string; 
}
type User = Name & { 
  age: number; 
}
```

## 不同点

###  type 可以而 interface 不行

* type 可以声明基本类型别名，联合类型，元组等类型<br>
``` typescript
// 基本类型别名
type Name = string

// 联合类型
interface Dog {
    wong();
}
interface Cat {
    miao();
}

type Pet = Dog | Cat

// 具体定义数组每个位置的类型
type PetList = [Dog, Pet]
```
* type 语句中还可以使用 typeof 获取实例的 类型进行赋值<br>
``` typescript
// 当你想获取一个变量的类型时，使用 typeof
let div = document.createElement('div');
type B = typeof div
```

* 其他骚操作
``` typescript
type StringOrNumber = string | number;  
type Text = string | { text: string };  
type NameLookup = Dictionary<string, Person>;  
type Callback<T> = (data: T) => void;  
type Pair<T> = [T, T];  
type Coordinates = Pair<number>;  
type Tree<T> = T | { left: Tree<T>, right: Tree<T> };
```

### interface 可以而 type 不行
interface 能够声明合并

``` typescript
interface User {
  name: string
  age: number
}

interface User {
  sex: string
}

/*
User 接口为 {
  name: string
  age: number
  sex: string 
}
*/
```


## 总结
一般来说，如果不清楚什么时候用interface/type，能用 interface 实现，就用 interface , 如果不能就用 type 。其他更多详情参看[官方规范文档](https://github.com/microsoft/TypeScript/blob/main/doc/spec-ARCHIVED.md)