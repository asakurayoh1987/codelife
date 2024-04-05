# TypeScript全面进阶指南

## 1. 开篇：用正确的方式学习

## 2. 工欲善其事：打造最舒适的TypeScript开发环境

### 2.1 VS Code配置与插件

#### 推荐插件

- TypeScript Importer
- Move TS
- ErrorLens

#### 配置

"typescript Inlay Hints"中配置

- Function Like Return Types，显示推导得到的函数返回值类型
- Parameter Names，显示函数入参的名称
- Parameter Types，显示函数入参的类型
- Variable Types，显示变量的类型

#### 创建TypeScript项目

```bash
npx --package typescript tsc --init
# 如果已全局安装了TypeScript或已作为项目依赖进行了安装
tsc --init
```

## 3. 进入类型的世界：理解原始类型与对象类型

### null 与 undefined

它们作为类型时，表示的是一个有意义的具体类型值。这两者在没有开启 `strictNullChecks` 检查的情况下，会**被视作其他类型的子类型**，比如 string 类型会被认为包含了 null 与 undefined 类型

### void

**void 操作符会执行后面跟着的表达式并返回一个 undefined**，如你可以使用它来执行一个立即执行函数（IIFE）

在TypeScript中表示一个内部没有`return`语句，或者没有显示`return`一个值的函数的返回值

### 数组与元组

当长度明确时，可以使用元组来代替数组，这样比如在进行`arr[index]`访问时，对于后者，在进行越界访问时会给出类型报错

对于标记为可选的成员，在 `--strictNullCheckes` 配置下会被视为一个 `string | undefined` 的类型。此时元组的长度属性也会发生变化，比如：

```ts
const arr6: [string, number?, boolean?] = ['linbudu'];
type TupleLength = typeof arr6.length; // 1 | 2 | 3
```

### type 与 interface

推荐的方式是，interface 用来描述**对象、类的结构**，而类型别名用来**将一个函数签名、一组联合类型、一个工具类型等等抽离成一个完整独立的类型**

### object、Object 以及 {}

在TypeScript中，Object包含了所有的类型

和 Object 类似的还有 Boolean、Number、String、Symbol，这几个**装箱类型（Boxed Types）** 同样包含了一些超出预期的类型。以 String 为例，它同样包括 undefined、null、void，以及代表的 **拆箱类型（Unboxed Types）** string，但并不包括其他装箱类型对应的拆箱类型，如 boolean 与 基本对象类型

**在任何情况下，都不应该使用这些装箱类型。**

object 的引入就是为了解决对 Object 类型的错误使用，它代表**所有非原始类型的类型，即数组、对象与函数类型这些**

可以认为使用`{}`作为类型签名就是一个合法的，但**内部无属性定义的空对象**，这类似于 Object（想想 `new Object()`），它意味着任何非 null / undefined 的值

为了更好地区分 `Object`、`object` 以及`{}`这三个具有迷惑性的类型，我们再做下总结：

- 在任何时候都**不要，不要，不要使用** Object 以及类似的装箱类型。
- 当你不确定某个变量的具体类型，但能确定它不是原始类型，可以使用 object。但我更推荐进一步区分，也就是使用 `Record<string, unknown>` 或 `Record<string, any>` 表示对象，`unknown[]` 或 `any[]` 表示数组，`(...args: any[]) => any`表示函数这样。
- 我们同样要避免使用`{}`。`{}`意味着任何非 `null / undefined` 的值，从这个层面上看，使用它和使用 `any` 一样恶劣。

## 4. 掌握字面量类型与枚举，让你的类型再精确一些

在 TypeScript 中要求赋值类型始终与原类型一致（如果声明了的话）。因此对于 let 声明，**只需要推导至这个值从属的类型即可**。而 const 声明的原始类型变量将不再可变，因此类型可以直接一步到位收窄到最精确的字面量类型，但对象类型变量仍可变（但同样会要求其属性值类型保持一致）。

这些现象的本质都是 TypeScript 的**类型控制流分析**

## 5. 函数与Class中的类型：详解函数重载与面向对象

### 函数

#### Callable Interface

```ts
interface FuncFooStruct {
  (name: string): number
}
```

#### void类型

