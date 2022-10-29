# Event Loop

## node与浏览器中Event Loop的不同

### 1. Node

### 1.1 node中的事件循环阶段：

```
     +------------------------------------+
     |                                    |
+--->+              timer                 | // 执行满足条件的setTimeout、setInterval回调
|    |                                    |
|    +----------------+-------------------+
|                     |
|                     v
|    +----------------+-------------------+
|    |                                    |
|    |         pending callbacks          | // 上一次事件循环中被挂起的任务
|    |                                    |
|    +----------------+-------------------+
|                     |
|                     v
|    +----------------+-------------------+
|    |                                    |
|    |            idle,prepare            | // 可忽略，一般场景用不到
|    |                                    |
|    +----------------+-------------------+
|                     |
|                     v
|    +----------------+-------------------+
|    |                                    |
|    |                poll                | // I/O轮询，会阻塞并等待I/O事件，等待超时时间是计算得来的
|    |                                    |
|    +----------------+-------------------+
|                     |
|                     v
|    +----------------+-------------------+
|    |                                    |
|    |               check                | // 执行setImmedia回调
|    |                                    |
|    +----------------+-------------------+
|                     |
|                     v
|    +----------------+-------------------+
|    |                                    |
+----+          close callbacks           | // 执行一些onclose事件的回调
     |                                    |
     +------------------------------------+
```

### 1.2 setTimout vs setImmediate

* Demo1

```javascript
setTimeout(() => {
  console.log('setTimout callback');
});

setImmediate(() => {
  console.log('setImmedia callback');
});
```

执行6次的结果如下：

```
// 第1次
setImmedia callback
setTimout callback

// 第2次
setTimout callback
setImmedia callback

// 第3次
setImmedia callback
setTimout callback

// 第4次
setImmedia callback
setTimout callback

// 第5次
setTimout callback
setImmedia callback
```