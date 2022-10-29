# Configuring ESLint

[原文](https://eslint.org/docs/user-guide/configuring)

ESLint为设计为完全可配置的，这意味着你可以关闭每一条规则并且只进行基础的语法检测，或者混用打包好的规则和你自定义的规则来使ESLint完美的适配你的项目，配置ESLint的方式有两种：

1. 使用注释：可以在文件中直接使用javascript的注释来内嵌配置信息
2. 配置文件：可以使用javascript、json、YAML文件来为整个目录文件夹及子文件夹指定配置信息。配置可能通过单独的`.eslintrc.*`文件或在`package.json`中的eslintConfig字段来指定。无论使用哪种方式，ESLint都会去查找并自动读取配置（优先级？），你也可以在使用[命令行工具](https://eslint.org/docs/user-guide/command-line-interface)时指定配置文件。

配置中可以指定以下几个方面的信息：

* 环境标识（Environments） - 你的脚本会在哪种环境下运行。每一种环境都会默认指定一组预定义好的全局变量
* 全局变量（Globals） - 你代码执行期间用到的额外（Environments之外）的全局变量
* 规则（Rules） - 你需要使用哪些规则以及这些规则的错误级别

## 配置解析器选项（ParserOptions）

ESLint允许你指定你期望支持的javascript语言配置项。默认情况下，ESLint只支持ES5的语法。你可以修改配置来使它支持其它的ECMAScript版本，同样也可以使用指定的解析器来支持JSX

注意：支持JSX语法并不等价于支持React，因为React在JSX的语法之上又添加了特定的语法，而这是ESLint无法识别的。如果你使用React并且想使用这些语法，我们推荐使用[eslint-plugin-react](https://github.com/yannickcr/eslint-plugin-react)插件。同样，支持ES6语法并不等价于支持新的ES6全局变量（比如像`Set`这样的新类型）。为了支持ES6语法，使用`{"parserOptions": {"ecmaVersion":6}}`，而对于ES6新增的全局变量，使用`{"env": {"es6": true}}`，使用后者会默认支持ES6语法 ，但仅使用前者是不会默认支持ES6新增的全局变量。解析器的配置可以在`.eslintrc.*`中通过配置项`parserOptions`指定，它包括以下这些选项：

* `ecmaVersion` - 可以设置为3，5(默认)，6，7，8，9，10或11来指定你想要使用的ECMAScript语法的版本，你也可以将其设置为2015(同ES6)，2016(ES7)，2017(ES8)，2018(ES9)，2019(ES10)，2020(ES11)这样基于年份的命名方式。
* `sourceType` - 可以设置为`"script"(默认)`或`"module"`，当你的代码是基于ECMAScript模块写法时使用后者。
* `ecmaFeatures` - 对象的配置形式，用来指定你所使用的语法版本的附加配置项（配置值为布尔型，默认值为false）
	* `globalReturn` - 允许在全局作用域中使用`return`
	* `impliedStrict` - 开启全局[strict mode](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Strict_mode)(严格模式)
	* `jsx` - 支持JSX语法

下面是一个`.eslintrc.json`配置文件的例子：

```json
{
    "parserOptions": {
        "ecmaVersion": 6,
        "sourceType": "module",
        "ecmaFeatures": {
            "jsx": true
        }
    },
    "rules": {
        "semi": "error"
    }
}
```
设置解析器选项可以帮助ESLint确认是什么解析错误。

## 配置解析器（Parser）

ESLint默认使用[Espree](https://github.com/eslint/espree)作为解析器，你可以在配置文件中指定不同的解析器，只要它满足以下要求：

1. 它必须是可以从配置文件中加载的node.js模块，通常这意味着，你应该使用npm来独立安装解析器模块包
2. 它必须遵循[解析器的接口定义](https://eslint.org/docs/developer-guide/working-with-custom-parsers)

注意：即使满足上面的要求，也不能保证一个外部的解析器能与ESLint正常工作，ESLint也无法修复与其它解析器不兼容相关的bug。

可以通过`.eslintrc`文件中的`parser`配置项来指定需要作为解析器的npm模块。下面是一个使用Esprima代替默认的Espree作为解析器的配置例子：

```json
{
    "parser": "esprima",
    "rules": {
        "semi": "error"
    }
}
```
下面的这些解析器能很好的与ESLint兼容：

* [Esprima](https://www.npmjs.com/package/esprima)
* [Babel-ESLint](https://www.npmjs.com/package/babel-eslint) - 封装了[babel](https://babeljs.io/)解析器来使它可以兼容ESLint
* [@typescript-eslint/parser](https://www.npmjs.com/package/@typescript-eslint/parser) - 用于将typescript转换成ESTree兼容的形式来使ESLint支持typescript的解析器

注意：当使用自定义的解析器时，为了让ESLint能支持默认的ES5之外的特性，依旧需要指定`parserOptions`配置项。`parserOptions`被传递给所有的解析器，解析器来决定是否使用它们来开启对某些特性的支持。

## 配置处理器（Processor）

ESLint插件可能提供了处理器，它可以用于从非javascript文件中提取出javascript的代码，然后使用ESLint来检查这些代码，也可以用于对javascript代码进行转换的预处理。

要在配置文件中指定处理器，可以使用`processor`选项，其值是格式为`{去除前缀的插件名称}/{处理理名称}`的字符串，例如，下面配置项中指定了名为`a-plugin`的插件提供的名为`a-processor`的处理器：

```json
{
    "plugins": ["a-plugin"],
    "processor": "a-plugin/a-processor"
}
```

如果需要为特定类型的文件指定处理器，需要将`processor`配置项与`overrides`配置项组合使用，比如下面的例子就是为*.md文件指定使用`markdown`处理器：

```json
{
    "plugins": ["a-plugin"],
    "overrides": [
        {
            "files": ["*.md"],
            "processor": "a-plugin/markdown"
        }
    ]
}
```

处理器可以生成命名的代码块，比如0.js、1.js，ESLint将这些作为原文件下的子文件处理，你可以在`overrides`节中对这些代码块指定额外的配置。比如下例中对以markdown文件中以.js结尾的命名代码块禁用严格模式：

```json
{
    "plugins": ["a-plugin"],
    "overrides": [
        {
            "files": ["*.md"],
            "processor": "a-plugin/markdown"
        },
        {
            "files": ["**/*.md/*.js"],
            "rules": {
                "strict": "off"
            }h
        }
    ]
}
```

ESLint会检查这些命名代码块的后缀名，如果在[命令行选项](https://eslint.org/docs/user-guide/command-line-interface#--ext)`--ext`中未指定对应的文件后缀名，则它会被ESLint忽略，所以如果你检查这些命名代码块，请确保在`--ext`选项指定了对应的后缀名。

## 配置环境（Environments）

每个环境中都定义了一些预定义好的全局变量，可指指定的环境如下：

* `browser` - 浏览器全局变量
* `node` - node.js全局变量和作用域
* `commonjs` - CommonJS全局变量和作用域（使用Browsify或Webpack构建的仅在浏览器中运行的代码使用这个）
* `shared-node-browser` - node.js和浏览器通用的全局变量
* `es6` - 除模块外的所有ES6特性（并且会自动将`ecmaVersion`设置为6）
* `es2017` - 添加所有ES2017的全局变量，包括自动将`ecmaVersion`设置为8
* `es2020` - 添加所有ES2020的全局变量，包括自动将`ecmaVersion`设置为11
* `worker` - web worker中的全局变量
* `amd` - 根据[amd规范](https://github.com/amdjs/amdjs-api/wiki/AMD)将`require()`和`define()`定义为全局变量
* `mocha` - 添加Mocha测试用的全局变量
* `jasmine` - 添加版本1.3和2.0的Jasmine测试用的全局变量
* `jest` - Jest的全局变量
* `phantomjs` - PhantomJS的全局变量
* `protractor` - Protractor的全局变量
* `qunit` - QUnit的全局变量
* `jquery` - jQuery的全局变量
* `prototypejs` - Prototype.js的全局变量
* `shelljs` - ShellJS的全局变量
* `meteor` - Meteor的全局变量
* `mongo` - MongoDB的全局变量
* `applescript` - AppleScript的全局变量
* `nashorn` - Java 8 Nashorn的全局变量
* `serviceworker` - Service Worker的全局变量
* `atomtest` - Atom测试助手的全局变量
* `embertest` - Ember测试助手的全局变量
* `webextensions` - WebExtensions的全局变量
* `greasemonkey` - GreaseMonkey的全局变量

这些环境并不是互斥的，所以你可以同时指定多个。环境配置可以在文件内部、配置文件中以及命令行参数`--env`来指定。
在javascript文件中可以使用注释的方式来指定，如下（指定环境为node.js和mocha）：
```javascript
/* eslint-env node, mocha */
```

在配置文件中可以使用`env`配置项来指定环境，想要开启某个环境，只需要将其对应的值配置为`true`，如下（指定环境为浏览器和node.js）：

```json
{
    "env": {
        "browser": true,
        "node": true
    }
}
```

或者在package.json中指定

```json
{
    "name": "mypackage",
    "version": "0.0.1",
    "eslintConfig": {
        "env": {
            "browser": true,
            "node": true
        }
    }
}
```
以及在 YAML格式的配置中

```YAML
---
  env:
    browser: true
    node: true
```

如果想使用插件中提供的环境，首先在`plugins`中指定要使用的插件，然后`env`中配置值的格式为`{去除前缀的插件名称}/{环境名称}`，比如：

```json
{
    "plugins": ["example"],
    "env": {
        "example/custom": true
    }
}
```

或者在package.json中

```json
{
    "name": "mypackage",
    "version": "0.0.1",
    "eslintConfig": {
        "plugins": ["example"],
        "env": {
            "example/custom": true
        }
    }
}
```

以及在YAML中

```YAML
---
  plugins:
    - example
  env:
    example/custom: true
```

## 配置全局变量（Globals）

当在访问在同一个文件中没有定义的变量时，`no-undef`规则会发出警告。如果需要在一个文件中使用全局变量，那就有必要定义这些全局变量来让ESLint不发出警告。你可以在文件内部或在配置文件中定义全局变量。

在javascript文件使用注释的方式定义全局变量，如下：

```javascript
/* global var1, var2 */
```

这里定义了两个全局变量，var1和var2，如果你期望选择性的指定这些全局变量可行，而不是只读，则可以对每个变量使用`writable`标致：

```javascript
/* global var1:writable, var2:writable */
```

在配置文件中配置全局变量时，可以使用`globals`配置项，其值是一个对象，对象的key则为你要使用的全局变量，对于这些变量，如果可写则对应的value配置为`writable`，如果只读则value配置为`readonly`，如下：

```javascript
{
    "globals": {
        "var1": "writable",
        "var2": "readonly"
    }
}
```

以及在YAML中

```YAML
---
  globals:
    var1: writable
    var2: readonly
```

通过设置vaue为`off`也可以禁用全局变量，比如，在一个支持Promise之外的ES2015全局变量的环境中，可以这样配置：

```javascript
{
    "env": {
        "es6": true
    },
    "globals": {
        "Promise": "off"
    }
}
```

由于历史原因，布尔值`false`以及字符串`readable`等价于字符串`readonly`，同样，布尔值`true`以及字符串`writeable`等价于字符串`writable`，但不推荐使用旧的值。

注意：开启`no-global-assign`规则会禁止在代码中修改只读的全局变量

## 配置插件（Plugins）

ESLint支持使用第三方插件，在使用插件之前，你需要通过npm安装它

可以通过`plugins`配置项在配置文件中指定插件，其值是一个插件名称的列表，配置值时可以省去`eslint-plugin-`这个前缀。

```javascript
{
    "plugins": [
        "plugin1",
        "eslint-plugin-plugin2"
    ]
}
```

在YAML中

```YAML
---
  plugins:
    - plugin1
    - eslint-plugin-plugin2
```

注意：插件是相对于ESLint进程当前工作目录解析的，换言之，ESLint加载一个插件就如同在用户在项目的根目录下通过node的REPL执行`require('eslint-plugin-pluginname')`的一样。

### 命名规范

#### 引入插件

对于非scoped的包，`eslint-plugin-`前缀可以省略

```javascript
{
    // ...
    "plugins": [
        "jquery", // means eslint-plugin-jquery
    ]
    // ...
}
```

对于scoped包，此规则也同样适用

```javascript
{
    // ...
    "plugins": [
        "@jquery/jquery", // means @jquery/eslint-plugin-jquery
        "@foobar" // means @foobar/eslint-plugin
    ]
    // ...
}
```

#### 使用插件

当使用插件中提供的规则、环境以及配置时，它们必须按如下约定进行引用：

* eslint-plugin-foo --> foo/a-rule
* @foo/eslint-plugin --> @foo/a-config
* @foo/eslint-plugin-bar --> @foo/bar/a-environment

比如：

```javascript
{
    // ...
    "plugins": [
        "jquery",   // eslint-plugin-jquery
        "@foo/foo", // @foo/eslint-plugin-foo
        "@bar"      // @bar/eslint-plugin
    ],
    "extends": [
        "plugin:@foo/foo/recommended",
        "plugin:@bar/recommended"
    ],
    "rules": {
        "jquery/a-rule": "error",
        "@foo/foo/some-rule": "error",
        "@bar/another-rule": "error"
    },
    "env": {
        "jquery/jquery": true,
        "@foo/foo/env-foo": true,
        "@bar/env-bar": true,
    }
    // ...
}
```

## 配置规则（Rules）

ESLint附带了大量的规则，在你的项目中可以通过注释或配置文件来修改你用到的规则。要修改规则，你需要将对应ID的规则设置为如下几个值：

* "off" 或 0 - 关闭规则
* "warn" 或 1 - 开启规则为警告级别（不影响exit code）
* "error" 或 2 - 开启规则为错误级别（当触发时exit code为1）

### 使用注释配置

可以通过注释的方式在文件内部配置规则，格式如下：

```javascript
/* eslint eqeqeq: "off", curly: "error" */
```

上面这个例子中，规则`eqeqeq`被关闭了，同时`curly`规则被设置为错误级别，你也可以通过数字形式来指定相同的规则级别

```javascript
/* eslint eqeqeq: 0, curly: 2 */
```
如果一个规则有附加的选项，你可以通过数组字面量的语法来指定，比如：

```javascript
/* eslint quotes: ["error", "double"], curly: 2 */
```

这里给`quotes`规则指定了"double"选项，记住，数组中的第一项永远是规则的级别（数字或字符串）

### 使用配置文件

在配置文件中配置规则，通过`rules`配置项，同时指定规则的级别以及你想要使用的选项，如下：

```json
{
    "rules": {
        "eqeqeq": "off",
        "curly": "error",
        "quotes": ["error", "double"]
    }
}
```

在YAML中

```YAML
---
rules:
  eqeqeq: off
  curly: error
  quotes:
    - error
    - double
```

如果是使用插件中定义的规则，你需要在规则名前添加插件名作为前缀，并以"/"隔开，如下：

```json
{
    "plugins": [
        "plugin1"
    ],
    "rules": {
        "eqeqeq": "off",
        "curly": "error",
        "quotes": ["error", "double"],
        "plugin1/rule1": "error"
    }
}
```

在YAML中

```YAML
---
plugins:
  - plugin1
rules:
  eqeqeq: 0
  curly: error
  quotes:
    - error
    - "double"
  plugin1/rule1: error
```

在上述配置文件中，规则`plugin1/rule1`来自于名为`plugin1`的插件，你也可以在注释中使用这种格式

```javascript
/* eslint "plugin1/rule1": "error" */
```

注意：当指定插件提供的规则时，确保省略了`eslint-plugin-`这个前缀，因为在ESLint同部，它仅使用不带前缀的名称来定位规则。

## 使用行注释来禁用规则

为了临时禁用文件中的规则警告，你可以使用如下块注释的方式

```javascript
/* eslint-disable */

alert('foo');

/* eslint-enable */
```

你也可以禁用或开启指定的规则

```javascript
/* eslint-disable no-alert, no-console */

alert('foo');
console.log('bar');

/* eslint-enable no-alert, no-console */
```

如果需要针对整个文件关闭规则警告，则可以在文件的头部使用`/* eslint-disable */`块注释

```javascript
/* eslint-disable */

alert('foo');
```

也可以针对整个文件禁用指定的规则

```javascript
/* eslint-disable no-alert */

alert('foo');
```

要针对某一行禁用所有规则，可以按如下格式之一使用行注释或块注释

```javascript
alert('foo'); // eslint-disable-line

// eslint-disable-next-line
alert('foo');

/* eslint-disable-next-line */
alert('foo');

alert('foo'); /* eslint-disable-line */
```

针对某一行禁用指定规则

```javascript
alert('foo'); // eslint-disable-line no-alert

// eslint-disable-next-line no-alert
alert('foo');

alert('foo'); /* eslint-disable-line no-alert */

/* eslint-disable-next-line no-alert */
alert('foo');
```

针对某一行禁用多条规则

```javascript
alert('foo'); // eslint-disable-line no-alert, quotes, semi

// eslint-disable-next-line no-alert, quotes, semi
alert('foo');

alert('foo'); /* eslint-disable-line no-alert, quotes, semi */

/* eslint-disable-next-line no-alert, quotes, semi */
alert('foo');

```

上面所提到的方法同样适用于插件提供的规则，比如禁用插件`eslint-plugin-example`提供的规则`rule-name`，可以这样写

```javascript
foo(); // eslint-disable-line example/rule-name
foo(); /* eslint-disable-line example/rule-name */
```

注意：这种只针对文件中某部分禁用规则的注释只是告诉ESLint不要对这部分内容进行告警，但ESLint依旧会解析整个文件，所以这部分代码依仍要保证是有效的javascript语法。

### 针对一组文件禁用规则

在配置文件中使用`overrides`和`files`配置项可以针对一组文件禁用规则，如下：

```javascript
{
  "rules": {...},
  "overrides": [
    {
      "files": ["*-test.js","*.spec.js"],
      "rules": {
        "no-unused-expressions": "off"
      }
    }
  ]
}
```

## 配置行内注释的行为

### 禁用行内注释

通过设置`noInlineConfig`为`true`可以禁用行内注释，如下：

```json
{
  "rules": {...},
  "noInlineConfig": true
}
```

这与命令行选项的`--no-inline-config`是同等作用

### 报告未使用的eslint-disable注释

通过设置`reportUnusedDisableDirectives`为`true`可以报告未使用的`esint-disable`注释，如下：

```json
{
  "rules": {...},
  "reportUnusedDisableDirectives": true
}
```

这与命令行选项的`--report-unused-disable-directives`是同等作用（报告是以warn级别的）

## 添加共享设置

ESLint支持在配置文件中添加共享配置，你可以在配置文件中添加`settings`对象，它将会被提供给将要执行的每个规则。如果你正在添加自定义规则，并且希望它们可以访问相同的信息且易于配置，这可能会很有用。

JSON格式：

```json
{
    "settings": {
        "sharedData": "Hello"
    }
}
```

YAML格式：

```
---
  settings:
    sharedData: "Hello"
```

## 使用配置文件

有两种使用配置文件的方式。
首先可以通过`.eslintrc.*`和`package.json`来使用配置文件。ESLint会自动在要检查的文件所在有目录查找配置文件，并顺着父级目录向上查找，一直到文件系统的根目录（除非指定了`root: true`）。当你希望可以为项目的不同部分提供不同的配置文件时，或者当你期望其他人可以直接使用ESLint而不需要记住传递配置文件时，这种方式非常有用。

第二种方式是你可将配置保存你喜欢的任务地方，并在使用时通过命令行参数`-c`来指定，如下：

```bash
eslint -c myconfig.json myfiletotest.js
```

如果你正在使用一个配置文件，并期望ESLint忽略其它的`.eslintrc.*`文件，请确传递`-c`标识时同时指定`--no-eslintrc`标识。

无论以哪种方式，配置文件中的设置都将覆盖默认配置项。

## 配置文件的格式

ESLint支持以下格式的配置文件：

* Javascript - 使用`.eslintrc.js`并导出包括配置项的对象
* Javascript(ESM) - `.eslintrc.cjs`，在package.json中指定了`"type":"module"`的javascript包中使用，注意目前ESLint尚不支持ESM格式的配置
* YAML - 使用`.eslintrc.yaml`或`.eslintrc.yml`来定义配置项结构
* JSON - 使用`.eslintrc.json`来定义配置项结构，ESLint的JSON文件支持javascript风格的注释
* Deprecated - 使用`.eslinrc`，其中即可以使用JSON格式也支持YAML，但现已不推荐使用
* package.json - 通过package.json中的`eslintConfig`字段来定义配置项

如果在同一个文件中有多个配置文件，ESLint将会按如下的优先级取第一个使用：

1. .eslintrc.js
2. .eslintrc.cjs
3. .eslintrc.yaml
4. .eslintrc.yml
5. .eslintrc.json
6. .eslintrc
7. package.json

## 配置的级联与继承

当使用`.eslintrc.*`和`package.json`作为配置文件时，你可以充分复用配置的级联，举个例子，假设你的目录结构如下：

```bash
your-project
├── .eslintrc
├── lib
│ └── source.js
└─┬ tests
  ├── .eslintrc
  └── test.js
```

配置级联的工作方式是离待检查文件最近的`.eslintrc`文件优先级最高，其次是其父级目录下配置文件，等等。当你在这个项目中运行ESLint时，lib目录下的所有文件使用项目根目录下的`.eslintrc`文件作为配置项，当检查执行到tests目录下时，则除了`your-project/.eslintrc`外还将使用`your-project/tests/.eslintrc`，因此，`your-project/tests/test.js`会根据其目录层级中的这两个`.eslintrc`配置项的组合进行检查，其中更接近的`.eslintrc`的优先级更高。通过这种方式，你可以指定项目级别的ESLint配置，也可以使用目录级的配置进行覆盖。

同样，如果根目录下的package.json中有eslintConfig字段，其中的配置项会对所有子目录生效，但`tests`目录下`.eslintrc`中的配置项具有更高的优先级。

```bash
your-project
├── package.json
├── lib
│ └── source.js
└─┬ tests
  ├── .eslintrc
  └── test.js
```

如果在同一目录下同时有`.eslintrc`和`package.json`，则`.eslintrc`的优先级更高，并且`package.json`中的配置将不会使用。

默认情况下，ESLint会在所有父目录中查找配置文件直到系统根目录，当你希望你所有的项目都有一个统一的规范时，这可能会很有用，但有时也会导致意料之外的结果。为了将ESLint限制在指定的项目目录中，可以在项目根目录中`package.json`的`eslintConfig`字段或`.eslintrc.*`中设置`"root":true`。ESLint一旦在配置中发现这个配置项则会终止向父级目录查找配置。

```json
{
    "root": true
}
```

在YAML中

```YAML
---
  root: true
```

举个例子，假设有一个项目projectA，其`lib`目录下的`.eslintrc`中设置了`"root":true`，此时，在检查`main.js`时，`lib`目录下的配置项会被使用，而projectA目录下的`.eslintrc`不会被用到。

```bash
home
└── user
    └── projectA
        ├── .eslintrc  <- Not used
        └── lib
            ├── .eslintrc  <- { "root": true }
            └── main.js
```

完整的配置项继承（从最高优先级到最低优先级）顺序如下：

1. 行内配置
	1. /*eslint-disable*/ and /*eslint-enable*/
	2. /*global*/
	3. /*eslint*/
	4. /*eslint-env*/

2. 命令行选项
	1. --global
	2. --rule
	3. --env
	4. -c, --config

3. 项目级配置文件
	1. 与被检查文件同目录下的`.eslintrc.*`或`package.json`
	2. 继续在父辈目录下查找`.eslintrc`和`package.json`文件（直接父级目录有最高优先级，再上级父目录优先级次之），一直向上直到系统根目录或直到查找到设置了`"root":true`的配置

## 扩展配置文件(Extends)

配置文件可以在基础配置启用的规则集上进行扩展。

`extends`属性的值可以为以下任意：

* 一个用于指定配置字符串（可以配置文件的路径，共享配置项的名称，`eslint:recommended`或`eslint:all`）
* 一个字符串数组：每一个附加的配置都扩展了前一个配置

ESLint递归扩展配置，所以一个基础配置也可以有一个`extends`属性。`extends`属性中的相对路径和共享配置名称都是根据它们在配置文件中出现的位置进行解析的。

`rules`属性可以进行下述操作来扩展或覆盖规则集：

* 添加额外的规则
* 修改所继承规则的警告级别而不修改它的选项
	* 基础配置：`"eqeqeq":["error","allow-null"]`
	* 派生配置：`"eqeqeq": "warn"`
	* 最终生效的配置：`"eqeqeq":["warn","allow-null"]`
* 覆盖基础配置中的选项
	* 基础配置：`"quotes":["error","singer","avoid-escape"]`
	* 派生配置：`"quotes":["error","single"]`
	* 最终生效的配置：`"quotes":["error","single"]`

### 使用"eslint:recommended"

`eslint:recommended`这个扩展值会启用核心规则的一个子集用来报告通用的错误，这些规则可以在[规则列表页面](https://eslint.org/docs/rules/)中查看，其中带“勾选”标记的规则就是推荐规则集，推荐规则集只会根据ESLint的大版本号而改变。如果你的配置项是基于推荐规则集扩展的，当你升级了ESLint的主版本号后，在使用命令行选项`--fix`前，先浏览一下问题报告，这样你才知道新修订的推荐规则集是否会修改你的代码

`eslint --init`命令可以创建配置以便你扩展推荐规则集，下面是一个javascript格式的配置文件：

```javascript
module.exports = {
    "extends": "eslint:recommended",
    "rules": {
        // enable additional rules
        "indent": ["error", 4],
        "linebreak-style": ["error", "unix"],
        "quotes": ["error", "double"],
        "semi": ["error", "always"],

        // override default options for rules from base configurations
        "comma-dangle": ["error", "always"],
        "no-cond-assign": ["error", "always"],

        // disable rules from base configurations
        "no-console": "off",
    }
}
```

### 使用共享配置集

所谓共享配置集是一个导出配置对象的npm包，使用前要确保包被安装到ESLint可以引入它的目录下。

`extends`属性在指定值时可以省去包名中的`eslint-config-`前缀。`eslint --init`可以创建配置以便你可以扩展一个热度高的style guide（比如`eslint-config-standard`），对应YAML配置的例子如下：

```YAML
extends: standard
rules:
  comma-dangle:
    - error
    - always
  no-empty: warn
```

### 使用插件中提供的配置

[ESLint插件](https://eslint.org/docs/developer-guide/working-with-plugins)就是一个npm包，通常会导出一些规则，有些插件也会提供一个或多个命名的[配置集](https://eslint.org/docs/developer-guide/working-with-plugins#configs-in-plugins)，配置前先确保安装的包能被ESLint引用到。

`plugins`属性在指定值时可以省略`eslint-plugin-`前缀，然后`extends`属性的值可以包括以下几个部分：

* plugin:
* 包名（可以省略前缀）
* /
* 配置集的名称

下面是一个JSON格式的例子

```json
{
    "plugins": [
        "react"
    ],
    "extends": [
        "eslint:recommended",
        "plugin:react/recommended"
    ],
    "rules": {
       "no-set-state": "off"
    }
}
```

### 使用配置项文件

`extends`的属性值可是一个基本配置文件的绝对路径或相对路径，对于相对路径，ESLint是基于它所使用的配置文件的位置来解析，下面是一个JSON格式的配置文件例子：

```json
{
    "extends": [
        "./node_modules/coding-standard/eslintDefaults.js",
        "./node_modules/coding-standard/.eslintrc-es6",
        "./node_modules/coding-standard/.eslintrc-jsx"
    ],
    "rules": {
        "eqeqeq": "warn"
    }
}
```

### 使用"eslint:all"

`extends`属性值可以设置为`eslint:all`来开启当前ESLint版本下所有核心规则，ESLint的大小版本号改变时，核心规则集都可能发生改变。

** 重要：这个配置集不推荐在生产环境使用，因为ESLint的大小版本号改变时它都会发生变化，需要自行评估影响。**

如果你设置了ESLint在升级之后自动启用新的规则，就有可能在代码未改动的前提下ESLint会报告出新的问题，因为ESLint任意版本号的变动都会影响这个规则集的变化。

在决定项目的配置时，你可以启用所有核心规则，作为探索规则和选项的快捷方式，特别是在很少覆盖选项或禁用规则的情况下。规则的默认选项不是代表ESLint的认可（比如`quotes`规则的默认选项并不意味着双引号比单引号好）

如果你的配置项是基于所有核心规则扩展的，当你升级了ESLint的任意版本号后，在使用命令行选项`--fix`前，先浏览一下问题报告，这样你才知道新修订的推荐规则集是否会修改你的代码，下面是一个javascript格式的配置样例：

```javascript
module.exports = {
    "extends": "eslint:all",
    "rules": {
        // override default options
        "comma-dangle": ["error", "always"],
        "indent": ["error", 2],
        "no-cond-assign": ["error", "always"],

        // disable now, but enable in the future
        "one-var": "off", // ["error", "never"]

        // disable
        "init-declarations": "off",
        "no-console": "off",
        "no-inline-comments": "off",
    }
}
```

## 基于Glob模式的匹配

v4.1.0+，有时你需要一个更精细控制的配置，比如在同一个目录下不同的文件期望使用不同的配置，因此你可以在配置文件通过`overrides`属性（值为数组）来使对应的配置只被应用于符合特定glob模式的文件，就像使用如命令行时的格式一样`（比如：app/**/*.test.js）`

### 它是怎么工作的

* glob模式对于文件路径的匹配是相对于配置文件所在的目录的，举个例子，如果你的配置文件位于`/Users/john/workspace/any-project/.eslintrs.js`，然后你希望检查的文件位于`/Users/john/workspace/any-project/lib/util.js`，那么`.eslintrc.js`中使用的模式是相对于路径`lib/util.js`执行的
* 同一配置文件中，glob模式的overrides相对于常规的配置有更高的优先级。同一个配置中有多个overrides时则按顺序应用。也就是说最后一个overrides块永远有最高的优先级。
* 通过glob模式指定的配置与其它ESLint的配置的工作方式几乎是一样的。override块可以包含任何在常规配置中有效的配置项，但`root`和`ignorePatterns`这两个配置项不一样
	* 通过glob指定的配置项中可以包含`extends`配置，但扩展配置中的`root`属性会被忽略，扩展配置中的`ignorePatterns`属性也只对通过glob模式匹配的文件生效。
	* 对于嵌套的`overrides`配置，只有当父配置和子配置的glob模式同时匹配时，才会被应用。当扩展配置中有`overrides`配置时也是这样。
* 在同一个`overrides`块中可以指定多个glob模式，只有当文件至少符合所提供的glob模式中任意一个时才会被应用配置项
* `overrides`块中也可以指定需要从匹配项中排除的模式（通过`excludedFiles`），如果文件匹配到任意排除的模式，则配置不会被应用到此文件。

### glob的相对匹配模式

```bash
project-root
├── app
│   ├── lib
│   │   ├── foo.js
│   │   ├── fooSpec.js
│   ├── components
│   │   ├── bar.js
│   │   ├── barSpec.js
│   ├── .eslintrc.json
├── server
│   ├── server.js
│   ├── serverSpec.js
├── .eslintrc.json
```

配置文件`app/.eslintrc.json`中定义的glob模式为`**/*Spec.js`，这个模式是相对于`app/.eslintrc.json`的，所以它将会匹配到`app/lib/fooSpec.js`和`app/components/barSpec.js`，但不会匹配到`server/serverSpec.js`，如果在项目根目录下的配置中指定同样的模式，则三个`*Spec`文件都会被匹配上

### 样例配置

```json
{
  "rules": {
    "quotes": ["error", "double"]
  },

  "overrides": [
    {
      "files": ["bin/*.js", "lib/*.js"],
      "excludedFiles": "*.test.js",
      "rules": {
        "quotes": ["error", "single"]
      }
    }
  ]
}
```

## 配置文件中的注释

JSON和YAML的配置文件都支持注释（`package.json`关在其列），你可以使用javascript风格的注释或YAML风格的注释，ESLint会无视它们，这会你的配置文件更加友好易理解，比如：

```json
{
    "env": {
        "browser": true
    },
    "rules": {
        // Override our default settings just for this directory
        "eqeqeq": "warn",
        "strict": "off"
    }
}
```

## 根据后缀名指定要检查的文件

目前告之ESLint只检查指定后缀名的文件的方法就是通过命令行参数`--ext`来指定一个逗号分隔的后缀名列表。注意，此标记仅在与目录一起使用时生效，如果是与文件名或glob模式一起使用时，则会被忽略。

## 忽略文件和文件夹

### 配置文件中的"ignorePatterns"属性

在配置文件中可以通过`ignorePatterns`来告诉ESLint去忽略指定的文件和文件夹。`ignorePatterns`（数组）中的每个值与下节要提到的`.eslintingore`文件中的每一行是相同的模式

```json
{
    "ignorePatterns": ["temp.js", "node_modules/"],
    "rules": {
        //...
    }
}
```

* `ignorePatterns`属性只会影响配置文件所在的目录
* `ignorePatterns`不能写在`overrides`属性下的配置下
* `.eslintignore`可以覆盖`ignorePatterns`属性

### .eslintignore

可能通过在项目的根目录下创建一个`.eslintignore`文件来告诉ESLint去忽略指定的文件和文件夹。这是一个纯文本的文件，每一行都是一个glob模式用来标识哪些路径需要被忽略。举个例子，下面的配置将会忽略所有的javascript文件：

```txt
**/*.js
```

当ESLint运行时，在决定哪些文件需要进行检查之前，会在当前工作目录下查找`.eslintignore`文件，如果找到这个文件，则在遍历目录时应用这些首选项。一次只能使用一个`.eslintignore`文件，因此当前工作目录之外的`.eslintignore`文件是不会使用

glob在匹配时是使用[node-ignore](https://github.com/kaelzhang/node-ignore)，所以会有下面这些可能的特性：

* 以"#"开头行会被视为注释，不会影响忽略的模式
* 模式是相对于`.eslintignore`文件的位置或当前工作目录，通过命令行参数`--ignore-pattern`指定路径时也是这样的。
* 以"!"开头的行表示一个反模式，会将被之前忽略的模式重新包括进来
* 忽略模式的行为与`.gitignore`的[规范](https://git-scm.com/docs/gitignore)是一致的

** 特别需要注意的是，与`.gitignore`文件一样，无论是在`.gitignore`还是通过`--ignore-pattern`来设置模式时，所有的路径必须使用正斜线作为路径的分隔符。 **

```txt
# 有效的
/root/src/*.js

# 无效的
\root\src\*.js
```

请参见`.gitignore`的规范来查看更多无效语法的例子。

除了忽略在`.eslintignore`指定的模式匹配到的文件之外，ESLint会始终忽略`/node_modules/*`和`/bower_components/*`下的文件。

举个例子，将下面的`.eslintignore`文件放到当前工作目录下，将会忽略所有项目根目录下的`node_modules`,`bower_components`以及`build/`下的所有文件，除了`build/index.js`。

```txt
# /node_modules/* and /bower_components/* in the project root are ignored by default

# Ignore built files except build/index.js
build/*
!build/index.js
```

** 重点关注：对于mono repo，`packages`文件下的node_modules文件夹默认情况下不会被忽略，需要显示指定 **

### 使用替代文件

如果你选择使用当前工作目录下另一个文件而非`.eslintignore`，则可以通过命令行参数`--ignore-path`来指定，比如，你可以使用`.jshintignore`，因为它有着一样的格式

```bash
eslint --ignore-path .jshintignore file.js
```

或者直接使用`.gitignore`文件

```bash
eslint --ignore-path .gitignore file.js
```

任何标准文件忽略格式的文件都可以使用，不过要记住，指定了`--ignore-path`意味着不会使用任何现有的`.eslintignore`文件。同时需要注意`.eslintignore`中的glob规则是遵循`.gitignore`的规则。

### 在package.json中使用eslintignore

如果找不到`.eslintignore`文件，且没有指定替代的文件，ESLint会到`package.json`中查询`eslintIgnore`的配置项

```json
{
  "name": "mypackage",
  "version": "0.0.1",
  "eslintConfig": {
      "env": {
          "browser": true,
          "node": true
      }
  },
  "eslintIgnore": ["hello.js", "world.js"]
}
```

### 警告被忽略的文件

当你将整个目录传给ESLint时，文件和文件夹是静默忽略的，但当你将指定的文件传给ESLint时，你就会看到一条警告，告诉你这个文件检查被跳过了，举个例子，比如你有一个这样的`.eslintignore`文件

```txt
foo.js
```

然后你执行如下命令：

```bash
eslint foo.js
```

你会看到这样的警告

```bash
foo.js
  0:0  warning  File ignored because of your .eslintignore file. Use --no-ignore to override.

✖ 1 problem (0 errors, 1 warning)
```

这条消息的出现是因为ESLint不确定你是否真的要检查这个文件，就如警告信息中提示的，你可以使用命令行参数`--no-ignore`来省略使用的忽略规则