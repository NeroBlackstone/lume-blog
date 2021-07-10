---
title: Svelte学习笔记（十）
description: ''
date: 2021-07-10
img: https://res.cloudinary.com/neroblackstone/image/upload/v1625101749/svelte_y2yhr6.png
tags:
- Svelte

---
## svelte:body

与 `<svelte:window>` 类似，`<svelte:body>` 元素允许您侦听在 `document.body` 上触发的事件。这对于不会在窗口上触发的 `mouseenter` 和 `mouseleave` 事件很有用。

将 `mouseenter` 和 `mouseleave` 处理程序添加到 `<svelte:body>` 标签：

``` html
<script>
	let hereKitty = false;

	const handleMouseenter = () => hereKitty = true;
	const handleMouseleave = () => hereKitty = false;
</script>

<svelte:body
	on:mouseenter={handleMouseenter}
	on:mouseleave={handleMouseleave}
/>

<!-- creative commons BY-NC http://www.pngall.com/kitten-png/download/7247 -->
<img
	class:curious={hereKitty}
	alt="Kitten wants to know what's going on"
	src="tutorial/kitten.png"
>

<style>
	img {
		position: absolute;
		left: 0;
		bottom: -60px;
		transform: translate(-80%, 0) rotate(-30deg);
		transform-origin: 100% 100%;
		transition: transform 0.4s;
	}

	.curious {
		transform: translate(-15%, 0) rotate(0deg);
	}

	:global(body) {
		overflow: hidden;
	}
</style>
```

## svelte:head

`<svelte:head>` 元素允许您在文档的 `<head>` 内插入元素：

``` html
<svelte:head>
	<link rel="stylesheet" href="tutorial/dark-theme.css">
</svelte:head>

<h1>Hello world!</h1>
```

> 在服务器端呈现 (SSR) 模式下， [svelte:head](svelte:head) 的内容与 HTML 的其余部分分开返回。

## svelte:options

[svelte:options](svelte:options) 元素允许您指定编译器选项。

我们将使用`immutable`选项作为示例。在这个应用程序中，`<Todo>` 组件在收到新数据时闪烁。单击其中一项会通过创建更新的 `todos` 数组来切换其`done`状态。这会导致其他 `<Todo>` 项目闪烁，即使它们最终没有对 DOM 进行任何更改。

``` js
// flash.js
export default function flash(element) {
	requestAnimationFrame(() => {
		element.style.transition = 'none';
		element.style.color = 'rgba(255,62,0,1)';
		element.style.backgroundColor = 'rgba(255,62,0,0.2)';

		setTimeout(() => {
			element.style.transition = 'color 1s, background 1s';
			element.style.color = '';
			element.style.backgroundColor = '';
		});
	});
}
```

我们可以通过告诉 `<Todo>` 组件期待不可变数据来优化它。这意味着我们承诺永远不会改变 `todo` prop，而是会在情况发生变化时创建新的 todo 对象。

``` html
<!-- App.svelte -->
<script>
	import Todo from './Todo.svelte';

	let todos = [
		{ id: 1, done: true, text: 'wash the car' },
		{ id: 2, done: false, text: 'take the dog for a walk' },
		{ id: 3, done: false, text: 'mow the lawn' }
	];

	function toggle(toggled) {
		todos = todos.map(todo => {
			if (todo === toggled) {
				// return a new object
				return {
					id: todo.id,
					text: todo.text,
					done: !todo.done
				};
			}

			// return the same object
			return todo;
		});
	}
</script>

<h2>Todos</h2>
{#each todos as todo}
	<Todo {todo} on:click={() => toggle(todo)}/>
{/each}
```

将此添加到 `Todo.svelte` 文件的顶部：

``` html
<!-- Todo.svelte -->
<svelte:options immutable={true}/>

<script>
	import { afterUpdate } from 'svelte';
	import flash from './flash.js';

	export let todo;

	let div;

	afterUpdate(() => {
		flash(div);
	});
</script>

<!-- the text will flash red whenever
     the `todo` object changes -->
<div bind:this={div} on:click>
	{todo.done ? '👍': ''} {todo.text}
</div>

<style>
	div {
		cursor: pointer;
		line-height: 1.5;
	}
</style>
```

> 如果您愿意，可以将其缩短为 `<svelte:options immutable/>`。

现在，当您通过单击来切换待办事项时，只有更新的组件会闪烁。

可以在此处设置的选项有：

* `immutable={true}` — 您从不使用可变数据，因此编译器可以进行简单的引用相等性检查以确定值是否已更改
* `immutable={false}` — 默认值。 Svelte 对于可变对象是否发生变化会更加保守
* `accessors={true}` — 为组件的 props 添加 getter 和 setter
* `accessors={false}` — 默认值
* `namespace="..."` — 将使用此组件的命名空间，最常见的是“svg”
* `tag="..."` — 将此组件编译为自定义元素时使用的名称

有关这些选项的更多信息，请参阅[ API 参考](https://svelte.dev/docs)。