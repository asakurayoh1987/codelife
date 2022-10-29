## 安装源的配置

正常安装时，在下载electron包时非常慢，可以通过如下方式使用淘宝的镜像源

### 1. 通过npm命令配置

```bash
# 全局设置下载源
npm config set registry https://registry.npm.taobao.org/
# 下载node源码加速
npm config set disturl https://npm.taobao.org/mirrors/node
# electron包下载地址注册位淘宝的镜像
npm config set ELECTRON_MIRROR https://npm.taobao.org/mirrors/electron/
```

### 2. 通过项目下的配置文件

通过在项目下创建.npmrc文件并写入如下配置

```
registry=https://registry.npm.taobao.org/
disturl=https://npm.taobao.org/mirrors/node
ELECTRON_MIRROR=https://npm.taobao.org/mirrors/electron/
```

对于mac系统，尝试了下，需要将ELECTRON_MIRROR写入到系统的环境变量中

## WSL2下开发环境搭建

### 1. 系统库安装

```bash
# system libraries needed for electron
sudo apt install libnss3-dev libgdk-pixbuf2.0-dev libgtk-3-dev libxss-dev
```

### 2. X Server安装

可参考[这里](https://www.beekeeperstudio.io/blog/building-electron-windows-ubuntu-wsl2)

要求以管理员权限启动power shell，如果是使用terminal来启动power shell，默认并不是以管理员权限，会导致安装失败，可以通过[这里](https://blog.csdn.net/weixin_39858881/article/details/107026065)提到的方法在terminal中添加power shell的管理员启动模式，以管理员权限运行power shell后，执行如下命令即可安装vcxsrv

```bash
choco install vcxsrv
```

### cli

[vue](https://vue3js.cn/)+[vue-cli](https://cli.vuejs.org/)+[vue cli plugin electron builder](https://github.com/nklayman/vue-cli-plugin-electron-builder)

### 文章

* [@vue/cli4创建electron-vue项目踩坑之路](https://juejin.im/post/6844904119837130760)
* [Vite源码分析](https://vite-design.surge.sh/)
* [Awesome Electron](https://github.com/sindresorhus/awesome-electron#documentation)
* [electron-wiki](https://github.com/poetries/electron-wiki)

### 库或工具

* [electron-vue](https://simulatedgreg.gitbooks.io/electron-vue/content/cn/)：electron+vue的脚手架
* [vue-cli-plugin-electron-builder](https://github.com/nklayman/vue-cli-plugin-electron-builder)：同上
* [vuex-electron](https://github.com/vue-electron/vuex-electron#installation)：在electron中使用vuex，同步两个redener之间的状态
* [wait-on](https://github.com/jeffbski/wait-on)：用于保证electron在前端页面可访问之后再启动
* [concurrently](https://github.com/kimmobrunfeldt/concurrently)：同时启动多条命令
* [@rollup/plugin-commonjs](https://github.com/rollup/plugins/tree/master/packages/commonjs)：rollup插件，用来解析cjs的模块
* [@rollup/plugin-node-resolve](https://github.com/rollup/plugins/tree/master/packages/node-resolve)：rollup插件，如果查找合适的模块（比如配置项中指定`browser`为`true`，则查找包时会根据包下package.json中的属性`browser`指定的文件）
* [@rollup/plugin-alias](https://github.com/rollup/plugins/tree/master/packages/alias)：别名，同webpack里的alias
* [rollup-plugin-esbuild](https://github.com/egoist/rollup-plugin-esbuild)：TS/ESNext to ES6 compilers and minifier
* [Node.js ECMAScript compatibility tables](https://github.com/williamkapke/node-compat-table)：node.js版本对ES特性的支持情况
* [electron-is-dev](https://github.com/sindresorhus/electron-is-dev)：判断当前环境是否为开发环境
* [electron-store](https://github.com/sindresorhus/electron-store)
* [electron-rebuild](https://github.com/electron/electron-rebuild)
* [robotjs](https://github.com/octalmage/robotjs)
* [electron-builder](https://github.com/electron-userland/electron-builder/)

### 示例

* [cnode-electron-demo](https://github.com/vipsunwei/cnode-electron-demo)
* [electron-vue-vite](https://github.com/caoxiemeihao/electron-vue-vite)
* [electron-vue3-vite](https://github.com/laterly/electron-vue3-vite)：比较新的模板，包括vue-router及vuex的使用
* [vite-electron-quick](https://github.com/MangoTsing/vite-electron-quick)：比较新的模板，同上，代码相对更完整一些
* [electron-vue-next](https://github.com/ci010/electron-vue-next)：比较新的模板
* [electron-vue-music](https://github.com/SmallRuralDog/electron-vue-music)：仿QQ音乐播放器
* [newbee-mall-vue3-app](https://github.com/newbee-ltd/newbee-mall-vue3-app)：基于vite的商城项目

### API

#### BrowserWindow

##### 属性

* simpleFullscreen：macOS上没测试出效果
* kiosk：所谓的展台模式？设置为true时，窗口全屏，且没有菜单，等于无法关闭窗口，除非使用快捷键直接退出
* opacity与transparent：前者是让整个窗体和内容都变得半透明，后者只影响窗体，内容不会变透明
* tabbingIdentifier：
* 