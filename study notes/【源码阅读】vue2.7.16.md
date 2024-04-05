# Vue 2.7.16 源码阅读

## 一、入口及调试环境搭建

还是从`package.json`入手，可以看到诸如`"dev:esm": "rollup -w -c scripts/config.js --environment TARGET:runtime-esm",`这样的代码，而构建的配置正是在`scripts/config.js`中，通过大概的阅读其中内容，大概意思就是根据`--environment`指定的变量值来构建一个rollup的配置

### 1. 修改npm script

与之前不同，为了方便阅读源码及在浏览器中调试，我们构建一个完整的基于ESM的版本，并添加sourcemap，所以在package.json添加npm script：`"dev:full-esm": "rollup -w -c scripts/config.js --sourcemap --environment TARGET:full-esm-browser-dev",`在`scripts/config.js`中可以找到`full-esm-browser-dev`对应的rollup配置，包括输入（`src/platforms/web/entry-runtime-with-compiler-esm.ts`）输出（`dist/vue.esm.browser.js`）

### 2. 添加测试页面

添加`examples/index.html`，内容如下：

```html
<!DOCTYPE html>
<html lang="en">

<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Vue Runtime Demo</title>
    <script type="importmap">
        {
            "imports": {
                "vue": "../dist/vue.esm.browser.js"
            }
        }
    </script>
</head>

<body>
    <div id="app">
        <h1>{{ msg }}</h1>
        <p>{{ counter }}</p>
        <button @click="increment">Increment</button>
        <button @click="decrement">Decrement</button>
    </div>
    <script type="module">
        import Vue from 'vue';
        import { ref } from 'vue';
        const app = new Vue({
            setup() {
                const msg = ref('Hello World!');
                const counter = ref(0);
                const increment = () => {
                    counter.value++;
                }
                const decrement = () => {
                    counter.value--;
                }
                return {
                    msg,
                    counter,

                    increment,
                    decrement
                }
            }
        });
        app.$mount('#app')
    </script>
</body>

</html>
```

### 3. 构建并调试

在命令中执行`pnpm dev:full-esm`以`watch`模式构建产物，再配置vscode的`Live Server`插件，打开index.html，就可以看到基于vue渲染之后的页面，在代码合适的位置添加`debugger;`即可命中断点，并配合sourcemap来调试代码了

根据npm script中的`TARGET`所指加上`script/config.js`中的解析，构建的入口文件位于：`src/platforms/web/entry-runtime-with-compiler-esm.ts`

## 4. 层级结构

1. `src/platforms/web/entry-runtime-with-compiler-esm.ts`，作为ESM格式的构建入口，依赖2
2. `src/platforms/web/runtime-with-compiler.ts`，在runtime的基础之上，添加了将模板编译成渲染函数的能力，依赖3
3. `src/platforms/web/runtime/index.ts`，在core的基础之上添加了runtime环境（也就是浏览器环境）中的方法，其中的`$mount`方法就是在浏览器中渲染虚拟节点的入口，依赖4
   - 这里的`Vue.prototype.$mount`函数下的`mountComponent`方法要细读一些，这是渲染的入口
4. `src/core/index.ts`，核心机制，与环境无关的逻辑，是对依赖5的一个包装，比如注册一些全局方法（比如`Vue.set`、`Vue.nextTick`等）
5. `src/core/instance/index.ts`，`function Vue`定义之处
