---
toc: true
title: 快速搭建一个MOCKSERVE服务器
date: 2021-7-5 20:30:22
toc: true
tags:  
- Node 
- JavaScript
- mockserve
- API
---


# 如何快速搭建一个mockserve服务器

## 什么是mock
如果将mock单独翻译过来，其意义为 “虚假、虚设”，因此在软件开发领域，我们也可以将其理解成 “虚假数据”，或者 “真实数据的替身”。

## mock的好处

* 团队可以更好地并行工作

当使用mock之后，各团队之间可以不需要再互相等待对方的进度，只需要约定好相互之间的数据规范（文档），即可使用mock构建一个可用的接口，然后尽快的进行开发和调试以及自测，提升开发进度的的同时，也将发现缺陷的时间点大大提前。

* 开启TDD（Test-Driven Development）模式，即测试驱动开发

单元测试是TDD实现的基石，而TDD经常会碰到协同模块尚未开发完成的情况，但是有了mock，这些一切都不是问题。当接口定义好后，测试人员就可以创建一个Mock，把接口添加到自动化测试环境，提前创建测试。

* 测试覆盖率

比如一个接口在各种不同的状态下要返回不同的值，之前我们的做法是复现这种状态然后再去请求接口，这是非常不科学的做法，而且这种复现方法很大可能性因为操作的时机或者操作方式不当导致失败，甚至污染之前数据库中的数据。如果我们使用mock，就完全不用担心这些问题。

* 方便演示

通过使用Mock模拟数据接口，我们即可在只开发了UI的情况下，无须服务端的开发就可以进行产品的演示。

* 隔离系统

在使用某些接口的时候，为了避免系统中数据库被污染，我们可以将这些接口调整为Mock的模式，以此保证数据库的干净。

## 如何快速实现一个mockserve

### 什么是mockserve
专门实现 mock 功能的一个服务。

### mockserve 的作用
的业务系统很少有孤立存在的，它们或多或少需要使用兄弟团队或是其他公司提供的服务，这给我们的联调和测试造成了麻烦。对于这种情况，我们常见的解决方案是搭建一个临时的server，模拟那些服务，提供数据进行联调和测试。这就是 mock server 出现原因。

