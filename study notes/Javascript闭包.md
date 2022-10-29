# 理解Javascript中的闭包

## 什么是闭包

闭包是一个函数，即使外部函数已经返回，它仍可以访问外部函数的作用域。这意味着闭包可以记住并访问外部函数的变量和参数，即使该函数已经完成。这深入讨论闭包之前，我们先来理解一下词法作用域的概念。

### 什么是词法作用域

javascript中的词法作用域或静态作用域指的是在代码中基于变量、函数以及对象的物理位置的可访问性。比如：

```javascript
let a = 'global';
function outer() {
  let b = 'outer';
  function inner() {
    let c = 'inner'
    console.log(c);   // prints 'inner'
    console.log(b);   // prints 'outer'
    console.log(a);   // prints 'global'
  }
  console.log(a);     // prints 'global'
  console.log(b);     // prints 'outer'
  inner();
}
outer();
console.log(a);         // prints 'global'
```

这里`inner`函数可以访问其自己作用域中、`outer`函数作用域中以及全局作用域中定义的变量，而`outer`函数可以访问自己作用域中以及全局作用域中定义的变量，上述代码的作用域链如下所示：

```javascript
Global {
	outer {
	  inner
	}
}
```

可以注意到，`inner`函数被`outer`函数的词法作用域所包围，而`outer`函数又被全局作用域所包围。这就是为什么`inner`函数可以访问`outer`函数以及全局作用域中定义的变量。

## 闭包是如何工作的

为了弄清javascript中闭包是如何工作的，我们先要理解两个重要的概念：执行上下文与词法环境

### 执行上下文

执行上下文是一个抽象的环境，javascript的代码就是在这个环境被评估和执行，全局代码会在全局执行上下文中运行，函数中的代码会在函数的执行上下文中运行，由于javascript的执行线程是单线程的，所以同一时间只会有一个执行上下文在运行，所有的执行上下文都在被称作执行栈或调用栈的栈（LIFO）数据结构中维护，栈顶元素即为当前运行中的执行上下文，当函数执行完毕，其执行上下文就会从栈顶出栈，控制权会交给栈中下一个执行上下文。

### 词法环境

每次javascript引擎创建执行上下文来执行函数或全局代码时，会同时创建一个新的词法环境来存储在函数执行期间定义的变量。关于词法环境的描述可以查看文档 [javascript执行上下文](javascript执行上下文.md) ，我们以如下代码示例来看一下词法环境大概的结构：

```javascript
let a = 'Hello World!';
function first() {
  let b = 25;  
  console.log('Inside first function');
}
first();
console.log('Inside global execution context');
```

这段代码执行时，首先会创建如下的全局词法环境：

```javascript
globalLexicalEnvironment = {
  environmentRecord: {
      a     : 'Hello World!',
      first : < reference to function object >
  }
  outer: null
}
```

当执行`first`函数时，会创建如下的函数词法环境：

```javascript
functionLexicalEnvironment = {
  environmentRecord: {
      b    : 25,
  }
  outer: <globalLexicalEnvironment>
}
```

此时`outer`指向外部的全局词法环境，因为在源码中`first`函数是处于全局作用域中的。

这里需要注意一点，当函数执行完闭，它的执行上下文从栈上移除，但它的词法环境是否被从内存中移除？这取决于它是否被其他词法环境中的`outer`词法环境属性所引用，下面我们看几个例子

## 具体的闭包例子

### 示例1

```javascript
function person() {
  let name = 'Peter';
  
  return function displayName() {
    console.log(name);
  };
}
let peter = person();
peter(); // prints 'Peter'
```

当`person`函数执行时，javascript引擎会创建一个新的执行上下文和词法环境，在函数执行完成后，它返回`displayName`函数并将它赋值给变量`peter`，所以`person`的词法环境如下：

```javascript
personLexicalEnvironment = {
  environmentRecord: {
    name : 'Peter',
    displayName: < displayName function reference>
  }
  outer: <globalLexicalEnvironment>
}
```

当`person`执行结束 ，它的执行上下文会从调用栈中移除，但它的词法环境依旧在内存中，因为它的词法环境被它内部的`displayName`函数的词法环境所引用。所以`person`中定义的变量依旧可访问。

请注意，创建`personLexicalEnvironment`时，javaScript引擎会将`personLexicalEnvironment`附加到该词法环境内部的所有函数定义中（**也就是说函数的`outer`所指向的词法环境是与函数定义的位置有关联的**）。 这样，以后如果调用任何内部函数，javaScript引擎便可以将外部词法环境设置为附加到该函数定义的词法环境。当`peter`函数执行时，javascript会创建一个新的执行上下文和词法环境，如下：

```javascript
displayNameLexicalEnvironment = {
  environmentRecord: {
    
  }
  outer: <personLexicalEnvironment>
}
```

由于`dispayName`函数中未定义任何变量，所以环境记录（environmentRecord）为空，当函数执行时，javascript引擎会尝试在函数的词法环境中查找变量`name`，由于当时函数未定义变量，所以它会在外部的词法环境中查询该变量，由于此时外部的词法环境依旧存储于内存，javascript引擎就能找到该变量并输出到控制台。

### 示例2

```javascript
function getCounter() {
  let counter = 0;
  return function() {
    return counter++;
  }
}
let count = getCounter();
console.log(count());  // 0
console.log(count());  // 1
console.log(count());  // 2
```

当执行`getCounter`函数时，其词法环境结构如下：

```javascript
getCounterLexicalEnvironment = {
  environmentRecord: {
    counter: 0,
    <anonymous function> : < reference to function>
  }
  outer: <globalLexicalEnvironment>
}
```

当执行`count`函数时，其词法环境结构如下：

```javascript
countLexicalEnvironment = {
  environmentRecord: {
  
  }
  outer: <getCountLexicalEnvironment>
}
```

如前一个例子一样，`count`函数执行时，会在`getCounterLexicalEnvironment`中查找`counter`变量，所以当第一次调用完成之后，`getCounterLexicalEnvironment`会变成如下：

```javascript
getCounterLexicalEnvironment = {
  environmentRecord: {
    counter: 1,
    <anonymous function> : < reference to function>
  }
  outer: <globalLexicalEnvironment>
}
```