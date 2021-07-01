---
title: Svelte学习笔记（三）
description: 声明响应式变量和属性，更新数组和对象
date: 2021-07-01
img: https://res.cloudinary.com/neroblackstone/image/upload/v1625101749/svelte_y2yhr6.png
tags: []

---
## 声明响应式变量

当你的组件状态改变时，Svelte 会自动更新 DOM。通常，组件状态的某些部分需要根据其他部分（例如从`名字`和`姓氏`派生的`全名`）进行计算，并在它们发生变化时重新计算。

对于这些需求，我们有反应式声明。它们看起来像这样：

``` js
let count = 0;
$: doubled = count * 2;
```

> 如果这看起来有点陌生，请不要担心。它是有效的（如果是非常规的）JavaScript，Svelte 将其解释为“只要引用的任何值发生变化就重新运行此代码”。你会习惯的。

让我们在标记中使用 `doubled`:

``` html
<p>{count} doubled is {doubled}</p>
```

当然，您可以只在标记中写入 {count * 2} - 您不必使用反应值。当您需要多次引用它们，或者您的值依赖于其他反应值时，反应值变得特别有价值。

## 状态

我们不仅限于声明反应值——我们还可以反应性地运行任意语句。例如，我们可以在 count 发生变化时记录它的值：

``` js
$: console.log(`the count is ${count}`);
```

您可以轻松地将语句与块组合在一起：

``` js
$: {
	console.log(`the count is ${count}`);
	alert(`I SAID THE COUNT IS ${count}`);
}
```

你甚至可以把 $: 放在 if 块之类的东西前面：

``` js
$: if (count >= 10) {
	alert(`count is dangerously high!`);
	count = 9;
}
```

## 更新数组和对象

因为 Svelte 的反应性是由赋值触发的，所以使用 `push` 和 `splice` 等数组方法不会自动导致更新。例如，单击按钮不会执行任何操作。

解决这个问题的一种方法是添加一个原本多余的赋值：

``` js
function addNumber() {
	numbers.push(numbers.length + 1);
	numbers = numbers;
}
```

但有一个更惯用的解决方案：

    function addNumber() {
    	numbers = [...numbers, numbers.length + 1];
    }

您可以使用类似的模式来替换 `pop`、`shift`、`unshift` 和 `splice`。

分配给数组和对象的属性——例如`obj.foo += 1` 或 `array[i] = x` — 与赋值本身的工作方式相同。

``` js
function addNumber() {
	numbers[numbers.length] = numbers.length + 1;
}
```

一个简单的经验法则：更新变量的名称必须出现在赋值的左侧。例如这个...

``` js
const foo = obj.foo;
foo.bar = 'baz';
```

...不会更新对 obj.foo.bar 的引用，除非你再加上 obj = obj 。

## 声明属性

到目前为止，我们只处理内部状态——也就是说，这些值只能在给定的组件内访问。

``` html
<!-- App.svelte -->
<script>
	import Nested from './Nested.svelte';
</script>

<Nested answer={42}/>
```

``` html
<!-- Nested.svelte -->
<script>
	let answer;
</script>

<p>The answer is {answer}</p>
```

在任何实际应用程序中，您都需要将数据从一个组件向下传递到其子组件。为此，我们需要声明属性，通常缩写为“props”。在 Svelte 中，我们使用 `export` 关键字来做到这一点。编辑 `Nested.svelte` 组件：

``` html
<script>
	export let answer;
</script>
```

> 就像 `$:` 一样，一开始可能会觉得有点奇怪。这不是`export`在 JavaScript 模块中的正常工作方式！你会习惯的。