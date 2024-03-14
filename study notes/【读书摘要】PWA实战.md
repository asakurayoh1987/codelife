# PWA实战：面向下一代Progressive Web App

## Part I 定义PWA

### 第1章 理解PWA

> 本章将介绍是什么让Web应用成为渐进式的，以及如何释放浏览器中已经存在的强大功能

#### 1.1 PWA有什么优势

PWA最棒的一点就是它真的是渐进式的

#### 1.2 PWA基础

PWA最棒的一点是可以渐进增强现有的Web应用

#### 1.3 Service Worker：PWA的关键

##### 1.3.1 理解Service Worker

Service Worker可以让你全权控制网站发起的每一个请求，这为许多不同的使用场景开辟了可能性

Service Worker的特点：

1. 运行在自己的全局脚本上下文中
2. 不绑定到具体的网页
3. 无法修改网页中的元素，因为它无法访问DOM（也不能访问localStorage之类的功能）
4. 只能使用HTTPS
5. 事件驱动

##### 1.3.2 Service Worker生命周期

当第一次加载页面时，Service Worker还没有激活，所以它不会处理任何请求。**只有当它安装和激活后，才能控制在其范围内的一切**。这意味着，只有刷新页面或者导航到另一个页面，Service Worker内的逻辑才会启动

##### 1.3.3 Service Worker基础示例

##### 1.3.4 案全考虑

只能使用https

#### 1.4 性能洞察：Flipkart





