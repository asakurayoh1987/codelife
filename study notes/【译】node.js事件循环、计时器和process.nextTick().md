# The Node.js Event Loop, Timers, and process.nextTick()

## 一、什么是事件循环

尽管事实上javascript引擎是单线程的，但node.js通过事件循环，尽可能将一些操作交给系统内核来处理，从而实现非阻塞的I/O操作。因为现代操作系统内核基本都是多线程的，它们可以在后台处理多个操作。但其中某个操作完成时，内核会通知node.js，从而将对应的回调添加到轮询队列中并最终被执行，接下来我们会进一步讨论这些细节。

## 二、事件循环

当node.js启动时，它会初始化事件循环，然后执行输入的javascript脚本，脚本中可能包括异步api的调用、计时器设置以及调用process.nextTick方法，然后就会开始执行事件循环，下图简单说明了事件循环中的各阶段（其中每个方块代表了事件循环中的一个阶段）：

```
   ┌───────────────────────────┐
┌─>│           timers          │
│  └─────────────┬─────────────┘
│  ┌─────────────┴─────────────┐
│  │     pending callbacks     │
│  └─────────────┬─────────────┘
│  ┌─────────────┴─────────────┐
│  │       idle, prepare       │
│  └─────────────┬─────────────┘      ┌───────────────┐
│  ┌─────────────┴─────────────┐      │   incoming:   │
│  │           poll            │<─────┤  connections, │
│  └─────────────┬─────────────┘      │   data, etc.  │
│  ┌─────────────┴─────────────┐      └───────────────┘
│  │           check           │
│  └─────────────┬─────────────┘
│  ┌─────────────┴─────────────┐
└──┤      close callbacks      │
   └───────────────────────────┘
```

每个阶段都有一个待执行的FIFO(先进行出)的队列，基本上来说，每个阶段有自己的特点，当事件循环进入指定阶段时，会执行这个阶段特有的一些操作，然后就会执行队列中的回调，直到队列被清空或者达到一个最大的回调执行数量。当满足这个条件时，事件循环就会进入下个阶段。

由于这些操作中的任意一个都可能调度更多的操作，然后系统内核会往poll阶段的队列中添加新的事件，在处理这些poll阶段的事件时可能会添加新的poll事件到队列中。因此，这些长时间执行的回调会导致poll阶段的执行时间远超过计时器的阈值，参见下文关于timers阶段与poll阶段的具体描述。

*Note：Windows和Unix/Linux在实现上会有些许不同，但这不影响本文的内容，因为最重要的部分就是这里提到的——Node.js实际使用的*

## 三、各阶段概览

* **timers：**此阶段执行通过setTimeout以及setInterval添加的回调。
* **pending callbacks：**执行那些被延迟到下一次事件循环执行的I/O回调
* **idle,prepare：**仅内部使用
* **poll：**获取新的I/O事件；执行I/O相关的回调(基本上除了close callbacks、timers的回调以及setImmediate的回调之外的所有)，适当的时候，node.js会阻塞在此阶段。
* **check：**执行setImmediate添加的回调
* **close callbacks：**一些用于关闭事件的回调，比如：`socket.on('close',...)`

## 四、各阶段详述

### timers

计时器可以指定一个时间阈值及一个回调，回调会在这个指定的阈值之后尽可能早的一个时间点执行，而不是像人们期望的那样在一个准确的时间点执行。但由于操作系统的调度或其它回调的执行，会导致它们被延迟。

*Note：从技术上说，poll阶段控制计时器何时被执行*

举个例子，假设你通过setTimeout设置了一个100ms后执行的回调，然后你的脚本会执行一个异步读取文件的操作，这个操作耗时95ms：

```javascript
const fs = require('fs');

function someAsyncOperation(callback) {
  // Assume this takes 95ms to complete
  fs.readFile('/path/to/file', callback);
}

const timeoutScheduled = Date.now();

setTimeout(() => {
  const delay = Date.now() - timeoutScheduled;

  console.log(`${delay}ms have passed since I was scheduled`);
}, 100);

// do someAsyncOperation which takes 95 ms to complete
someAsyncOperation(() => {
  const startCallback = Date.now();

  // do something that will take 10ms...
  while (Date.now() - startCallback < 10) {
    // do nothing
  }
});
```

当事件循环进入poll阶段时，它对应的队列是空的(`fs.readFile()`尚未完成)，所以它接下来会等待时间上最临近的计时器(100ms)，但当它等待95ms时，`fs.readFile()`完成了文件读取的操作，并且它的回调(耗时10ms)会被添加到队列中并执行，当它执行完毕后，队列中没有其它的回调，此时事件循环发现之前设置的计时器已经到达阈值，于是折返回timers阶段来执行计时器的回调。在这个例子中，你会看从设置计时器到其回调被执行中间的间隔时间为105ms。

