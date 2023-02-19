# AST在解析咪咕js-sdk中的应用

## 1. 背景

针对特定的业务场景（短验B模式），为优化用户体验、提升业务订购转化率，需要了解咪咕的js-sdk中的具体业务逻辑，从而针对性的优化，但咪咕js-sdk所涉及的两个核心js文件进行了一些混淆处理，直接阅读源码比较困难，这里提供了其中一个js文件（代码量较少，方便分析）的[链接](https://m.12530.com/order/pub-ui/js/and/migus.js)

*如果访问不了，请使用手机网络代理，咪咕做了些处理，触发特定规则之后可能会封IP*

下载该js文件，通过vscode打开并格式化，大概浏览下（咪咕的js代码偶有调整，当前的版本格式化后为1700余行），整个代码的结构分为两部分：变量声明+IIFE，在IIFE中通过一个工厂函数来给暴露到全局的`migusdk`对象添加属性和方法：

![image-20230215092111687](../media/image-20230215092111687.png)

而这个工厂函数里，主要是结合for循环+switch...case语句的嵌套（这种混淆方式的特点）来声明私有函数及给migusdk添加属性和方法：

![image-20230215092550875](../media/image-20230215092550875.png)

更多细节有兴趣的同学可以直接查看源码

## 2. 思路

首先，靠人工“翻译”的方式肯定不可行，费时费力，而且源码一旦调整就意味着要从头开始翻译……

转换一下思路，无论使用何种方式进行了混淆，它都是一段可以在浏览器正常执行的js代码，那是否可以模拟js执行过程来进行“还原”，举个简单的例子：

```javascript
// 这里只是举例
var dsbtviyOR4Dp136Xb = '_s',
  JHK8gqk0LxL1 = 'pyzy',
  UfLd4MMf0LHM3EQn = 'xywwyMGu|kw 0',
// ... 
sdk.log(dsbtviyOR4Dp136Xb + JHK8gqk0LxL1 + UfLd4MMf0LHM3EQn)
```

其执行结果与下面的代码是等价的

```javascript
// ... 
sdk.log('_spyzyxywwyMGu|kw 0')
```

并且源码中头部声明的这些变量，在使用上也是符合这种预期的（只读常量），那我们不妨先将头部这些变量声明替换掉，说不定会豁然开朗

说到这里，对于日常和“构建”这一过程打交道的前端同学一定能想到，这个模式不就是我们使用babel将ES6代码转译ES5代码的过程吗：输入一段代码，经过一系列处理，输出另一段代码

babel正是我们需要的工具，虽然大家日常工作中经常使用到babel，但有些同学可能对于它的原理不是那么了解，这里先简单介绍下

## 3. Babel与AST

babel转译代码的过程分为三步：parse、transform、generate

### 1. parse

parse用于将源码转换成一种方便程序理解和处理的数据结构，这种数据结构正是抽象语法树（Abstract Syntax Tree，AST），比如对于`const fruit = "apple";`对应的AST大致如：

![ast-demo](../media/ast-demo.svg)

图中省略了一些细节，比如这里使用了`const`来声明变量，那对于`VariableDeclaration`节点的`kind`属性，其值为`const`，即表示当前定义的是一个常量

完整的AST可以使用[astexplorer](https://astexplorer.net/)来查看，比如上面的例子

![image-20230215122338598](../media/image-20230215122338598.png)

### 2. transform

在拿到parse阶段输出的AST后，就可以遍历AST，找到目标节点，然后修改它。如何遍历？babel使用**访问者模式**，它的作用是将操作特定类型数据结构的逻辑与数据结构本身解藕，babel提供的用于遍历AST的API如下：

```javascript
traverse(ast, visitor)
```

这里的visitor对象的key对应节点的类型名称，value就是操作该节点的函数（这其实是一种省略写法，如果需求更细粒度的控制，这里的value可以定义为以enter和exit为key的对象，它们对应的value是操作节点的函数，分别表示进入节点和退出节点时的执行逻辑，当使用省略写法时，默认作为enter来使用了），比如：

```javascript
traverse(ast, {
  VariableDeclarator: {
    enter: path => {},
    exit: path => {},
  }
})
```

或

```javascript
traverse(ast, {
	VariableDeclarator: path => {},
})
```



这里的path表示遍历时的路径信息（它的功能很强大），通过它可以方便的访问当前节点、父路径、作用域以及修改节点信息（我们这里就会大量用到）等等
