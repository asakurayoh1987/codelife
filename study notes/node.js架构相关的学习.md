# Node.js学习

## 一、文章列表

* [stackoverflow上关于node.js架构的提问](https://stackoverflow.com/questions/36766696/which-is-correct-node-js-architecture/37512766#37512766)
* [libuv设计概述](http://docs.libuv.org/en/v1.x/design.html)
* [libuv设计概述译](https://segmentfault.com/a/1190000005873917)
* [node.js事件循环官方文档](https://nodejs.org/en/docs/guides/event-loop-timers-and-nexttick/)
* [Don't Block the Event Loop (or the Worker Pool)](https://nodejs.org/en/docs/guides/dont-block-the-event-loop/)
* [Node进程通信](https://segmentfault.com/a/1190000007343993)

## 二、知识点

### 1. 事件循环

> libuv中的事件循环概览：

![libuv中事件循环流程](/Users/ycyu/Documents/Typora Notes/media/libuv-event-loop.png)

>The loop concept of ‘now’ is updated. The event loop caches the current time at the start of the event loop tick in order to reduce the number of time-related system calls.
If the loop is alive an iteration is started, otherwise the loop will exit immediately. So, when is a loop considered to be alive? If a loop has active and ref’d handles, active requests or closing handles it’s considered to be alive.
Due timers are run. All active timers scheduled for a time before the loop’s concept of now get their callbacks called.
Pending callbacks are called. All I/O callbacks are called right after polling for I/O, for the most part. There are cases, however, in which calling such a callback is deferred for the next loop iteration. If the previous iteration deferred any I/O callback it will be run at this point.
Idle handle callbacks are called. Despite the unfortunate name, idle handles are run on every loop iteration, if they are active.
Prepare handle callbacks are called. Prepare handles get their callbacks called right before the loop will block for I/O.
Poll timeout is calculated. Before blocking for I/O the loop calculates for how long it should block. These are the rules when calculating the timeout:

	* If the loop was run with the UV_RUN_NOWAIT flag, the timeout is 0.
	* If the loop is going to be stopped (uv_stop() was called), the timeout is 0.
	* If there are no active handles or requests, the timeout is 0.
	* If there are any idle handles active, the timeout is 0.
	* If there are any handles pending to be closed, the timeout is 0.
	* If none of the above cases matches, the timeout of the closest timer is taken, or if there are no active timers, infinity.

>The loop blocks for I/O. At this point the loop will block for I/O for the duration calculated in the previous step. All I/O related handles that were monitoring a given file descriptor for a read or write operation get their callbacks called at this point.
Check handle callbacks are called. Check handles get their callbacks called right after the loop has blocked for I/O. Check handles are essentially the counterpart of prepare handles.
Close callbacks are called. If a handle was closed by calling uv_close() it will get the close callback called.
Special case in case the loop was run with UV_RUN_ONCE, as it implies forward progress. It’s possible that no I/O callbacks were fired after blocking for I/O, but some time has passed so there might be timers which are due, those timers get their callbacks called.
Iteration ends. If the loop was run with UV_RUN_NOWAIT or UV_RUN_ONCE modes the iteration ends and uv_run() will return. If the loop was run with UV_RUN_DEFAULT it will continue from the start if it’s still alive, otherwise it will also end.
Important libuv uses a thread pool to make asynchronous file I/O operations possible, but network I/O is always performed in a single thread, each loop’s thread.

> node.js中事件循环各阶段：

```
     +------------------------------------+
     |                                    |
+--->+              timer                 |
|    |                                    |
|    +----------------+-------------------+
|                     |
|                     v
|    +----------------+-------------------+
|    |                                    |
|    |         pending callbacks          |
|    |                                    |
|    +----------------+-------------------+
|                     |
|                     v
|    +----------------+-------------------+
|    |                                    |
|    |            idle,prepare            |
|    |                                    |
|    +----------------+-------------------+
|                     |
|                     v
|    +----------------+-------------------+
|    |                                    |
|    |                poll                |
|    |                                    |
|    +----------------+-------------------+
|                     |
|                     v
|    +----------------+-------------------+
|    |                                    |
|    |               check                |
|    |                                    |
|    +----------------+-------------------+
|                     |
|                     v
|    +----------------+-------------------+
|    |                                    |
+----+          close callbacks           |
     |                                    |
     +------------------------------------+

```

> **<u>Each phase has a FIFO queue of callbacks to execute</u>**. While each phase is special in its own way, generally, when the event loop enters a given phase, it will perform any operations specific to that phase, then execute callbacks in that phase's queue until the queue has been exhausted or the maximum number of callbacks has executed. **<u>When the queue has been exhausted or the callback limit is reached, the event loop will move to the next phase, and so on</u>**.

#### 1.1 Poll阶段

Poll阶段主要做两件事：
1. 计算需要阻塞多久来进行I/O轮询
2. 处理Poll队列中的事件

```
         +-----------------------+        +------------------------+          +---------------------------+
         |                       |  Yes   |                        |   Yes    |                           |
         | poll queue is empty?  +------->+ have timers shceduled? +--------->+ wrap back to timers phase |
         |                       |        |                        |          |                           |
         +-----------+-----------+        +------------+-----------+          +---------------------------+
                     |                                 |
                     | No                              | No
                     v                                 v
         +-----------+------------+          +---------+---------+
         |                        |          |                   |
         | iterate the queue and  |          | have scheduled by +----------------+
+------->+ synchronously execute  |          | setImmediate()?   |                |
|        | callback               |          |                   |                |
|        |                        |          +---------+---------+                | No
|        +-----------+------------+                    |                          |
| No                 |                                 | Yes                      |
|                    v                                 v                          |
|           +--------+--------+              +---------+---------+   +------------v-------------+
|           |                 |              |                   |   |                          |
|           | queue exhausted |              | end poll phase    |   | wait for callbacks to    |
+-----------+       or        |              | enter check phase |   | be added to queue,then   |
            | reached limit?  |              |                   |   | execute them immediately |
            |                 |              +-------------------+   |                          |
            +--------+--------+                                      +--------------------------+
                     |
                     | Yes
                     v
           +---------+---------+
           |                   |
           | enter check phase |
           |                   |
           +-------------------+

```

#### 1.2 process.nextTick() vs setImmediate()

* process.nextTick() fires immediately on the same phase
* setImmediate() fires on the following iteration or 'tick' of the event loop

