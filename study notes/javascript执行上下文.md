# Javascript 执行上下文

## 概念

javascript代码评估与执行环境的一个抽象概念

## 分类

- 全局执行上下文
	默认或者说是基础的执行上下文，任何不在函数中的代码都属于全局执行上下文。它做两件事：创建一个全局对象（比如浏览器中的window对象）并且将`this`指向这个全局对象，全局对象仅允许存在一个
	
- 函数执行上下文
  每次函数执行时，都会创建一个全新的执行上下文，每个函数拥有自己的执行上下文。函数执行上下文可以有任意多个，函数执行上下文在创建时会经历一系列预定义好的步骤（后文会提到）
  
- eval的执行上下文
  eval函数中执行的代码也有自己的执行上下文，但eval现在用得越来越少，也不推荐使用，所以暂不深入讨论

## 执行栈

执行栈也称作“调用栈”，是一个后进先出的栈结构，用来存储代码执行期间的执行上下文。当js引擎首次解析到你的代码时，它会创建一个全局执行上下文，并压入执行栈，后面每当引擎遇到函数调用，就会再创建一个新的执行上下文并压入到栈顶，引擎执行栈顶的执行上下文对应的函数，当函数执行完之后，该执行上下文会出栈，控制权交给执行栈中下一个执行上下文。

## 执行上下文的创建过程

执行上下文的创建分为两个阶段：创建阶段与执行阶段

### 创建阶段

会创建如下的数据结构

```javascript
// 执行上下文
ExecutionContext = {
  // 词法环境
  LexicalEnvironment = <ref. to LexicalEnvironment in memory>,
  // 变量环境
  VariableEnvironment = <ref. to VariableEnvironment in  memory>,
}
```

#### 词法环境

ES6官方文档对词法环境的描述如下：

>A Lexical Environment is a specification type used to define the association of Identifiers to specific variables and functions based upon the lexical nesting structure of ECMAScript code. A Lexical Environment consists of an Environment Record and a possibly null reference to an outer Lexical Environment.

词法环境是一种规范类型，用于根据ECMAScript代码的词法嵌套结构来定义标识符与特定变量和函数的关联。 词法环境由环境记录和对外部词法环境的可能为空的引用组成。

简而言之，词法环境是一个保存了标识符与变量映射关系的结构（这里“标识符”是指变量或函数的名字，而“变量”是指对实际对象（包括函数对象与数组对象）或原始值的引用），比如下面的代码：

```javascript
var a = 20;
var b = 40;
function foo() {
  console.log('bar');
}
```

对应的词法环境如下：

```javascript
lexicalEnvironment = {
  a: 20,
  b: 40,
  foo: <ref. to foo function>
}
```

每个词法环境由以下三个部分组成：

1. 环境记录
2. 对外层环境的引用
3. `this`的绑定

#### 环境记录

环境记录是词法环境中存储变量与函数声明的地方，有两类环境记录：

- 声名式的环境记录——存储变量和函数声明。函数代码的词法环境包含一个声明性环境记录。
- 对象环境记录——全局代码的词法环境包含一个对象环境记录，除了变量与函数声明之外，对象环境记录还存储了一个全局绑定对象（比如浏览器中的window）。因此，对于每个绑定对象的属性（对于浏览器，它包含浏览器提供给window对象的属性和方法），将在记录中创建一个新条目。

注意，对于函数代码，环境记录还包含一个`arguments`对象，它包含了传递给该函数的参数与其索引的映射关系以及参数的长度。

#### 对外层环境的引用

这意味着可以访问外层的词法环境，也就是说当javascript引擎在当前的词法环境中找不到某个变量时，可以通过它在外层的环境中查找变量

#### `this`绑定

用来设置或决定`this`的值，在全局执行上下文中，它的值指向全局对象（比如浏览器中它指向window对象），在函数执行上下文中，`this`的值取决于函数的调用方式。如果它由对象引用调用，则`this`指向该对象，否则它指定全局对象或`undefined`（严格模式下）

通过伪代码来进行抽象，词法环境如下所示：