*Note：为了防止poll阶段耗时太久，libuv(实现node.js事件循环以及各平台下所有异步行为的c语言开发的库)会设置一个硬值上限(区分系统)来阻止它轮询更多的事件*

### pending callback

此阶段执行诸如TCP错误这样系统操作的回调。比如，当一个TCP套接字尝试建立连接时可能会得到ECONNREFUSED的错误，而一些*nix系统会等待并报告这个错误，获取这个错误信息的回调会被加到pending阶段的队列中去执行。

### poll

poll阶段主要做两件事：

1. 计算它需要阻塞多久来进行I/O的轮询
2. 然后处理队列中的事件

当事件循环进入到poll阶段，并且此时没有设置计时器，会进行如下操作：

* 如果队列非空，则队列中的回调会被依次同步执行，直到队列为空或达到设置的硬上限(前文描述的区分系统设置的值)
* 如果队列为空，则会进行如下操作：
	* 如果通过setImmediate设置了回调，则事件循环会结束poll阶段，并进入check阶段来执行这些回调
	* 否则事件循环会等待新的回调被添加到队列中，并立刻执行它们

一旦队列空了，事件循环将会检查是否有计时器的时间阈值已经到期，如果有则事件循环会折返到timers阶段来执行这些计时器的回调。

### check

此阶段用于在poll阶段结束后立刻执行回调，如果poll阶段空闲(任务队列空了)，并且已经通过setImmediate设置了回调，则事件循环并不停留在poll阶段等待新的事件，而是进入check阶段。setImmediate实际上是一个特殊的计时器，它运行在事件循环的一个独立的阶段。它使用了libuv提供的API，此API调度的回调会在poll阶段结束后执行。

通常，在代码执行时，事件循环最终将进入poll阶段，等待传入的链接、请求等等。但如果通过setImmediate设置了回调，并且poll阶段空闲，则事件循环将不再等待新的poll事件，而是结束poll阶段，进入check阶段。

### close callbacks

如果一个套接字或句柄突然关闭(比如`socket.destroy()`)，则`close`事件将会在此阶段触发，否则它将通过`process.nextTick()`触发。

## setImmediate() vs setTimeout()

二者很相似，但根据调用时间的不同，二者的表现也不一样

* `setImmediate()`被设计为一旦poll阶段完成就立刻执行一段脚本
* `setTimeout()`被设计为在一段时间(单位：ms)之后尽早执行一段脚本

关于二者执行的先后顺序与其被调用时的上下文相关，表现不一。如果二者都是在主模块中调用的，则计时将会受进程性能的影响（可能受到机器上运行的其他应用程序的影响）

举个例子，如果我们在非I/O周期(主模块)中执行下面的脚本，则两个计时器的顺序是不确定的。

```javascript
// timeout_vs_immediate.js
setTimeout(() => {
  console.log('timeout');
}, 0);

setImmediate(() => {
  console.log('immediate');
});

// 打印结果可能timeout在前，也可能immediate在前
```

但如果将这两个调用放到I/O周期中，则`setImmediate()`的回调会总是被先调用：

```javascript
// timeout_vs_immediate.js
const fs = require('fs');

fs.readFile(__filename, () => {
  setTimeout(() => {
    console.log('timeout');
  }, 0);
  setImmediate(() => {
    console.log('immediate');
  });
});
```

与`setTimeout()`相比，`setImmediate()`的优先是，如果在I/O周期，无论设置了多少计时器，`setImmediate()`永远最先执行

## process.nextTick()

### 理解`process.nextTick()`

你肯定已经注意到，尽管`process.nextTick()`是异步API的一部分，但并未显示在事件循环各阶段的示意图中，这是因为从技术上而言，它并不是事件循环的一部分，取而代之的是，无论当前处于事件循环的哪个阶段，`nextTickQueue`会在当前“操作”完成后就被处理。这里提到的“一次操作”被定义为一次从底层C/C++句柄的转换和处理需要被执行的javascript

再回头看各阶段的示意图，在给定的阶段的任何时刻所所有通过`process.nextTick()`添加的回调会在事件循环继续之前都会被处理。这可能会带来一些不好的情况，因为这将允许嵌套的调用`process.nextTick()`，从而让事件循环无法进入到poll阶段。

### 为什么允许这样设计

node.js中有这样的机制，部分原因是一种设计哲学：API应当总是异步的，即便在一些不必强制要求的场景。看一下下面的代码片段

```javascript
function apiCall(arg, callback) {
  if (typeof arg !== 'string')
    return process.nextTick(callback,
                            new TypeError('argument should be string'));
}
```

上面的代码中做了一个参数校验，当校验不通过时，会创建一个错误并传递给回调函数。`process.nextTick`这个API最近进行了更新，允许在callback之后传递参数来作为调用callback时的参数，这样你就不必像下面这样进行函数的嵌套了

