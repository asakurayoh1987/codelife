

# Web Vitals Analysis

## 一、基础

### 1.1 核心页面指标

#### [LCP](https://web.dev/articles/lcp?hl=zh-cn)——衡量加载性能

<img src="https://oss.kuyinyun.com/11W2MYCO/rescloud1/9b6b6995f67641c0ad048534320ba064.svg" alt="LCP" style="width: 50%;" />

##### 概念

Largest Contentful Paint (LCP) 是一种[稳定的](https://web.dev/articles/vitals?hl=zh-cn#lifecycle) Core Web Vitals 指标，用于衡量[感知的加载速度](https://web.dev/articles/user-centric-performance-metrics?hl=zh-cn#types_of_metrics")用于标记网页加载时间轴中可能加载了网页主要内容的时间点

LCP 报告的是视口中可见最大[图片或文本块](https://web.dev/articles/lcp?hl=zh-cn#what-elements-are-considered)（img\svg\video\通过`url()`加载的背景图片\包括文本节点或其他内嵌文本元素子元素的块级元素）的呈现时间（相对于用户首次导航到相应网页的时间）

##### 数据收集

通过Performance API获取`name`为`largest-contentful-paint`的PerformanceEntry

```js
new PerformanceObserver((entryList) => {
  for (const entry of entryList.getEntries()) {
    console.log('LCP candidate:', entry.startTime, entry);
  }
}).observe({type: 'largest-contentful-paint', buffered: true});
```

##### 注意事项

1. 页面加载过程中，可能会多次触发LCP

2. 对于跨域请求的图片，需要在图片的响应头中添加`Timing-Allow-Origin`，否则是无法获取图片的渲染时间戳

   未添加`Timing-Allow-Origin`响应头时，获取不到`renderTime`

   <img src="https://oss.kuyinyun.com/11W2MYCO/rescloud1/0168247a12c446e492490074a585b854.png" alt="image-20240312152640813" style="zoom:50%;" />

   添加`Timing-Allow-Origin`响应头之后

   <img src="https://oss.kuyinyun.com/11W2MYCO/rescloud1/a1d3016d35144884b737d119eacef0b0.png" alt="image-20240312153517829" style="zoom:50%;" />

3. 指标与API的[区别](https://web.dev/articles/lcp?hl=zh-cn#differences-metric-api)



#### [FID](https://web.dev/articles/fid?hl=zh-cn)——衡量互动

<img src="https://oss.kuyinyun.com/11W2MYCO/rescloud1/3fadf6009fbc4536b9061c4d7bac69b7.svg" alt="FID" style="width:50%;" />

##### 概念

FID 衡量的是从用户**首次**与网页互动（即，点击链接、点按按钮或使用由 JavaScript 提供支持的自定义控件）到浏览器实际能够开始处理事件处理脚本以响应相应互动的时间（如浏览器正在解析javascript文件，此时是无法响应用户交互的）

首次输入延迟通常发生在[首次内容绘制 (FCP)](https://web.dev/articles/fcp?hl=zh-cn) 和[可交互时间 (TTI)](https://web.dev/articles/tti?hl=zh-cn) 之间，衡量的是**收到输入事件与主线程下次空闲之间的增量**

##### 数据收集

```js
new PerformanceObserver((entryList) => {
  for (const entry of entryList.getEntries()) {
    // 这个就是响应用户操作的时间
    const delay = entry.processingStart - entry.startTime;
    console.log('FID candidate:', delay, entry);
  }
}).observe({type: 'first-input', buffered: true});
```



#### [CLS](https://web.dev/articles/cls?hl=zh-cn)——衡量视觉稳定性

<img src="https://oss.kuyinyun.com/11W2MYCO/rescloud1/d059a104aed346ebbbcb6b03ec074af3.svg" alt="CLS" style="width:50%;" />

##### 概念

CLS 衡量的是网页生命周期内发生的每次意外布局偏移的**最大布局偏移得分**（比如一个未加载的图片，在加载后因其尺寸的变更将原先该位置的内容挤到下方），只要可见元素的位置从一个渲染的帧更改为下一个渲染帧，就会发生布局偏移

##### 数据收集

```js
new PerformanceObserver((entryList) => {
  for (const entry of entryList.getEntries()) {
    console.log('Layout shift:', entry);
  }
}).observe({type: 'layout-shift', buffered: true});
```

#### [INP](https://web.dev/articles/inp?hl=zh-cn)——衡量用户交互响应

![](https://oss.kuyinyun.com/11W2MYCO/rescloud1/b7877178285240cb853ad1940cf02a4c.svg)

##### 概念

INP 会在网页生命周期内观察用户与网页进行的所有点击、点按和键盘互动的延迟时间，并报告最长持续时间，并忽略离群值。

###### 互动的定义：

![](https://oss.kuyinyun.com/11W2MYCO/rescloud1/eef479e541c7450688180e2c81feafdb.svg)

互动是指在同一逻辑用户手势期间触发的一组事件处理脚本。例如，触摸屏设备上的“点按”互动包括多个事件，如 `pointerup`、`pointerdown` 和 `click`。互动可由 JavaScript、CSS、表单元素等内置浏览器控件或这些元素的组合促成。

互动的主要驱动因素通常是 JavaScript，但浏览器确实会通过并非由 JavaScript 提供支持的控件（例如复选框、单选按钮和由 CSS 提供支持的控件）提供互动性

**系统会在用户离开页面时计算 INP，从而生成单个值，代表网页在其整个生命周期内的整体响应情况**

###### 与FID不同之处

FID仅测量了页面上首次互动的输入延迟；INP 通过考虑所有页面互动（从输入延迟到运行事件处理程序所需的时间，再到浏览器绘制下一帧）来改进 FID

##### 数据收集

```js
const observer = new PerformanceObserver((list) => {
  list.getEntries().forEach((entry) => {
    // Alias for the total duration.
    const duration = entry.duration;
    // Calculate the time before processing starts.
    const delay = entry.processingStart - entry.startTime;
    // Calculate the time to process the interaction.
    const lag = entry.processingEnd - entry.processingStart;

    console.log(`INP Duration: ${duration}`);
    console.log(`INP Delay: ${delay}`);
    console.log(`Event handler duration: ${lag}`);
  });
});

// Call the Observer.
observer.observe({ type: "event", buffered: true });

```



### 1.2 非核心指标

#### [TTFB](https://web.dev/articles/ttfb?hl=zh-cn)——首个响应字节

<img src="https://oss.kuyinyun.com/11W2MYCO/rescloud1/acf75d1388f644538f1cafda3e9bc4e4.svg" style="width:50%;" />

#### 概念

![ResourceTiming](https://oss.kuyinyun.com/11W2MYCO/rescloud1/35c1aba445ce4a70a969d0790d8dca0c.svg)

TTFB 是以下请求阶段的总和：

- 重定向时间
- Service Worker 启动时间（如果适用）
- DNS 查找
- 连接和 TLS 协商
- 请求，直到响应的第一个字节到达（即**responseStart**）

缩短连接设置时间和后端的延迟时间将有助于降低 TTFB

##### 数据收集

```js
new PerformanceObserver((entryList) => {
  const [pageNav] = entryList.getEntriesByType('navigation');

  console.log(`TTFB: ${pageNav.responseStart}`);
}).observe({
  type: 'navigation',
  buffered: true
});
```

#### [FCP](https://web.dev/articles/fcp?hl=zh-cn)——衡量加载速度

![](https://oss.kuyinyun.com/11W2MYCO/rescloud1/54e215a159b142388684278574c60447.svg)

##### 概念

FCP 衡量的是从用户首次导航到相应网页到该网页的任何部分呈现在屏幕上所用的时间。对于此指标，“内容”是指文本、图片（包括背景图片）、`<svg>` 元素或非白色 `<canvas>` 元素

##### 数据收集

```js
new PerformanceObserver((entryList) => {
  for (const entry of entryList.getEntriesByName('first-contentful-paint')) {
    console.log('FCP candidate:', entry.startTime, entry);
  }
}).observe({type: 'paint', buffered: true});
```

### 1. 3底层API

1. LCP——[LargestContentfulPain](https://developer.mozilla.org/en-US/docs/Web/API/LargestContentfulPaint)，通过传递type为`largest-contentful-paint`给PerformanceObserver

1. FID——[PerformanceEventTiming](https://developer.mozilla.org/en-US/docs/Web/API/PerformanceEventTiming)，通过传递type为`first-input`给PerformanceObserver

1. CLS——[LayoutShift](https://developer.mozilla.org/en-US/docs/Web/API/LayoutShift)，通过传递type为`layout-shift`给PerformanceObserver

1. TTFB——[PerformanceNavigationTiming](https://developer.mozilla.org/en-US/docs/Web/API/PerformanceNavigationTiming)，通过传递type为`navigation`给PerformanceObserver，拿到整个navigation各阶段的耗时

1. FCP——[PerformancePaintTiming](https://developer.mozilla.org/en-US/docs/Web/API/PerformancePaintTiming)，通过传递type为`paint`给PerformanceObserver，然后根据name属性过滤`first-contentful-paint`

1. INP——[PerformanceEventTiming](https://developer.mozilla.org/en-US/docs/Web/API/PerformanceEventTiming)，通过传递type为`event`给PerformanceObserver，计算duration最大的eventEntry

1. 资源请求——[PerformanceResourceTiming](https://developer.mozilla.org/en-US/docs/Web/API/PerformanceResourceTiming)，通过传递type为`resource`给PerformanceObserver，然后根据initiatorType来进行区分，可以获取诸如图片资源、AJAX请求等耗时

   - [各种指标的计算方法](https://developer.mozilla.org/en-US/docs/Web/API/PerformanceResourceTiming#typical_resource_timing_metrics)

   - 关于[Timing-Allow-Origin](https://developer.mozilla.org/en-US/docs/Web/API/PerformanceResourceTiming#cross-origin_timing_information)

   - 比如ajax请求的耗时统计

     ```js
     new PerformanceObserver((entryList) => {
       for (const entry of entryList.getEntries()) {
         if(entry.initiatorType==='xmlhttprequest') console.log(entry);
       }
     }).observe({type: 'resource', buffered: true});
     ```

1. Long-Task——[PerformanceLongTaskTiming](https://developer.mozilla.org/en-US/docs/Web/API/PerformanceLongTaskTiming)，通过传递type为`longtask`给PerformanceObserver，**但暂无有意义的数据可供收集**

   ```js
   const observer = new PerformanceObserver((list) => {
     for (const entry of entryList.getEntries()) {
       console.log(entry);
     }
   }).observe({ type: "longtask", buffered: true });
   ```

1. Long-Animation-Frame——PerformanceLongAnimationFrameTiming，但存在兼容性问题，大多数浏览器都不支持

   ```js
   new PerformanceObserver((entryList) => {
     for (const entry of entryList.getEntries()) {
       console.log(entry);
     }
   }).observe({type: 'long-animation-frame', buffered: true});
   ```

   

## 二、web-vitals包

### 2.1 基本使用

```ts
import { send } from '@/perf/analytics';
import { onFCP, onLCP, onFID, onCLS } from 'web-vitals/attribution';

export default function init() {
  onFCP(send);
  onLCP(send);
  onFID(send);
  onCLS(send);
}
```

### 2.2 关于上报

#### 可能不会上报的指标

- FID及INP只有在用户与页面有交互时才会上报（移动端触及屏幕）
- CLS、FCP、FID、LCP在页面以后台方式加载时不会上报

#### 可能会上报多次的指标

- CLS及INP会在页面的`visibilityState`每次变更为`hidden`时上报
- 当页面通过“前进”、“后退”这样的操作从缓存里恢复页面时，所有的指标都会再上报一次

#### 上报API

##### 1. navigator.sendBeacon

将少量数据以异步的方式发送POST请求至服务器，专门用于发送分析数据，主要解决老方法中的**不足之处**，比如通过XMLHttpRequest发送数据时，如果页面被关闭了，则浏览器会取消请求的发送

```js
if (navigator.sendBeacon) {
  const blob = new Blob([JSON.stringify(data)], {
    type: 'application/json',
  });
  navigator.sendBeacon(reqUrl, blob);
}
```

[更多介绍](https://developer.mozilla.org/en-US/docs/Web/API/Navigator/sendBeacon)

##### 2. fetch with keepalive

当参数中添加`keepalive: true`，其实效果与`navigator.sendBeacon`类似，不会在页面退出时就取消请求的发送 

```js
if (fetch) {
  fetch(reqUrl, {
    method: 'POST',
    headers: {
      'Content-Type': 'application/json',
    },
    // 注意这里的keepalive
    keepalive: true,
    body: JSON.stringify(data),
  });
}
```

[更多介绍](https://developer.mozilla.org/en-US/docs/Web/API/fetch#keepalive)



