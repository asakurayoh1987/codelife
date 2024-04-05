# 创建基于TypeScript的Library

## 一、环境搭建

### 1. 基础环境配置

```bash
# 初始化package.json
pnpm init

# 设置node版本，基于nvmrc
echo 'v20' > .nvmrc

# 设置registry
echo "registry=https://depend.iflytek.com/artifactory/api/npm/npm-repo/" > .npmrc

# 安装typescript相关
pnpm add -d typescript @types/node@v20

# 初始化tsconfig.json
pnpm exec tsc --init
```

###  2. 配置ESLint

```bash
# 通过ESLint提供的命令行引导工具来创建ESLint及相关的配置
pnpm create @eslint/config

# 操作及输出结果
✔ How would you like to use ESLint? · style
✔ What type of modules does your project use? · esm
✔ Which framework does your project use? · none
✔ Does your project use TypeScript? · typescript
✔ Where does your code run? · browser, node
✔ Which style guide do you want to follow? · standard
The config that you've selected requires the following dependencies:

eslint, globals, eslint-config-standard-with-typescript, @typescript-eslint/eslint-plugin@^6.4.0, eslint@^8.0.1, eslint-plugin-import@^2.25.2, eslint-plugin-n@^15.0.0 || ^16.0.0 , eslint-plugin-promise@^6.0.0, typescript@*, typescript-eslint, @eslint/eslintrc, @eslint/js
✔ Would you like to install them now? · No / Yes
✔ Which package manager do you want to use? · pnpm
☕️Installing...
 WARN  deprecated eslint-config-standard-with-typescript@43.0.1: Please use eslint-config-love, instead.
```

#### 所涉及需要安装的依赖项说明

##### [globals](https://www.npmjs.com/package/globals)

包含了ESLint中各种环境相关的配置项，它的内容就是一个json文件

```js
// index.js
'use strict';
module.exports = require('./globals.json');
```

##### [eslint-config-love](https://www.npmjs.com/package/eslint-config-love)

**用于替换eslint-config-standard-with-typescript**

