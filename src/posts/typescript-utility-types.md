---
title: TypeScript 补充学习笔记
description: 以前学过，现在补上一些零碎知识
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

``` ts
    type Animal = {
      name: string
    }
    
    type Bear = Animal & { 
      honey: boolean 
    }
    
    const bear = getBear();
    bear.name;
    bear.honey;
```
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

1. 您可以通过在任一位置添加类型断言来更改推理：

``` ts
// Change 1:
const req = { url: "https://example.com", method: "GET" as "GET" };
// Change 2
handleRequest(req.url, req.method as "GET");
```

更改 1 表示“我打算让 `req.method` 始终具有文字类型`“GET”`”，从而防止之后可能将`“GUESS”`分配给该字段。更改 2 的意思是“我知道出于其他原因 `req.method` 的值为`“GET”`”。

1. 您可以使用 `as const` 将整个对象转换为类型文字：

``` ts
const req = { url: "https://example.com", method: "GET" } as const;
handleRequest(req.url, req.method);
```

`as const` 后缀的作用类似于 `const`，但对于类型系统，确保为所有属性分配文字类型，而不是更通用的版本，如string或number。
  
## 索引签名

有时您事先并不知道类型属性的所有名称，但您确实知道值的形状。
  
在这些情况下，您可以使用索引签名来描述可能值的类型，例如：
  
``` ts
interface StringArray {
  [index: number]: string;
}
 
const myArray: StringArray = getStringArray();
const secondItem = myArray[1];
          
//const secondItem: string
```
  
上面，我们有一个 `StringArray` 接口，它有一个索引签名。此索引签名指出，当 `StringArray` 用`number`索引时，它将返回一个字符串。
  
索引签名属性类型必须是“字符串”或“数字”。

可以支持两种类型的索引器，但从数字索引器返回的类型必须是从字符串索引器返回的类型的子类型。这是因为当使用 `number` 进行索引时，JavaScript 实际上会在索引到对象之前将其转换为 `string`。这意味着使用`100`（一个`数字`）进行索引与使用`“100”`（一个`字符串`）进行索引是一样的，因此两者需要保持一致。
  
``` ts
interface Animal {
  name: string;
}
 
interface Dog extends Animal {
  breed: string;
}
 
// Error: indexing with a numeric string might get you a completely separate type of Animal!
interface NotOkay {
  [x: number]: Animal;
'number' index type 'Animal' is not assignable to 'string' index type 'Dog'.
  [x: string]: Dog;
}
```
  
虽然字符串索引签名是一种描述“字典”模式的强大方式，但它们也强制所有属性匹配它们的返回类型。这是因为字符串索引声明 `obj.property` 也可用作 `obj["property"]`。在下面的例子中，`name` 的类型与字符串索引的类型不匹配，类型检查器给出了一个错误：
  
``` ts
interface NumberDictionary {
  [index: string]: number;
 
  length: number; // ok
  name: string;
//Property 'name' of type 'string' is not assignable to 'string' index type 'number'.
}
```
  
但是，如果索引签名是属性类型的并集，则可以接受不同类型的属性：
  
``` ts
interface NumberOrStringDictionary {
  [index: string]: number | string;
  length: number; // ok, length is a number
  name: string; // ok, name is a string
}
```
  
最后，您可以将索引签名设为`readonly`，以防止对其索引进行赋值：
  
``` ts
interface ReadonlyStringArray {
  readonly [index: number]: string;
}
 
let myArray: ReadonlyStringArray = getReadOnlyStringArray();
myArray[2] = "Mallory";
//Index signature in type 'ReadonlyStringArray' only permits reading.
```
  
你不能设置 `myArray[2]` 因为索引签名是`readonly`的。
  
总结：Index Signatures出现的需求是有必要描述key未定的object的类型（而且key本身也有两个类型）。相当于定义`类型：类型`。但是由于number的key类型在js实际上会被转化为string，那么“number类型的key对应的value类型”必须为“string类型key对应的value类型“的子类型。或者是index signatures的value为union type，其他value为union type的子集。index signatures可以为只读的。
  
