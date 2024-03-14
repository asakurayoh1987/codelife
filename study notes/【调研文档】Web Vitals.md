

# Web Vitals Analysis

## 一、基础

### 1.1 核心页面指标

#### [LCP](https://web.dev/articles/lcp?hl=zh-cn)——衡量加载性能

<img src="../media/largest-contentful-paint-ea2e6ec5569b6.svg" alt="LCP" style="width: 50%;" />

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

   <img src="../media/image-20240312152640813.png" alt="image-20240312152640813" style="zoom:50%;" />

   添加`Timing-Allow-Origin`响应头之后

   <img src="../media/image-20240312153517829.png" alt="image-20240312153517829" style="zoom:50%;" />

3. 指标与API的[区别](https://web.dev/articles/lcp?hl=zh-cn#differences-metric-api)



#### [FID](https://web.dev/articles/fid?hl=zh-cn)——衡量互动

<img src="../media/first-input-delay-thresho-4329fd6d1129a.svg" alt="FID" style="width:50%;" />

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

<img src="../media/cumulative-layout-shift-t-5d49b9b883de4.svg" alt="CLS" style="width:50%;" />

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

#### [INP](https://web.dev/articles/inp?hl=zh-cn)

Interaction to Next Paint (INP)，将于2024年3月12日取代FID，它使用Event Timing API的数据评估响应能力。INP 较低意味着网页始终能够快速响应大多数用户互动

### 1. 2底层API

#### Event Timing API

#### Performance API





## 二、web-vitals包

### 2.1 基本使用

#### From CDN



### 2.2 关于上报

#### 关于LCP的上报

LCP的上报需要用户与页面有交互、切换Tab、退出页面（unload）

#### 关于`navigator.sendBeacon`