```javascript
function apiCall(arg, callback) {
  if (typeof arg !== 'string') {
    return process.nextTick(() => {
      callback(new TypeError('argument should be string'));
    });
  }
}
```

我们在这里做的将错误返回给调用者，但这是在执行完剩下的用户代码之后才会进行。通过使用`process.next()`，可以保证`apiCall()`总是在执行完剩余代码并且在事件循环继续之前调用`callback`。为了实现这一点，js的调用栈被允许展开然后立刻执行提供的回调，这就允许嵌套调用`process.nextTick()`而不会触发`RangeError：Maximum call stack size exceeded from v8`的错误(超过v8最大调用栈大小)

这种设计哲学也可能导致一些潜在的问题，比如下面这个例子：

```javascript
let bar;

// this has an asynchronous signature, but calls callback synchronously
function someAsyncApiCall(callback) { callback(); }

// the callback is called before `someAsyncApiCall` completes.
someAsyncApiCall(() => {
  // since someAsyncApiCall has completed, bar hasn't been assigned any value
  console.log('bar', bar); // undefined
});

bar = 1;
```

这里从函数名可推测`someAsyncApiCallback`是异步执行回调的，但实际上是同步操作的。当调用它时，提供给`someAsyncApiCallback`的回调是在事件循环的同一个阶段中被调用的，因为实际上`someAsyncApiCallback`并没有进行任何异步的操作，于是乎，脚本并没有执行完，此时变量还未赋值，回调拿到的`bar`是undefined。

通过把`callback`放到`process.nextTick()`中，脚本就可以在`callback`执行前执行完，这样变量会先于`callback`进行初始化。不允许事件循环继续执行也存在一些好处，比如在事件循环继续执行前通知用户某些错误信息。下面是之前代码的修改版本(使用`process.nextTick()`)

```javascript
let bar;

function someAsyncApiCall(callback) {
  process.nextTick(callback);
}

someAsyncApiCall(() => {
  console.log('bar', bar); // 1
});

bar = 1;
```

下面是一个真实场景的例子：

```javascript
const server = net.createServer(() => {}).listen(8080);

server.on('listening', () => {});
```

当只传递端口时，这个端口会立刻被绑定，所以`listening`的回调会立刻被调用，但问题在于`.on('listening')`的回调在端口绑定时并未设置。为了解决这个问题，`listening`事件是被到`nextTickQueue`中的，这样就允许代码执行完毕后才会被触发，所以`listening`的回调可以在代码的任何地方进行设置了

## process.nextTick() vs setImmediate()

就用户而言，我们有两个类似的调用，但它们的名称却令人困惑

* `process.nextTick()`：在同一阶段立刻触发
* `setImmediate()`：在事件循环的下一次迭代或'tick'中触发

实际上二者的名称应该交换一下，`process.nextTick()`比`setImmediate()`要更及时的触发，但由于历史原因，这个几乎不会改变。因为如果交换则会破坏npm上的大部分包的逻辑，并且每天都有新的模块添加到npm仓库，这就意味着我们每等一天，就会发生更多潜在的故障。所以尽管这个命名让人困惑，但却不会被更改。

我们推荐开发者在所有场景下使用`setImmediate()`，因为它更容易被理解和推导。

## 为什么会使用process.nextTick()

主要有两个原因：

1. 允许用户处理错误、清理任何接下来用不到的资源，或者也可能在事件循环继续之前再请求一次
2. 当需要让回调在调用栈展开之后但在事件循环继续之前被执行

下面是一个符合用户预期的简单例子：

```javascript
const server = net.createServer();
server.on('connection', (conn) => { });

server.listen(8080);
server.on('listening', () => { });
```

假设`listen()`在事件循环开始时执行，但`listening`的回调是放在`setImmediate()`中执行的。在不传递域名的情况下，端口的绑定操作会立刻被触发，随着事件循环的进行，它必定会进入到poll阶段，这就意味着可能会出现在触发`listening`事件(check阶段)前就已经接收到连接从而触发`connection`事件(poll阶段)。

另一个例子是执行一个构造函数，这个构造函数继承自`EventEmiiter`，并且它想在构造函数中调用一个事件

```javascript
const EventEmitter = require('events');
const util = require('util');

function MyEmitter() {
  EventEmitter.call(this);
  this.emit('event');
}
util.inherits(MyEmitter, EventEmitter);

const myEmitter = new MyEmitter();
myEmitter.on('event', () => {
  console.log('an event occurred!');
});
```

这里你无法在构造函数中立刻触发这个事件，因为此时脚本还没有执行到你声明此事件回调的位置。因此，在构造函数中，可以使用`process.nextTick()`，并在它的回调中去触发这个事件，就像下面这样：

```javascript

```