## 索引访问类型  

我们可以使用索引访问类型来查找另一种类型的特定属性：

``` ts
type Person = { age: number; name: string; alive: boolean };
type Age = Person["age"];
     
//type Age = number
```
  
索引类型本身就是一种类型，因此我们可以完全使用 unions、`keyof` 或其他类型：

``` ts
type I1 = Person["age" | "name"];
     
//type I1 = string | number
 
type I2 = Person[keyof Person];
     
//type I2 = string | number | boolean
 
type AliveOrName = "alive" | "name";
type I3 = Person[AliveOrName];
     
//type I3 = string | boolean
```

如果您尝试索引不存在的属性，您甚至会看到错误：
  
``` ts
type I1 = Person["alve"];
//Property 'alve' does not exist on type 'Person'.
```
  
使用任意类型进行索引的另一个示例是使用 `number` 来获取数组元素的类型。我们可以将其与 `typeof` 结合使用，以方便地捕获数组字面量的元素类型：
  
``` ts
const MyArray = [
  { name: "Alice", age: 15 },
  { name: "Bob", age: 23 },
  { name: "Eve", age: 38 },
];
 
type Person = typeof MyArray[number];
       
//type Person = {
//    name: string;
//    age: number;
//}
type Age = typeof MyArray[number]["age"];
     
//type Age = number
// Or
type Age2 = Person["age"];
      
//type Age2 = number
```

您只能在索引时使用类型，这意味着您不能使用 `const` 来进行变量引用：
  
``` ts
const key = "age";
type Age = Person[key];
//Type 'any' cannot be used as an index type.
//'key' refers to a value, but is being used as a type here. Did you mean 'typeof key'?
```
  
但是，您可以为类似的重构风格使用类型别名： 
  
``` ts
type key = "age";
type Age = Person[key];
```

总结：Indexed Access Types这种类型，主要是用来根据字面量去取object里字段的值的类型。但是要注意只能根据type去引索。
  
## Keyof 类型运算符
  
keyof 运算符采用对象类型并生成其键的字符串或数字文字联合：
  
``` ts
type Point = { x: number; y: number };
type P = keyof Point;
    
//type P = keyof Point
```
  
如果类型具有`string`或`number`索引签名，则 `keyof` 将返回这些类型：
  
``` ts
type Arrayish = { [n: number]: unknown };
type A = keyof Arrayish;
    
//type A = number
 
type Mapish = { [k: string]: boolean };
type M = keyof Mapish;
    
//type M = string | number
```
  
请注意，在此示例中，`M` 是`string | number` — 这是因为 JavaScript 对象键总是被强制转换为字符串，因此 `obj[0]` 始终与 `obj["0"]` 相同。
  
当与映射类型结合使用时，`keyof` 类型变得特别有用，我们稍后会详细了解。
  
总结：keyof这个关键字主要是用于提取object的key，如果key是已知的，那么keyof的类型就是key的literal type（数字或者字符串），但是如果key是未知的，那么提取的就是number或string类型。如果key是string类型，那么keyof会提取string和number的union type，因为js里number也会被当作string，所以实际上有string的key对应了两个可能的类型。

## 模板文字类型

