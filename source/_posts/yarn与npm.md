---
title: NPM与YARN
date: 2021-7-4 22:12:22
tags:  
- Node 
- JavaScript
---

## NPM
### 什么是npm？
npm（全称 Node Package Manager，即“node包管理器”）是Node.js默认的、用JavaScript编写的软件包管理系统
> 相关历史
npm完全有JavaScript编写，最初由艾萨克·施吕特（Isaac Z. Schlueter）开发。艾萨克表示自己意识到“模块管理很糟糕”的问题，并看到了PHP的PEAR与Perl的CPAN等软件的缺点，于是编写了npm。（[参考内容](https://github.com/nodejs/node-v0.x-archive/issues/5132#issuecomment-15432598)）<br>
2020年3月16 日，GitHub CEO Nat Friedman 宣布 GitHub 已签署收购 NPM（npm 背后的公司）的协议，并表示 npm 加入 GitHub 后会继续免费提供公共软件注册中心服务。

### 相关运行机制
npm会随着Node.js自动安装。npm模块仓库提供了一个名为“registry”的查询服务，用户可通过本地的npm命令下载并安装指定模块。此外用户也可以通过npm把自己设计的模块分发到registry上面。

registry上面的模块通常采用CommonJS格式，而且都包含一个JSON格式的元文件。

npm的模块以“先到先得”的原则注册，各模块作者不会发生混乱。然而一旦有人撤回自己发布的模块，那么不仅会使依赖那个模块的项目出现问题，还会带来安全风险。例如有一个模块叫做“left-pad”，其中只有一个字符串对齐的功能。但是，当作者把它从registry里面移除之后，许多模块便无法正确构建。

npm的registry没有审核机制，因此会存在一些低质量、不安全甚至有害的模块，不过npm服务器的管理员也可以删除有害模块并阻止不怀好意的用户。

另外也有人为npm制作了统计功能，这样可以让开发者了解各模块的使用情况，帮助他们选择合适的模块。

### 依赖安装过程
① 发出 npm install 命令
② 查询node_modules目录之中是否已经存在指定模块，若存在，不再重新安装
③ 若不存在，npm 向 registry 查询模块压缩包的网址④ 下载压缩包，存放在根目录下的.npm目录里⑤ 解压压缩包到当前项目的node_modules目录注意： 一个模块安装以后，本地其实保存了两份。一份是.npm目录下的压缩包，另一份是node_modules目录下解压后的代码。但是，运行npm install 的时候，只会检查node_modules目录，而不会检查.npm目录。也就是说，如果一个模块在.npm下有压缩包，但是没有安装在node_modules目录中，npm 依然会从远程仓库下载一次新的压缩包。


### package 和 package.json文件

package（包）是用javascript代码编写的功能包 package.json文件是包的描述文件，在每个项目的根目录下面。

* 描述包的信息(比如名称、版本、入口文件等)，以便发布到npm registry
* 描述项目所需的依赖包，方便通过npm install下载安装 package.json文件的字段详情介绍参考：
[https://javascript.ruanyifeng.com/nodejs/packagejson.html](https://javascript.ruanyifeng.com/nodejs/packagejson.html)

npm 共享 JS 代码的过程就是：
![npm](https://greenhaha.oss-cn-beijing.aliyuncs.com/frontend/assets/img/package.jpg)

1. 一个统一的package代码仓库 (npm官网)
2. 编写自己的package和package.json文件(参考npm官方文档介绍)
3. 通过 npm publish 把package放到这个仓库里
4. 其他人的项目里想要使用某些package就写到package.json文件中，然后运行npm install，就会自动将这些代码下载下来，统一放到node_modules目录中。

### npm 常用命令

* 查看当前npm版本： npm -v
* 安装包 
* * npm install 包名 
* * npm install 包名 -g : 全局安装，安装后在命令行任意目录下可直接使用包命令

* 更新包： npm install 包名@latest
* 卸载包： npm uninstall 包名
* 根据guide创建一个package.json文件： npm init
* 换源：npm --registry 源地址

## YARN

### 什么是yarn
正如官网主页写的yarn是快速、可靠、安全的依赖管理工具
* 快速： Yarn 缓存了每个下载过的包，所以再次使用时无需重复下载。 同时利用并行下载以最大化资源利用率，因此安装速度更快。
* 可靠：使用详细、简洁的锁文件格式和明确的安装算法，Yarn 能够保证在不同系统上无差异的工作。
* 安全： 在执行代码之前，Yarn 会通过算法校验每个安装包的完整性。

而对于[官方文档](https://engineering.fb.com/2016/10/11/web/yarn-a-new-package-manager-for-javascript/)中所说：
Yarn 是为了弥补 npm 的一些缺陷而出现的。

> Yarn is a new package manager that replaces the existing workflow for the npm client or other package managers while remaining compatible with the npm registry. It has the same feature set as existing workflows while operating faster, more securely, and more reliably. --- (engineering.fb)[https://engineering.fb.com/2016/10/11/web/yarn-a-new-package-manager-for-javascript/]
中文大意就是：yarn是一个新的包管理工具并取代现有npm端以及其他包管理器的现有工作流，并同时保持与 npm 注册表兼容。它具有与现有工作流程相同的功能等。

### yarn 优点
* 速度快。速度快的主要原因：

1. 并行安装：无论 npm 还是 Yarn 在执行包的安装时，都会执行下载任务。npm 是按照队列执行每个 package，也就是说必须要等到当前 package 安装完成之后，才能继续后面的安装。而 Yarn 是同步执行所有任务，提高了性能。
2. 离线模式：如果之前已经安装过一个软件包，用Yarn再次安装时之间从缓存中获取，就不用像npm那样再从网络下载了。减少对网络的依赖防止了在网速不好的情况下下载出现问题或者缓慢等情况

* 安装版本统一：为了防止拉取到不同的版本，Yarn 有一个锁定文件 (lock file) 记录了被确切安装上的模块的版本号。每次只要新增了一个模块，Yarn 就会创建（或更新）yarn.lock 这个文件。这么做就保证了，每一次拉取同一个项目依赖时，使用的都是一样的模块版本。npm 其实也有办法实现处处使用相同版本的 packages，但需要开发者执行 npm shrinkwrap 命令。这个命令将会生成一个锁定文件，在执行 npm install 的时候，该锁定文件会先被读取，和 Yarn 读取 yarn.lock 文件一个道理。npm 和 Yarn 两者的不同之处在于，Yarn 默认会生成这样的锁定文件，而 npm 要通过 shrinkwrap 命令生成 npm-shrinkwrap.json 文件，只有当这个文件存在的时候，packages 版本信息才会被记录和更新。
* 更简洁的输出：npm 的输出信息比较冗长。在执行 npm install 的时候，命令行里会不断地打印出所有被安装上的依赖。相比之下，Yarn 简洁太多：默认情况下，结合了 emoji直观且直接地打印出必要的信息，也提供了一些命令供开发者查询额外的安装信息。
* 多注册来源处理：所有的依赖包，不管他被不同的库间接关联引用多少次，安装这个包时，只会从一个注册来源去装，要么是 npm 要么是 bower, 防止出现混乱不一致。
* 更好的语义化： yarn改变了一些npm命令的名称，比如 yarn add/remove，感觉上比 npm 原本的 install/uninstall 要更清晰。

### 安装过程的步骤
1. 分析: Yarn 通过向每个注册库（registry）发出请求并递归查找每个依赖项来开始解析依赖项。
2. 获取: 接下来，Yarn 在全局缓存目录中查找所需的包是否已经下载。如果没有，Yarn 会获取压缩包 并将其放在全局缓存中，这样它就可以脱机工作并且不需要多次下载依赖项。依赖项也可以作为压缩包 放置在源代码管理中，用于完全离线安装。
3. 链接: 最后，Yarn 通过将全局缓存中所需的所有文件复制到本地node_modules目录中来将所有内容链接在一起。



### yarn 与 npm 命令对比

``` cmd
npm install === yarn 
npm install xxx --save === yarn add xxx
npm uninstall xxx --save === yarn remove xxx
npm install xxx --save-dev === yarn add xxx --dev
npm update --save === yarn upgrade
```