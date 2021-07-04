---
title: Svelte学习笔记（六）
description: Await块，事件，内联控制器，事件修饰符
date: 2021-07-04
img: https://res.cloudinary.com/neroblackstone/image/upload/v1625101749/svelte_y2yhr6.png
tags:
- Svelte

---
## await块

大多数 Web 应用程序必须在某些时候处理异步数据。 Svelte 可以很容易地直接在你的标记中_await_ [promises](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Guide/Using_promises)的值：

``` html
<script>
	async function getRandomNumber() {
		const res = await fetch(`tutorial/random-number`);
		const text = await res.text();

		if (res.ok) {
			return text;
		} else {
			throw new Error(text);
		}
	}

	let promise = getRandomNumber();

	function handleClick() {
		promise = getRandomNumber();
	}
</script>

<button on:click={handleClick}>
	generate random number
</button>

{#await promise}
	<p>...waiting</p>
{:then number}
	<p>The number is {number}</p>
{:catch error}
	<p style="color: red">{error.message}</p>
{/await}
```

> 仅考虑最近的 `promise`，这意味着您无需担心竞争条件。

如果你知道你的promise不能reject，你可以省略 `catch`块。如果在 promise 解决之前不想显示任何内容，也可以省略第一个块：

``` html
{#await promise then value}
	<p>the value is {value}</p>
{/await}
```

## 事件

正如我们已经简要看到的，您可以使用 `on:` 指令侦听元素上的任何事件：

``` html
<script>
	let m = { x: 0, y: 0 };

	function handleMousemove(event) {
		m.x = event.clientX;
		m.y = event.clientY;
	}
</script>

<div on:mousemove={handleMousemove}>
	The mouse position is {m.x} x {m.y}
</div>

<style>
	div { width: 100%; height: 100%; }
</style>
```

## 内联控制器

您还可以内联声明事件处理程序：

``` html
<div on:mousemove="{e => m = { x: e.clientX, y: e.clientY }}">
	The mouse position is {m.x} x {m.y}
</div>
```

> 在某些框架中，您可能会看到出于性能原因避免使用内联事件处理程序的建议，尤其是在循环内部。该建议不适用于 Svelte — 编译器将始终做正确的事情，无论您选择哪种形式。

## 事件修饰符

DOM 事件处理程序可以具有改变其行为的修饰符。例如，带有 `once` 修饰符的处理程序只会运行一次：

``` html
<script>
	function handleClick() {
		alert('no more alerts')
	}
</script>

<button on:click|once={handleClick}>
	Click me
</button>
```

完整的修饰符列表：

* `preventDefault` - 在运行处理程序之前调用 `event.preventDefault ()`。例如，对于客户端表单处理很有用。
* `stopPropagation` — 调用 event.stopPropagation()，阻止事件到达下一个元素
* `passive` — 提高触摸/滚轮事件的滚动性能（Svelte 会在安全的地方自动添加它）
* `nonpassive` — 显式设置passive：false
* `capture` — 在capture阶段而不是冒泡阶段触发处理程序（[MDN 文档](https://developer.mozilla.org/en-US/docs/Learn/JavaScript/Building_blocks/Events#Event_bubbling_and_capture)）
* `once` - 在第一次运行后删除处理程序
- `self` — 如果 event.target 是元素本身，则仅触发处理程序

您可以将修饰符链接在一起，例如`on:click|once|capture={...}`。