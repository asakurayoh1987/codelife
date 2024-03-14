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

  
