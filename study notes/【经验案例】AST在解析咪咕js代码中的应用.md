# AST在解析咪咕js-sdk中的应用

## 1. 背景

原始的诉求是基于业务场景下，我们需要去“破解”咪咕（移动运营商）提供的用于开通业务的js-sdk，说是破解，其实主要是做以下两件事：

1. 咪咕的js-sdk其实是经过了多层封装，我方想绕过外层的封装，通过一些优化手段直接调用底层的sdk，缩短整体的调用耗时，提升业务转化率
2. 咪咕的js-sdk中涉及一些“防作弊”的操作，通过收集用户侧的一些指纹信息来判断当前订购是否源自用户的真实操作，我方需要获取到它具体收集了哪些信息

无论出于哪个目的，大的前提都是需要能“读懂”源代码，咪咕的js-sdk代码经过了“特殊”的混淆处理并且会定期更新，直接阅读非常困难，同时它分为静态与动态两部分，前面提到的两个目的，都是需要去解读动态生成的这部分代码（未处理的代码差不多4万行）。如标题所示，本文的目的仅在于分享一种思路来解决当前的问题，所以我会以这部分静态的代码（代码量比较少）为例，来讲述如何运用AST来让这些混淆后的代码变得方便阅读

## 2. 初步分析js-sdk的代码结构