模板文字类型建立在[字符串文字类型](https://www.typescriptlang.org/docs/handbook/2/everyday-types.html#literal-types)之上，并且能够通过联合扩展为多个字符串。

它们与 JavaScript 中的模板文字字符串具有相同的语法，但用于类型位置。当与具体文字类型一起使用时，模板文字通过连接内容产生新的字符串文字类型。
  
``` ts
type World = "world";
 
type Greeting = `hello ${World}`;
        
// type Greeting = "hello world"
```
  
当在插值位置使用联合时，类型是每个联合成员可以表示的每个可能的字符串文字的集合：  
  
``` ts
type EmailLocaleIDs = "welcome_email" | "email_heading";
type FooterLocaleIDs = "footer_title" | "footer_sendoff";
 
type AllLocaleIDs = `${EmailLocaleIDs | FooterLocaleIDs}_id`;
          
//type AllLocaleIDs = "welcome_email_id" | "email_heading_id" | "footer_title_id" | "footer_sendoff_id"
```
  
对于模板文字中的每个插值位置，联合交叉相乘：
  
``` ts
type AllLocaleIDs = `${EmailLocaleIDs | FooterLocaleIDs}_id`;
type Lang = "en" | "ja" | "pt";
 
type LocaleMessageIDs = `${Lang}_${AllLocaleIDs}`;
            
//type LocaleMessageIDs = "en_welcome_email_id" | "en_email_heading_id" | "en_footer_title_id" | "en_footer_sendoff_id" | "ja_welcome_email_id" | "ja_email_heading_id" | "ja_footer_title_id" | "ja_footer_sendoff_id" | "pt_welcome_email_id" | "pt_email_heading_id" | "pt_footer_title_id" | "pt_footer_sendoff_id"
```
  
我们通常建议人们对大型字符串联合使用提前生成，但这在较小的情况下很有用。
  
### 类型中的字符串联合
  
模板文字的强大之处在于根据类型中的现有字符串定义新字符串。
  
例如，JavaScript 中的一个常见模式是根据对象当前拥有的字段来扩展对象。我们将为函数提供一个类型定义，该函数增加了对 `on` 函数的支持，让您知道值何时发生变化：
  
``` ts
const person = makeWatchedObject({
  firstName: "Saoirse",
  lastName: "Ronan",
  age: 26,
});
 
person.on("firstNameChanged", (newValue) => {
  console.log(`firstName was changed to ${newValue}!`);
});
```
  
请注意，在侦听事件“firstNameChanged”，而不仅仅是“firstName”时，模板文字提供了一种在类型系统内处理此类字符串操作的方法：
  
```ts
type PropEventSource<Type> = {
    on(eventName: `${string & keyof Type}Changed`, callback: (newValue: any) => void): void;
};
 
/// Create a "watched object" with an 'on' method
/// so that you can watch for changes to properties.
declare function makeWatchedObject<Type>(obj: Type): Type & PropEventSource<Type>;
```
  
## Intersection Types

`interface`允许我们通过扩展其他类型来构建新类型。 TypeScript 提供了另一种称为intersection types的构造，主要用于组合现有的对象类型。
  
Intersection Types使用 `&` 运算符定义。
  
``` ts
interface Colorful {
  color: string;
}
interface Circle {
  radius: number;
}
 
type ColorfulCircle = Colorful & Circle;
```
  
在这里，我们将 `Colorful` 和 `Circle` 相交以产生一种新类型，该类型具有 `Colorful` 和 `Circle` 的所有成员。
  
``` ts
function draw(circle: Colorful & Circle) {
  console.log(`Color was ${circle.color}`);
  console.log(`Radius was ${circle.radius}`);
}
 
// okay
draw({ color: "blue", radius: 42 });
 
// oops
draw({ color: "red", raidus: 42 });
//Argument of type '{ color: string; raidus: number; }' is not assignable to parameter of type 'Colorful & Circle'.
//  Object literal may only specify known properties, but 'raidus' does not exist in type 'Colorful & Circle'. Did you mean to write 'radius'?
```
  
总结：intersection type的`&`符号相当于加号，只不过加的不是数字，是类型。
  
## Interfaces vs. Intersections
  
我们只是研究了两种组合相似但实际上略有不同的类型的方法。对于接口，我们可以使用 extends 子句从其他类型扩展，并且我们能够对交集做类似的事情并用类型别名命名结果。两者之间的主要区别在于如何处理冲突，而这种区别通常是您在接口和交叉类型的类型别名之间选择一个而不是另一个的主要原因之一。