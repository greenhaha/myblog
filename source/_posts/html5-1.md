---
title: Html5.1 新属性
date: 2018-12-19 11:35:03
tags:
---


自2014年10月发布html标准制定至今，html的规范一直没有停止。在2016年的11月，W3C 发布了最新的html规范：&#39;html 5.1&#39;。在这个5.1当中，添加了很多新的特性同时html的规范得到重大改进。我们可以使用html来创建更加灵活的网络体验。

## 为响应式设计定义了多个图片资源

在html5.1中，我们可以使用 [&lt;picture&gt;](https://www.w3.org/TR/html/semantics-embedded-content.html#elementdef-picture)标签和srcset属性来使响应式图片选择成为可能。&lt;picture&gt;标签标识图片的容器，允许开发者声明不同的图像资源用来适应UA的视口大小，屏幕像素密度，屏幕类型和在响应式设计中使用的其他参数。

注：如果想要设置图片大小  你需要在img标签设置大小  其他自适应的图片都会按照img设置的大小展示

&lt;picture&gt;

    <source srcset="./image/0.jpeg" media="(max-width:760px)">

    <source srcset="./image/1.jpg" media="(max-width: 1024px)">

    <img src="./image/004.jpg" style="width:300px;height:400px"

 alt="mypicture" srcset="default-pic.jpg">

&lt;/picture&gt;

## 显示或者隐藏额外信息

使用&lt;details&gt;和&lt;summary&gt;标签，可以向内容项添加扩展信息。默认情况下是不显示额外信息。如果用户对额外信息感兴趣可以点击查看。在使用的时候&lt;details&gt;标签需要放到&lt;summary&gt;标签外部。你可以在&lt;summary&gt;标签之后添加要隐藏的额外信息。

&lt;details&gt;

     <summary>这里含有隐藏信息，请点击查看</summary>

             <div&gt;这条信息是一个隐藏信息</div>

                <img

                        src="./image/004.jpg"

                        alt="mypicture"

                        srcset="default-hd.jpg 2x"

                    >

&lt;/details&gt;

## 将功能添加到浏览器的上下文菜单

使用&lt;menuitem&gt;标签和type = "context"属性，可以使自定义的菜单添加到浏览器的上下文菜单中。使用时需要把&lt;menuitem&gt;标签放到&lt;menu&gt;标签中。在使用的时&lt;menu&gt;标签中的id必须要与我们添加的上下文菜单的元素的&lt;contextmenu&gt;属性保持相同。

注：浏览器对这个功能不太友好，firefox支持使用  chrome已经禁用这个功能

本实例仅限firefox 浏览器使用有效

&lt;div contextmenu="rightclickmenu" style="width: 200px; height: 300px; backgroundColor: green"&gt;Right click me&lt;/div&gt;

                <menu type="context" id="rightclickmenu">

                    <menuitem type="checkbox" label="I HTML5.1.">

                   </menu>

## 对样式和脚本使用加密随机数

使用html5.1中 我们可以通过&lt;script&gt;和&lt;style&gt;元素中使用nonce属性添加加密随机数到样式和脚本中。由于随机数是随机生成的并且只能使用一次。当每次请求时，都会重新生成。网站的[安全策略](https://www.cspplayground.com/home)可以使用随机数来决定是否应在网页上使用特定样式。等多内容查看[Google开发者网页基础](https://developers.google.com/web/fundamentals/security/csp/)

&lt;script nonce=&#39;adfjaf8eda64U7&#39;&gt;

            // some JavaScript

            </script>

## 使用零宽度图像

在html5.1中 允许开发人员创建width为0的零宽度图片。如果想要包含不想向用户展示的图像，这个特性很好用。建议零宽度图像与空alt属性一起使用。

&lt;img src="yourpicture.jpg" width="0" height="0" alt=""&gt;

## 分离浏览器上下文防止网络钓鱼攻击

HTML 5.1已经标准化了rel ="noopener"属性的用法，它消除了分隔浏览器上下文的问题，你可以在&lt;a&gt;和&lt;area&gt;元素中使用rel ="noopener"。

注：本特性针对的时[target ="_ blank"漏洞](https://www.jitbit.com/alexblog/256-targetblank---the-most-underestimated-vulnerability-ever/)

&lt;a href="#" target="_blank" rel="noopener"&gt;

                Your link that won&#39;t make troubles

            </a>

## 空选项

HTML 5.1允许开发人员创建一个空的&lt;option&gt;元素。&lt;option&gt;标签可以是&lt;select&gt;，&lt;optgroup&gt;或&lt;datalist&gt;元素的子元素。 如果你不想建议用户应该选择哪个选项，以及在想要设计用户友好的表单时，使用空的&lt;option&gt;可能很有用。

## 图片图形标题

[&lt;figcaption&gt;](https://www.w3.org/TR/html/grouping-content.html#elementdef-figcaption)标签表示[&lt;figure&gt;](https://www.w3.org/TR/html/grouping-content.html#elementdef-figure)元素的题注或说明，其是用于视觉（例如插图，照片和图表）的容器。以前，&lt;figcaption&gt;标记只能用作&lt;figure&gt;元素的第一个或最后一个子元素。HTML 5.1已放松此规则，现在&lt;figcaption&gt;可以出现在其&lt;figure&gt;容器中的任何位置。