**在 TypeScript 中，undefined 类型是一个实际的、有意义的类型值，而 void 才代表着空的、没有意义的类型值**

#### 重载

```ts
function func(foo: number, bar: true): string;
function func(foo: number, bar?: false): number;
function func(foo: number, bar?: boolean): string | number {
  if (bar) {
    return String(foo);
  } else {
    return foo * 599;
  }
}

const res1 = func(599); // number
const res2 = func(599, true); // string
const res3 = func(599, false); // number
```

拥有多个重载声明的函数在被调用时，**是按照重载的声明顺序往下查找的**。因此在第一个重载声明中，为了与逻辑中保持一致，即在 bar 为 true 时返回 string 类型，这里我们需要将第一个重载声明的 bar 声明为必选的字面量类型

### Class

TypeScript 4.3 新增了 `override` 关键字，来确保派生类尝试覆盖的方法一定在基类中存在定义：

```ts
class Base {
  printWithLove() { }
}

class Derived extends Base {
  override print() {
    // ...
  }
}
```

在这里 TS 将会给出错误，因为**尝试覆盖的方法并未在基类中声明**。通过这一关键字我们就能确保首先这个方法在基类中存在，同时标识这个方法在派生类中被覆盖了

 #### Newable Interface

```ts
class Foo { }

interface FooStruct {
  new(): Foo
}

declare const NewableFoo: FooStruct;

const foo = new NewableFoo();

```

## 6. 探秘内置类型：any、unknown、never与类型断言

### 内置类型：any、unknown 与 never

**any 的本质**是类型系统中的**顶级类型**，即 Top Type，这是许多类型语言中的重要概念

unknown 类型和 any 类型有些类似，一个 unknown 类型的变量可以再次**赋值为**任意其它类型，但**只能赋值给** any 与 unknown 类型的变量。**在类型未知的情况下，更推荐使用 unknown 标注**

**never 类型不携带任何的类型信息**，因此会在联合类型中被直接移除，它是整个类型系统层级中**最底层的类型**，即Bottom Type，它是所有类型的子类型，但**只有never类型的变量可以赋值给另一个never类型的变量**

### 类型层级初探

- 最顶级的类型，any 与 unknown
- 特殊的 Object ，它也包含了所有的类型，但和 Top Type 比还是差了一层
- String、Boolean、Number 这些装箱类型
- 原始类型与对象类型
- 字面量类型，即更精确的原始类型与对象类型嘛，需要注意的是 null 和 undefined 并不是字面量类型的子类型
- 最底层的 never

而实际上类型断言的工作原理也和类型层级有关，在判断断言是否成立，即差异是否能接受时，实际上判断的即是这两个类型是否能够找到一个公共的父类型。比如 `{ }` 和 `{ name: string }` 其实可以认为拥有公共的父类型 `{}`（一个新的 `{}`！你可以理解为这是一个基类，参与断言的 `{ }` 和 `{ name: string }` 其实是它的派生类）

如果找不到具有意义的公共父类型呢？这个时候就需要请出 **Top Type** 了，如果我们把它先断言到 **Top Type**，那么就拥有了公共父类型 **Top Type**，再断言到具体的类型也是同理。你可以理解为**先向上断言，再向下断言**

## 7. 类型编程好帮手：TypeScript类型工具（上）

### 类型别名

类型工具可以分为 **类型创建** 与 **类型安全保护** 两类

类型别名 + 泛型 = 工具类型

```ts
type Factory<T> = T | number | string;
```

**工具类型**就像一个函数一样，**泛型是入参**，内部逻辑基于入参进行某些操作，再返回一个新的类型

### 联合类型与交叉类型

联合类型的符号是`|`，它代表按位或，即只需要符合联合类型中的一个类型，既可以认为实现了这个联合类型，如`A | B`，**只需要实现 A 或 B 即可**

交叉类型的符号是`&`，代表着按位与，需要符合这里的所有类型，才可以说实现了这个交叉类型，即 `A & B`，**需要同时满足 A 与 B 两个类型**才行

### 索引类型

#### 1. 索引签名类型

```typescript
interface AllStringTypes {
	[key: string]: string;
}
```

#### 2. 索引类型查询

