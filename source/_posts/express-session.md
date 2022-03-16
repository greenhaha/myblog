---
toc: true
title: express-session 原理
pic: http://greenhaha.oss-cn-beijing.aliyuncs.com/frontend/assets/img/images.jpg
date: 2022-03-16 21:35:36
tags:
- Node
- express
- 中间件
- express-session
category: 
- 知识剖析
---

## Express-session解析

### session 是什么
ession一般译作会话，牛津词典对其的解释是进行某活动连续的一段时间。在web应用的用户看来，session是另一种记录客户状态的机制，不同的是Cookie保存在客户端浏览器中，而session保存在服务器上。
客户端浏览器访问服务器的时候，服务器把客户端信息以某种形式记录在服务器上.客户端浏览器再次访问时只需要从该Session中查找该客户的状态就可以了。
> Session是有状态的，而HTTP协议是无状态的，二者是否矛盾呢？

Session和HTTP协议属于`不同层面的事物`，HTTP属于ISO七层模型的最高层`应用层`，<strong>前者Session不属于后者</strong>，前者HTTP是具体的动态页面技术来实现的，但同时它又是<strong>基于后者的</strong>。

### Session的原理：摘录
基本原理是服务端为每一个session维护一份会话信息数据, 而客户端和服务端依靠一个全局唯一的标识来访问会话信息数据。用户访问web应用时，服务端程序决定何时创建session，创建session可以概括为三个步骤：

```
1. 生成全局唯一标识符（sessionid）；
2. 开辟数据存储空间。一般会在内存中创建相应的数据结构，但这种情况下，系统一旦掉电，所有的会话数据就会丢失，如果是电子商务网站，这种事故会造成严重的后果。不过也可以写到文件里甚至存储在数据库中，这样虽然会增加I/O开销，但session可以实现某种程度的持久化，而且更有利于session的共享；
3. 将session的全局唯一标示符发送给客户端。

```

问题的关键就在<strong>服务端如何发送这个session的唯一标识上</strong>。联系到HTTP协议，数据无非可以放到请求行、头域或Body里，基于此，一般来说会有两种常用的方式：`cookie`和`URL重写`。

### session和cookie 优缺点和各自的应用场景：
```
1.   应用场景
Cookie的典型应用场景是Remember Me服务，即用户的账户信息通过cookie的形式保存在客户端，当用户再次
请求匹配的URL的时候，账户信息会被传送到服务端，交由相应的程序完成自动登录等功能。当然也可以保存
一些客户端信息，比如页面布局以及搜索历史等等。
Session的典型应用场景是用户登录某网站之后，将其登录信息放入session，在以后的每次请求中查询相应
的登录信息以确保该用户合法。当然还是有购物车等等经典场景；
2.   安全性
cookie将信息保存在客户端，如果不进行加密的话，无疑会暴露一些隐私信息，安全性很差，一般情况下敏
感信息是经过加密后存储在cookie中，但很容易就会被窃取。而session只会将信息存储在服务端，如果存
储在文件或数据库中，也有被窃取的可能，只是可能性比cookie小了太多。
Session安全性方面比较突出的是存在会话劫持的问题，这是一种安全威胁。总体来讲，session的安全性
要高于cookie；
3.   性能
Cookie存储在客户端，消耗的是客户端的I/O和内存，而session存储在服务端，消耗的是服务端的资源。
但是session对服务器造成的压力比较集中，而cookie很好地分散了资源消耗，就这点来说，cookie是要优于session的；
4.   时效性
Cookie可以通过设置有效期使其较长时间内存在于客户端，而session一般只有比较短的有效期（用户主动销毁
session或关闭浏览器后引发超时）；
5.   其他
Cookie的处理在开发中没有session方便。而且cookie在客户端是有数量和大小的限制的，而session的大小
却只以硬件为限制，能存储的数据无疑大了太多。
```

