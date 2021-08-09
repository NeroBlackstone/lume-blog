---
title: TypeScript Utility Types 学习笔记
description: ''
date: 2021-08-07
img: https://res.cloudinary.com/neroblackstone/image/upload/v1628328399/proxy-image_talsae.png
tags:
- TypeScript

---
TypeScript 提供了几种utility型来促进常见的类型转换。这些utility类型可在全局范围内使用。

## Partial<Type>

构造一个 Type 的所有属性都设置为 optional 的类型。此utility将返回表示给定类型的所有子集的类型。

``` ts
interface Todo {
  title: string;
  description: string;
}
 
function updateTodo(todo: Todo, fieldsToUpdate: Partial<Todo>) {
  return { ...todo, ...fieldsToUpdate };
}
 
const todo1 = {
  title: "organize desk",
  description: "clear clutter",
};
 
const todo2 = updateTodo(todo1, {
  description: "throw out trash",
});
```

## Required<Type>

构造一个由 `Type` 设置为 required 的所有属性组成的类型。与[`Partial`](https://www.typescriptlang.org/docs/handbook/utility-types.html#partialtype)相反。

``` ts
interface Props {
  a?: number;
  b?: string;
}
 
const obj: Props = { a: 5 };
 
const obj2: Required<Props> = { a: 5 };
//Property 'b' is missing in type '{ a: number; }' but required in type 'Required<Props>'.
```

## Readonly<Type>

构造一个类型，其中 `Type` 的所有属性都设置为 `readonly`，这意味着不能重新分配构造类型的属性。

``` ts
interface Todo {
  title: string;
}
 
const todo: Readonly<Todo> = {
  title: "Delete inactive users",
};
 
todo.title = "Hello";
//Cannot assign to 'title' because it is a read-only property.
```

此utility可用于表示将在运行时失败的赋值表达式（即尝试重新分配的[ frozen object](https://developer.mozilla.org/docs/Web/JavaScript/Reference/Global_Objects/Object/freeze)属性时）。

``` ts
function freeze<Type>(obj: Type): Readonly<Type>;
```

## Record<Keys,Type>

构造一个对象类型，其属性键为 `Keys`，属性值为 `Type`。此实用程序可用于将一种类型的属性映射到另一种类型。

``` ts
interface CatInfo {
  age: number;
  breed: string;
}
 
type CatName = "miffy" | "boris" | "mordred";
 
const cats: Record<CatName, CatInfo> = {
  miffy: { age: 10, breed: "Persian" },
  boris: { age: 5, breed: "Maine Coon" },
  mordred: { age: 16, breed: "British Shorthair" },
};
 
cats.boris;
 
const cats: Record<CatName, CatInfo>
```

## Pick<Type, Keys>

通过从 Type 中选取一组属性键（字符串文字或字符串文字的并集）来构造一个类型。

``` ts
interface Todo {
  title: string;
  description: string;
  completed: boolean;
}
 
type TodoPreview = Pick<Todo, "title" | "completed">;
 
const todo: TodoPreview = {
  title: "Clean room",
  completed: false,
};
 
todo;
 
const todo: TodoPreview
```

## Omit<Type, Keys>

通过从 `Type` 中选取所有属性然后删除键（字符串文字或字符串文字的并集）来构造一个类型。

``` ts
interface Todo {
  title: string;
  description: string;
  completed: boolean;
  createdAt: number;
}
 
type TodoPreview = Omit<Todo, "description">;
 
const todo: TodoPreview = {
  title: "Clean room",
  completed: false,
  createdAt: 1615544252770,
};
 
todo;
 
const todo: TodoPreview
 
type TodoInfo = Omit<Todo, "completed" | "createdAt">;
 
const todoInfo: TodoInfo = {
  title: "Pick up kids",
  description: "Kindergarten closes at 5pm",
};
 
todoInfo;
   
const todoInfo: TodoInfo
```

## Exclude<Type, ExcludedUnion>

通过从 `Type` 中排除所有可分配给 `ExcludedUnion` 的联合成员来构造一个类型。

``` ts
type T0 = Exclude<"a" | "b" | "c", "a">;
     
type T0 = "b" | "c"
type T1 = Exclude<"a" | "b" | "c", "a" | "b">;
     
type T1 = "c"
type T2 = Exclude<string | number | (() => void), Function>;
     
type T2 = string | number
```

## Extract<Type, Union>

通过从 `Type` 中提取可分配给 `Union` 的所有联合成员来构造一个类型。

``` ts
type T0 = Extract<"a" | "b" | "c", "a" | "f">;
     
type T0 = "a"
type T1 = Extract<string | number | (() => void), Function>;
     
type T1 = () => void
```

## NonNullable<Type>

通过从 `Type` 中排除 `null` 和 `undefined` 来构造一个类型。

``` ts
type T0 = NonNullable<string | number | undefined>;
     
type T0 = string | number
type T1 = NonNullable<string[] | null | undefined>;
     
type T1 = string[]
```

## Parameters<Type>

根据函数类型 `Type` 的参数中使用的类型构造元组类型。

``` ts
declare function f1(arg: { a: number; b: string }): void;
 
type T0 = Parameters<() => string>;
     
type T0 = []
type T1 = Parameters<(s: string) => void>;
     
type T1 = [s: string]
type T2 = Parameters<<T>(arg: T) => T>;
     
type T2 = [arg: unknown]
type T3 = Parameters<typeof f1>;
     
type T3 = [arg: {
    a: number;
    b: string;
}]
type T4 = Parameters<any>;
     
type T4 = unknown[]
type T5 = Parameters<never>;
     
type T5 = never
type T6 = Parameters<string>;
//Type 'string' does not satisfy the constraint '(...args: any) => any'.
     
type T6 = never
type T7 = Parameters<Function>;
//Type 'Function' does not satisfy the constraint '(...args: any) => any'.
  //Type 'Function' provides no match for the signature '(...args: any): any'.
     
type T7 = never
```

## ConstructorParameters<Type>

从构造函数类型的类型构造元组或数组类型。它生成一个包含所有参数类型的元组类型（或者如果 `Type` 不是函数，则类型 `never` ）。

``` ts
type T0 = ConstructorParameters<ErrorConstructor>;
     
type T0 = [message?: string]
type T1 = ConstructorParameters<FunctionConstructor>;
     
type T1 = string[]
type T2 = ConstructorParameters<RegExpConstructor>;
     
type T2 = [pattern: string | RegExp, flags?: string]
type T3 = ConstructorParameters<any>;
     
type T3 = unknown[]
 
type T4 = ConstructorParameters<Function>;
//Type 'Function' does not satisfy the constraint 'abstract new (...args: any) => any'.
  //Type 'Function' provides no match for the signature 'new (...args: any): any'.
     
type T4 = never
```

## ReturnType<Type>

构造一个由函数 `Type` 的返回类型组成的类型。

``` ts
declare function f1(): { a: number; b: string };
 
type T0 = ReturnType<() => string>;
     
type T0 = string
type T1 = ReturnType<(s: string) => void>;
     
type T1 = void
type T2 = ReturnType<<T>() => T>;
     
type T2 = unknown
type T3 = ReturnType<<T extends U, U extends number[]>() => T>;
     
type T3 = number[]
type T4 = ReturnType<typeof f1>;
     
type T4 = {
    a: number;
    b: string;
}
type T5 = ReturnType<any>;
     
type T5 = any
type T6 = ReturnType<never>;
     
type T6 = never
type T7 = ReturnType<string>;
//Type 'string' does not satisfy the constraint '(...args: any) => any'.
     
type T7 = any
type T8 = ReturnType<Function>;
//Type 'Function' does not satisfy the constraint '(...args: any) => any'.
  //Type 'Function' provides no match for the signature '(...args: any): any'.
     
type T8 = any
```

## InstanceType<Type>

构造一个由 Type 中构造函数的实例类型组成的类型。

``` ts
class C {
  x = 0;
  y = 0;
}
 
type T0 = InstanceType<typeof C>;
     
type T0 = C
type T1 = InstanceType<any>;
     
type T1 = any
type T2 = InstanceType<never>;
     
type T2 = never
type T3 = InstanceType<string>;
//Type 'string' does not satisfy the constraint 'abstract new (...args: any) => any'.
     
type T3 = any
type T4 = InstanceType<Function>;
//Type 'Function' does not satisfy the constraint 'abstract new (...args: any) => any'.
  //Type 'Function' provides no match for the signature 'new (...args: any): any'.
     
type T4 = any
```

## ThisParameterType<Type>

提取函数类型的 [this](https://www.typescriptlang.org/docs/handbook/functions.html#this-parameters) 参数的类型，如果函数类型没有 `this` 参数，则[unknown](https://www.typescriptlang.org/docs/handbook/release-notes/typescript-3-0.html#new-unknown-top-type)。

``` ts
function toHex(this: Number) {
  return this.toString(16);
}
 
function numberToString(n: ThisParameterType<typeof toHex>) {
  return toHex.apply(n);
}
```

关于function和箭头函数对this的不同绑定，查看[这里](https://www.typescriptlang.org/docs/handbook/functions.html#this)。

原文有一个非常好的总结：Arrow functions capture the `this` where the function is created rather than where it is invoked

## OmitThisParameter<Type>

从 Type 中删除 this 参数。如果 Type 没有显式声明此参数，则结果只是 Type。否则，会从 Type 创建一个没有此参数的新函数类型。泛型被擦除，只有最后一个重载签名被传播到新的函数类型中。

``` ts
function toHex(this: Number) {
  return this.toString(16);
}
 
const fiveToHex: OmitThisParameter<typeof toHex> = toHex.bind(5);
 
console.log(fiveToHex());
```

## 类型别名和接口之间的差异

类型别名和接口非常相似，在很多情况下你可以自由选择它们。`interface`的几乎所有功能都可以在`type`中使用，关键区别在于无法重新打开类型以添加新属性与始终可扩展的接口。

### Interface

扩展接口

``` ts
interface Animal {
  name: string
}

interface Bear extends Animal {
  honey: boolean
}

const bear = getBear() 
bear.name
bear.honey
```

向现有接口添加新字段

``` ts
interface Window {
  title: string
}

interface Window {
  ts: TypeScriptAPI
}

const src = 'const a = "Hello World"';
window.ts.transpileModule(src, {});
```

### Type

通过交叉点扩展类型

    type Animal = {
      name: string
    }
    
    type Bear = Animal & { 
      honey: boolean 
    }
    
    const bear = getBear();
    bear.name;
    bear.honey;

类型创建后不可更改

``` ts
type Window = {
  title: string
}

type Window = {
  ts: TypeScriptAPI
}

 // Error: Duplicate identifier 'Window'.
```

大多数情况下，您可以根据个人喜好进行选择，TypeScript 会告诉您是否需要其他类型的声明。如果您想要启发式，请使用 interface 直到您需要使用 type 中的功能。

## any和unknow类型的区别

参考[stackoverflow](https://stackoverflow.com/questions/51439843/unknown-vs-any)

``` ts
const a: any = 'a'; // OK
const b: unknown = 'b' // OK

const v1: string = a; // OK
const v2: string = b; // ERROR
const v3: string = b as string; // OK

a.trim() // OK
b.trim() // ERROR
```

unknow用于描述未知的类型。即使用前要强制做type cheke。

## Literal Types

除了一般类型的`string`和`number`外，我们还可以在类型位置引用特定的字符串和数字。

考虑这一点的一种方法是考虑 JavaScript 如何以不同的方式声明变量。`var` 和 `let` 都允许更改变量中保存的内容，而 `const` 则不允许。这反映在 TypeScript 如何为文字创建类型上。

``` ts
let changingString = "Hello World";
changingString = "Olá Mundo";
// Because `changingString` can represent any possible string, that
// is how TypeScript describes it in the type system
changingString;
      
let changingString: string
 
const constantString = "Hello World";
// Because `constantString` can only represent 1 possible string, it
// has a literal type representation
constantString;
      
const constantString: "Hello World"
```

就其本身而言，文字类型并不是很有价值：

``` ts
let x: "hello" = "hello";
// OK
x = "hello";
// ...
x = "howdy";
Type '"howdy"' is not assignable to type '"hello"'.
```

拥有一个只能有一个值的变量并没有多大用处！

但是通过将文字组合成联合，你可以表达一个更有用的概念——例如，只接受一组特定已知值的函数：

``` ts
function printText(s: string, alignment: "left" | "right" | "center") {
  // ...
}
printText("Hello, world", "left");
printText("G'day, mate", "centre");
Argument of type '"centre"' is not assignable to parameter of type '"left" | "right" | "center"'.
```

数字文字类型的工作方式相同：

``` ts
function compare(a: string, b: string): -1 | 0 | 1 {
  return a === b ? 0 : a > b ? 1 : -1;
}
```

当然，您可以将这些与非文字类型结合使用：

``` ts
interface Options {
  width: number;
}
function configure(x: Options | "auto") {
  // ...
}
configure({ width: 100 });
configure("auto");
configure("automatic");
//Argument of type '"automatic"' is not assignable to parameter of type 'Options | "auto"'.
```

还有一种文字类型：布尔文字。只有两种布尔文字类型，正如您可能猜到的，它们是 `true` 和 `false` 类型。类型 `boolean` 本身实际上只是union `true | false`的别名。

### Literal Inference

当您使用对象初始化变量时，TypeScript 假定该对象的属性稍后可能会更改值。例如，如果你写了这样的代码：

``` ts
const obj = { counter: 0 };
if (someCondition) {
  obj.counter = 1;
}
```

TypeScript 不认为将 `1` 分配给以前为 `0` 的字段是错误的。另一种说法是 `obj.counter` 必须具有类型`number`，而不是 `0`，因为类型用于确定读取和写入行为。

这同样适用于字符串：

``` ts
const req = { url: "https://example.com", method: "GET" };
handleRequest(req.url, req.method);
//Argument of type 'string' is not assignable to parameter of type '"GET" | "POST"'.
```

在上面的例子中，`req.method` 被推断为`string`，而不是`“GET”`。因为代码可以在 `req` 的创建和 `handleRequest` 的调用之间进行评估，它可以为 `req.method` 分配一个像`“GUESS”`这样的新字符串，TypeScript 认为这段代码有错误。

有两种方法可以解决这个问题。

1\.您可以通过在任一位置添加类型断言来更改推理：

``` ts
// Change 1:
const req = { url: "https://example.com", method: "GET" as "GET" };
// Change 2
handleRequest(req.url, req.method as "GET");
```

更改 1 表示“我打算让 `req.method` 始终具有文字类型`“GET”`”，从而防止之后可能将`“GUESS”`分配给该字段。更改 2 的意思是“我知道出于其他原因 `req.method` 的值为`“GET”`”。

2\.您可以使用 `as const` 将整个对象转换为类型文字：

``` ts
const req = { url: "https://example.com", method: "GET" } as const;
handleRequest(req.url, req.method);
```

`as const` 后缀的作用类似于 `const`，但对于类型系统，确保为所有属性分配文字类型，而不是更通用的版本，如string或number。