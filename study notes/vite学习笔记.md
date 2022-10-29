# Vite学习笔记（基于vite2）

## 一、知识储备

### 1.1 关于`npm init`

[npm文档](https://docs.npmjs.com/cli/v7/commands/npm-init)

`npm init <initializer>`命令中的`initializer`实际上对应于一个名为`create-<initializer>`的npm包，当执行相关的`npm init`命令时，实际上执行了如下操作：

- `npm init foo` --> `npm exec create-foo`
- `npm init @usr/foo` --> `npm exec @usr/create-foo`
- `npm init @usr` --> `npm exec @usr/create`	

## 二、创建项目

### 2.1 直接创建

- `npm init vite@latest`
- `yarn create vite`

### 2.2 指定模板

- `npm init vite@latest my-vue-app -- --template vue-ts`
- `yarn create vite my-vue-app --template vue`

### 2.3 可选模板

#### 2.3.1 官方模板

- `vanilla`
- `vanilla-ts`
- `vue`
- `vue-ts`
- `react`
- `react-ts`
- `preact`
- `preact-ts`
- `lit-element`
- `lit-element-ts`
- `svelte`
- `svelte-ts`

#### 2.3.2 社区模板

[awesome-vite#templates](https://github.com/vitejs/awesome-vite#templates)

[degit工具](https://github.com/Rich-Harris/degit)

## 三、使用及特性

### 3.1 启动

使用`npm run dev`即可，默认情况下vite会使用当前目录作为根目录启动项目，也可以使用`vite serve some/sub/dir`来指定根目录启动项目

还可以指定`--port`和`--https`，更多选项可以通过`npx vite --help`查看

### 3.2 npm依赖解析与预打包

浏览器原生的es module是不支持如下这种模块导入方式的

```javascript
import { someMethod } from 'my-dep'
```

所以vite会进行如下操作

- 通过pre-bundle来提升页面加载速度，并将Commonjs/UMD的包转换成ESM，pre-building的操作是基于esbuild（官方宣称其打包速度最高是其它打包器的100倍），从来使得vite的冷启动速度远超其它基于js的打包器
- 重写import语句来使浏览器可以正常加载

**vite会通过设置http头来缓存请求的依赖资源**，所以如果需要修改或调试某个依赖，请参照[这里](https://vitejs.dev/guide/dep-pre-bundling.html#file-system-cache)

### 3.3 热更新

vite提供了基于原生ESM的HMR API，那些支持HMR的框架可以利用这个API来提供快速、精确的热更新能力；vite为vue、react及preact都提供了官方的HMR集成，当你使用vite官方脚手架创建项目时，这些已经被预置好了

### 3.4 Typescript支持

vite为typescript提供了开箱即用的支持，不过它仅仅针对.ts文件进行了转换，并不会进行类型检查，它假定类型检查的工作是由IDE或构建来处理的（可以通过在构建脚本中执行`tsc --noEmit`来进行类型检查，对于.vue文件可以通过安装vue-tsc并执行`vue-tsc --noEmit`来进行类型检查）

vite使用esbuild来将ts转换成js，它比普通的tsc要快上20～30倍

#### 3.4.1 Typescript构建选项使用注意

##### 1. isolateModules

**此项须设置为true**

因为esbuild仅进行转换操作，而不会携带类型信息，所以它并不支持某些特定的特性，比如`const enum`、隐式的仅类型的导入操作，通过在tsconfig.json中将isolateModules设置为true，当你使用这些不支持的特性时，TS会给出相应的警告信息

## 四、配置Vite

### 4.1 配置文件

当在命令行使用vite时，它会默认从配置项root指定的目录下查找名为`vite.config.js`的配置文件，也可通过`--config`选项来指定配置文件，vite支持在配置文件中直接使用ES的模块语法，vite内部会在加载配置文件前进行预处理

vite基于typescript，所以使用中可以通过以下方式开启IDE的智能代码提示功能

```javascript
import { defineConfig } from 'vite';

export default defineConfig({
  // ...
})
```



 