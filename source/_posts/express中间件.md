---
toc: true
title: express中间件
pic: https://greenhaha.oss-cn-beijing.aliyuncs.com/frontend/assets/img/expresslog.png
date: 2021-08-27 15:35:36
tags:
- Node
- express
- 中间件
category: 
- 知识剖析
---

## 什么是express 中间件？
在express 的官网中有一段对中间件函数的解释：
> Middleware functions are functions that have access to the request object (req), the response object (res), and the next middleware function in the application’s request-response cycle. The next middleware function is commonly denoted by a variable named next.

理解这句话可以看一下下图：
![middleware 流程](https://greenhaha.oss-cn-beijing.aliyuncs.com/frontend/assets/img/middleware.png)
 
可以把express 理解成一个水管，用户的请求内容（request）可以理解为进水口，返回的内容（response）可以理解为出水口。而中间件在其中起到的 可以理解为这个水管的过滤器对请求的内容进行过滤处理之后排出过滤或者处理的结果。

那么对于在日常开发中使用中间的好处的话可以总结3点：
* 逻辑清楚，层次分明。 正如TCP/IP中的分层一样，通过分层，可以使得每一个部分各司其职，更好的干事情。
* 便于维护。如果觉得其中一个做的不好，还可以换一个中间件，而其他的不用替换。
* 可复用。我们写好了一个中间件之后，就可以直接拿来在别的地方用了，就比如，我们在使用第三方中间件的时候，直接npm install somemiddleware，然后 require，最后直接 app.use 即可，非常方便。

正如官网对中间件的解释：Middleware function 中间件的本质就是一个函数，这时候在理解上面的好处的时候实际上就是函数组件化的优点。

在express里中间件可以大致分为4类：
* 应用级中间件 Application-level middleware 
* 路由级中间件 Router-level middleware
* 错误级中间件 Error-handling middleware
* 内置级中间件 Built-in middleware
* 第三方中间件 Third-party middleware


### 应用级中间件（Application-level middleware）
其实应用级中间件可以理解为直接作用于express实例上的，这么说可能不太理解，但是`expresss().ues()`一定用过，其实这个就是应用级中间件
``` javascript
let express = require('express')
let app = express()
app.use(middleware1)   // app.use() 方法中可以直接加载中间件
```

### 路由级中间件（Router-level middleware）
路由级中间件和一般的应用级中间件使用的方法是一样的，它们之间的区别是功能上职责区别。路由级中间件是为进行路由的管理，这里的路由并非是前端页面的路由，可以理解为单纯的url管理
``` javascript
let app = express()
let router = express.Router()
// a middleware function with no mount path. This code is executed for every request to the router
router.use(function (req, res, next) {
  console.log('Time:', Date.now())
  next()
})
```

### 错误级中间件（Error-handling middleware）
这个中间件可以理解为拦截请求错误的。有四个参数，所以你必须提供四个参数来表明这个中间件是一个错误处理中间件，即使也许你不需要Next对象，你也得明确说明，否则的话，这就会被解析为一个普通的中间件，而非错误处理中间件了。
``` javascript
app.use(function (err, req, res, next) {
  console.error(err.stack)
  res.status(500).send('Something broke!')
})
```

### 内置级中间件（Built-in middleware）
express.js本身就是一个基于node的web应用程序。一个web应用就会涉及到静态资源的加载，express也不例外。因此对于静态资源的加载问题，express在4.4之后就不再依赖connect。因此目前express中唯一的一个内置应用级中间件就是`express.static`。这个函数是基于server-static的。目的是为了提供类似html，css，img，js等静态资源的。
``` javascript
express.static(root, [options])
```
具体使用方法：
``` javascript
var options = {
  dotfiles: 'ignore',
  etag: false,
  extensions: ['htm', 'html'],
  index: false,
  maxAge: '1d',
  redirect: false,
  setHeaders: function (res, path, stat) {
    res.set('x-timestamp', Date.now())
  }
}

app.use(express.static('public', options))
```
如果有多个静态文件的目录
``` javascript
app.use(express.static('public'))
app.use(express.static('file'))
app.use(express.static('image'))

```

### 第三方级中间件（Third-party middleware）
第三方级中间件按照字面理解的话就是第三方中间件。这些中间件非官方开发，有独立开发者自己开发的中间件。这种中间件有很多比如：
* cookie-parser
* body-parser 等等

## 中间件实现原理
上面介绍的express的中间件，其实中间件就是一个函数，在相应发送之前对请求进行一些操作
``` javascript
function middleware(req, res, next){
    // do someting

    // run next function
    next()
}
```
中间件函数它有一个next参数，而这个next参数也是一个函数，他表示函数数组中的下一个函数。那么就会有另外一个问题：函数数组又是什么呢？
其实就是express内部维护一个函数数组，这个函数数组表示在发出响应之前要执行的所有函数，也就是中间件数组

使用`app.use(fn)`后，传进来的fn就会被扔到这个数组里，执行完毕后调用`next()`方法执行函数数组里的下一个函数，如果没有调用next()的话，就不会调用下一个函数了，也就是说调用就会被终止

## 简单实现一个express中间件
``` javascript
/**
 * 仿照express实现中间件的功能
 *
 * Created by BadWaka on 2017/3/6.
 */

var http = require('http');

/**
 * 仿express实现中间件机制
 *
 * @return {app}
 */
function express() {

    var funcs = []; // 待执行的函数数组

    var app = function (req, res) {
        var i = 0;

        function next() {
            var task = funcs[i++];  // 取出函数数组里的下一个函数
            if (!task) {    // 如果函数不存在,return
                return;
            }
            task(req, res, next);   // 否则,执行下一个函数
        }

        next();
    }

    /**
     * use方法就是把函数添加到函数数组中
     * @param task
     */
    app.use = function (task) {
        funcs.push(task);
    }

    return app;    // 返回实例
}

// 下面是测试case

var app = express();
http.createServer(app).listen('3000', function () {
    console.log('listening 3000....');
});

function middlewareA(req, res, next) {
    console.log('middlewareA before next()');
    next();
    console.log('middlewareA after next()');
}

function middlewareB(req, res, next) {
    console.log('middlewareB before next()');
    next();
    console.log('middlewareB after next()');
}

function middlewareC(req, res, next) {
    console.log('middlewareC before next()');
    next();
    console.log('middlewareC after next()');
}

app.use(middlewareA);
app.use(middlewareB);
app.use(middlewareC);
```