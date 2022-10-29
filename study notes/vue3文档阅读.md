

# Vue3文档阅读

## 一、基础

- 传递给createApp的选项用于配置**根组件**，当我们**挂载**应用时，该组件被用作渲染的起点

- 应该实例暴露的大多数方法都返回同一实例，允许链式调用，但mount方法返回的是根组件实例

- 不要在选项property或回调上使用**箭头函数**，比如`created:()=>console.log(this.a)`

- 生命周期

  ![示意图](https://v3.cn.vuejs.org/images/lifecycle.svg)

- 通过**v-once**指令，只执行一次性插值，后续当数据变化时，插值处的内容不会更新

- 指令的动态参数使用**方括号**括起来，同时不要在其中使用空格或引号（可用计算属性替代），也要避免使用大写字符来命名，因为浏览器会将attribute名全部强制转为小写

- 模板表达式都被放在沙盒中，只能访问一个**[受限的全局变量列表](https://github.com/vuejs/core/blob/main/packages/shared/src/globalsWhitelist.ts#L3)**

- vue3中，组件的data选项均为一个函数，包括根组件，组件实例中可以通过**$data**来访问data函数返回的对象

- 直接将不包含在 data中的新 property 添加到组件实例是可行的。但由于该 property 不在背后的响应式$data对象内，所以**[Vue 的响应性系统](https://v3.cn.vuejs.org/guide/reactivity.html)**不会自动跟踪它。

- 在methods选项中不要使用箭头函数，因为这会阻止Vue绑定恰当的**this指向**

- class绑定时的对象语法、数组语法，style内联时也支持对象语法与数组语法

- 对于style绑定中的property提供一个包含多个值的数组，比如`<div :style="{ display: ['-webkit-box', '-ms-flexbox', 'flex'] }"></div>`只会沉浸数组中最后一个被浏览器支持的值

- 在组件中使用class的绑定语法时，对于带有单个根元素的自定义组件，class会被添加到该元素上，并且不会覆盖该元素上现有的class，但如果有多个根元素，则需要通过$attrs.class来将组件上的class指定绑定到根元素上

- v-for在遍历数组时，第一个元素是数组元素，第二个元素是索引，在遍历对象时，第一个元素是对象中属性的Value，第二个元素是对象中属性的Key，第三个元素是索引，但遍历对象时，会按Object.keys()的结果遍历，不能保证在不同的javascript引擎下结果都一致

- 不要使用对象或数组之类的非基本类型值作为v-for的key，因为在对比时，两个对象的key或数组的元素都一样，它们在对比时也是不同的，所以需要使用字符串或数值类型的值作为key

- **不推荐**v-if与v-for一起使用，原因参照[这里](https://v3.cn.vuejs.org/style-guide/#%E9%81%BF%E5%85%8D-v-if-%E5%92%8C-v-for-%E4%B8%80%E8%B5%B7%E4%BD%BF%E7%94%A8%E5%BF%85%E8%A6%81)，关于它俩的优先级，在vue2与vue3中不同：

  - 在vue2中，v-for的优先级更高，也就是说，会先遍历元素，再判断v-if，即使所有的元素在v-if中为假值，循环也会执行，可以考虑使用计算属性先过滤一遍元素，再进行遍历
  - vue3中，v-if优先级更高，所以会导致v-if没有权限访问v-for里的变量，从而报错

- 事件处理时可通过$event将DOM事件对象传入，多事件处理器使用逗号分隔

- 事件修饰符：stop\prevent\capture\self\once\passive，可以串联

  - 使用修饰符时，顺序很重要，相应的代码会以相同的顺序产生
  - once修饰符与其他只能对原生的DOM事件起作用的修饰符不同，它还能被用到自定义组件事件上
  - passive修饰符能提升移动端性能，但注意不要和prevent一起使用，后者将被忽略，同时浏览器可能会向你展示一个警告
  - 按键修饰符以及exact修饰符（精确按下某个按键或按键组合时才触发事件）

- 表单元素值绑定时的v-model修饰符

  - lazy：将input事件替换为change事件
  - number：将输入值转为数值类型，若无法被parseFloat()解析，则返回原始值
  - trim：去除首尾空白字符

- v-model只是一个语法糖，对于input元素，v-model绑定就相当于通过value绑定值，然后通过input事件来更新值，而对于自定义组件，通过model-value绑定值，通过update:model-value事件更新值，另一种方式是通过computed property（通过getter与setter），具体参见**[这里](https://v3.cn.vuejs.org/guide/component-basics.html#%E5%9C%A8%E7%BB%84%E4%BB%B6%E4%B8%8A%E4%BD%BF%E7%94%A8-v-model)**

- 动态组件component的is属性可以是：

  - 已注册组件的名字
  - 一个组件选项对象

- is属性还可用于在DOM中使用Vue模板：

  某些元素只能出现在特定元素内，比如在table下不能出现自定义的组件，但可以通过is属性绕过，比如`<tr is="vue:blog-post-row"></tr>`，**is的值必须以“vue:”开头，以区分原生自定义元素**

## 二、深入组件

- 组件的prop会在一个组件实例创建之前进行验证，所以实例的property（如data、computed等）在default或validator函数中是不可用的

- 类型检查时type还可以是一个自定义的构造函数，并且通过instanceof来进行检查确认

- 关于attribute的继承，当组件返回单个节点时，非prop的attribute将自动添加到根节点的attribute中，这一点同样适用到事件监听器，如果不希望开启attribute继承，在组件的选项中设置inheritAttrs为false即可，然后配合v-bind="$attrs"来进行显示的绑定操作

- 当在组件的emits选项中定义了原生事件（如click）时，将使用组件中的事件**替代**原生事件侦听器

- 默认情况下，组件上的v-model使用modelValue作为prop，以update:modelValue作为事件，但通过向v-model传递**参数**可以修改这些名称，比如v-model:title="xxx"，则组件中就可使用title为作prop，并以update:title作为事件

- v-model自定义修饰符，以v-model.xxx的方式指定修饰符，然后通过modelModifiers prop提供给组件，其值为**布尔**类型，对于带参数的v-model，自定义修饰符的属性名为**参数名+"Modifiers"**，比如`v-model:description.capitalize="myTest"`，则通过descriptionModifiers可以拿到定义的包修符的内容（此时为{ capitalize: true}）

- **父级模板里的所有内容都是在父级作用域中编译的；子模板里的所有内容都是在子作用域中编译的。**

- **具名插槽**，组件模板中，通过在slot上指定name属性来给插槽命名（不带name属性的slot隐含名字为default），在使用组件时，可以通过v-slot:xxx的参数方式指定渲染的位置

  ```html
  <!--base-layout组件-->
  <div class="container">
    <header>
      <slot name="header"></slot>
    </header>
    <main>
      <slot></slot>
    </main>
    <footer>
      <slot name="footer"></slot>
    </footer>
  </div>

  <!--父组件中使用base-layout使用具名插槽-->
  <base-layout>
    <template v-slot:header>
      <h1>Here might be a page title</h1>
    </template>

    <template v-slot:default>
      <p>A paragraph for the main content.</p>
      <p>And another one.</p>
    </template>

    <template v-slot:footer>
      <p>Here's some contact info</p>
    </template>
  </base-layout>
  ```



- **作用域插槽**，为了让插槽内容能够访问子组件中才有的数据，可以在子组件中使用`<slot :xxx="数据"></slot>`，即将数据绑定到xxx属性上，该属性被称为**插槽属性**，在父级作用域中，则可以使用具名插槽类似的语法来指定使用子级暴露出来的数据，并且这里支持使用对象解构，同时如果**只有一个插槽**，那v-slot:default可以简写为v-slot，否则不能使用缩略语法

  ![示意图](../media/scoped-slot.png)

- **动态插槽名**也可用在v-slot上，形名"template v-slot:[dynamicSlotName]"

- **具名插槽的缩写**，v-slot:header缩写成#header，v-slot:default缩写成#default

- 在使用provide传递组件实例的property时，需要将provide转换为**返回对象的函数**，否则不生效

- 默认情况下，provide/inject绑定**并不是响应式的**，但可以通过传递一个ref property或reactive对象给provide来实现

- 通过向defineAsyncComponent方法传递一个返回Promise的工厂函数，可以定义一个**异步组件**，然后再通过app.component方法注册该组件，局部注册时也可以使用

- **与Suspense一起使用**，异步组件默认情况下是可挂起的，如果父链中有一个<Suspense>，则异步组件会作为<Suspense>的异步依赖，此时加载状态会由<Suspense>控制，组件自身的加载、错误、延迟和超时选项会被忽略，但可以通过在组件选项中指定suspensible: false来退出Suspense控制

- **$refs只会在组件渲染完成之后生效**

- **【慎用】**通过$forceUpdate来进行强制更新

## 三、过渡 & 动画

- transform和opacity不会触发任何几何形状变化或绘制，是由合成器线程在GPU的帮助下执行的，有较好的性能表现

- perspective、backface-visibility和transform:translateZ(x)等property将让浏览器知道你需要硬件加速

- 对于简单的过渡动画，从一个状态到另一个没有中间状态的状态，0.25s是一个最佳的选择

- css过渡示意图，其中"v"是默认前缀，可以通过过transition上的name属性进行变更

  ![image-20220308172753517](https://gitee.com/asakurayoh/image-host/raw/master/img/202203081727600.png)

- 可以通过 `appear` attribute 设置节点在初始渲染的过渡

- 要使用 `type` attribute 并设置 `animation` 或 `transition` 来显式声明你需要 Vue 监听的类型

- 可以用 `<transition>` 组件上的 `duration` prop 显式指定过渡持续时间 (以毫秒计)

## 四、可复用 & 组合

- 组合式API中，setup在组件创建之前执行，此时组件还未实例化，data\computed\methods都未被解析

- setup的第一个参数props是响应式的，不能使用解构，**否则会消除prop的响应性，如果需要解构，需要使用toRefs函数**，如果props中某个属性可能不存在，则可使用toRef单独处理

- setup的第二个参数context是普通的javascript对象，包含了attrs（等同于\$attrs）、slots（等同于\$slots）、emit（等同于\$emit）、expose（暴露公共property）

- **attrs和slots是有状态的对象**，它们总是会随组件本身的更新而更新。这意味着你应该**避免对它们进行解构**，并始终以attrs.x或slots.x的方式引用 property，与props不同，attrs和slots的 property 是**非**响应式的

- 当setup返回一个渲染函数，同时又希望暴露公共property以便外部调用时，就需要用到expose了，它是一个方法，参数为一个对象，其中包含了需要暴露的property

- 生命周期对比

  | 选项式 API        | Hook inside `setup` |
  | ----------------- | ------------------- |
  | `beforeCreate`    | Not needed*         |
  | `created`         | Not needed*         |
  | `beforeMount`     | `onBeforeMount`     |
  | `mounted`         | `onMounted`         |
  | `beforeUpdate`    | `onBeforeUpdate`    |
  | `updated`         | `onUpdated`         |
  | `beforeUnmount`   | `onBeforeUnmount`   |
  | `unmounted`       | `onUnmounted`       |
  | `errorCaptured`   | `onErrorCaptured`   |
  | `renderTracked`   | `onRenderTracked`   |
  | `renderTriggered` | `onRenderTriggered` |
  | `activated`       | `onActivated`       |
  | `deactivated`     | `onDeactivated`     |

- 在组合式API中，通过provide配置ref及reactive可以将响应式对象传递到子孙组件中，同时配合readonly方法可避免子孙组件中直接修改先辈组件中的值，如需修改，应由先辈组件provide一个修改对应属性的方法

- 在组合式API中，为了获得对模板内元素或组件的引用，可以直接使用与模板中ref属性指定的值同名的ref变量，比如模板中的div上使用了ref="root"，则在setup中使用const root = ref(null)则可引用对应的div，这是因为在虚拟 DOM 补丁算法中，如果 VNode 的 `ref` 键对应于渲染上下文中的 ref，则 VNode 的相应元素或组件实例将被分配给该 ref 的值

- 在v-for中使用模板引用，则需要手动处理绑定

  ```vue
  <template>
    <div v-for="(item, i) in list" :ref="el => { if (el) divs[i] = el }">
      {{ item }}
    </div>
  </template>

  <script>
    import { ref, reactive, onBeforeUpdate } from 'vue'

    export default {
      setup() {
        const list = reactive([1, 2, 3])
        const divs = ref([])

        // 确保在每次更新之前重置ref
        onBeforeUpdate(() => {
          divs.value = []
        })

        return {
          list,
          divs
        }
      }
    }
  </script>
  ```

- 关于侦听模板引用，默认情况下watch和watchEffect会在DOM挂载或更新**之前**运行副作用，此时模板引用还未更新，需要使用设置flush: 'post'选项，使其在DOM更新后运行副作用，**从 Vue 3.2.0 开始，`watchPostEffect` 和 `watchSyncEffect` 别名也可以用来让代码意图更加明显。**

- **Mixin的优先级**：

  - data函数：各自执行，将结果合并返回，**组件自身的数据为优先**
  - 钩子函数：同名钩子函数合并为一个数组，**mixin对象的钩子优先调用**
  - 值为对象的选项：合并为同一个对象，键名冲突时，**以组件对象的键值对为优先**
  - 自定义选项：通过app.config.optionMergeStrategies添加函数来指定合并策略，参见[这里](https://v3.cn.vuejs.org/guide/mixins.html#%E8%87%AA%E5%AE%9A%E4%B9%89%E9%80%89%E9%A1%B9%E5%90%88%E5%B9%B6%E7%AD%96%E7%95%A5)

- 关于渲染函数中使用插槽，比较难理解，参见[这里](https://v3.cn.vuejs.org/guide/render-function.html#%E6%8F%92%E6%A7%BD)，需要多次阅读

- 在底层实现里，模板使用resolveDynamicComponent来实现is attribute

## 五、高阶指南

- Vue与Web Components，依赖对[Web Components](https://developer.mozilla.org/zh-CN/docs/Web/Web_Components)的了解，先跳过，以后再[回来](https://v3.cn.vuejs.org/guide/web-components.html#vue-%E4%B8%8E-web-components)

- 响应式原理可以参见[Vue Mastery上的视频](https://www.vuemastery.com/courses/vue-3-reactivity/vue3-reactivity/)非常形象

- 响应式相关的几个概念（依赖于上一条的基础知识）：

  - effect——副作用，是一个函数的包裹器，在函数被调用之前就启动跟踪，当函数执行时，就会收集依赖，此时，vue就知道当前所执行的函数以及所依赖的数据（利用ES6的Proxy拦截），从而建立依赖关系，当数据变化时，就可重新执行这个函数

  - track——收集器，如effect中提到，在数据被访问时，由Proxy进行拦截，然后调整用收集器，此时Vue知道当前运行的是哪个副作用，并将其与参数target、property记录在一起

  - trigger——触发器，如effect提到，在数据被修改时，同样由Proxy进行拦截，通过参数target、property从记录中找到必须执行的副作用并执行

  - 示意图：

    ![image-20220307095025125](https://gitee.com/asakurayoh/image-host/raw/master/img/image-20220307095025125.png)

    ![image-20220307095243855](https://gitee.com/asakurayoh/image-host/raw/master/img/image-20220307095243855.png)

    ![image-20220307095700452](https://gitee.com/asakurayoh/image-host/raw/master/img/image-20220307095700452.png)

- 当从一个响应式代理中访问一个嵌套对象时，该对象在被返回之前也被转换为一个代理

- Proxy的引入会导致被代理对象与原始对象不相等（===），所以最佳实践就是永远不要持有对原始对象的引用，而只使用响应式版本，同时Vue不会在Proxy中包裹数字或字符串等原始值，所以仍可以对这些值直接使用全等号

- Vue响应性系统的本质，就是当从组件data()返回一个对象时，在内部交由reactive()使其成为响应式对象，而模板会被编译成能够使用这些响应式property的**渲染函数**

- setup返回的ref声明属性，在模板中被访问时会**自动浅层解包**，也就是你无需写xxx.value，但如果通过嵌套的方式来访问，就需要添加.value，同理当ref作为响应对象的property被访问或更改时，也会自动解包内部值，无需写.value，但这仅限于被响应式Object嵌套的时候，如果是从Array或原生集合（如Map）访问ref时，不会进行解包

- 计算属性，通过computed函数，它接受getter函数并为getter返回的值返回一个不可变的响应式ref对象，或接受一个带有get和set函数的对象来创建一个可写的ref对象

- watch与watchEffect的区别：

  - 懒执行副作用
  - 更具体地说明什么状态应该触发侦听器重新运行
  - 访问侦听状态变化前后的值

- \$nextTick()返回一个Promise对象，所以可以使用async/await语法，以同步的写法来执行原本写在\$nextTick中的回调函数

## 六、工具

- SFC文件格式，必须由@vue/compiler-sfc预编译为标准的JS与CSS，编译后的SFC是一个标准的JS模块
- 如果在组件渲染期间发生运行时错误，它将被传递到全局的 `app.config.errorHandler` 配置函数，如果它已经被设置。

## 七、风格指南

- 组件名为多个单词，可以避免与现有及未来的HTML元素冲突
- prop定义应该详细，至少指定类型
- v-for中一定要指定key
- 避免v-if与v-for一起使用
  - vue3中v-if优先级更高，会因没权限访问v-for中定义的变量而报错
  - vue2中v-for优先级更高，但可能会浪费循环沉浸，比如判断v-if时发现全为false
- 为组件设置作用域
  - 对于应用来说，样式在顶层App组件和布局组件中可以是全局的，但在其他所有组件中都应该是有作用域的，不过记住这条规则只适用于单文件组件
  - 对于组件库来说，更倾向于选用基于class的策略，而不是scoped attribute，这会让覆写内部样式更容易
- 关于私有属性的命名，尽量使用“$_”作为前缀
- 单文件组件的文件名应该要么始终是单词大写开头，要么始终是横线连接的形式
- 应用特定样式和约定的基础组件应该全部以一个特定的前缀开头
- 在单文件组件、字符串模板及JSX中，没有内容的组件应该是自闭合的，但在DOM模板里不要这样做，因为HTML不支持自闭合的自定义元素
- 对于绝大多数项目来说，单文件组件和字符串模板中，组件名称应该始终是PascalCase的，但在DOM模板中是kebab-case的，因为HTML是大小写不敏感的
- 元素选择器避免在scoped中出现，因为scoped会给样式设置作用域，添加特定的attribute，比如'data-v-f3f3eg9'，同时会修改样式，比如原样式中button会变成button[data-v-f3f3eg9]，导致无法选中，同时大量元素和attribute组合的选择器会比类与attribute组合的选择器要更慢
- 理想的Vue应用是prop向下传递，事件向上传递的

## 八、Vue2迁移指南

- v-for中使用:ref绑定函数的方式来手动处理DOM元素

- 注册异步组件时，不再支持调用resovle的工厂函数，必须始终返回一个Promise

- 在3.x中，应该使用 `null` 或 `undefined` 以显式移除 attribute

- 在2.x中，$attrs中不包括class与style，但3.x中被包含在内了

- 在3.x中，`$children` property 已被移除，且不再支持。如果你需要访问子组件实例，我们建议使用 [$refs](https://v3.cn.vuejs.org/guide/component-template-refs.html#模板引用)

- 3.x中自定义指令的钩子函数被重命名，以便更好的与组件的生命周期保持一致

- 3.x中is attribute被限制在component标签中，同时当需要将元素解析为Vue组件时，需要添加一个`vue:`前缀

- 3.x中Mixin合并将被浅层地执行

- 新增了emits选项，用来定义一个组件可以向其父组件触发的事件，

- 强烈建议使用 `emits` 记录每个组件所触发的所有事件。这尤为重要，因为我们[移除了 `.native` 修饰符](https://v3.cn.vuejs.org/guide/migration/v-on-native-modifier-removed.html)。任何未在 `emits` 中声明的事件监听器都会被算入组件的 `$attrs`，并将默认绑定到组件的根节点上

- \$on, \$off和\$once实例方法已被移除，所以2.x版本中的事件总线的实现方式无法再使用，在3.x中需要使用外部的实现了事件触发器接口的库，比如mitt与tiny-emitter

- 3.x移除了过滤器，建议使用方法调用或计算属性来替换，对于之前的全局过滤器，可以通过全局属性来让它被所有组件使用到：`app.config.globalProperties.$filters=xxx`

- 2.x中的很多全局API在3.x中都需要使用实例上的API来代替

  - 关于判断是否是自定义元素的API: app.config.compilerOptions.isCustomElement，只在使用运行时编译器版本才会生效，如果是仅运行时的版本，在构建时需要通过vue-loader中的compilerOptions选项来传递些配置
  - Vue.extend移除，均使用Vue.createApp来代替，同时关于组件继承，建议使用CompositionAPI来代替mixin，如果仍需要使用继承，可以使用extends选项来代替Vue.extend

- 3.x中对于ES模块构建版本来说，全局API是通过具名导出进行访问的

- 3.x移除了内联模板的特性：inline-template

- 3.x的条件分支中，会自动生成key，无须添加，如果手动添加则要保证key唯一，而对于template配合v-for使用时，则要将key设置在<template>标签上

- 不再支持使用数字作为v-on修饰符，使用kebab-cased的键名，比如PageDown：`v-on:keyup.page-down="xxx"`

- \$listeners被移除，已作为\$attrs的一部分

- 挂载逻辑的变更，2.x中，当应用包括template时，其内容会直接替换要挂载到的dom元素，在3.x中，则是将template的内容作为要挂载到的dom元素的innerHtml

- propsData移除，2.x中是用来向根组件传递props的值，在3.x中作为createApp的第二个参数传递

- prop默认值的工厂函数不再能访问this，替代方法：

  - 组件接收到的原始prop将作为参数传递给默认函数
  - Inject API可以在默认函数中使用

- 渲染函数API的变更

  - h函数改为全局导入，不再作为参数传递给渲染函数
  - VNode prop的结构都是扁平的
  - 组件渲染方式不同，3.x中，由于VNode是上下文无关的，组件的字符串id需要先传递给resolveComponent来拿组件的构造函数，再将其传递了h函数

- 插槽统一（大部分变更都在vue2.6中已支持）：

  - this.\$slots现在将插槽作为函数公开，在h函数中，插槽以对象的形式定义为当前节点的子节点：

    ```javscript
    h(LayoutComponent, {}, {
    	header: ()=> h('div', this.header),
    	content: ()=> h('div', this.content),
    })
    ```



  - 移除this.\$scopedSlots

- Suspense为实验特性，**生产环境勿用**

-

