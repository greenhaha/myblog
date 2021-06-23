---
title: REST与RESTful的故事
date: 2019-03-20 11:33:09
tags:
---
# REST是什么？
 
REST英文全称：Representational State Transfer。中文：表现层状态转换。（第一次看到这个词的时候，一脸懵逼中 \_(:3LZ)\_）。它是由美国的[Roy Thomas Fielding](https://zh.wikipedia.org/w/index.php?title=Roy_Thomas_Fielding&amp;action=edit&amp;redlink=1)博士提出的万维网软件架构风格。他的目的就是在不同的软件/程序在网络中传递信息时，提供一组约束和属性。他是一种为万维网络服务设计的软件构件风格。而RESTful就是&quot;URL定位资源，用HTTP动词（GET,POST,DELETE,DETC）描述操作&quot;
 
## RESTful API是什么？
 
其实总结一句话就是 按照REST风格做成的API。（这个真不是废话。 =。=）。由于互联网的应用在前端越来越丰富。电脑，笔记本，手机，平板电脑等越来越丰富。前端展示媒体信息的情况也越来越多。用户在前端发送请求，同意用后台接受处理并返回给不同的前端展示成为开发运营者最科学也是最经济的方式。因此RESTful API 就是一套协议来规范多种形式的前端和同一个后台交互方式。
 
## RESTful API设计原则与规范
 
1. 资源
 
资源时网络上的一个实体，可以是一段文本，一张图片一段视频。资源需要载体来展示内容。文本的载体可以用TXT，HTML等。而资源最常用，用的最广泛的则是json
 
1. 统一接口
 
统一的接口是RESTful系统设计的基本出发点。
 
1. 无状态
 
所有资源都可以用url来定位。而且这个定位也是唯一定位与其他的资源无关。不会影响其他资源的变化。更不会因为其他资源的改变而改变。如果后面的步骤是依靠前一个步骤那么这就是有状态的。如果直接输入url得到结果 那么就是无状态的。
 
1. URL段
 
一个好的url段对后期版本的维护升级是很有帮助的。可以把api版本号放到url中
 
[https://api.example.com/v1/](https://api.example.com/v1/)
 
api的设计并不是一致不变的。需要根据不同情况来进行开发。而restful 提供了一个通用广泛的设计风格。