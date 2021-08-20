---
toc: true
title: ELM 编程语言
date: 2021-08-20 10:11:51
pic: https://greenhaha.oss-cn-beijing.aliyuncs.com/frontend/assets/img/elmlogo.png
tags:
- 函数式编程
- ELM
- JavaScript
category: 
- 文档教程
---

在一个晴朗的午后，我正在拿我的专属神器‘保温杯’在快乐的划水摸鱼中，此时一个惊天阴谋将发生。项目经理面带微笑的走过来说：接下来开发会需要用到 ‘ELM’，你好好看看学一学！说完就离开了，留下了一脸楞逼的我。 ‘ELM’是啥？ELM能干什么？ELM应该怎么干？ 素质三连问？----- 中二病又犯了_(:3 LZ)_

## ELM 是什么？
对于第一次接触的小伙伴来说估计也是一脸懵逼中，‘ELM’？==》‘element’ ==》‘饿了么’！机智如我{% blur 智障如我 %}。不用学，不久‘饿了么’，简单小菜一碟。然而google一下之后发现事情远远没有那么简单。
> Elm是一个领域特定编程语言，用于声明式的创建基于web浏览器的图形用户界面。Elm是纯函数式的，开发它时强调了易用性、性能和健壮性。它宣传为“实际上没有运行时间异常”，Elm编译器的静态类型检查使之成为可能。   -----维基百科

一个领域特定编程语言？纯函数式？runningtime no error？静态类型检查？看介绍好高大上。机智的我搜索打开[ELM官网](https://elm-lang.org/)。官网中第一句话就是：A delightful language
for reliable web applications.（对于开发web应用程序这是一个令开发者兴奋的语言 --- 抱歉中文是我翻译的，然后之后我会对delightful这个词有一个新的理解 无论精神上还是肉体上）

 ![elm 首页](https://greenhaha.oss-cn-beijing.aliyuncs.com/frontend/assets/img/elmindex.png)

那么回到正题：ELM是什么？
官网给出了回答：ELM是一个可以编译成JavaScript的纯函数语言。其实说直白一点就是他也算是前端语言吧，不过他需要自己的解释器将对应的语法翻译成JavaScript最终在浏览器上执行。并且redux的产生于设计思想其实是借鉴ELM。
EML和JavaScript不同，虽然最终解释运行在浏览器上，但是开发的语法却和JavaScript不同。ELM的编译器使用的Haskell语法，你可以把它理解成是一个纯函数语法，和JavaScript有点相似但是与JavaScript又完全不同。之后会说到的。他有自己的运行环境：elm-live 或者是elm-go。是实施提供了elm的热更功能。不要问我为啥知道，这就是我在上面提到的 ‘delightful’的原因

## ELM能干什么
这个是重点：ELM有自己的语法，因为他是一门语言并且最终解释称JavaScript并在浏览器上运行。因此ELM有自己的数据更新机制，这套机制和redux相似：
![elm 更新机制](https://greenhaha.oss-cn-beijing.aliyuncs.com/frontend/assets/img/buttons.svg)

ELM生成的HTML可以在屏幕上显示，而这种运行其实可以分为三个部分
- Model  --- 可以理解为应用的状态（打比方的话可以认为是redux的全局state）
- View   --- 可以认为是视图层，把state放知道html上
- Update --- 用于更新应用的状态的（打比方的话可以认为是reducer）

其实仔细一品的话其实就会发现，这特喵的不就和react 一个组件的内部更新一样吗。（原谅我不厚道的笑出了猪声）

## ELM应该怎么干？
一个就简单的例子（hasekell语法需要自己学习）：
``` javascript
import Browser
import Html exposing (Html, button, div, text)
import Html.Events exposing (onClick)

-- MAIN


main =
  Browser.sandbox { init = init, update = update, view = view }

-- MODEL

type alias Model = Int

init : Model
init =
  0

-- UPDATE

type Msg = Increment | Decrement

update : Msg -> Model -> Model
update msg model =
  case msg of
    Increment ->
      model + 1

    Decrement ->
      model - 1

-- VIEW

view : Model -> Html Msg
view model =
  div []
    [ button [ onClick Decrement ] [ text "-" ]
    , div [] [ text (String.fromInt model) ]
    , button [ onClick Increment ] [ text "+" ]
    ]
```
### Main
`Main`函数对于ELM来说是特别的，它是用来在屏幕上描述显示HMLT的，你可以理解为它是一个沙盒。在程序init值初始化后，`view`函数会把view中的所有内容描述在屏幕上。

### Model
state在ELM中时极其重要的存在，Model存在的重点是捕获收集当前应用的所有详细信息数据，而在上面的例子中则是当作一个计数器来使用
``` javascript
type alias Model = init
```
由于ELM是静态类型检验，所有在定义类型后需要给予初始值
``` javascript
init : Model
init = 0
```

### View
画面是如何呈现在屏幕上，这是则需要`View`函数。
上面的例子中：
``` javascript
view : Model -> Html Msg
view model =
  div []
    [ button [ onClick Decrement ] [ text "-" ]
    , div [] [ text (String.fromInt model) ]
    , button [ onClick Increment ] [ text "+" ]
    ]
```
他接受一个Model作为参数 并且输出HTML信息。（在ELM中静态类型校验是无处不在的）

### Update
`update`函数则是描述了Model中state的值如何更新
更具例子：
``` javascript
type Msg = Increment | Decrement
```
当收到触发的Msg的时候就会在更新函数中对Model中state进行更新
``` javascript
update : Msg -> Model -> Model
update msg model =
  case msg of
    Increment ->
      model + 1

    Decrement ->
      model - 1
```

上面的例子就是一个最基础的ELM的demo。通过点击按钮更新页面上的内容。
运行机制的话如下图所示：
![elm 数据流](https://greenhaha.oss-cn-beijing.aliyuncs.com/frontend/assets/img/elm%E6%95%B0%E6%8D%AE%E6%B5%81.png)