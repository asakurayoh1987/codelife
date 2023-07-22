# vue2 源码阅读（基于vue 2.7.14）

## 一、查找入口

首先定位到package.json中的scripts部分的`dev:cjs`脚本（webpack等工具在打包vue时就是使用该版本——commonjs模块规范）：

```bash
rollup -w -c scripts/config.js --environment TARGET:runtime-cjs-dev
```

也就是说通过`scripts/config.js`作为入口可以查看`dev:cjs`版本的构建流程，所以接下来就要查看该文件，注意下面的内容

```javascript
/**
 * 内联到这里方便查看：
 * {
      vue: resolve('src/platforms/web/entry-runtime-with-compiler'),
      compiler: resolve('src/compiler'),
      core: resolve('src/core'),
      shared: resolve('src/shared'),
      web: resolve('src/platforms/web'),
      server: resolve('packages/server-renderer/src'),
      sfc: resolve('packages/compiler-sfc/src')
    }
 */
const aliases = require('./alias')
const resolve = p => {
  const base = p.split('/')[0]
  if (aliases[base]) {
    return path.resolve(aliases[base], p.slice(base.length + 1))
  } else {
    return path.resolve(__dirname, '../', p)
  }
}
const builds = {
  // Runtime only (CommonJS). Used by bundlers e.g. Webpack & Browserify
  'runtime-cjs-dev': {
    // 通过解析，可以知道入口位置：../src/platforms/web/entry-runtime.ts
    entry: resolve('web/entry-runtime.ts'),
    dest: resolve('dist/vue.runtime.common.dev.js'),
    format: 'cjs',
    env: 'development',
    banner
  },
  // 其它省略
}

// 中间省略

if (process.env.TARGET) {
  // 此处读取了环境变量，按照前文，设定其值为`runtime-cjs-dev`
  module.exports = genConfig(process.env.TARGET)
} else {
  exports.getBuild = genConfig
  exports.getAllBuilds = () => Object.keys(builds).map(genConfig)
}
```

可知整个构建的入口文件在“../src/platforms/web/entry-runtime.ts”，而接下阅读源码就可以以此为入口进行

注意项目中tsconfig.json中的一些配置，比如sourcemap以及paths字段

## 二、从entry-runtime.ts开始

入口文件：`src/platforms/web/entry-runtime.ts`

```typescript
import Vue from './runtime/index'
import * as vca from 'v3'
import { extend } from 'shared/util'

// 将导出的vue3想着的方法，添加到Vue构造函数上
extend(Vue, vca)

export default Vue
```

所以如果想了解vue3新增的api，可以直接看v3这个模块里的细节