![mockserve](https://greenhaha.oss-cn-beijing.aliyuncs.com/frontend/assets/img/mockserve.png)

### mock数据的几种方式

1. 直接在页面写死数据，后期等接口来了，再改成动态的; 

2. 在js里直接声明变量，并给变量赋值，在逻辑脚本中使用，并渲染到dom;

3. 将模拟数据编辑成json数据或者是零碎的js脚本中，通过请求，取回数据，并进行业务逻辑处理，渲染到dom

4. 最理想化的
- 前后台在需求分解之后，一起定义好接口api，包含：请求url（项目前缀+具体的接口名称）、请求方式、请求参数、数据响应；

- 前端研发人员根据接口约定，模拟请求返回对应的数据，完成对应的交互；

- 后台人员根据接口约定，完成对应的api，并完成对应的自测；

- 待后台人员交付接口api后，前端人员直接修改接口项目前缀，切换到对应的环境，即可进入项目提测。



### 如何使用web框架快速搭建mockserve

这个mockserve是基于 express+mock.js+nodemon 来构建的

Express 是一个基于 Node 平台的 Web 开发框架，使用它可以很方便的搭建本地Web 服务。用来部署我们的 Mock 数据，Express 可以通过 npm 来安装，其官网地址如下：http://www.expressjs.com.cn/ 其中有详细安装方法

Mock.js 是一个模拟数据结构、生成随机数据的 JavaScript 库，有一套语法规则用来模拟结构和生成数据。其安装过程也很简单，官网地址：http://mockjs.com/

nodemon node进程自动重启，监听文件

### 搭建步骤：


首先本地创建项目工程文件夹，名字随意 然后进入到创建的文件夹中
``` cmd
mkdir express_mockserve
```

利用npm或者yarn 本地初始化工程
```
yarn init
```
```
npm init
```
安装相关依赖包依赖包

```
npm install express mockjs nodemon
```

```
yarn add express mockjs nodemon
```

安装完成后，打开package.json文件，在script对象中加入下列代码
```json
"mock:serve": "nodemon express_mockserve/index.js --watch express_mockserve/*"
```
>作用：利用nodemon 自动启动node并监听目标文件更改，如果更改自动热更新

然后创建`index.js` 工程启动文件

``` javascript
var express = require('express');
var app = express();
var data = {
    name : "jack",
    age : 25
}

app.get('/mock', function (req,res) {
    res.send(JSON.stringify(data));
});
var server = app.listen(8001, function(){

});
```
上面的代码使用 Express 搭建了一个简单的本地服务器，在终端使用 Node 执行此 JavaScript 程序，在浏览器中输入如下地址，进行 GET 请求，就可以看到已经获取到本地服务器返回的数据。

```
http://localhost:8001/mock
```

下面借助 Mock.js 来随机生成模拟数据，修改上面代码如下

``` javascript
var express = require('express');
var Mock = require('mockjs');
var app = express();
var data = Mock.mock({

    'people|1-10' : [{
        name : "@cname",
        'age|10-40' : 0,
        // 属性 id 是一个自增数，起始值为 1，每次增 1
        'id|+1': 1
    }]
})

app.get('/people', function (req,res) {
    res.send(JSON.stringify(data));
});
var server = app.listen(8001, function(){

});
```

上面代码使用了 Mock.js 中生成随机数据的一些语法，重启Node服务，再次刷新浏览器，就会看到效果。。
具体的 Mock.js 模拟数据的语法和规则可以在 [https://github.com/nuysoft/Mock/wiki/Getting-Started](https://github.com/nuysoft/Mock/wiki/Getting-Started) 查看

项目demo：[地址](https://github.com/greenhaha/exprssmockserve)

### 关于express搭建mockserve跨域问题
如果express搭建mockserve产生跨域问题 通常有2中解决方案：
1. 安装解决跨域中间件 `cors`
使用发放
```
npm install cors --save-dev
```
然后再工程的入口文件里 加入下列代码
```
const cors = require('cors');
app.use(cors());
```

2. 手动实现解决方案

```javascript
app.use((req, res, next) => {
res.header('Access-Control-Allow-Origin', '*')
res.header('Access-Control-Allow-Headers', 'Authorization,X-API-KEY, Origin, X-Requested-With, Content-Type, Accept, Access-Control-Request-Method' )
res.header('Access-Control-Allow-Methods', 'GET, POST, OPTIONS, PATCH, PUT, DELETE')
res.header('Allow', 'GET, POST, PATCH, OPTIONS, PUT, DELETE')
next();
});
```

注：方案2中不同的值在解决跨域中起到不同的作用
* Access-Control-Allow-Origin
`Access-Control-Allow-Origin` 是一个html5中添加的CORS(Cross-Origin Resource Sharing)头
跨域访问时，B站点 通过在响应头中添加 Access-Control-Allow-Origin:http://siteA 向浏览器表示该资源可被A站点正常访问使用。除非添加了Access-Control-Allow-Origin响应头，否则默认情况下一个站点的资源不允许来自于其他域的任何XMLHttpRequest请求。

对于B站点任意页面或者资源，如果想要允许被A站点访问，则应在页面或者资源请求的响应中添加相应头： Access-Control-Allow-Origin: http://siteA.com
* Simple请求
现代浏览器不会完全阻止跨域请求。如果A站点请求B站点的一个页面P，浏览器实际上会在网络级拉取页面P，然后检查页面P响应头中A站点是否在允许列表中。如果响应中没有声明A站点具有访问权限，则浏览器会触发XMLHttpRequest's error事件，并且阻止响应数据的执行。

* Non-Simple 请求
真实发生的网络级请求实际上会稍微复杂一点。如果一个请求是Non-Simple的，浏览器首先会发送一个不包含数据的OPTIONS http请求，以此来验证服务器是否接受该站点的相应请求（GET,POST,PUT..etg.），一个Simple的请求需要同时满足以下两点：

    * 只能使用HTTP的GET,POST或者HEAD方法
    * 只能使用Simple的请求头：
        * Accept
        * Accept-Language
        * Content-Language
        * Last-Event-ID
        * Content-Type (Simple的请求定义中，content-type只能是application/x-www-form-urlencoded, multipart/form-data,或者 text/plain)
不符合这种情况的请求则是一个Non-Simple的请求，除此之外的http方法和http头叫做Non-Simple方法和头

如果服务端对OPTIONS预请求的响应中包含了恰当的Non-Simple响应头(Access-Control-Allow-Headers,Access-Control-Allow-Methods),并且响应头的值中包含了真实要发生的请求的Non-Simple请求头和方法，浏览器才会继续发送真正的Non-Simple请求。

* Access-Control-Allow-Headers
响应首部 `Access-Control-Allow-Headers` 用于 preflight request （预检请求）中，列出了将会在正式请求的 Access-Control-Request-Headers 字段中出现的首部信息。

简单首部，如 simple headers、Accept、Accept-Language、Content-Language、Content-Type （只限于解析后的值为 application/x-www-form-urlencoded、multipart/form-data 或 text/plain 三种MIME类型（不包括参数）），它们始终是被支持的，不需要在这个首部特意列出。

* Access-Control-Allow-Methods
响应首部 `Access-Control-Allow-Methods` 在对 preflight request.（预检请求）的应答中明确了客户端所要访问的资源允许使用的方法或方法列表。

* Allow
Allow 首部字段用于枚举资源所支持的 HTTP 方法的集合。

若服务器返回状态码 405 Method Not Allowed，则该首部字段亦需要同时返回给客户端。如果 Allow  首部字段的值为空，说明资源不接受使用任何 HTTP 方法的请求。这是可能的，比如服务器需要临时禁止对资源的任何访问。

> 为什么访问url 没有端口号也可以访问对应页面工程
只有IP地址只可以发链路层的PING包侦测地址是否可达，页面传输是应用层的HTTP协议，从TCP/IP网络层开始，高层次的协议通信是要有ip加端口号的。浏览器默认80端口，一般而言tomcat也不会直接面对用户，前面会加一层代理层（Apache nginx），代理层的默认端口就是80。