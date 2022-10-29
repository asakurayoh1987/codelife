# Timers in Node.js and beyond

node.js中的计时器模块包括可以在一段时间后执行代码的功能。计时器模块不需要通过require来导入，因为所有方法都可全局使用来模拟浏览器中的Javascript API，为了全面理解计时器何时会被执行，可以阅读下指南中关于node.js[事件循环的部分](https://nodejs.org/en/docs/guides/event-loop-timers-and-nexttick/)

## 使用node.js来控制时间线

node.js的api中提供了几个用来调度代码来在将来的某个时间点执行。下面这几个方法看上去很相似，因为它们在绝大多数浏览器中都是可用的，但node.js实际上对这些方法提供了自己的实现。计时器与系统结合得非常紧密，并且尽管这些API与浏览器中对应的API一样，但在实际上是有一些不一样的地方的

### “当我这样说时”(在什么的时候) 执行~setTimeout()

`setTimeout()`可用于在指定的时间(毫秒)后执行代码，这个方法与浏览器中的`window.setTimout()`相似，但不能像浏览器中那样执行一个代码的字符串形式。`setTimeout()`接受一个待执行函数作为第一个参数以及一个表示延迟多少毫秒执行的数字为第二个参数。其余额外的参数会被传递给待执行的函数，这里是一个例子。

```javascript
function myFunc(arg) {
  console.log(`arg was => ${arg}`);
}

setTimeout(myFunc, 1500, 'funky');
```
上面的函数`myFunc()`会在1500毫秒后尽可能早的去执行。这个时间间隔并不是一个完成准确的时间，因为其它正在执行的代码会阻塞或保持事件循环从而将计时器回调的执行推迟，唯一的保证是计时器不会早于这个指定的时间执行

`setTimeout()`方法会返回一个`Timeout`对象用于引用刚才设定的计时器，它可以用于撤消这个计时器(通过`clearTimeout()`)，同时也可用于修改执行的行为(参见下面将要提到的`unref()`)

### “在这之后立刻” 执行~setImmediate()

`setImmediate()`会在当前事件循环的结束时执行代码。代码会当前事件循环的异步I/O操作之后并且在下一次事件循环的timers阶段之前执行。这段代码的执行可以认为是“在这之后立刻”发生。这也就意味着，在`setImmediate()`之后的代码会在作为参数传递`setImmediate()`的代码之前执行

`setImmediate()`的第一个参数是将要执行的函数，后续的任务参数会作为参数传递给这个函数，下面是一个例子

```javascript
console.log('before immediate');

setImmediate((arg) => {
  console.log(`executing immediate: ${arg}`);
}, 'so immediate');

console.log('after immediate');
```

上面传递给`setImmediate()`的代码会在所有可执行代码之后执行，控制台的输出结果如下：

```
before immediate
after immediate
executing immediate: so immediate
```

`setImmediate()`会返回一个`Immediate`对象，可以被用来撤消计时器的调度(`clearImmediate()`)

*注意：不要将`setImmediate()`与`process.nextTick()`混淆了，它们在使用方面有着较大的不同，首先，`process.nextTick()`会在任意设置的`Immediate`之前执行，同时也会在任何调度的I/O操作之前执行。其次，`process.nextTick()`是不可撤消的，也就意味着一旦代码通过`process.nextTick()`进行调度，其执行是不可撤消的，就像一个普通函数一样，可以在[这篇](https://nodejs.org/en/docs/guides/event-loop-timers-and-nexttick/#process-nexttick)指南里了解`process.nextTick()`的详情*

### "无限循环" 执行~setInterval()

如果有一个代码块需要多次执行，可以用`setInterval()`来调度这段代码，此方法的第一参数是待执行的函数，第二参数是一个以毫秒为单位的时间延迟，函数会被无限次的执行，每次按第二参数指定的延迟时间来执行。与`setTimout()`一样，附加的参数会被传递给要执行的函数作为参数，同时也如`setTimeout()`一样，这个延迟也是无法准确保证的，因为可能会有一些操作会占用事件循环，因此只能将其视为一个近似的延迟。下面是一个调用的例子

```javascript
function intervalFunc() {
  console.log('Cant stop me now!');
}

setInterval(intervalFunc, 1500);
```

在上面的代码里，`intrvalFunc`将会每隔1500毫秒执行一次，直接停止(下文会提及如何停止)。如`setTimeout()`和`setInterval()`一样，它也会返回一个`Timeout`对象，用于引用和修改这个计时器。

## 清除计时器

如果需要停止`Timeout`和`Immediate`对象时该怎么做呢？`setTimeout()`、`setImmediate()`和`setInterval()`返回的计时器对象可以被用来引用通过这些方法设置的`Timeout`对象和`Immediate`对象，通过将这些对象传递给对应的`clear`方法，这些对象的执行就可以被完全停止。这些对应的`clear`方法是`clearTimout()`、`clearImmediate()`和`clearInterval()`，下面是对应的调用例子：

```javascript
const timeoutObj = setTimeout(() => {
  console.log('timeout beyond time');
}, 1500);

const immediateObj = setImmediate(() => {
  console.log('immediately executing immediate');
});

const intervalObj = setInterval(() => {
  console.log('interviewing the interval');
}, 500);

clearTimeout(timeoutObj);
clearImmediate(immediateObj);
clearInterval(intervalObj);
```

## 最后来谈谈Timeouts

`Timout`对象是通过`setTimout()`和`setInterval()`返回的。它提供了两个方法来操作`Timeout`的行为，这两个方法就是`unref()`和`ref()`。对于一个通过set方法调度的`Timeout`对象，则可以在其上调用`unref()`方法，它会对对行为产生轻微的影响(调用后，这个激活的`Timeout`对象就不会要求事件循环来使它一直保持激活状态，如果没有其它活动保持事件循环的运行，则进程就会在计时器回调被触发前退出 )，并且如果它是最后执行的代码，`Timeout`对象将不会被执行，因为`Timeout`不会让进程一直存活着等待它的执行。

类似的方式，在一个已经调用了`unref()`的对象上，可以通过其上调用`ref()`来移除`unref()`的行为，它将恢复这个计时器对象的执行。但需要注意，出于性能原因，这并不能完全恢复初始行为。下面的例子

```javascript
const timerObj = setTimeout(() => {
  console.log('will i run?');
});

// if left alone, this statement will keep the above
// timeout from running, since the timeout will be the only
// thing keeping the program from exiting
timerObj.unref();

// we can bring it back to life by calling ref() inside
// an immediate
setImmediate(() => {
  timerObj.ref();
});
```

## 进一步深入事件循环

关于事件循环和计时器，本指南中还有很多未提及，想要了解更多关于node.js事物循环的内部机制以及计时在运行时是如何操作的，可以阅读这篇指南：[The Node.js EventLoop, Timers, and process.nextTick()](https://nodejs.org/en/docs/guides/event-loop-timers-and-nexttick/)

