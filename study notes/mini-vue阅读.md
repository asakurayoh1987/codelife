# mini-vue 阅读记录

- `const app = createApp(rootComponent)`，创建对象字面量，包括两个字段
  - \_component：保存 rootComponent 的引用
  - mount(rootContainer)方法
- `app.mount(rootContainer)`，调用上一步实例对象的 mount 方法，挂载至页面 DOM 元素'rootContainer'之下

  - 创建 vnode 实例对象，调用`function createVNode(type, props, children)`，`var vnode = createVNode(rootComponent)`

    - vnode 对象实例定义如下

      ```javascript
      if (props === void 0) {
        props = {};
      }
      var vnode = {
        el: null,
        component: null,
        key: props.key || null,
        type: type,
        props: props,
        children: children,
        shapeFlag: getShapeFlag(type), // 根据type是否为string类型设置标记位
      };
      ```

    - 根据 children 是否是数组（`Array.isArray`）、字符串、对象（主要用来设置是否为插槽子节点）来设置相关的标记位

  - 调用`render(vnode, rootContainer);`

    - 通过`patch(null, vnode, container)`，调用方法`function patch(n1, n2, container, parentComponent)`，此时是首次创建，所以 n1 为 null

          - 根据 n2 节点的 type 和 shapeFlag 来决定如何 patch 节点，type 为`Text`调用`processText(n1, n2, container)`，其它类型则根据 shapeFlag 来确定，分别通过`processElement(n1, n2, container)`和`processComponent(n1, n2, container, parentComponent);`来处理
          - `processText(n1,n2,container)`
            - n1 不存在时，表示进行新建操作，此时调用`hostCreateText`（其内部是调用 document.createTextNode 方法）以 n2.children（其内容为文本字符串）为内容，创建文本节点，并调用`hostInsert`（内部调用 element.insertBefore 或 element.appendChild 实现）
            - n1 存在时，表示进行更新操作，此时对比 n1 与 n2 的 children，即两者的文本内容，并调用`hostSetText`（通过文本节点的 nodeValue 来修改值）更新文本为 n2.children
          - `processElement(n1,n2,container)`

            - n1 不存在，表示首次挂载节点，调用`mountElement(n2, container)`，调用`hostCreateElement`（内部调用`document.createElement`）创建 element

              1. 根据子节点的类型来处理子节点

              - `ShapeFlags.TEXT_CHILDREN`：子节点为文本，即 children 为字符串，调用`hostSetElementText`（内部直接使用 innerText 来设置文本），比如`h("div",{},"test")`最终渲染的是`<div>test</div>`
              - `ShapeFlags.ARRAY_CHILDREN`：子节点为数组形式，调用`mountChildren`，则遍历子节点列表来递归调用`patch`方法，只不过此时参数中的 vnode 为子节点元素，而 container 为刚才创建的 element 元素

              1. 处理 props 属性的设置

              - 遍历 props（过滤 vue 生命周期相关的 key），调用`hostPatchProp(el, key, null, props[key])`，后两个参数的意义为“旧值”与“新值”，主要通过 switch...case...语句来处理属性，比如通过`element.removeAttribute`和`element.setAttribute`来删除（新值为 null 时）或添加属性，包括事件绑定（通过`element.addEventListener`）

              1. 调用`hostInsert`插件元素，同时在之前和之后触发如下钩子：

              - vnodeHook -> onVnodeBeforeMount
              - DirectiveHook -> beforeMount
              - transition -> beforeEnter
              - vnodeHook -> onVnodeMounted
              - DirectiveHook -> mounted
              - transition -> enter

            - n1 存在，执行更新操作，调用`updateElement`
              - 调用`patchProps`来对比新旧属性
                1. 新旧属性都存在，但值不同，遍历新节点中的属性，调用`hostPatchProp(el, key, prevProp, nextProp)`来更新属性的值
                1. 旧属性存在，但新属性中不存在，遍历旧节点中的属性，对应的 key 不存在于新属性，调用`hostPatchProp(el, key, prevProp, nextProp)`来更新属性的值
              - 调用`patchChildren`来对比新旧子节点
                1. 子节点为文本类型时，直接使用`hostSetElementText`设置当前节点的文本为新值
                1. 子节点为数据，则调用`patchKeyedChildren`，此过程略复杂，根据子节点的 type 与 key 值，分别由左向右或由右向左，递归调用`patch`方法，然后考虑新旧节点数量不一致是新增节点还是删除节点，最后对比节点列表的中间部分比如：`a,b,[c,d,e],f,g`与`a,b,[e,c,d],f,g`

          - `processComponent(n1,n2,container,parentComponent)`
            - n1 不存在，调用`mountComponent(n2, container, parentComponent);`
              - 调用`createComponentInstance`创建组件实例instance：
                ```javascript
                const instance = {
                  type: vnode.type,
                  vnode,
                  props: {},
                  parent,
                  provides: parent ? parent.provides : {}, // 获取 parent 的 provides 作为当前组件的初始化值 这样就可以继承 parent.provides   的属性了
                  proxy: null,
                  isMounted: false,
                  attrs: {}, // 存放 attrs 的数据
                  slots: {}, // 存放插槽的数据
                  ctx: {}, // context 对象
                  setupState: {}, // 存储 setup 的返回值
                  emit: () => {},
                };
                instance.ctx = {
                  _: instance,
                };
                instance.emit = emit.bind(null, instance) as any;
                ```
              - 调用`setupComponent(instance)`
                - 处理props，调用`initProps`将n2（vnode）的props挂载到instance上
                - 处理slots，调用`initSlots`,根据instance.vnode的shapeFlag，如果为ShapeFlags.SLOTS_CHILDREN，表示子组件是一个component，则调用`normalizeObjectSlots`，遍历子节点，如果其值为函数类型，则挂载到instance.slots中
                - 调用`setupStatefulComponent(instance)`处理通过options创建的组件，即stateful component
                  使用ES6的proxy对instance.ctx创建代理，创建proxy的第二个参数为`PublicInstanceProxyHandlers`，其中就用于定义各种拦截操作，通过instance.type可以拿到组件的定义，其中就包括了setup函数，如果存在：
                    1. 调用`setCurrentInstance(instance)`设置currentInstance的值（所以在setup中可以通过getCurrentInstance获取当前的实例）
                    1. 调用`createSetupContext`，抽取当前实例上的attrs、slots、emit等来组建setup中的参数context
                    1. 调用setup函数，参数为`shallowReadonly(instance.props)`以及刚才创建的`setupContext`，`shallowReadonly`引自库"@vue/reactivity"
                    1. `setCurrentInstance(null)`
                    1. `handleSetupResult(instance, setupResult)`，处理setup的返回值，如果返回的是一个函数，则作为instance上的render，如果是一个对象，则使用`proxyRefs`处理后作为instance上的setupState（proxyRefs 的作用就是把 setupResult 对象做一层代理，方便用户直接访问 ref 类型的值，比如 setupResult 里面有个 count 是个 ref 类型的对象，用户使用的时候就可以直接使用 count 了，而不需要在 count.value）
                      - 如果实例上没有设置render，则对调用compile对组件的template进行编译生成render，并将其挂载到实例上（instance.render = Component.render），这里包括对vue 2.x语法的兼容处理
              - `setupRenderEffect(instance, container)`
                - 通过@vue/reactivity提供的effect方法来设置组件更新时的effect函数，这里会根据组件的isMounted属性来区分逻辑，未挂载则调用实例的render函数生成渲染树（vnode），然后触发`beforeMount`与`onVnodeBeforeMount`钩子，再调用patch函数进行挂载节点；如果已挂载，则调用render生成新的渲染树，与组件当前的渲染树交换，并通过patch函数来进行数据的更新。

            - n1 存在，调用`updateComponent(n1, n2, container);`


