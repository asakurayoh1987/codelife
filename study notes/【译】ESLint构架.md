# Architecture

[原文](https://eslint.org/docs/developer-guide/architecture)

![架构图](https://eslint.org/docs/developer-guide/architecture/dependency.svg)

俯览ESLint架构，有以下几个关键部分组成：

* `bin/eslint.js` - 这是在命令行工具执行时实际调用的文件。它只是一个简单的封装，除了启动ESLint外什么都不做，只是将命令行参数传递给`cli`，它被特意设计的很简单，因为不需要进行繁重的测试。
* `lib/api.js` - 这个是`require('eslint')`的入口。该文件公开了一个对象，该对象包含公共类Linter、CLIEngine、RuleTester和SourceCode。
* `lib/cli.js` - 这个是ESLint CLI的核心。它接受一个参数数组，然后使用eslint来执行命令。通过把它作为一个独立的单元，它允许调用者通过另一个node.js应用高效的调用ESLint，就像是在命令行中操作一样，主要调用的是`cli.execute()。它也是进行文件读取、目录遍历、输入和输出的部分。
* `lib/init/` - 这个模块包括了命令行`--init`的功能，用来为终端用户初始化一个ESLint的配置。
* `lib/cli-engine/` - 这个模块为`CLIEngine`类，它负责查找代码文件和配置文件，然后使用`Linter`类来进行代码校验。这包括了配置文件、解析器、插件以及格式化程序的加载逻辑。
* `lib/linter/` - 这个模块为核心的`Linter`类，它负责基于配置项来进行代码的校验，它不做文件的I/0操作也不与控制台交互。对于基于进行javascript文本校验的node.js应用，可以直接使用这个接口。
* `lib/rule-tester/` - 这个模块为`RuleTester`类，它是Mocha的包装器，用于对规则进行单元测试。这个类帮助我们为实现的每条规则编写格式一致的测试用例，从而确保每条规则能正常工作。`RuleTester`接口是模仿Mocha并使用它的全局测试方法。它也可以被修改来用于其它的测试框架。
* `lib/source-code/` - 这个模块为`SourceCode`类，用于表示已解析的源代码。它接受源代码和其对应的AST编程节点。
* `lib/rules/` - 这里包括了内建的一些规则。

## cli对象

cli对象为命令行接口提供了API，正如字面上的意思，`bin/eslint.js`只是简单的将参数传递给cli对象，然后将`process.exitCode`设置为要返回的`exit code`。

cli对象最主要的方法是`cli.execute()`，它接受代表命令行选项的字符串数组（就像`process.argv`除去了前两个参数一样），如果你希望在另一个应用中运行ESLint，并希望让它像CLI一样工作，那么就可以使用cli对象。

这个对象的职责包括以下：

* 解析命令行参数
* 从文件系统中读取
* 输出到控制台
* 输出到文件系统
* 使用格式化程序
* 返回正确的`exit code`

这个对象不能进行以下操作：

* 直接调用`process.exit()`
* 进行任何异步操作

## CLIEngine对象

`CLIEngine`类型代表了CLI的核心功能，但它默认情况下不从命令行读信息也不输出任何信息。相对的，它接受传递给CLI的大多数参数（不是全部），它会读取配置信息和源文件，同时会管理传递给`Linter`对象的环境。

`CLIEngine`的主要方法是`executeOnFiles()`，它接收一个由文件名和目录名组成的数组，并对这些文件和目录进行检查。

这个对象的职责包括以下：

* 管理`Linter`的执行环境
* 读取文件系统
* 从配置文件中读取配置信息（包括`.eslintrc`和`package.json`）

这个对象不能进行以下操作：

* 直接调用`process.exit()`
* 进行任何异步操作
* 输出到控制台
* 使用格式化程序

## Linter对象

`Linter`对象的主要方法是`verify()`，它接收两个参数：需要校验的原文本和配置信息对象（由配置文件中的配置信息加上命令行操作的选项）。这个方法首先使用`espree`（或者默认的别的解析器）对输入的文本进行解析来获得AST。生成的AST包括行/列和范围位置信息，分别用于报告问题的位置以及获取AST节点对应的源文本。

一旦AST生成好，`estraverse`就会有上到下遍历AST，对于每个节点，`Linter`对象会触发与节点类型相同名称的事件（比如"Identifier"、"WithStatement"等等）。在返回到AST子树时，会触发一个以AST类型名加上":exit"后缀的事件，比如"Identifier:exit" - 这就允许规则在向下和向上遍历AST时都可以进行操作。每个被触发的事件都会携带上对应可用的AST节点。

这个对象的职责包括以下：

* 检查javascript代码字符串
* 将代码转换成AST
* 在AST上执行规则
* 报告执行的结果

这个对象不能进行以下操作：

* 直接调用`process.exit()`
* 进行任何异步操作
* 使用node.js特有的功能
* 访问文件系统
* 调用`console.log()`或其它类似的方法

## Rules

独立的规则是ESLint架构中最独特的部分。规则能做的事非常少，它们只是一组针对所提供的AST执行的指令。尽管它们可以获得一些上下文信息，但一条规则的主要职责是检查AST并告警。

这个对象的职责包括以下：

* 在AST中检查是否存在特定的模式
* 当某种模式发现时告警

这个对象不能进行以下操作：

