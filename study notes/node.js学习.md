# Node.js 学习

## 什么是*node.js*

### 官方关键词

* Chrome V8引擎
* 事件驱动
* 非阻塞式I/O

### 应用场景

* 服务端渲染——前后端同构、SEO
* 服务端API封装
* 前端工程构建——gulp、webpack
* Electron（node.js+chrome）——复用node.js的系统控制能力+chrome的页面渲染能力
* ……

## 关于版本

具体可参见[官方文档](https://github.com/nodejs/Release)

### 版本类型

* LTS：长期维护版本（Long Term Support）
* Current：当前版本（即正在开发的最新版本）

### 发布时间

* 主版本号每6个月发布一次，其中偶数版本为每年4月，奇数版本为每年10月
* 偶数版本发布之后，经过6个月（即每年10月）会转变为LTS版本，进入12个月的ACTIVE阶段，随后进入18个月的MAINTENANCE阶段。（在node.js 12之前的版本是18个月的ACTIVE阶段和12个月的MAINTENANCE）
* 一旦版本进入LTS，新特性只有在发布工作组同意的情况下才能发布，除了关键安全修复所需要的更改外（bug修复、安全漏洞修复、文档更新、对现有应用影响小的性能提升等），不会有大的改动。
* 一旦版本进入MAINTENANCE阶段，只允许进行关键bug修复、关键安全漏洞修复、文档更新以及N-API一致性或可用性问题的更新
* 奇数版本将会在下一个偶数版本

| Release                                                   | Status         | Codename                                                     | Initial Release | Active LTS Start | Maintenance Start | End-of-life |
| --------------------------------------------------------- | -------------- | ------------------------------------------------------------ | --------------- | ---------------- | ----------------- | ----------- |
| [10.x](https://nodejs.org/download/release/latest-v10.x/) | **Active LTS** | [Dubnium](https://nodejs.org/download/release/latest-dubnium/) | 2018-04-24      | 2018-10-30       | April 2020        | April 2021  |
| [12.x](https://nodejs.org/download/release/latest-v12.x/) | **Active LTS** | [Erbium](https://nodejs.org/download/release/latest-erbium/) | 2019-04-23      | 2019-10-21       | October 2020      | April 2022  |
| 13.x                                                      | **Current**    |                                                              | 2019-10-22      |                  |                   | June 2020   |
| 14.x                                                      | **Pending**    |                                                              | April 2020      | October 2020     | October 2021      | April 2023  |



![schedule](/Users/ycyu/Documents/Typora Notes/media/schedule.svg)

## 架构一览

* node.js v12开始，默认http解析器改为llhttp，性能相对http_parser提升

| |input size       | bandwidth  | reqs/sec     | time               |
| ---------------- | ---------- | ------------ | ------------------ | ------ |
| **llhttp** *(C)* | 8192.00 mb | 1777.24 mb/s | 3583799.39 ops/sec | 4.61 s |
| **http_parser**  | 8192.00 mb | 694.66 mb/s  | 1406180.33 req/sec | 11.79 s |

* 通过process.versions可以查看各组件的版本

```bash
node -p 'process.versions'                                                            13:44:38  2020-03-03
{
  node: '12.15.0',
  v8: '7.7.299.13-node.16',
  uv: '1.33.1',
  zlib: '1.2.11',
  brotli: '1.0.7',
  ares: '1.15.0',
  modules: '72',
  nghttp2: '1.40.0',
  napi: '5',
  llhttp: '2.0.4',
  http_parser: '2.9.3',
  openssl: '1.1.1d',
  cldr: '35.1',
  icu: '64.2',
  tz: '2019c',
  unicode: '12.1'
}
```
## 引子

* 什么是node.js——脚本语言？
* require一个模块时发生了什么
* 当在try...catch...中有异步操作时，比如setTimout，为什么无法捕获异常
* node.js是怎么实现异步非阻塞的
* 有的api中比如ajax请求，写代码时，可以先调send，再去监听事件



### 版本的区别

* 奇偶版本
* LTS与Current

### process.argv

### buffer

* 内存分配——在v8堆内存之外
* buffer.slice()操作，不像数据那样得到一个新的数组，buffer.slice的结果引用的是同一段内存，也就是说如果修改了，会影响原内容

## 模块

### 浏览器端的模块

### commonjs

### 模块加载

* 两个关键的模块：require 和 module，两者都是全局可访问了，即你不需要require('require')
* 

* 第一次加载本地文件时：require('./xxx/xx.js')
	* resolving：查找的绝对路径
	* loading：决定文件的类型
	* wrapping：针对文件创建一个私有的作用域，
	* evaluating：引擎执行代码
	* caching：再一次require模块时，不会去执行模块中的代码，而是直接使用结果

* module模块
	`console.log(module)`
	* id：模块标识，如果是文件模块，这里就是文件的路径
	* paths：模块查找的路径（核心模块除外，为什么？）

* 模块入口文件解析的顺序
	* package.json中的main
	* index.js

* require.resolve：只查找模块，但不执行

* 模块的循环依赖

* 不指定文件后缀名时，按.js->.json->.node的优先级顺序查找文件，可以通过require.extensions查看，其中每一个是个函数
* 

* exports 与 module.exports（参照webpack）
	* require('module').wrapper具体做了什么
	* require.main的作用
	* require.cache的内容
	
* 默认没有导出时，引入的模块是什么
* __dirname、__filename是怎么来的 

### npm——模块管理

* 常用命令
* package.json 与 package-lock.json

### 架构图

### 内置版块 以及 调整流程

## 异步和事件循环机制

### node.js中的I/O主要指什么

* 访问磁盘
* 访问网络

### 异步模型

* 一个请求一个线程处理：Apache
* 单线程：Nginx

### 什么是非阻塞I/O

### 事件循环——非阻塞I/O的基础

* [事件循环](https://nodejs.org/en/docs/guides/event-loop-timers-and-nexttick/)
	* The entity that handles external events and converts them into callback invocations
	* A loop that picks events from the event queue and pushes their callbacks to the call stack
	* 图片说明

* 调用栈
* node 11 前后事件循环机制的变化：https://mp.weixin.qq.com/s/qEmR-N6cANSkKuJt2QO_eg

* libuv

## 子进程

### 进程：

操作系统挂载运行程序的单元，拥有一些独立的资源，比如内存

### 线程：

进行运算调试的单元，进程内的线程共享进程内的资源

### 创建子进程的四种方式及区别

#### 1. spawn

`spawn()`返回的是一个ChildProcess的实例，它继承了EventEmitter，可绑定的事件有`disconnect`, `error`, `close`, and `message`.

#### 2. exec

默认情况下`spawn`不会创建shell来执行传递的命令（可以通过spawn的第二参数的配置项指定`shell:true`来开启支持），而`exec`则是如此，所以它支持shell脚本的语法，同时其结果是缓存下来并通过回调函数的方式输出（所以输出的数据要尽量小，因为它会先将期缓存在内存中，再一起输出）

#### 3. execFile

仅需要在非shell中执行一个文件时使用，它与`exec`类似，只是不使用shell

#### 4. fork

它是`spawn`的变体，与`spawn`最大的不同是当使用`fork`时，与会子进程建立一个消息管道，就可以使用`send`方法和`on('message')`事件

## cluster

### cluster下listen的处理

多个进程同时监听一个端口号，怎么实现的？

## 源码调试

### build

```bash
./configure --debug
# -j指定同时进行的编译任务数，可与cpu核数一致
make -j8
# 或如下，此时不会编译release的代码
make -C out BUILDTYPE=Debug -j8
# linux下要注意使用sudo来执行
```

### xcode的调试方法

```bash
./configure -- -f xcode
```

## 内存管理

### 内存占用情况

```bash
# 查看内存占用情况
node -p 'process.memoryUsage()'
```

通过启动node时指定`--expose-gc`，就可以通过`global.gc()`来手动gc

```bash
# 修改新生代的内存上限，减少GC的次数，建议分配64MB或128MB比较合适
node --max-semi-space-size=128 app.js
```