定义了一堆适用于typescript场景下的配置项，具体可以看[这里](https://github.com/mightyiam/eslint-config-love/blob/main/src/index.ts)

它的依赖项包括：

- typescript
- eslint
- @typescript-eslint/eslint-plugin
- eslint-plugin-import
- eslint-plugin-n
- eslint-plugin-promise

##### [@eslint/eslintrc](https://www.npmjs.com/package/@eslint/eslintrc)

ESLint已经废弃旧的`.eslintrc`这种格式的配置文件，而这个库的作用就是用来支持旧的ESLintRC配置文件格式的，其中最重要的一个类就是`FlatCompat`，用来将旧的ESLintRC风格的配置转成扁平化的配置

```js
import { FlatCompat } from "@eslint/eslintrc";
import js from "@eslint/js";
import path from "path";
import { fileURLToPath } from "url";

// mimic CommonJS variables -- not needed if using CommonJS
const __filename = fileURLToPath(import.meta.url);
const __dirname = path.dirname(__filename);

const compat = new FlatCompat({
    baseDirectory: __dirname,                  // optional; default: process.cwd()
    resolvePluginsRelativeTo: __dirname,       // optional
    recommendedConfig: js.configs.recommended, // optional unless you're using "eslint:recommended"
    allConfig: js.configs.all,                 // optional unless you're using "eslint:all"
});

export default [

    // mimic ESLintRC-style extends
    ...compat.extends("standard", "example"),

    // mimic environments
    ...compat.env({
        es2020: true,
        node: true
    }),

    // mimic plugins
    ...compat.plugins("airbnb", "react"),

    // translate an entire config
    ...compat.config({
        plugins: ["airbnb", "react"],
        extends: "standard",
        env: {
            es2020: true,
            node: true
        },
        rules: {
            semi: "error"
        }
    })
];
```

##### [@typescript-eslint/eslint-plugin](https://www.npmjs.com/package/@typescript-eslint/eslint-plugin)

ESLint插件用于让ESLint支持处理TypeScript代码

##### [eslint-plugin-import](https://www.npmjs.com/package/eslint-plugin-import)

用于添加到ES2015+之后`import`/`export`语法的ESLint检测

##### [eslint-plugin-n](https://www.npmjs.com/package/eslint-plugin-n)

添加了针对Node.js的相关ESLint的规则

##### [eslint-plugin-promise](https://www.npmjs.com/package/eslint-plugin-promise)

强制JavaScript中Promise的一些最佳实践

##### [typescript-eslint](https://typescript-eslint.io/packages/typescript-eslint/)

同样也是添加了适合于TypeScript中的推荐规则，并且支持ESLint新的扁平风格的[配置项](https://eslint.org/docs/latest/use/configure/configuration-files)

#### vscode配置

因为使用的是ESLint flat风格的配置项，所以vscode中要设置`"eslint.experimental.useFlatConfig": true`，如果将ESLint插件升级到`v3.0.5 (pre-release)`版本，则使用配置项`"eslint.useFlatConfig": true,`，不然会提示报错

不过目录vscode中的ESLint插件尚有问题，使用flat风格的配置时，无法启动ESLint Server，但命令行中执行`eslint src --fix `是生效的，坐等官网ESLint插件升级吧

### 3. 配置prettier

```bash
# 锁定prettier的版本
pnpm add -DE prettier 
pnpm add -D eslint-config-prettier 
```

在上面生成的`eslint.config.mjs`配置文件中配置prettier，用于当ESLint与Prettier规则冲突时，优先使用Prettier的规则

```js
... // 其它import
import eslintConfigPrettier from 'eslint-config-prettier';

... // 其它代码
export default [
	... // 其它配置项
	eslintConfigPrettier,
]
```

然后配置`.prettierrc`和`.prettierignore`

```json
{
  "printWidth": 58,
  "tabWidth": 2,
  "useTabs": false,
  "semi": true,
  "singleQuote": true,
  "quoteProps": "preserve",
  "trailingComma": "all",
  "bracketSpacing": true,
  "arrowParens": "avoid",
  "endOfLine": "lf"
}
```

### 4. husky & lint-stage

```bash
pnpm add -D husky lint-staged
pnpm exec husky init 
```

编辑`.husky/pre-commit`

```bash
pnpm exec lint-staged
```

编辑package.json下的文件

```json
"lint-staged": {
  "*.ts": [
    "prettier --write",
    "eslint --fix --max-warnings 0"
  ]
}
```

### 5. commitlint

```bash
pnpm add -D @commitlint/{cli,config-conventional}
echo "export default { extends: ['@commitlint/config-conventional'] };" > commitlint.config.js
echo "pnpm exec commitlint --edit $1" > .husky/commit-msg
# 如果发现commit-msg hook不生成，就执行下面的命令
pnpm prepare
```

### 6. commitizen

```bash
pnpm add -D commitizen
pnpm exec commitizen init cz-conventional-changelog --pnpm --save-dev --save-exact
```

### 7. ts-patch

在使用typescript开发时，在引入模块时，经常使用alias来简单路径，如下：

```ts
import { MobileSystem } from '@model';
```

这个是通过在tsconfig.json中的配置实现的：

```json
"baseUrl": "./",
"paths": {
  "@util": [
    "src/util/index"
  ],
  "@model": [
    "src/model/index"
  ],
  "@service": [
    "src/service/index"
  ],
  "@mock": [
    "tests/mock/index"
  ]
}
```

但这种方式，如果直接使用`tsc`来生成js代码，并不会将这个别名替换成原始路径，这就导致这个js文件在直接运行时会提示找不到模块，官方貌似并没给出解决方案，这里使用了第三方给的方案：ts-patch + typescript-transform-paths

```bash
pnpm add -D ts-patch typescript-transform-paths
```

tsconfig.json的compilerOptions节添加如下配置

```json
"plugins": [
  // Transform paths in output .js files
  {
    "transform": "typescript-transform-paths"
  },
  // Transform paths in output .d.ts files (Include this line if you output declarations files
  {
    "transform": "typescript-transform-paths",
    "afterDeclarations": true
  }
]
```

再将构建命令由`tsc`替换为`tspc`，在生成的js代码中，就会将这个别名的形式替换为原相对路径的形式了

```ts
import { MobileSystem } from "../model";
```



## 二、文档-TypeDoc

```bash
pnpm add -D typedoc
```

添加`typedoc.json`配置文件

```json
{
  "entryPoints": [
    "src/index.ts",
  ],
  "useTsLinkResolution": true,
  "out": "docs"
}
```

添加npm script

```json
"scripts": {
  "doc": "typedoc"
}
```

## 三、单元测试-Jest

```bash
pnpm add -D jest @types/jest ts-jest ts-node jest-environment-jsdom jest-location-mock
pnpm create jest@latest

The following questions will help Jest to create a suitable configuration for your project

✔ Would you like to use Typescript for the configuration file? … yes
✔ Choose the test environment that will be used for testing › jsdom (browser-like)
✔ Do you want Jest to add coverage reports? … yes
✔ Which provider should be used to instrument code for coverage? › v8
✔ Automatically clear mock calls, instances, contexts and results before every test? … yes

📝  Configuration file created at /Users/yudiechao/Desktop/ts-lib/jest.config.ts
```

编辑创建的`jest.config.ts`

```ts
const config: Config = {
  ... // 其它配置
  preset: 'ts-jest',
  moduleNameMapper: {
    '@util': '<rootDir>/src/util',
    '@model': '<rootDir>/src/model',
  },
  setupFilesAfterEnv: ['./config/jest-setup.ts'],
}
```

config目录下的jest-setup.ts

```ts
import "jest-location-mock"
```

`jest-environment-jsdom` 、`jest-location-mock`及上面这个文件都是为了处理在jest测试用例中模拟修改`window.location`来验证一些操作浏览器DOM API的场景

类似，当模拟`navigator.userAgent`时可以使用`jest-useragent-mock`

## 四、构建方案

#### 1. tsc + eslint

项目使用tsc进行构建，通过配置，将代码转成ES5语法，但tsc不会注入polyfill，所以如`Array.prototype.includes`这样的API不会进处理，像Lodash这样的库是尽量使用广泛支持的JavaScript特性，而我们的工具函数也可以按这个思路，同时配置ESLint的插件来实现不兼容性API使用的检查

对于typescript配置，除了基础的tsconfig.json，增加tsconfig.cjs.json与tsconfig.esm.json，分别用于构建commjs与esm的包

```json
// tsconfig.cjs.json
{
  "extends": "./tsconfig.json",
  "compilerOptions": {
    "target": "ES5",
    "outDir": "lib/cjs",
    "module": "CommonJS"
  },
  "include": [
    "src/**/*.ts",
  ],
}

// tsconfig.esm.json
{
  "extends": "./tsconfig.json",
  "compilerOptions": {
    "target": "ES5",
    "outDir": "lib/esm",
    "module": "ESNext"
  },
  "include": [
    "src/**/*.ts",
  ],
}
```

然后通过`eslint-plugin-compat`配合`.browserslistrc`来校验不得使用非兼容性的API

#### 2. 生产方 rollup + babel

```bash
pnpm add -D rollup @rollup/{plugin-typescript,plugin-node-resolve,plugin-commonjs,plugin-babel} @babel/{core,preset-env,preset-typescript,plugin-transform-runtime}

pnpm add core-js@latest @babel/runtime
```

rollup的配置如下：

```js
```



#### 3. 消费方 rollup + babel

## 四、使用Library

### 配置rollup

```bash
pnpm add -D rollup
```

```js
import { defineConfig } from 'rollup';
import { nodeResolve } from '@rollup/plugin-node-resolve';
import commonjs from '@rollup/plugin-commonjs';
import babel from '@rollup/plugin-babel';

export default defineConfig({
  input: 'src/index.ts',
  output: {
    file: 'dist/index.js',
    format: 'iife',
  },
  plugins: [
    // ...
  ],
});

```

### 插件的使用

#### 1. [@rollup/plugin-node-resolve](https://www.npmjs.com/package/@rollup/plugin-node-resolve) 与 [@rollup/plugin-commonjs](https://www.npmjs.com/package/@rollup/plugin-commonjs)

这两个插件的介绍以及为什么使用参见[这里](https://rollupjs.org/tools/#with-npm-packages)

#### 2. [@rollup/plugin-babel](https://www.npmjs.com/package/@rollup/plugin-babel)

##### babelHelpers