```javascript
GlobalExectionContext = {
  LexicalEnvironment: {
    EnvironmentRecord: {
      Type: "Object",
      // Identifier bindings go here
    }
    outer: <null>,
    this: <global object>
  }
}
FunctionExectionContext = {
  LexicalEnvironment: {
    EnvironmentRecord: {
      Type: "Declarative",
      // Identifier bindings go here
    }
    outer: <Global or outer function environment reference>,
    this: <depends on how function is called>
  }
}
```

#### 变量环境

变量环境其实也是一个词法环境，它的环境记录保存了当前执行上下文中通过变量声明语句创建的绑定，它也拥有词法环境中定义的属性，在ES6中，词法环境与变量环境的一个区别就是，前者用来存储函数声明以及通过`let`和`const`声明的变量绑定，而后者仅用于存储`var`声明的变量绑定

### 执行阶段

在这个阶段，将完成对所有这些变量的赋值，并最终执行代码。

### 示例

如下代码：

```javascript
let a = 20;
const b = 30;
var c;
function multiply(e, f) {
 var g = 20;
 return e * f * g;
}
c = multiply(20, 30);
```

上述代码执行时，javascript引擎会创建一个全局执行上下文来执行全局代码，在创建阶段全局执行上下文如下：

```javascript
GlobalExectionContext = {
  LexicalEnvironment: {
    EnvironmentRecord: {
      Type: "Object",
      // Identifier bindings go here
      a: < uninitialized >,
      b: < uninitialized >,
      multiply: < func >
    }
    outer: <null>,
    ThisBinding: <Global Object>
  },
  VariableEnvironment: {
    EnvironmentRecord: {
      Type: "Object",
      // Identifier bindings go here
      c: undefined,
    }
    outer: <null>,
    ThisBinding: <Global Object>
  }
}
```

在执行阶段，完成变量的赋值，全局执行上下文如下：

```javascript
GlobalExectionContext = {
LexicalEnvironment: {
    EnvironmentRecord: {
      Type: "Object",
      // Identifier bindings go here
      a: 20,
      b: 30,
      multiply: < func >
    }
    outer: <null>,
    ThisBinding: <Global Object>
  },
VariableEnvironment: {
    EnvironmentRecord: {
      Type: "Object",
      // Identifier bindings go here
      c: undefined,
    }
    outer: <null>,
    ThisBinding: <Global Object>
  }
}
```

接着当执行到`multiply(20, 30)`，会创建一个如下的函数执行上下文，其在创建阶段如下所示：

```javascript
FunctionExectionContext = {
LexicalEnvironment: {
    EnvironmentRecord: {
      Type: "Declarative",
      // Identifier bindings go here
      Arguments: {0: 20, 1: 30, length: 2},
    },
    outer: <GlobalLexicalEnvironment>,
    ThisBinding: <Global Object or undefined>,
  },
VariableEnvironment: {
    EnvironmentRecord: {
      Type: "Declarative",
      // Identifier bindings go here
      g: undefined
    },
    outer: <GlobalLexicalEnvironment>,
    ThisBinding: <Global Object or undefined>
  }
}
```

之后，进入执行阶段，完成函数中的变量赋值，函数执行上下文的结构如下：

```javascript
FunctionExectionContext = {
LexicalEnvironment: {
    EnvironmentRecord: {
      Type: "Declarative",
      // Identifier bindings go here
      Arguments: {0: 20, 1: 30, length: 2},
    },
    outer: <GlobalLexicalEnvironment>,
    ThisBinding: <Global Object or undefined>,
  },
VariableEnvironment: {
    EnvironmentRecord: {
      Type: "Declarative",
      // Identifier bindings go here
      g: 20
    },
    outer: <GlobalLexicalEnvironment>,
    ThisBinding: <Global Object or undefined>
  }
}
```

当函数执行完成，其返回值存储在变量`c`中，全局执行上下文进行更新，之后全局代码执行完成，程序结束。

注意，你可能已经发现，通过`let`和`const`声明的变量在创建阶段没有指定任何值，而`var`声明的变量则指定为`undefined`。这里因为在创建阶段，会扫描代码查找变量及函数声明，函数的声明会完整的存储在环境记录中，`var`声明的变量会被初始化为`undefined`，而`let`或`const`声明的变量会保持未初始化的状态，这就是为什么你可以在声明变量之前访问var定义的变量(虽然是`undefined`)，但访问`let`和`const`变量会出现引用错误的原因——这也就是我们说的`var`变量以及函数声明的提升。