Session工作的大致步骤
```
1. 用户提交包含用户名和密码的表单，发送HTTP请求。
2. 服务器验证用户发来的用户名密码。
3. 如果正确则把当前用户名（通常是用户对象）存储到redis中，并生成它在redis中的ID。
    这个ID称为Session ID，通过Session ID可以从Redis中取出对应的用户对象， 敏感数据（比如authed=true）都存储在这个用户对象中。
4. 设置Cookie为sessionId=xxxxxx|checksum并发送HTTP响应， 仍然为每一项Cookie都设置签名。
5. 用户收到HTTP响应后，便看不到任何敏感数据了。在此后的请求中发送该Cookie给服务器。
6. 服务器收到此后的HTTP请求后，发现Cookie中有SessionID，进行放篡改验证。
7. 如果通过了验证，根据该ID从Redis中取出对应的用户对象， 查看该对象的状态并继续执行业务逻辑。
```

## express-session中间件
express-session中间件将会话数据存储在服务器上；它仅将会话标识（而非会话数据）保存在 cookie 中。从1.5.0版本开始, express-session不再依赖`cookie-parser`,直接通过`req/res`读取/写入;默认存储位置`内存存储`(服务器端),

安装:  `npm install express-session`

主要方法 : <strong>session(options)</strong>

通过option来设置session存储，除了session ID外，session中的任何数据都不存储在cookie中。
options可选参数:
```
1. name - cookie的名字（原属性名为 key）。（默认：’connect.sid’）
2. store - session存储实例
3. secret - 用它来对session cookie签名，防止篡改
4. cookie - session cookie设置 （默认：{ path: ‘/‘, httpOnly: true,secure: false, maxAge: null }）
5. genid - 生成新session ID的函数 （默认使用uid2库）
6. rolling - 在每次请求时强行设置cookie，这将重置cookie过期时间（默认：false）
7. resave - 强制保存session即使它并没有变化 （默认： true）
8. proxy - 当设置了secure cookies（通过”x-forwarded-proto” header ）时信任反向代理。当设定为true时，
”x-forwarded-proto” header 将被使用。当设定为false时，所有headers将被忽略。当该属性没有被设定时，将使用Express的trust proxy。
9. saveUninitialized - 强制将未初始化的session存储。当新建了一个session且未设定属性或值时，它就处于
未初始化状态。在设定一个cookie前，这对于登陆验证，减轻服务端存储压力，权限控制是有帮助的。（默认：true）
10. unset - 控制req.session是否取消（例如通过 delete，或者将它的值设置为null）。这可以使session保持存储
状态但忽略修改或删除的请求（默认：keep）
```

### express-session的一些方法:
```
1. Session.destroy():删除session，当检测到客户端关闭时调用。
2. Session.reload():当session有修改时，刷新session。
3. Session.regenerate()：将已有session初始化。
4. Session.save()：保存session。
```

### 内存存储 方式实例代码:
一旦我们将express-session中间件用use挂载后，我们可以很方便的通过req参数来存储和访问session对象的数据。req.session是一个JSON格式的JavaScript对象，我们可以在使用的过程中随意的增加成员，这些成员会自动的被保存到option参数指定的地方，默认即为内存中去。
```
var express = require('express');
var session = require('express-session');
var app = express();// Use the session middleware 
app.use(session({ 
////这里的name值得是cookie的name，默认cookie的name是：connect.sid
  //name: 'hhw',
  secret: 'keyboard cat', 
  cookie: ('name', 'value', { path: '/', httpOnly: true,secure: false, maxAge:  60000 }),  //重新保存：强制会话保存即使是未修改的。默认为true但是得写上
  resave: true, 
  //强制“未初始化”的会话保存到存储。 
  saveUninitialized: true,  
  
}))
// 只需要用express app的use方法将session挂载在‘/’路径即可，这样所有的路由都可以访问到session。//可以给要挂载的session传递不同的option参数，来控制session的不同特性 
app.get('/', function(req, res, next) {  
var sess = req.session//用这个属性获取session中保存的数据，而且返回的JSON数据
  if (sess.views) {
    sess.views++
    res.setHeader('Content-Type', 'text/html')
    res.write('<p>欢迎第 ' + sess.views + '次访问       ' + 'expires in:' + (sess.cookie.maxAge / 1000) + 's</p>')
    res.end();
  } else {
    sess.views = 1
    res.end('welcome to the session demo. refresh!')
  }
});

app.listen(3000);
```