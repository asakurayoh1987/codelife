# TypeScript学习笔记

## 1. TypeScript 5.3 New Features

### 1.1 Import Attributes 

在使用`import`添加一些标记信息用来告诉运行时如何处理导入的资源，比如对于JSON文件在导入时可以这样：

```typescript
// 静态导入
import obj from "./something.json" with { type: "json" };

// 动态导入
const obj = await import("./something.json", {
    with: { type: "json" }
});
```

此特性会逐步替代旧的称为`import assertions`的特性

### 1.2 Stable Support `resolution-mode` in Import Types

用于当进行类型导入时决定是什么传统的`require`方式还是现代的`import`方式

```typescript
// 传统方式
import type { TypeFromRequire } from "pkg" with {
    "resolution-mode": "require"
};

// 现代方式
import type { TypeFromImport } from "pkg" with {
  "resolution-mode": "import"
};
```

但此特性最初不能用于`import assertions`，但在TypeScript 5.3中，此特性也支持`import types`了，即：

```typescript
export type TypeFromRequire =
    import("pkg", { with: { "resolution-mode": "require" } }).TypeFromRequire;

export type TypeFromImport =
    import("pkg", { with: { "resolution-mode": "import" } }).TypeFromImport;

export interface MergedType extends TypeFromRequire, TypeFromImport {}
```

### 1.3 `resolution-mode` Supported in All Module Modes

之前只能在指定的模块解析选项下才能使用`resolution-mode`，比如`node16`或`nodenext`，但在TypeScript 5.3开始，你可以在其它所有模块解析选项下使用该特性

### 1.4 `switch (true)` Narrowing

当`swtich(true)`时，TypeScript可以根据`case`语句中的条件来智能推测类型，比如：

```typescript
function f(x: unknown) {
    switch (true) {
        case typeof x === "string":
            // 'x' is a 'string' here
            console.log(x.toUpperCase());
            // falls through...

        case Array.isArray(x):
            // 'x' is a 'string | any[]' here.
            console.log(x.length);
            // falls through...

        default:
          // 'x' is 'unknown' here.
          // ...
    }
}
```

### 1.5 Narrowing On Comparisons to Booleans

```typescript
interface A {
    a: string;
}

interface B {
    b: string;
}

type MyType = A | B;

function isA(x: MyType): x is A {
    return "a" in x;
}

function someFn(x: MyType) {
    if (isA(x) === true) {
        console.log(x.a); // TypeScript gets it now!
    }
}
```

### 1.6 `instanceof` Narrowing Through `Symbol.hasInstance`

在对自定义类似使用`instanceof`时类型推断更加精准

```typescript
class Point {
  	// 注意这里的类型守卫
    static [Symbol.hasInstance](val: unknown): val is PointLike {
        // Your custom type guard logic goes here
    }
}

function f(value: unknown) {
    if (value instanceof Point) {
        // Now, you can access properties defined in PointLike,
        // but you won't have access to specific Point methods or properties.
    }
}
```

### 1.7 Checks for `super` Property Accesses on Instance Fields

正常情况：

```typescript
class Base {
    someMethod() {
        console.log("Base method called!");
    }
}

class Derived extends Base {
    someMethod() {
        console.log("Derived method called!");
        super.someMethod();
    }
}

new Derived().someMethod();
// Prints:
//   Derived method called!
//   Base method called!
```

但当`Base`中的`someMethod`是一个类字段时，情况就不一样了

```typescript
class Base {
  	// 这里的someMethod是一个类字段，这会让someMethod位于每一个实例上，而不是在原型链上
    someMethod = () => {
        console.log("someMethod called!");
    }
}

class Derived extends Base {
    someOtherMethod() {
        super.someMethod(); // This will throw an error now!
    }
}

new Derived().someOtherMethod();
// 💥
// Doesn't work because 'super.someMethod' is 'undefined'.
```

在TypeScript5.3中，它可以提前告之这里存在错误

### 1.8 Settings to Prefer `type` Auto-Imports

```typescript
export let p: Person;
```

通常TypeScript会添加一个类型导入：

```typescript
import { Person } from "./types";

export let p: Person;
```

但现在TypeScript提供了配置项用来改变这种行为，比如`verbatimModuleSyntax`，会添加如下类型导入方式：

```typescript
import { type Person } from "./types";

export let p: Person;
```

### 1.9 other optimizations

- 跳过部分JSDoc的转译，编译耗时减少
- 优化对交叉类型的处理
- 

