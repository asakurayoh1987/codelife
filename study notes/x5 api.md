# x5内核API

## 缓存清理

```javascript
// 清理 [Cookie]
window.ProxyStatus.clearCookieAndCache(1);
// 清理 [文件缓存]
window.ProxyStatus.clearCookieAndCache(2);
// 清理 [广告过滤缓存]
window.ProxyStatus.clearCookieAndCache(4);
// 清理 [DNS缓存]
window.ProxyStatus.clearCookieAndCache(8);
```