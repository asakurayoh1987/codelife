# Vue.js 源码学习

## 一、准备工作

- 仓库地址：https://github.com/vuejs/vue
- npm script中的dev修改，添加`--sourcemap`
  - "dev": "rollup -w -c scripts/config.js --sourcemap --environment

## 二、代码入口位置

1. 通过debug模式断点定位

   - 在examples下创建目录ycyu，并添加index.html，并添加如下内容

     ```html
     <!DOCTYPE html>
     <html lang="en">
     
     <head>
       <meta charset="UTF-8">
       <meta http-equiv="X-UA-Compatible" content="IE=edge">
       <meta name="viewport" content="width=device-width, initial-scale=1.0">
       <title>ycyu</title>
     </head>
     
     <body>
       <script src="../../dist/vue.js"></script>
       <div id="app">{{msg}}</div>
       <script>
         debugger
         new Vue({
           data: {
             msg: "welcome!!"
           }
         }).$mount('#app')
       </script>
     </body>
     
     </html>
     ```

   - 在chrome中单步调度，即可查看Vue构造函数定义的位置：src/core/instance/index.js中导出的Vue构造函数

2. 通过查看rollup.config.js

   比如npm scripts中的dev脚本，指定了rollup的配置文件scripts/config.js，并通过`--environment TARGET:web-full-dev`指定了环境变量TARGET为web-full-dev，对应到config.js中的逻辑，可知得到的配置文件如下：

   ```javascript
     'web-full-dev': {
       entry: resolve('web/entry-runtime-with-compiler.js'),
       dest: resolve('dist/vue.js'),
       format: 'umd',
       env: 'development',
       alias: { he: './entity-decoder' },
       banner
     },
   ```

   结合`resolve方法`可以找到源文件位置为`src/platforms/web/entry-runtime-with-compiler.js`，通过该文件可以进一步跟踪到`src/platforms/web/runtime/index.js`，并继续找到`src/core/index.js`，最后找到`src/core/instance/index.js`中，至此，与方法一中得到的结果相同

3. 关于代码中用到的一些alias，比如`import config from 'core/config'`，在rollup编译时会通过`scripts/alias.js`中找到真实的文件，如下：

   ```javascript
   const path = require('path')
   
   const resolve = p => path.resolve(__dirname, '../', p)
   
   module.exports = {
     vue: resolve('src/platforms/web/entry-runtime-with-compiler'),
     compiler: resolve('src/compiler'),
     core: resolve('src/core'),
     shared: resolve('src/shared'),
     web: resolve('src/platforms/web'),
     weex: resolve('src/platforms/weex'),
     server: resolve('src/server'),
     sfc: resolve('src/sfc')
   }
   ```

   但我们看代码时，就无法进行直接跳转到相应模块了，此时需要安装“Flow Language Support”这个插件，然后在`.vscode/settings.json`中添加如下配置，重启vscode后即可实现代码跳转

   ```json
   {
     "javascript.validate.enable": false,
     "flow.useLSP": false
   }
   ```

## 三、runtime版本

先从`src/platforms/web/entry-runtime.js`版本看，这个是大家平时最常用的版本，只包含运行时