咪咕的js-sdk会不定期的更新，我这边保存了一个[版本](https://github.com/asakurayoh1987/ast-migu/blob/main/js-sdk/migus.js)，代码已格式化，方便大家下载并与文中的截图进行对比

整个代码可以分为两个部分：变量（以及函数的）声明和一个IIFE，IIFE中通过一个工厂函数来将属性和方法挂载到全局对象`migusdk`上：

![image-20230707095752824](https://oss.kuyinyun.com/11W2MYCO/rescloud1/937c16a515ba4ef0bf695317df4ed086.png)

仔细查看工厂函数里的代码，虽然经过了特殊的混淆处理，但依然可以看到我们熟悉的代码片段，比如下面这段：

![image-20230707101345513](https://oss.kuyinyun.com/11W2MYCO/rescloud1/ec5543e9c76546b1a9443c83054cf96b.png)

相信大家都已看出这是一段发送ajax请求的代码，由此也可以揭开咪咕代码混淆方式的第一层面纱：**字面量替换**

比如上面的代码`xhr.setRequestHeader(...)`，我们直接在浏览器中调试，对相关的表达式进行求值：

![image-20230707102424025](https://oss.kuyinyun.com/11W2MYCO/rescloud1/1fa2bf84a287470bbc7cc854799bcaf6.png)

在进行变量及表达式值替换后，实际的代码如下：

```javascript
xhr.setRequestHeader('Content-Type', 'application/x-www-form-urlencoded');
```

接下来我们来看另一段代码：

![image-20230707103104833](https://oss.kuyinyun.com/11W2MYCO/rescloud1/8eedab2eab574ec7942e40a1e6ee8142.png)

我们先不去看每条`case`语句中的的具体逻辑，只着眼于整个`for`循环+`switch...case`语句的结构，这段代码所做的事如下：

1. 声明一个数组：['6', '2', '3', '0', '1', '4', '5']（通过'6|2|3|0|1|4|5'.split('|')得到）
2. 按数组中元素的顺序来执行`switch...case`语句

整个工厂函数的代码中充斥着这样的结构，甚至存在嵌套，只不过其它地方的`for`循环语句的条件声明部分以及每条`case`语句的值都使用了前面提到的字面量替换的方式进行了混淆处理

由此揭开咪咕代码混淆方式的第二层面纱：**使用for循环+switch...case的结构将原代码拆成一个个片段，在保持逻辑执行顺序不变的前提下调整其物理位置，从而增加阅读的难度**

在知晓了代码混淆的方式之后，其实就可以着手去解析代码了，只不过人工的解析方式不仅低效，而且在源代码变动之后，又要重新进行解析工作，理想的方式自然是自动化，但自动化的前提就是需要机器能”读懂“代码，方能”重构“代码，这就引出来了今天的主角——AST（抽象语法树）

## 3. AST与Babel

### 3.1 AST（抽象语法树）

所谓AST，简单来说就是使用预定义好的数据结构来表示代码中的各个部分，从而可以让程序”读懂“代码和”修改“代码，这就像dom与virtual dom的关系，通过将dom抽象成virtual dom这样的数据结构，让程序可以解析dom结构。想直观的查看某段javascript代码所对应的AST是什么样的，可以使用[AST Explorer](https://astexplorer.net/)这个工具，比如`const fruit = 'apple';`所对应的AST如下图：

![image-20230711214253429](https://oss.kuyinyun.com/11W2MYCO/rescloud1/5a5dd88ab65c4434bd7f4c1c80ded47d.png)

并且在左侧将光标定位到代码的不同位置时，右侧对应的区块就会高亮（反之亦然），这样方便查看代码各部分所对应的数据结构

仔细观察右侧部分，可以发现这个数据结构其实是如下这样的一个树形结构（抽象语法树的由来）：

![ast-demo](https://oss.kuyinyun.com/11W2MYCO/rescloud1/39d74ad504a74ec4b5ba821459d92e0c.svg)

通过遍历该树就能让程序“读懂”代码，但我们的目的当然不会止步于此，既然能解析这棵语法树，自然可以去修改它，比如，假设我们将`StringLiteral`节点替换为另一个`value`为`'orange'`的`StringLiteral`节点，再将这个语法树还原成代码，则可得到下面的代码

```javascript
const fruit = 'orange';
```

### 3.2 Babel

相信做前端开发的同学对Babel都不陌生（尽管现在的脚手架都对其进行了很好的封装，可能你都感知不到它的存在），我们日常写的ES6及之后的代码通过Babel转译为ES5的代码，从而可以在一些老旧的设备上运行，而上述的三个过程：解析代码得到AST、（遍历AST）修改AST的节点、将修改后的AST转换为代码，正好对应了babel转译代码的三个步骤：parse、transform、generate，babel也提供相应的库来方便我们执行这三个步骤

#### 3.2.1 parse

关于parse就是将代码转换为AST，方便后续的transform阶段，至于AST前文已有介绍，这里就不再赘述

#### 3.2.2 transform

在拿到parse阶段输出的AST后，就可以遍历AST，找到目标节点，然后修改它。那么如何遍历？babel使用了**访问者模式**，该模式将操作于特定类型数据结构的逻辑与数据结构本身解藕，babel提供的用于遍历AST的API如下：

```javascript
traverse(ast, visitor)
```

这里的`visitor`对象的`key`对应节点的类型名称，`value`则对应为操作该节点的函数，`value`也可以是一个对象，以`enter`与`exit`为`key`，各自对应的`value`则是操作节点的函数，分别表示在遍历过程中**进入节点**和**退出节点**时需要执行的操作（后面的例子中会说明如何使用），两种格式如下：

```javascript
traverse(ast, {
  VariableDeclarator: {
    enter: path => {
      // ...
    },
    exit: path => {
      // ...
    },
  }
})
```

或

```javascript
traverse(ast, {
  // 这种模式就相当于是enter
	VariableDeclarator: path => {
    // ...
  },
})
```

关于API的更多说明可以参见[Babel官方文档](https://github.com/jamiebuilds/babel-handbook/blob/master/translations/en/plugin-handbook.md)

### 3.2.3 generate

在该阶段，我们将修改后的AST再次还原成目标代码（字符串）

```javascript
// code为修改后生成的代码
const { code } = generator(ast);
```

## 4. 实战

### 4.1 环境声明

[项目仓库地址](https://github.com/asakurayoh1987/ast-migu)，通过`package.json`可以查看项目对node.js及pnpm的要求（node.js 16.18.1, pnpm 8.6.6），这里仍使用了commonjs的模块，因为代码里会使用到`eval`函数，所以**不能开启严格模式**

### 4.2 代码框架

```javascript
const parse = require('@babel/parser').parse;
const traverse = require('@babel/traverse').default;
const generator = require('@babel/generator').default;
const { resolve } = require('node:path');
const { readFileSync, outputFileSync } = require('fs-extra');

// 读取咪咕js-sdk的源码
const source = readFileSync(resolve(__dirname, './js-sdk/migus.js'), {
  encoding: 'utf8',
});

// 转换为AST
const ast = parse(source);

// 遍历AST，针对特点的节点进行处理
traverse(ast, { 
	// 针对各类型节点的处理逻辑
});

// 将修改后的AST生成代码，并输出产物到文件
const { code } = generator(ast);

outputFileSync(resolve(__dirname, './dist/migu.js'), code, {
  encoding: 'utf8',
});

```

我们的转译逻辑将作为模块被引入，并作为`traverse`第二个参数传入（参见后面的实例）

### 4.3 小试牛刀——字面量替换

按前文描述，咪咕js-sdk中代码存在大量的变量声明，并且基本是作为常量用于后续代码中进行字面量拼接，我们的思路为：查找所有变量声明的节点，如果它是一个常量，就查找所有被引用的地方，并将其替换为字面量，为此我们需要在添加如下代码：

```javascript
const { VariableDeclarator } = require('./traverse/index.js');

// 中间代码省略

// 遍历变量声明的节点
traverse(ast, { VariableDeclarator });
```

关于`VariableDeclarator`的逻辑如下：

```javascript
/**
 * @type {import("@babel/traverse").TraverseOptions['VariableDeclarator']}
 */
const VariableDeclarator = {
  enter: path => {
    debug(`VariableDeclarator path: ${path.toString()}`);
    // 获取变量名和其初始化值
    const {
      id: { name },
      init,
    } = path.node;

    // 根据变量名查找binding
    const binding = path.scope.getBinding(name);

    // 如果初始化的值是字面量，并且未被修改过
    if (types.isLiteral(init) && binding.constant) {
      // 进行字面量的替换，遍历所有被引用的地方，将节点替换为一个字面量节点
      // 这里的types.valueToNode是babel提供的工具函数，用来根据具体的值创建对应的节点
      for (let referPath of binding.referencePaths) {
        referPath.replaceWith(types.valueToNode(init.value));
      }

      // 全部替换之后则可以将变量声明给移除了，可选
      path.remove();
    }

    // 全局函数声明
    if (types.isFunctionExpression(init) && path.scope.uid === 0) {
      try {
        // 拿到函数的字符串形式，复用eval来进行全局声明，后续的操作中会用到
        eval(path.toString());
        path.remove();
      } catch (err) {
        debug(err);
      }
    }
  },
};
```

咪咕js-sdk中的变量声明部分会有如下这样的全局函数的声明：

![image-20230713095214251](https://oss.kuyinyun.com/11W2MYCO/rescloud1/0ba393239dba472782c9b4e6a9cb5c76.png)

![image-20230713095227230](https://oss.kuyinyun.com/11W2MYCO/rescloud1/0841611f08ec4618af092a3c9ecc3416.png)

并且这些函数都是幂等的（在代码层面我这里偷了个懒，其实可以拿到函数体的字符串形式，然后写正则去匹配，看是否存在诸如`Date`、`Math.random`等这种导致函数非幂等的标识存在），为了后续的处理过程——将代码中函数的调用替换为其执行的结果，这里就需要先将这些全局函数其声明到当前node.js执行环境下（具体可以参见后续步骤）

执行`node main.mjs`则可以查看进行字面量替换之后的js-sdk是何模样

![image-20230713105234306](https://oss.kuyinyun.com/11W2MYCO/rescloud1/e6f9df546afa4a55aa963cb50bf6fe75.png)

代码中的变量也都直接替换为字面量了

![image-20230713105558381](https://oss.kuyinyun.com/11W2MYCO/rescloud1/c698915e3f5f406899b735adae0030df.png)

接下来就是将图中这些字符串的拼接及函数或方法的调用进一步处理

### 4.4 字符串字面量拼接

我们先利用[AST Explorer](https://astexplorer.net/)来查看这种字符串字面量拼接的AST结构，通过下图可知，我们需要对`BinaryExpression`（二元表达式）类型节点进行处理，第一印象会觉得逻辑很简单：如果表达的左值与右值都是字面量，则将整个表达式替换为执行结果，但实际上真的如此吗？

![image-20230228172527642](https://oss.kuyinyun.com/11W2MYCO/rescloud1/fd5f4c581752445eaffe04a806b28f4f.png)

可以发现对于`'a'+'b'+'c'`这种，它的树结构其实是这样的：

![binaryexpression](https://oss.kuyinyun.com/11W2MYCO/rescloud1/c369ec67d28749caa0517207d7b0639c.svg)

这里涉及树的遍历相关的知识 ，默认情况下是如图中数字`1`、`2`的顺序来处理`BinaryExpression`节点，即**先序遍历**，但这个处理节点的顺序无法满足我们的场景，理想的情况是按图中`3`、`4`的顺序，先处理左下方`BinaryExpression`节点，将其替换为运算后的字符串字面量，再处理顶端的`BinaryExpression`节点，即**后序遍历**，如何实现这样的遍历顺序？这就要用到前文提及的处理节点的时机，即`enter`与`exit`，这里需要将处理逻辑写在`exit`内：

```javascript
/**
 * @type {import("@babel/traverse").TraverseOptions['BinaryExpression']}
 */
const BinaryExpression = {
  exit: path => {
    debug(`BinaryExpression path: ${path.toString()}`);
    // 左值、右值、运算符
    const { left, right, operator } = path.node;
    // 如果左值与右值都是字面量，则进行处理（这里先不考虑全量变量的场景）
    if (types.isLiteral(left) && types.isLiteral(right)) {
      let result = null;
      const leftValue = left.value;
      const rightValue = right.value;

      switch (operator) {
        case '+':
          result = leftValue + rightValue;
          break;
        case '-':
          result = leftValue - rightValue;
          break;
        case '*':
          result = leftValue * rightValue;
          break;
        case '/':
          result = leftValue / rightValue;
          break;
        case '<<':
          result = leftValue << rightValue;
          break;
        case '==':
          result = leftValue == rightValue;
          break;
        case '===':
          result = leftValue === rightValue;
          break;
        case '!=':
          result = leftValue != rightValue;
          break;
        case '!==':
          result = leftValue !== rightValue;
          break;
        default:
          throw new Error(
            `unhandled operator(${operator}) in BinaryExpression(${path.toString()})!`
          );
      }

      // 使用字面量运算结果替换原节点
      path.replaceWith(types.valueToNode(result));
    }
  },
};
```

```javascript
traverse(ast, { VariableDeclarator, BinaryExpression });
```

运行代码，看看本次转译之后的结果

![image-20230713112957089](https://oss.kuyinyun.com/11W2MYCO/rescloud1/c8d3a2251fea40a3842691afa4b3e9f7.png)

### 4.5 函数及方法调用的替换

这个场景稍微复杂一些，我们用通过[AST Explorer](https://astexplorer.net/)来查看源代码`_l1$1L1lL.oQuG36(sdk.umark, cm$wMDxg("IFYo^", 22))`的AST结构，如下图：

![call_member_expression](https://oss.kuyinyun.com/11W2MYCO/rescloud1/3493ebe044104cc98040921402ffeea7.svg)

而这里的`_l1$1L1lL.oQuG36`可以在代码中找到声明的地方：

```javascript
var _l1$1L1lL = {
    oQuG36: function (x, d) {
      return x == d;
    },
    olGi20: function (K, r) {
      return K + r;
    },
    WbfN74: function (V, l) {
      return V != l;
    },
    kpFl$0: function (X, U) {
      return X - U;
    },
    GlFU85: function (V, e) {
      return V * e;
    },
    qGPe52: function (O, R) {
      return O / R;
    },
    fvKc99: function (H, g) {
      return H === g;
    },
    pMZL50: function (G, N) {
      return G < N;
    },
    NSFw10: function (M, P) {
      return M !== P;
    }
  };
```

也就是说这句应该转译为：`sdk.umark == cm$wMDxg("IFYo^", 22)`这样的表达式（源码中这句正是作为`if`语句的条件表达式的一部分）

还有诸如这种：`"1|2|0|3|5|4".split(cm$wMDxg("F", 41))`，为了后续方便调整`for`循环中代码段的物理位置，需要转译为`['1', '2', '0', '3', '5', '4']`（`cm$wMDxg("F", 41)`的执行结果是`'|'`）

综上所述，可以分场景进行处理（定制逻辑），同时可以看到`CallExpression`是存在嵌套的，所以这里我们依旧使用后序遍历的处理方式

```javascript
/**
 * @type {import("@babel/traverse").TraverseOptions['CallExpression']}
 */
const CallExpression = {
  exit: path => {
    debug(`CallExpression path: ${path.toString()}`);
    // 获取被调用方以及参数
    const { callee, arguments: args } = path.node;
    let result = null;

    // 如果callee是一个标识符，说明它是一个函数
    if (types.isIdentifier(callee)) {
      // 获取标识符的变量名
      const { name } = callee;
      // 如果在全局环境已经存在，则直接
      if (name in global) {
        result = evalExp(path.toString());
      } else {
        // 如果是代码中的标识符，比如新定义的函数
        const binding = path.scope.getBinding(name);
        if (binding) {
          // 声明函数
          evalExp(binding.path.toString());
          // 执行函数并用其结果值替换节点
          result = evalExp(path.toString());
        }
      }
    }

    // 如果callee是一个MemberExpression
    if (types.isMemberExpression(callee)) {
      // 获取对象及所访问的属性
      const { object, property } = callee;
      // 比如："1|2|0|3|5|4".split('|')
      if (types.isLiteral(object)) {
        result = evalExp(path.toString());
      }

      // 对象是一个标识符
      if (types.isIdentifier(object)) {
        const { name } = object;
        // 全局对象
        if (name in global) {
          result = evalExp(path.toString());
        } else {
          // 如果是代码中的标识符，比如：_l1$1L1lL.oQuG36
          const binding = path.scope.getBinding(name);
          if (binding) {
            // 这里目的是用来定制处理sdk中的工具函数，用来将执行结果替换一个二元表达式，并作为if语句中的条件部分
            if (types.isVariableDeclarator(binding.path)) {
              if (types.isObjectExpression(binding.path.node.init)) {
                binding.path.traverse({
                  ObjectProperty(propPath) {
                    if (propPath.node.key.name !== property.name) {
                      return;
                    }
                    propPath.traverse({
                      BinaryExpression(returnPath) {
                        if (types.isReturnStatement(returnPath.parentPath)) {
                          const { operator } = returnPath.node;
                          result = types.BinaryExpression(
                            operator,
                            args[0],
                            args[1]
                          );
                        }
                      },
                    });
                  },
                });
              }
            }
          }
        }
      }
    }

    if (result === null) {
      return;
    }

    path.replaceWith(types.isNode(result) ? result : types.valueToNode(result));
  },
};
```

执行代码，查看生成的文件，可以对比处理前后的同段代码如下：

![image-20230713112957089](https://oss.kuyinyun.com/11W2MYCO/rescloud1/64306bf2747a4cf2b939d48e45f690f1.png)

![image-20230713173300699](https://oss.kuyinyun.com/11W2MYCO/rescloud1/c34d7af8feb54b31b4183dacc206eff1.png)

### 4.6 调整for语句及swtich...case语句中代码段的物理顺序

经过前面步骤的处理，这里的处理逻辑就相对明了一些了，查看上一步生成的代码中最外层的`for`循环及`switch...case`语句（经过前面的步骤，`case`语句中的值已均为字面量了）：

![image-20230713173856845](https://oss.kuyinyun.com/11W2MYCO/rescloud1/8c122efb49b141f5bc57176d291de01a.png)

而我们要做的就是模拟`for`语句的执行，拿到每轮循环得到的值，然后在众多的`case`语句中找到匹配的那条，并将`case`语句中`continue`之外的代码取出来，依次塞入一个数组，最后将数组生成新的节点并替换原节点

这里的`for`循环及`switch...case`结构也存在嵌套，所以我们依旧使用后序遍历

```javascript
/**
 * @type {import("@babel/traverse").TraverseOptions['ForStatement']}
 */
const ForStatement = {
  exit: path => {
    debug(`ForStatement path: ${path.toString()}`);
    const { init, body } = path.node;
    if (types.isBlockStatement(body) && types.isSwitchStatement(body.body[0])) {
      try {
        // 这里偷了个懒，将节点转成代码后，通过eval来拿到变量
        eval(generator(init).code);
        let statementStr = [];
        const { discriminant, cases } = body.body[0];
        // 同上，通过eval拿到当前的case的值
        let caseNo = eval(generator(discriminant).code);
        while (caseNo) {
          const caseExpression = cases.find(
            c => types.isLiteral(c.test) && c.test.value === caseNo
          );
          if (!caseExpression) {
            continue;
          }
          caseExpression.consequent
            // FIXME: 这里偷懒了，应该是获取第一条continue之前的代码，只不过源码比较规范，可不处理
            .filter(item => !types.isContinueStatement(item))
            .forEach(item => {
              // 将得到的语句按序放入数组
              statementStr.push(item);
            });

          caseNo = eval(generator(discriminant).code);
        }

        path.replaceWithMultiple(statementStr);
      } catch {
        return;
      }
    }
  },
};
```

执行脚本后对比前后调整：

![image-20230714093304877](https://oss.kuyinyun.com/11W2MYCO/rescloud1/8be04e6c933742ba944e4ae0a284704c.png)

![image-20230714093346454](https://oss.kuyinyun.com/11W2MYCO/rescloud1/7dac95e56afc4e36bdc122833dd33533.png)

### 4.7 小优化

至此关于这个js-sdk的转译过程已完成，转译后的代码已经可以方便的阅读，这里有个小的优化点，即：

![image-20230714100130504](https://oss.kuyinyun.com/11W2MYCO/rescloud1/a55f78e60a2b45529016a64c1eab0fde.png)

在将生成后的代码写入文件之前可以进行如下操作：

```javascript
// 省略

let { code } = generator(ast);

// 通过正则将unicode代码匹配出来，然后进行处理
code = code.replaceAll(/(\\u[\d\w]+)+/g, m => {
  return eval(`'${m}'`)
});

// 省略
```

重新执行脚本，效果如下：

![image-20230714103818450](https://oss.kuyinyun.com/11W2MYCO/rescloud1/7fc69414e3cc4f4a82b0566d9723452c.png)

## 5. 总结

本文旨在结合实际案例分享AST及Babel相关的使用，提供一种解决问题的思路，相关的代码已上传github仓库，其中涉及很多场景定制的逻辑，以及解决特定问题时所使用的技巧（比如`eval`），不具备普适性，仅供参考