```ts
interface Foo {
  linbudu: 1,
  599: 2
}

type FooKeys = keyof Foo; // "linbudu" | 599
// 在 VS Code 中悬浮鼠标只能看到 'keyof Foo'
// 看不到其中的实际值，你可以这么做：
type FooKeys = keyof Foo & {}; // "linbudu" | 599
```

#### 3. 索引类型访问

```ts
interface NumberRecord {
  [key: string]: number;
}

type PropType = NumberRecord[string]; // number
```

```ts
interface Foo {
  propA: number;
  propB: boolean;
  propC: string;
}

type PropTypeUnion = Foo[keyof Foo]; // string | number | boolean
```

使用字面量联合类型进行索引类型访问时，其结果就是**将联合类型每个分支对应的类型进行访问后的结果，重新组装成联合类型**。**索引类型查询、索引类型访问通常会和映射类型一起搭配使用**

| 类型工具                             | 创建新类型的方式                                             | 常见搭配           |
| ------------------------------------ | ------------------------------------------------------------ | ------------------ |
| 类型别名（Type Alias）               | 将一组类型/类型结构封装，作为一个新的类型                    | 联合类型、映射类型 |
| 工具类型（Tool Type）                | 在类型别名的基础上，基于泛型去动态创建新类型                 | 基本所有类型工具   |
| 联合类型（Union Type）               | 创建一组类型集合，满足其中一个类型即满足这个联合类型（\|\|） | 类型别名、工具类型 |
| 交叉类型（Intersection Type）        | 创建一组类型集合，满足其中所有类型才满足映射联合类型（&&）   | 类型别名、工具类型 |
| 索引签名类型（Index Signature Type） | 声明一个拥有任意属性，键值类型一致的接口结构                 | 映射类型           |
| 索引类型查询（Indexed Type Query）   | 从一个接口结构，创建一个由其键名字符串字面量组成的联合类型   | 映射类型           |
| 索引类型访问（Indexed Access Type）  | 从一个接口结构，使用键名字符串字面量访问到对应的键值类型     | 类型别名、映射类型 |
| 映射类型 （Mapping Type）            | 从一个联合类型依次映射到其内部的每一个类型                   | 工具类型           |

## 8. 类型编程好帮手：TypeScript类型工具（下）

### 类型查询操作符：typeof

typeof 返回的类型就是当你把鼠标悬浮在变量名上时出现的推导后的类型，并且是**最窄的推导程度（即到字面量类型的级别）**

### 类型守卫

TypeScript 中提供了非常强大的类型推导能力，它会随着你的代码逻辑不断尝试收窄类型，这一能力称之为**类型的控制流分析**（也可以简单理解为类型推导）

```ts
function isString(input: unknown): input is string {
  return typeof input === "string";
}

function foo(input: string | number) {
  if (isString(input)) {
    // 正确了
    (input).replace("linbudu", "linbudu599")
  }
  if (typeof input === 'number') { }
  // ...
}
```

isString 函数称为**类型守卫**，在它的返回值中，我们不再使用 boolean 作为类型标注，而是使用 `input is string` 这么个奇怪的搭配，拆开来看它是这样的：

- input 函数的某个参数；
- `is string`，即 **is 关键字 + 预期类型**，即如果这个函数**成功返回为 true**，那么 is 关键字前这个入参的类型，就会**被这个类型守卫调用方后续的类型控制流分析收集到**。

## 9. 类型编程基石：TypeScript中无处不在的泛型

### 泛型约束与默认值 

在泛型中，我们可以使用 `extends` 关键字来**约束**传入的泛型参数必须符合要求。关于 extends，`A extends B` 意味着 **A 是 B 的子类型**，这里我们暂时只需要了解非常简单的判断逻辑，也就是说 A 比 B 的类型更精确，或者说更复杂。具体来说，可以分为以下几类。

- 更精确，如**字面量类型是对应原始类型的子类型**，即 `'linbudu' extends string`，`599 extends number` 成立。类似的，**联合类型子集均为联合类型的子类型**，即 `1`、 `1 | 2` 是 `1 | 2 | 3 | 4` 的子类型。
- 更复杂，如 `{ name: string }` 是 `{}` 的子类型，因为在 `{}` 的基础上增加了额外的类型，基类与派生类（父类与子类）同理。

箭头函数的泛型：

```ts
const handle = <T>(input: T): T => {}
```

