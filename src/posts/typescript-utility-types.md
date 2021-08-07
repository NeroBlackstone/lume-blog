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

```
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