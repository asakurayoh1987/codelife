# create-vue源码学习

## 一、前置知识

### 1. npm cli

npm init用法示例，更全的参见[官网](https://docs.npmjs.com/cli/v8/commands/npm-init)

- npm init foo 等价于 npm exec create-foo
- npm init @usr/foo 等价于 npm exec @usr/create-foo
- npm init @usr 等价于 npm exec @usr/create

### 2. package.json配置项

- type: "module" : 表示默认使用ES Module语法来加载.js文件，除此之外，还可以通过文件后缀名来指定模块语法，.mjs采用ES Module语法，.cjs采用CommonJS的模块语法，更多可以参见[这里](https://www.ruanyifeng.com/blog/2020/08/how-nodejs-use-es6-module.html)及[官方文档](https://nodejs.org/dist/latest-v16.x/docs/api/packages.html#determining-module-system)

### 3. 第三方依赖

- [prompts](https://www.npmjs.com/package/prompts)-处理cli中的问题选项交互
- [enquirer](https://github.com/enquirer/enquirer)-同上
- [zx](https://github.com/google/zx)-以js脚本的方式写shell脚本
- [minimist](https://www.npmjs.com/package/minimist)-命令行参数读取

## 二、代码解读

通过package.json中的scripts`"build": "zx ./scripts/build.mjs",`及bin中`"create-vue": "outfile.cjs"`可知，项目通过zx直接执行build.mjs脚本来生成基于commonjs规范的outfile.cjs来作为命令行执行时的入口文件，查看build.mjs中配置可以看到`entryPoints: ['index.js']`，从来找到代码的入口文件——**项目根目录下的index.js**

通过快捷键cmd+k+1，折叠所有代码段，可以看到代码的执行是从init方法开始的，整个代码的执行分为以下几个步骤：~~PS：从init方法的签名可知这里使用了top level async/await特性，所以前面build.mjs中的配置target才会是node14，因为从这个版本起才支持此特性~~

- 参数解析及用户输入项收集

  这里主要是利用minimist读取命令行入参，通过prompts进行交互式的收集用户输入的选项，为后续构建项目模板提供依据，同时创建初始的package.json文件

  ```javascript
    const pkg = { name: packageName, version: '0.0.0' }
    fs.writeFileSync(path.resolve(root, 'package.json'), JSON.stringify(pkg, null, 2))
  ```

- 根据前文收集到的配置项，“渲染”模板

  ```javascript
  const templateRoot = path.resolve(__dirname, 'template')
  const render = function render(templateName) {
    const templateDir = path.resolve(templateRoot, templateName)
    renderTemplate(templateDir, root)
  }
  ```

  项目的template文件夹下放置了各种“渲染”用的模板，这些模板的路径会被传入render函数来创建项目工程化配置的文件，而这里最主要的就是renderTemplate的逻辑：

  - 是文件夹
    - 是node_modules文件夹，则跳过
    - 递归创建目标目录
    - 遍历当前目录下的文件和文件夹，递归调用renderTemplate函数进行处理
  - 是文件
    - 如果是package.json，且目标位置有同名文件，则将两者内容合并，并写入目标位置，返回
    - 如果文件以“_”开头，则改为以"."开头
    - 将文件拷贝到目标位置

  关于模板目录下的部分文件夹说明：

  - base

    基础目前，包括vscoder插件配置、.gitignore、基础的package.json（只包含vue\vite\@vitejs/plugin-vue）和vite对应的scripts脚本、vite.config.js

  - config/jsx

    jsx相关的配置，没研究

  - config/router

    package.json中增加vue-router的dependencies依赖

  - config/pinia

    package.json增加pinia的的dependencies依赖，同时添加到src/stores/counter.js的示例代码，用来初始化store

  - config/vitest

    添加vitest等依赖及src下添加单元测试代码

  - config/typescript

    添加typescript及vue-tsc依赖，同时修改package.json中的构建脚本，增加对vue-tsc的调用

  - tsconfig/base

    添加tsconfig.json相关的配置

- ESLint配置构建

- 清理工作

  - 使用typescript，则递归检查目录下，如果同时存在.ts与.js文件，则删除后者，如果只有.js文件，则重命名为.ts文件，包括html中的main.js的处理
  - 否则则清理.ts文件

- 生成README.md