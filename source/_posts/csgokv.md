---
title: CSGO模型 kv模型配置参数解
date: 2019-06-27 19:24:32
tags:
---

使用自定义的人物模型需要写kv文件，放在maps文件夹下，名称为【地图名.kv】.

下面介绍一下各参数作用：

de\_ruby   //地图名字
{
name          de\_ruby   //地图名字
t\_arms        models/weapons/t\_arms\_anarchist.mdl   //T手模，此项无效
ct\_arms       models/weapons/ct\_arms\_sas.mdl    //CT手模，此项无效

t\_models   //T模型，可以填写多个，将随机分配
{
models/player/custom\_player/voikanaa/rayman.mdl 
}

ct\_models   //CT模型，同T模型一样
{
ctm\_sas
ctm\_sas\_variantA
ctm\_sas\_variantB
ctm\_sas\_variantC
ctm\_sas\_variantD
ctm\_sas\_variantE
}
}
