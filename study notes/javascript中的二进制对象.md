# javascript中的二进制对象

[谈谈JS二进制：File、Blob、FileReader、ArrayBuffer、Base64](https://juejin.cn/post/7148254347401363463)

先引用一张图：

![img](../media/aa83846a988842ad8656a68331207bef~tplv-k3u1fbpfcp-zoom-in-crop-mark.awebp)

![img](../media/de779e1b97f24013a387890fd63e983e~tplv-k3u1fbpfcp-zoom-in-crop-mark.awebp)

在 JavaScript 中，有两个函数被分别用来处理解码和编码 *base64* 字符串：

- `atob()`：解码，解码一个 Base64 字符串；
- `btoa()`：编码，从一个字符串或者二进制数据编码一个 Base64 字符串。
