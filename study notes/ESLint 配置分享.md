# ESLint 配置讲解

## 一、环境搭建

### 1. 约定

* 配置项使用单独的配置文件形式，即不要使用以下几种方式：
	* 通过命令行指定
	* 不要放在package.json中的eslintConfig节
	* 配置文件统一使用.eslintrc.js文件（多个配置文件存在时js格式的优先级最高）， 不要使用yml及json格式（不灵活，包括注释不方便等等）

### 2. eslint引导

一般新的环境可以直接使用`eslint --init`来启动一个引导，会通过一系列的问答来自动生成一个配置文件。
** 如果需要使用typescript相关的eslint插件，需要提前安装好typescript**

```bash
# 初始化项目，主要是生成package.json
npm init -y

# 依赖模块安装
npm i -D eslint typescript

# 使用引导初始化eslint配置
npx eslint --init
```

![eslint guide](/Users/ycyu/Documents/Typora Notes/media/eslint guide.png)

```javascript
module.exports = {
	// 指定环境
  env: {
    browser: true,
    es6: true,
    node: true,
  },
  // 规则扩展
  extends: [
    'plugin:vue/essential',
    'airbnb-base',
  ],
  // 附加的全局变量
  globals: {
    Atomics: 'readonly',
    SharedArrayBuffer: 'readonly',
  },
  // 解析器选项
  parserOptions: {
    ecmaVersion: 2018,
    // 与官方的说明以及npm包说明不一致
    parser: '@typescript-eslint/parser',
    sourceType: 'module',
  },
  // 插件
  plugins: [
    'vue',
    '@typescript-eslint',
  ],
  // 规则
  rules: {
  },
};

```

## 二、配置项说明

### 0. 项目中使用的一个样例

```javascript
module.exports = {
  root: true,
  env: {
    browser: true,
    es6: true,
    node: true,
  },
  extends: [
    'airbnb-base',
    'plugin:@typescript-eslint/recommended',
    'plugin:prettier/recommended',
    'prettier/@typescript-eslint',
  ],
  plugins: ['import', '@typescript-eslint', 'prettier'],
  settings: {
    'import/parsers': {
      '@typescript-eslint/parser': ['.ts', '.tsx'],
    },
    'import/extendisons': ['.js', '.ts', '.tsx'],
    'import/resolver': {
      typescript: {
        directory: '.',
      },
    },
  },
  parser: '@typescript-eslint/parser',
  globals: {
    Atomics: 'readonly',
    SharedArrayBuffer: 'readonly',
  },
  parserOptions: {
    ecmaVersion: 2018,
    sourceType: 'module',
  },
  rules: {
    eqeqeq: [
      'error',
      'always',
      {
        null: 'ignore',
      },
    ],
    'no-shadow': 'off',
    'no-param-reassign': 'off',
    'no-use-before-define': 'off',
    'no-plusplus': 'off',
    'no-underscore-dangle': 'off',
    'no-unused-expressions': 'off',
    'no-useless-constructor': 'off',
    'no-shadow': 'off',
    'no-bitwise': 'off',
    'func-names': 'off',
    'class-methods-use-this': 'off',
    'import/extensions': [
      'error',
      'ignorePackages',
      {
        js: 'never',
        mjs: 'never',
        jsx: 'never',
        ts: 'never',
        tsx: 'never',
      },
    ],
    '@typescript-eslint/no-use-before-define': 'off',
    '@typescript-eslint/explicit-function-return-type': 'off',
    '@typescript-eslint/no-explicit-any': 'off',
  },
};
```

### 1. root——涉及配置文件的继承

默认情况下eslint在读取配置文件时，会顺着目录层级向上寻找，越靠近的优先级越高，但如果指定了`root:true`，则不会再继续向上寻找

### 2. env——指定代码运行的环境，同时会预置一批指定环境中的全局变量

* [官方预置](https://eslint.org/docs/user-guide/configuring#specifying-environments)：设置对应项为`true`则开启
* 插件中：`插件名（去除前缀）/环境名`为`true` 

** 关于es6以及es2017、es2020等，设置为true时，会同时静默设置parserOptions中的ecmaVersion为对应的值，但反之则不会**

### 3. globals——指定额外的全局变量

当变量不在env所覆盖的范围，同时又希望全局访问时在此处定义，以对象形式指定，key为环境变量名的字符串形式，value为“writeable”、“readlonly”、"off",

### 4. parser和parserOptions——指定解析器及解析器选项

* eslint进行代码检查的流程可以理解：source code --(parser)-->AST--(遍历节点)-->rule校验
* parser的作用就是告诉eslint如何将源码解析成AST
* [兼容的解析器](https://eslint.org/docs/user-guide/configuring#specifying-parser)
* parserOptions是用来传递给parser的，用于一些特性的使用，比如`jsx`

### 5. rules——用于检测的规则

* 每条规则的值可以是一个字符串、一个数字或一个数组，字符串、数字以及数组的第一元素（后续元素是用于定制该规则的其它参数）可以是以下值：
```
// 关闭规则
"off"/0,
// 开启仅提示告警
"warn"/1,
// 开启且提示错误
"error"/2,
```

* 规则除了在配置文件中设置外还可以在文件中以注释的方式设置
* 使用插件提供的规则时，规则的ID为：`插件名（除去前缀）/插件提供的规则ID`

### 6. extends——配置的继承

* 用途：继承现有的配置
* 值：
	* 字符串（配置文件的路径或共享配置的名称、[eslint:recommended](https://eslint.org/docs/rules/)或eslint:all）
	* 字符串数组（后指定的配置会覆盖前面的配置）
* 来源：
	* eslint-config-xxx：`extends:['xxx']`
	* eslint-plugin-xxx：`extends:['plugin:xxx/配置名']`
* 举例：
	* [airbnb-base](https://github.com/airbnb/javascript/blob/master/packages/eslint-config-airbnb-base/package.json)
	* [@typescript-eslint/eslint-plugin](https://www.npmjs.com/package/@typescript-eslint/eslint-plugin)
	* [eslint-config-prettier](https://www.npmjs.com/package/eslint-config-prettier)
	* [eslint-plugin-prettier](https://www.npmjs.com/package/eslint-plugin-prettier)

### 7. plugins——添加第三方插件

