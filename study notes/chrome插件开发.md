# Chrome插件开发

## 0. 涉及的知识点

![img](https://oss.kuyinyun.com/11W2MYCO/rescloud1/53fc9315d6474f8ba0c327105a407007.awebp)

![img](https://oss.kuyinyun.com/11W2MYCO/rescloud1/24d8b2984d8d4cc087348bc6b8f0c8d9.awebp)

### 参考资料

- [官方文档](https://developer.chrome.com/docs/extensions?hl=zh-cn)
- [官方样例库](https://github.com/GoogleChrome/chrome-extensions-samples)
- [Web API](https://developer.mozilla.org/zh-CN/docs/Web/API)



## 1. manifest.json

```json
{
  "name": "Getting Started Example",  // 插件名称
  "description": "Build an Extension!", // 插件描述
  "version": "1.0", // 版本
  "manifest_version": 3, // 指定插件版本，这个很重要，指定什么版本就用什么样的api，不能用错了
  "background": {
    "service_worker": "background.js" // 指定background脚本的路径
  },
  "action": {
    "default_popup": "popup.html", // 指定popup的路径
    "default_icon": {  // 指定popup的图标，不同尺寸
      "16": "/images/icon16.png",
      "32": "/images/icon32.png",
      "48": "/images/icon48.png",
      "128": "/images/icon128.png"
    }
  },
  "icons": { // 指定插件的图标，不同尺寸
    "16": "/images/icon16.png",
    "32": "/images/icon32.png",
    "48": "/images/icon48.png",
    "128": "/images/icon128.png"
  },
  "permissions": [],// 指定应该在脚本中注入那些变量方法，后文再详细说
  "options_page": "options.html",
  "content_scripts": [ // 指定content脚本配置
    {
      "js": [ "content.js"], // content脚本路径
      "css":[ "content.css" ],// content的css
      "matches": ["<all_urls>"] // 对匹配到的tab起作用。all_urls就是全部都起作用
    }
  ]
}

```

## 2. background.js

## 3. popup.html

当用户点击标签栏里插件图标时展示用户的界面

## 4. content_scripts

插件提供了内容脚本 Content Scripts（**CS**）的概念，当用户打开并访问某个网站时，浏览器将**CS**注入网站的文档里执行