#   Svelte学习

## 一、快速入门

[60秒快速开始svelte](https://www.sveltejs.cn/blog/the-easiest-way-to-get-started)

[Svelte REPL](https://www.sveltejs.cn/repl/hello-world?version=3.45.0)

[degit—a project scaffolding tool](https://github.com/Rich-Harris/degit)

[Svelte Template](https://github.com/sveltejs/template)

[Work with TypeScript](https://www.sveltejs.cn/blog/svelte-and-typescript)

发布工具：

- [Vercel](https://vercel.com/)
- [Surge](https://surge.sh/)

构建工具：

- [rollup-plugin-svelte](https://github.com/sveltejs/rollup-plugin-svelte)
- [svelte-loader](https://github.com/sveltejs/svelte-loader)

VS Code插件

- [svelte for vscode](https://marketplace.visualstudio.com/items?itemName=svelte.svelte-vscode)

## 二、语法层面

- 反应式声明

  ```javascript
  // 声明反应式的值
  let count = 0;
  $: doubled = count * 2;
  
  // 运行反应式的语句
  // demo-1
  $: console.log(`the count is ${count}`);
  // demo-2
  $: {
    console.log(`the count is ${count}`);
    alert(`I SAID THE COUNT IS ${count}`);
  }
  // demo-3
  $: if (count >= 10) {
  	alert(`count is dangerously high!`);
    count = 9;
  }
  ```

- 数组和对象的更新

  svelte的反应性是由赋值语句触发的，因此使用数组的push、splice等方法是不会触发自动更新的，解决方法如下：

  1. 添加多余的赋值

     numbers.push(1);

     numbers = numbers;

  2. 使用扩展表达式

     numbers = [...numbers, 1];

  3. 对数组或对象的属性进行赋值

     numbers[numbers.length] = 1;

  **被更新的变量的名称必须出现在赋值语句的左侧，否则不会更新**

- 组件属性

  在.svelte中通过export声明属性，在另一个组件中引入该组件，并以这个应导出的变量名作为属性并进行赋值，从而实现值的传递

  属性的传递还可以通过...语法+对象的形式：

  ```javascript
  <Info {...pkg}/>
  ```

  被引用的组件还可以通过$$props直接获取所有传递的值，但通常不建议这么做，因为Svelte难以优化

- if语句

  ```html
  {#if user.loggedIn}
    <button on:click={toggle}/>
    	Log out
  	</button>
  {:else}
  	<button on:click={toggle}/>
  		Log in
  	</button>
  {/if}
  ```

- each语句

  ```html
  // 括号中的cat.id用于作为key，可以将任何对象用作key，但字符串或数字通常更安全，因为能确保它的唯一性
  {#each cats as cat (cat.id)}
  	<li>{cat.name}</li>
  {/each}
  ```

- await块

  ```html
  {#await promise}
  	<p>...waiting</p>
  {:then number}
  	<p>The number is {number}</p>
  {:catch error}
  	<p style="color: red">{error.message}</p>
  {/await}
  ```

- 事件

  通过“on:事件名”来绑定事件，同时可以在事件名后通过“|修饰符”来添加额外信息，比如“once”表示事件处理程序只运行一次，所有修饰符列表：

  - preventDefault
  - stopPropagation
  - passive
  - capture
  - once
  - self

  还可以组合使用，比如“on:click|once|capture={...}”

- 组件事件

  ```javascript
  import { createEventDispatcher } from 'svelte';
  // 必须在首次实例化组件时调用
  const dispatch = createEventDispatcher();
  
  function sayHello() {
  	dispatch('message', {
  		text: 'Hello!'
  	})
  }
  ```

  

- 组件事件转发

  组件事件不支持冒泡，所以对于App->Outer->Inner的组件结构中，Inner抛出的message事件，必须在Outer层做一次转发才能被App捕获到，方法如上一节描述，但svelte为了简写，可以在Outer组件上添加“on:message”属性，这样Outer组件会转发所有message事件

  ```html
  <Inner on:message/>
  ```

  **原生的DOM事件也支持这种方式进行转发**

- 双向绑定

  ```html
  <!--普通输入框-->
  <input bind:value={name}>
  <!--针对number类型的处理-->
  <input type=number bind:value={a} min=0 max=10>
  <input type=range bind:value={a} min=0 max=10>
  <!--checkbox的处理-->
  <input type=checkbox bind:checked={yes}>
  <!--radio group处理，绑定到同一个值，单选互斥-->
  <input type=radio bind:group={scoops} value={3}>
  <!--checkbox group处理，绑定到同一个值，多选，值为一个数组-->
  <input type=checkbox bind:group={flavours} value={flavour}>
  <!--文本域的处理-->
  <textarea bind:value={value}></textarea>
  <!--值与变量名相同时可简写-->
  <textarea bind:value></textarea>
  <!--select的处理，如果selected没值，则直接使用option中的第一个-->
  <select bind:value={selected} on:change="{() => answer = ''}">
  <!--可编辑div的处理-->
  <div
  	contenteditable="true"
  	bind:innerHTML={html}
  ></div>
  <!--媒体标签的绑定-->
  <video
  	poster="https://sveltejs.github.io/assets/caminandes-llamigos.jpg"
  	src="https://sveltejs.github.io/assets/caminandes-llamigos.mp4"
  	on:mousemove={handleMousemove}
  	on:mousedown={handleMousedown}
  	bind:currentTime={time}
  	bind:duration
  	bind:paused
  ></video>
  ```

## 三、生命周期

- onMount

  组件第一次渲染为DOM时触发，如果返回一个函数，则该函数会在组件销毁时调用

  ```javascript
  import { onMount } from 'svelte';
  
  let photos = [];
  
  onMount(async () => {
  	const res = await fetch(`https://jsonplaceholder.typicode.com/photos?_limit=20`);
  	photos = await res.json();
  });
  ```

- onDestroy

  组件销毁时调用

  关于在何处调用onDestroy并没有明确的要求，也就是说，你可以按[这里](https://svelte.dev/tutorial/ondestroy)所示的方式，将生命周期函数封装在utils.js中，然后在组件中引入使用，在组件销毁时一样生效

  **怎么实现的?**

- beforeUpdate/afterUpdate

  前者在DOM更新之前立即执行（这里要注意beforeUpdate第一次执行是在组件挂载之前，所以在获取DOM元素时要判断是否存在），后者则是在异步数据加载完成后执行，二者配合使用可以实现一个在纯状态驱动模式下比较难实现的指令式操作，比如更新元素的滚动位置

- tick

  在Svelte中每当组件状态变更时，DOM不会立刻更新，而是会等待直到下次微任务的执行，以此来查看是否有其他变化或组件需要更新，这与Vue中的nextTick原理一致，只不过API层面不同，tick返回的是一个Promise，这个Promise会在之前挂起的状态变更被应用到DOM之后立刻resolve，所以当更新状态后，想要立刻获取新的状态，可以在tick返回的promise被resolve之后，也可以结合async...await语法

## 四、Store

- writable stores

  一个拥有subscribe方法的简单对象，通过这个方法可以让其他部分来订阅通知，这样在store中的数据发生变化时，可以得到回调

  ```javascript
  // store.js 声明store
  import { writable } from 'svelte/store';
  // 初始值为0
  export const cout = writable(0);
  
  // Decrementer.svelte中操作该store
  import { count } from './store.js';
  // 该组件通过store的update方法来更新值
  function decrement() {
    count.update(n => n - 1);
  }
  
  // App.svelte中订阅store，当Decrementer对store进行操作后，会拿到订阅通知并更新count_value
  import { count } from './store.js;
  import Decrementer from './Decrementer.svelte;
  
  let count_value;
  count.subscrible(value => {
    count_value = value;
  });
  ```

  **subscribe方法返回一个unsubscribe方法，调用后可以取消订阅（比如在组件的onDestroy生命周期里）**

- 自动订阅

  对于在在组件最外层导入的store，可以通过前缀$+store名的方式进行自动订阅

  ```html
  <script>
    import { count } from 'store.js';
  </script>
  <h1>The count is {$count}</h1>
  ```

  **使用这种方式进行订阅，可以不用在onDestroy中进行unsubscribe操作**

  **所有以\$开头的变量都会认为是引用一个store值，所以Svelte会禁止你以$为前缀声明自己的变量**

- readable stores

  用于只读场景

  ```javascript
  import { readable } from 'svelte/store';
  
  // readable的第一个参数是初始值，如果不需要可以设为null或undefined
  // start函数会在store拿到第一个订阅时被调用
  // 返回的stop函数会在最后一个订阅者调用unsubscribe时调用
  export const time = readable(new Date(), function start(set) {
  	const interval = setInterval(() => {
    	set(new Date());
    }, 1000);
    
    return function stop() {
    	clearInterval(interval);
    }
  }
  ```

- store的继承

  可以通过derived关键字来创建一个基本其他store值的store，类型于vue中的计算属性

  ```javascript
  import { derived } from 'svelte/store';
  //这里关于time的定义省略，参见前一个例子
  
  const start = new Date();
  export const elapsed = derived(time, $time => Math.round(($time - start) / 1000));
  ```

  关于如何继承多store，参见API文档

- 创建自定义store

  只要一个对象正确的实现了subscribe方法，它就是一个store，所以可以定制store的逻辑，进行封装

  ```javascript
  import { writable } from 'svelte/store';
  
  function createCount() {
  	const { subscribe, set, update } = writable(0);
  
  	return {
  		subscribe,
  		increment: () => update(n => n + 1),
  		decrement: () => update(n => n - 1),
  		reset: () => set(0)
  	};
  }
  
  export const count = createCount();
  ```

- 绑定store

  如果是writable store，即它拥有一个set方法，则可以使用前文提到bind语法，就像绑定本地组件的状态，也可以在组件内直接赋值，效果就相当于调用了store的set方法

  ```html
  <button on:click="{() => $name += '!'}">
  	Add exclamation mark!
  </button>
  ```

## 五、动效

- tweened——补间动画

  使用tweened代替store，可以创建一个补间动画，同时配置缓动函数可以实现华丽的效果，如下

  ```javascript
  import { tweened } from 'svelte/motion';
  import { cubicOut } from 'svelte/easing';
  
  const progress = tweened(0, {
    duration: 400,
    easing: cubicOut,
  });
  ```

  svelte/easing模块包含了[Penner easing equations](https://web.archive.org/web/20190805215728/http://robertpenner.com/easing/)提到各种缓动函数

  tweened的所有配置项如下：

  - delay——延迟时间，单位毫秒
  - duration——动画时长，单位毫秒，或者是一个函数：`(from, to) => milliseconds`
  - easing——函数`p => t`，其中p和t的值都介于0到1之间
  - interpolate——插件函数`(from, to) => t => value`默认情况下Svelte会在数字、日期或者特定类型的数组和对象（只要它们仅包括数字、日期以及其它合法的数组和对象），对比变换矩阵

  tweened也具有set与update方法，其返回值为一个promise，会在动画结束后resolved

- spring——弹性动画

  在一些场景下代替tweened，实现一个弹性动画，模拟物理惯性，[参见这里的Demo](https://svelte.dev/tutorial/spring)

  ```javascript
  import { spring } from 'svelte/motion';
  
  let coords = spring({ x: 50, y: 50 }, {
  	stiffness: 0.1,
  	damping: 0.25
  });
  ```

  它具有两个属性：stiffness（刚性）damping（阻尼），通过修改这两个值来实现不同的动效，两值均有默认值

## 六、过渡

- transition指令

  svelte提供过渡指令及一系列特殊，来实现一个优雅的过渡动画效果

  ```html
  <script>
    import { fade } from 'svelte/transition';
    let visible = true;
  </script>
  {#if visible}
  	<p transition:fade>Fade in and out</p>
  {/if}
  ```

  并且支持参数传递

  ```html
  <script>
  	import { fly } from 'svelte/transition';
    let visible = true;
  </script>
  
  {#if visible}
    <!--注意这里的语法-->
  	<p transition:fly="{{ y:200, duration: 20000}}">Fade in and out</p>
  {/if}
  ```

- in和out指令

  使用in和out指令替换transition指令，可以分别指定组件加载和移除时的动画

  ```html
  <p in:fly="{{ y: 200, duration: 2000 }}" out:fade>
  		Flies in, fades out
  </p>
  ```

- 自定义css过渡

  svelte/transition模块提供了一系列内置的过渡效果，同时我们也可自定义，比如内置的fade过渡其实现如下：

  ```javascript
  function fade(node, {
    delay = 0,
    duration = 400,
  }) {
    const o = +getComputedStyle(node).opacity;
    
    return {
      delay,
      duration,
      css: t => `opacity: ${t * o}`
    };
  }
  ```

  过渡函数具有两个参数，第一个为要使用过渡动画的DOM元素，第二个则是一个对象，里面可定义任意属于，用于将来在使用时传递额外参数，函数的返回值是一个对象，其中包含以下属性：

  - delay——延迟时间，单位毫秒
  - duration——动画时长，单位毫秒，或者是一个函数：`(from, to) => milliseconds`
  - easing——函数`p => t`，其中p和t的值都介于0到1之间
  - css——函数`(t, u) => css`，其中`u === 1 - t`
  - tick——函数`(t, u) => {...}`，用于给DOM元素添加一些效果

  这里的t的值，对于in指令，由0到1的变化，对于out指令，则从1到0变化

  绝大多数情况下都应该使用css属性而不是tick，因为css会在主线程中执行，尽可能的阻止jank（指浏览器无响应），svelte会模拟这个过渡并构造css动画，比如上面的fade最终生成的css动画如下：

  ```css
  0% { opacity: 0 }
  10% { opacity: 0.1 }
  20% { opacity: 0.2 }
  /* ... */
  100% { opacity: 1 }
  ```

  [自定义的css过渡](https://svelte.dev/tutorial/custom-css-transitions)

- 自定义js过渡

  针对一些无法通过css实现的过渡效果，这时就要通过js的方式来实现，也就是使用到上一节中提到的tick函数，比如实现打字机效果：

  ```javascript
  function typewriter(node, { speed = 1 }) {
  	const valid = (
  		node.childNodes.length === 1 &&
  		node.childNodes[0].nodeType === Node.TEXT_NODE
  	);
  
  	if (!valid) {
  		throw new Error(`This transition only works on elements with a single text node child`);
  	}
  
  	const text = node.textContent;
  	const duration = text.length / (speed * 0.01);
  
  	return {
  		duration,
  		tick: t => {
  			const i = Math.trunc(text.length * t);
  			node.textContent = text.slice(0, i);
  		}
  	};
  }
  ```

- 过渡事件

  svelte定义了四种过渡相关的事件，分别为

  - introstart
  - introend
  - outrostart
  - outroend

  ```html
  <p
  	transition:fly="{{ y: 200, duration: 2000 }}"
  	on:introstart="{() => status = 'intro started'}"
  	on:outrostart="{() => status = 'outro started'}"
  	on:introend="{() => status = 'intro ended'}"
  	on:outroend="{() => status = 'outro ended'}"
  >
  	Flies in and out
  </p>
  
  ```

- 局部过渡——local

  比如我一个列表，通过each遍历生成，并添加了slide动效，但此时对于整个列表可见性的变化也被应用了slide过渡效果，但期望的是仅对列表项的可见性添加slide过渡，此时就可以使用local关键字

  ```html
  <div transition:slide|local>
  	{item}
  </div>
  ```

- 延时过渡

  直接看[Demo](https://www.sveltejs.cn/tutorial/deferred-transitions)比较直观

- key block

  默认情况下过渡动画是在元素在进入或离开DOM时执行，但通过key block，可以指定当表达式的值变化时就执行过度动画

  ```html
  // 当number变化时，则会执行过渡动画
  {#key number}
  		<span style="display: inline-block" in:fly={{ y: -20 }}>
  			{number}
  		</span>
  {/key}
  ```

- animation指令

  直接看[Demo](https://www.sveltejs.cn/tutorial/animate)，这里类型vue中的[FLIP](https://aerotwist.com/blog/flip-your-animations/)动画

## 七、Action

Action本质上是元素级的生命周期函数，它适用于以下场景：

- 与第三方库交互
- 图片懒加载
- 工具提示信息
- 添加自定义事件处理器

Action的定义是一个函数与Transition函数一样，接受一个node节点（应用Action的元素）和一些可选项作为参数，返回一个Action对象，对象中可以定义一些生命周期函数，比如destroy函数会在元素卸载时被调用，如果传递参数，并希望参数是动态更新的，在返回对象时，要定义一个update函数，入参为新的参数

## 八、Class指令

```html
<button
	class="{current === 'foo' ? 'selected' : ''}"
	on:click="{() => current = 'foo'}"
>foo</button>

<!--改为-->

<button
	class:selected="{current === 'foo'}"
	on:click="{() => current = 'foo'}"
>foo</button>
```

## 九、Slots

基本概念与vue中的插槽一节类似

## 十、 Context API

与React中的Context概念类型，在有父子层级关系中进行跨节点传递信息，比如祖先节点与子孙节点之间

## 十一、svelte开头的特殊节点

## 十二、模块上下文：Module Context

```html
// 组件中通过此属性设置的代码，只执行一次，可以理解为共用的上下文，并且声明的值可以在组件的script与html部分访问，但反之则不行
<script context="module">
	let current;
</script>
```

**注意：在此模块中定义的变量不是响应式的**

## 十三、@debug Tag

```html
// 当user的值发生变化时，会触发debug，并在控制台打印user的内容
{@debug user}
```

