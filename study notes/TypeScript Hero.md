# TypeScript Hero

[官网](https://typehero.dev/)

- 数组转对象

  ```ts
  type TupleToObject<T extends readonly (string | number | symbol)[]> = {
  	[K in T[number]]: K;
  };
  ```

- 获取数组的第一个元素的类型

  ```ts
  type First<T extends any[]> = T extends [infer first, ...any[]] ? first : never;
  ```

- 获取数组元素的长度

  ```ts
  type Length<T extends readonly any[]> = T["length"];
  ```

- Exclude实现

  ```ts
  type MyExclude<T, U> = T extends U ? never : T;
  ```

- Awaited实现

  ```ts
  type MyAwaited<T extends Promise<any> | { then: (onfulfilled: (arg: any) => any) => any }> =
  	T extends Promise<infer R>
  		? R extends Promise<any>
  			? MyAwaited<R>
  			: R
  		: T extends { then: (onfulfilled: (arg: infer U) => any) => any }
  			? U
  			: never;
  
  // 或者直接使用PromiseLike
  type MyAwaited<T extends Promise<any> | PromiseLike<any>> = T extends Promise<infer R>
  	? R extends Promise<any>
  		? MyAwaited<R>
  		: R
  	: T extends { then: (onfulfilled: (arg: infer U) => any) => any }
  		? U
  		: never;
  
  // 其中PromiseLike的定义
  interface PromiseLike<T> {
      then<TResult1 = T, TResult2 = never>(
          onfulfilled?: ((value: T) => TResult1 | PromiseLike<TResult1>) | null,
          onrejected?: ((reason: any) => TResult2 | PromiseLike<TResult2>) | null
      ): PromiseLike<TResult1 | TResult2>;
  }
  ```

- If实现类型选择

  ```ts
  type If<C extends boolean, T, F> = C extends true ? T : F;
  ```

- Concat实现

  ```ts
  type Concat<T extends readonly any[], U extends readonly any[]> = [...T,...U]
  ```

-  Equal实现

  ```ts
  type Equal<X, Y> = (<T>() => T extends X ? 1 : 2) extends <T>() => T extends Y ? 1 : 2
  	? true
  	: false;
  ```

- Includes 实现

  ```ts
  type Includes<T extends readonly any[], U> = T extends [infer R, ...infer Rest]
  	? Equal<R, U> extends true
  		? true
  		: Includes<Rest, U>
  	: false;
  ```

- Array.push实现

  ```ts
  type Push<T extends readonly any[], U> = [...T, U];
  ```

- Array.unshift

  ```ts
  type Unshift<T extends readonly any[], U> = [U, ...T];
  ```

- 获取函数的参数

  ```ts
  type MyParameters<T extends (...args: any[]) => any> = T extends (...args: infer P)=>any ? P : never
  ```

- Pick的实现

  ```ts
  type MyPick<T, K extends keyof T> = {
  	[P in K]: T[P];
  };
  ```

- ReturnType实现

  ```ts
  type MyReturnType<T> = T extends (...ary: any[]) => infer R ? R : never;
  ```

- Omit实现

  ```ts
  type MyOmit<T, K extends keyof T> = Pick<T, Exclude<keyof T, K>>;
  ```

- MyReadonly2

  当K不传递时，效果等同于Readonly，传递时则只将对应的key转成readonly

  ```ts
  type MyReadonly2<T, K extends keyof T = keyof T> = Readonly<T> & Omit<T, K>;
  ```

- DeepReadonly

  ```ts
  type DeepReadonly<T> = {
  	readonly [P in keyof T]: T[P] extends Function
  		? T[P]
  		: T[P] extends object
  			? DeepReadonly<T[P]>
  			: T[P];
  };
  ```

- TupleToUnion

  ```rust
  type TupleToUnion<T extends any[]> = T[number];
  ```

- Chainable

  ```ts
  type Chainable<T = {}> = {
    option: <K extends string, V>(
      key: K extends keyof T ? never : K,
      value: V)
    => K extends keyof T ? Chainable<Omit<T, K> & Record<K, V>> : Chainable<T & Record<K, V>>
    get: () => T
  }
  ```

- Last of Array

  ```ts
  type Last<T extends any[]> = T extends [...other: any, infer Last] ? Last : never;
  // 下面这种拿到数组的第length-1处的元素也是NB的构思
  type Last<T extends any[]> = [any, ...T][T["length"]];
  ```

- Pop

  ```ts
  type Pop<T extends any[]> = T extends [...infer Remain, any] ? Remain : [];
  ```

- Promise.all

  ```ts
  declare function PromiseAll<T extends any[]>(
  	values: readonly [...T],
  ): Promise<{
  	[key in keyof T]: Awaited<T[key]>;
  }>;
  
  type Awaited<T> = T extends null | undefined ? T : T extends object & {
      then(onfulfilled: infer F, ...args: infer _): any;
  } ? F extends (value: infer V, ...args: infer _) => any ? Awaited<...> : never : T
  ```

- Type Lookup

  ```ts
  type LookUp<U, T extends string> = {
  	[K in T]: U extends { type: T } ? U : never;
  }[T];
  ```
  
- Trim Left

  ```ts
  type TrimLeft<S extends string> = S extends `${" " | "\n" | "\t"}${infer Last}`
  	? TrimLeft<Last>
  	: S;
  ```
  
- Trim

  ```ts
  type Space = " " | "\n" | "\t";
  type TrimLeft<T> = T extends `${Space}${infer Right}` ? TrimLeft<Right> : T;
  type TrimRight<T> = T extends `${infer Left}${Space}` ? TrimRight<Left> : T;
  type Trim<S extends string> = TrimRight<TrimLeft<S>>;
  ```

- Capitalize

  ```ts
  type MyCapitalize<S extends string> = S extends `${infer First}${infer Last}`
  	? `${Uppercase<First>}${Last}`
  	: Uppercase<S>;
  ```

- Replace

  ```ts
  type Replace<S extends string, From extends string, To extends string> = From extends ""
  	? S
  	: S extends `${From}${infer Last}`
  		? `${To}${Last}`
  		: S extends `${infer Left}${From}${infer Last}`
  			? `${Left}${To}${Last}`
  			: S extends `${infer Left}${From}`
  				? `${Left}${To}`
  				: S;
  ```

- ReplaceAll

  ```ts
  type ReplaceAll<S extends string, From extends string, To extends string> = From extends ""
  	? S
  	: S extends `${From}${infer Last}`
  		? `${To}${ReplaceAll<Last, From, To>}`
  		: S extends `${infer Left}${From}${infer Last}`
  			? `${Left}${To}${ReplaceAll<Last, From, To>}`
  			: S extends `${infer Left}${From}`
  				? `${Left}${To}`
  				: S;
  ```

- Append Argument

  ```ts
  type AppendArgument<Fn extends Function, A> = Fn extends (...arg: infer Argument) => infer Result
  	? (...arg: [...Argument, A]) => Result
  	: never;
  ```

- Permutation

  ```ts
  type Permutation<T, K = T> = [T] extends [never]
  	? []
  	: K extends K
  		? [K, ...Permutation<Exclude<T, K>>]
  		: never;
  ```
  
  这篇[文章](https://juejin.cn/post/7165170011282079751)有很好的解释
  
- Length of String

  ```ts
  type LengthOfString<
  	S extends string,
  	T extends any[] = [],
  > = S extends `${infer First}${infer Last}` ? LengthOfString<Last, [...T, First]> : T["length"];
  ```

  将字符串转成字符数组，然后获取数组的length
  
- Append to object

  ```ts
  type AppendToObject<T extends {}, U extends string, V> = {
  	[K in keyof T | U]: K extends keyof T ? T[K] : V;
  };
  ```

- Absolute

  ```ts
  type Absolute<T extends number | string | bigint> = `${T}` extends `-${infer A}` ? A : `${T}`;
  ```

- String to Union

  ```ts
  type StringToUnion<T extends string> = T extends `${infer First}${infer Last}`
  	? First | StringToUnion<Last>
  	: never;
  ```

- Merge

  ```ts
  type Merge<F extends {}, S extends {}> = {
  	[K in keyof F | keyof S]: K extends keyof S ? S[K] : K extends keyof F ? F[K] : never;
  };
  ```

- KebabCase

  ```ts
  type KebabCase<S extends string> = S extends `${infer S1}${infer S2}`
  	? S2 extends Uncapitalize<S2>
  		? `${Uncapitalize<S1>}${KebabCase<S2>}`
  		: `${Uncapitalize<S1>}-${KebabCase<S2>}`
  	: S;
  ```

- Diff

  ```ts
  type Diff<O, O1> = Omit<O & O1, keyof (O | O1)>;
  ```

- Anyof

  ```ts
  type AnyOf<T extends readonly any[]> = T[number] extends
  	| 0
  	| ""
  	| false
  	| []
  	| undefined
  	| null
  	| { [key: string]: never }
  	? false
  	: true;
  ```

  关于`{[key: string]: never}`为什么是false，可以参见[这里](https://stackoverflow.com/questions/71799555/why-1-extends-key-string-never-is-false)

- IsNever

  ```ts
  type IsNever<T> = [T] extends [never] ? true : false;
  ```

- IsUnion

  ```ts
  type IsUnionImpl<T, C extends T = T> = (
  	T extends T ? (C extends T ? true : unknown) : never
  ) extends true
  	? false
  	: true;
  type IsUnion<T> = IsUnionImpl<T>;
  ```

  这里有[解析](https://github.com/type-challenges/type-challenges/issues/1140)，关键点：

  1. 对于`type Distributive<T> =  T extends T ? ConditionTrue<T> : never`，根据官方的[条件分配类型](https://www.typescriptlang.org/docs/handbook/2/conditional-types.html#distributive-conditional-types)，如果T为一个联合类型，则会对该联合类型中的每一个执行`ConditionTrue`，即`Distributive<string|number>`会按`ConditionTrue<string>|ConditionTrue<number>`来执行
  
  2. 基于上述构造一个分布式条件类型：`type DoubleDistribute<T, TRUE, FALSE, C = T> = T extends T ? C extends T ? TRUE : FALSE : never;`，因为对于任意类型`T extends T`一定为`true`，所以会走到`C extends T`，而`C`是`T`的一个拷贝，如果`C`是一个联合体，则它中的单个类型无法`extends`其它所有类型，所以`DoubleDistribute<T>`的结果为`TRUE`（`T`是单体类型）或`TRUE|FALSE`（`T`是联合类型）
  
  3. 基于上述
  
     ```ts
     type DoubleDistribute<T, TRUE, FALSE, C = T> = T extends T ? C extends T ? TRUE : FALSE : never;
     type IsUnionDetailed<T> = DoubleDistribute<T, true, unknown> extends true ? false : true;
     ```
  
     对于单体类型的推导过程：
  
     ```ts
     IsUnion<string>
     => IsUnionImpl<string, string>
     => (string extends string ? string extends string ? true : unknown : never) extends true ? false : true
     => (string extends string ? true : unknown) extends true ? false : true
     => (true) extends true ? false : true
     => false
     ```
  
     对于联合体类型的推导过程：
  
     ```ts
     IsUnion<string|number>
     => IsUnionImpl<string|number, string|number>
     => (string|number extends string|number ? string|number extends string|number ? true : unknown : never) extends true ? false : true
     => (
       (string extends string|number ? string|number extends string ? true : unknown : never) |
       (number extends string|number ? string|number extends number ? true : unknown : never)
     ) extends true ? false : true
     => (
       (string|number extends string ? true : unknown) |
       (string|number extends number ? true : unknown)
     ) extends true ? false : true
     => (
       (
         (string extends string ? true : unknown) |
         (number extends string ? true : unknown)
       ) |
       (
         (string extends number ? true : unknown) |
         (number extends number ? true : unknown)
       )
     ) extends true ? false : true
     => (
       (
         (true) |
         (unknown)
       ) |
       (
         (unknown) |
         (true)
       )
     ) extends true ? false : true
     => (unknown|unknown) extends true ? false : true
     => (unknown) extends true ? false : true
     => true
     ```
  
  4. unknown和除any之外的类型做联合类型时，只留下unknown；unknown和其它类型做交叉类型时得到去掉unknown的类型
  
- ReplaceKeys

  ```ts
  type ReplaceKeys<U, T, Y> = {
  	[K in keyof U]: K extends T ? (K extends keyof Y ? Y[K] : never) : U[K];
  };
  ```

  ```ts
  type NodeA = {
    type: 'A'
    name: string
    flag: number
  }
  
  type NodeB = {
    type: 'B'
    id: number
    flag: number
  }
  
  type A<T> = {
  	readonly [K in keyof T]: T[K];
  };
  
  type B = NodeA | NodeB
  // 此时，C的类型为 A<NodeA> | A<NodeB>
  type C = A<B>
  ```

- Remove Index Signature

  ```ts
  type RemoveIndexSignature<T, P = PropertyKey> = {
  	[K in keyof T as P extends K ? never : K extends P ? K : never]: T[K];
  };
  ```

  重点在内置的PropertyKey，它表示`string|number|symbol`，而`as`语法是对K进行过滤的，只有当K为索引签名时，这里的`P extends K`才成立，成立时返回`never`就表示该K被丢弃了

- Percentage Parser

  ```ts
  type checkPrefix<T> = T extends "+" | "-" ? T : never;
  type checkSuffix<T> = T extends `${infer S}%` ? [S, "%"] : [T, ""];
  type PercentageParser<A extends string> = A extends `${checkPrefix<infer F>}${infer L}`
  	? [F, ...checkSuffix<L>]
  	: ["", ...checkSuffix<A>];
  ```
  
- Drop Char

  ```ts
  type DropChar<S, C, Arr extends string = ''> = S extends `${infer F}${infer L}`
  	? F extends C
  		? DropChar<L, C, Arr>
  		: DropChar<L, C, `${Arr}${F}`>
  	: Arr;
  ```
  
- MinusOne

  ```ts
  type ParseInt<T> = T extends `${infer X extends number}` ? X : never;
  
  type RemoveLeadingZeros<T extends string> = T extends "0"
  	? T
  	: T extends `${0}${infer Rest}`
  		? RemoveLeadingZeros<Rest>
  		: T;
  
  type InnerMinusOne<T extends string> = T extends `${infer X extends number}${infer Y}`
  	? X extends 0
  		? `9${InnerMinusOne<Y>}`
  		: `${[-1, 0, 1, 2, 3, 4, 5, 6, 7, 8][X]}${Y}`
  	: "";
  
  type Reverse<T extends string> = T extends `${infer X}${infer Y}` ? `${Reverse<Y>}${X}` : "";
  
  type MinusOne<T extends number> = T extends 0
  	? -1
  	: ParseInt<RemoveLeadingZeros<Reverse<InnerMinusOne<Reverse<`${T}`>>>>>;
  
  ```

- PickByType

  ```ts
  type PickByType<T, U> = {
  	[K in keyof T as T[K] extends U ? K : never]: T[K];
  };
  ```

- StartsWith

  ```ts
  type StartsWith<T extends string, U extends string> = T extends `${U}${string}` ? true : false;
  ```

- EndsWith

  ```ts
  type EndsWith<T extends string, U extends string> = T extends `${string}${U}` ? true : false;
  ```

- PartialByKeys

  ```ts
  type PartialByKeys<T, K extends keyof T = keyof T> = Omit<Partial<Pick<T, K>> & Omit<T, K>, never>;
  ```

- RequiredByKeys

  ```ts
  type RequiredByKeys<T, K extends keyof T = keyof T> = Omit<
  	Required<Pick<T, K>> & Omit<T, K>,
  	never
  >;
  ```

- Mutable

  ```ts
  type Mutable<T extends object> = {
  	-readonly [K in keyof T]: T[K];
  };
  ```

- OmitByType

  ```ts
  type OmitByType<T, U> = {
  	[P in keyof T as T[P] extends U ? never : P]: T[P];
  };
  ```

- ObjectEntries

  ```ts
  type ObjectEntries<T> = {
    [P in keyof Required<T>]: [P, Required<T>[P] extends never ? undefined : Required<T>[P]]
  }[keyof T]
  
  type ObjectEntries<T> = {
    [K in keyof Required<T>]: [K, [T[K]] extends [undefined] ? undefined : Required<T>[K]]
  }[keyof T]
  ```

- Shift

  ```ts
  type Shift<T extends any[]> = T extends [infer F, ...infer L] ? L : [];
  ```

- Tuple to Nested Object

  ```ts
  type TupleToNestedObject<T, U> = T extends [infer F, ...infer L]
  	? { [key in F & string]: TupleToNestedObject<L, U> }
  	: U;
  ```

- Reverse

  ```ts
  type Reverse<T extends any[]> = T extends [infer F, ...infer L] ? [...Reverse<L>, F] : [];
  ```

- Flip Arguments

  ```ts
  type Reverse<T extends any[]> = T extends [infer F, ...infer L] ? [...Reverse<L>, F] : T;
  type FlipArguments<T extends Function> = T extends (...args: infer Args) => infer R
  	? (...args: Reverse<Args>) => R
  	: never;
  ```

- FlattenDepth

  ```ts
  type FlattenDepth<T, N = 1, Arr extends any[] = []> = Arr["length"] extends N
  	? T
  	: T extends [infer F, ...infer L]
  		? F extends any[]
  			? [...FlattenDepth<F, N, [...Arr, 1]>, ...FlattenDepth<L, N, Arr>]
  			: [F, ...FlattenDepth<L, N, Arr>]
  		: [];
  ```

- BEM style string

  ```ts
  type BEM<B extends string, E extends string[], M extends string[]> = E["length"] extends 0
  	? M["length"] extends 0
  		? B
  		: `${B}--${M[number]}`
  	: M["length"] extends 0
  		? `${B}__${E[number]}`
  		: `${B}__${E[number]}--${M[number]}`;
  ```

- Flip

  ```ts
  type Flip<T extends Record<string, string | number | boolean>> = {
  	[P in keyof T as `${T[P]}`]: P;
  };
  ```

- Fibonacci Sequence

  ```ts
  type Fibonacci<
  	T extends number,
  	CurrentIndex extends any[] = [1],
  	Prev extends any[] = [],
  	Current extends any[] = [1],
  > = CurrentIndex["length"] extends T
  	? Current["length"]
  	: Fibonacci<T, [...CurrentIndex, 1], Current, [...Prev, ...Current]>;
  ```

- AllCombinations

  ```ts
  type StringToUnion<T extends string> = T extends `${infer F}${infer L}` ? F | StringToUnion<L> : T;
  type AllCombinations<
  	S extends string,
  	T extends string = StringToUnion<S>,
  	U extends string = T,
  > = S extends `${infer F}${infer L}`
  	? U extends U
  		? `${U}${AllCombinations<L, U extends "" ? U : Exclude<T, U>>}`
  		: never
  	: "";
  ```

- Greater Than

  ```ts
  type ParseInt<T> = T extends `${infer X extends number}` ? X : never;
  
  type RemoveLeadingZeros<T extends string> = T extends "0"
  	? T
  	: T extends `${0}${infer Rest}`
  		? RemoveLeadingZeros<Rest>
  		: T;
  
  type InnerMinusOne<T extends string> = T extends `${infer X extends number}${infer Y}`
  	? X extends 0
  		? `9${InnerMinusOne<Y>}`
  		: `${[-1, 0, 1, 2, 3, 4, 5, 6, 7, 8][X]}${Y}`
  	: "";
  
  type Reverse<T extends string> = T extends `${infer X}${infer Y}` ? `${Reverse<Y>}${X}` : "";
  
  type MinusOne<T extends number> = ParseInt<
  	RemoveLeadingZeros<Reverse<InnerMinusOne<Reverse<`${T}`>>>>
  >;
  
  type InnerGreaterThan<T extends number, U extends number> = T extends U
  	? true
  	: T extends 0
  		? false
  		: InnerGreaterThan<MinusOne<T>, U>;
  
  type GreaterThan<T extends number, U extends number> = T extends U
  	? false
  	: U extends 0
  		? true
  		: InnerGreaterThan<T, U>;
  ```

- Zip

  ```ts
  type Zip<T extends any[], U extends any[], Result extends any[] = []> = Result["length"] extends
  	| T["length"]
  	| U["length"]
  	? Result
  	: Zip<T, U, [...Result, [T[Result["length"]], U[Result["length"]]]]>;
  ```

- IsTuple

  ```ts
  type IsTuple<T> = [T] extends [never]
  	? false
  	: T extends readonly any[]
  		? number extends T["length"]
  			? false
  			: true
  		: false;
  ```

  Exact number extends `number` and `number` does not extends exact number but extends itself. So we can check if `T['length']` is exact number or not by checking if `number` extends `T['length']` or not

- Chunk

  ```ts
  // 我的实现
  type Chunk<
  	T extends any[],
  	U extends number,
  	S extends any[] = [],
  	Result extends any[] = [],
  > = T extends [infer F, ...infer L]
  	? S["length"] extends U
  		? Chunk<T, U, [], [...Result, S]>
  		: Chunk<L, U, [...S, F], Result>
  	: S["length"] extends 0
  		? Result
  		: [...Result, S];
  
  // 别的实现，少使用到一个缓冲区
  type Chunk<T extends any[], N extends number, Swap extends any[] = []> = Swap["length"] extends N
  	? [Swap, ...Chunk<T, N>]
  	: T extends [infer K, ...infer L]
  		? Chunk<L, N, [...Swap, K]>
  		: Swap extends []
  			? Swap
  			: [Swap];
  ```

- Fill

  ```ts
  type Fill<
    T extends unknown[],
    N,
    Start extends number = 0,
    End extends number = T['length'],
    Count extends any[] = [],
    Flag extends boolean = Count['length'] extends Start ? true : false
  > = Count['length'] extends End
    ? T
    : T extends [infer R, ...infer U]
      ? Flag extends false
        ? [R, ...Fill<U, N, Start, End, [...Count, 0]>]
        : [N, ...Fill<U, N, Start, End, [...Count, 0], Flag>]
      : T
  ```

- TrimRight

  ```ts
  type Space = " " | "\n" | "\t";
  type TrimRight<S extends string> = S extends `${infer Left}${Space}` ? TrimRight<Left> : S;
  ```
  
- Without

  ```ts
  type WithoutNumber<T, U> = T extends [infer F, ...infer L]
  	? F extends U
  		? WithoutNumber<L, U>
  		: [F, ...WithoutNumber<L, U>]
  	: [];
  
  type Without<T, U> = U extends number
  	? WithoutNumber<T, U>
  	: U extends [infer F, ...infer L]
  		? Without<WithoutNumber<T, F>, L>
  		: T;
  
  // 别人的实现
  type ToUnion<T> = T extends any[] ? T[number] : T
  type Without<T, U> = 
    T extends [infer R, ...infer F]
      ? R extends ToUnion<U>
        ? Without<F, U>
        : [R, ...Without<F, U>]
      : T
  ```

- Trunc

  ```ts
  type Trunc<T extends number | string> = `${T}` extends `${infer F}.${infer L}`
  	? F extends ""
  		? "0"
  		: F
  	: `${T}`;
  ```

- IndexOf

  ```ts
  type IsEqual<T, U> = T extends U ? (U extends T ? true : false) : false;
  type IndexOf<T extends readonly any[], U, Result extends number[] = []> = T extends [
  	infer F,
  	...infer L,
  ]
  	? IsEqual<F, U> extends true
  		? Result["length"]
  		: IndexOf<L, U, [...Result, 0]>
  	: -1;
  ```

- Equals

  ```ts
  export type Equals<X, Y> =
      (<T>() => T extends X ? 1 : 2) extends
      (<T>() => T extends Y ? 1 : 2) ? true : false;
  ```

  [链接](https://github.com/microsoft/TypeScript/issues/27024#issuecomment-421529650)

- Join

  ```ts
  type Join<T extends any[], U extends string | number> = T extends [infer F, ...infer R]
  	? `${F & string}${R["length"] extends 0 ? "" : U}${Join<R, U>}`
  	: "";
  ```

- LastIndexOf

  ```ts
  type IsEqual<T, U> = T extends U ? (U extends T ? true : false) : false;
  type LastIndexOf<
  	T extends readonly unknown[],
  	U,
  	Result extends number = -1,
  	Count extends number[] = [],
  > = T extends [infer F, ...infer R]
  	? LastIndexOf<R, U, IsEqual<F, U> extends true ? Count["length"] : Result, [...Count, 0]>
  	: Result;
  ```

- Unique

  ```ts
  type Equal<X, Y> = (<T>() => T extends X ? 1 : 2) extends <T>() => T extends Y ? 1 : 2
  	? true
  	: false;
  type Include<T, U extends unknown[]> = U extends [infer F, ...infer R]
  	? Equal<T, F> extends true
  		? true
  		: Include<T, R>
  	: false;
  type Unique<T, U extends unknown[] = []> = T extends [infer F, ...infer R]
  	? Include<F, U> extends true
  		? Unique<R, U>
  		: Unique<R, [...U, F]>
  	: U;
  ```

- MapTypes

  ```ts
  type MapTypes<T extends object, R extends { mapFrom: any; mapTo: any }> = {
  	[P in keyof T]: T[P] extends R["mapFrom"]
  		? R extends { mapFrom: T[P] }
  			? R["mapTo"]
  			: never
  		: T[P];
  };
  ```

- Construct Tuple

  ```ts
  type ConstructTuple<L extends number, R extends unknown[] = []> = L extends R["length"]
  	? R
  	: ConstructTuple<L, [...R, unknown]>;
  ```

- Number Range

  ```ts
  type NumberRange<
  	L extends number,
  	H extends number,
  	Count extends unknown[] = [],
  	R extends number[] = [],
  	Flag extends boolean = Count["length"] extends L ? true : false,
  > = Count["length"] extends H
  	? [...R, H][number]
  	: Flag extends true
  		? NumberRange<L, H, [...Count, 0], [...R, Count["length"]], Flag>
  		: NumberRange<L, H, [...Count, 0], R>;
  ```

- Combination

  ```ts
  type Combination<T extends string[], All = T[number], Item = All>
    = Item extends string
      ? Item | `${Item} ${Combination<[], Exclude<All, Item>>}`
      : never
  ```

- Subsequence

  ```ts
  type Subsequence<T extends any[], Prefix extends any[] = []> = T extends [infer F, ...infer R]
  	? Subsequence<R, Prefix | [...Prefix, F]>
  	: Prefix;
  ```

  
