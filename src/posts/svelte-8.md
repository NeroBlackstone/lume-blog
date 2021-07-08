---
title: Svelte学习笔记（九）
description: ''
date: 2021-07-08
img: https://res.cloudinary.com/neroblackstone/image/upload/v1625101749/svelte_y2yhr6.png
tags:
- Svelte

---
## 自定义js变换

虽然您通常应该尽可能多地使用 CSS 进行过渡，但有些效果没有 JavaScript 就无法实现，例如打字机效果：

``` html
<script>
	let visible = false;

	function typewriter(node, { speed = 50 }) {
		const valid = (
			node.childNodes.length === 1 &&
			node.childNodes[0].nodeType === Node.TEXT_NODE
		);

		if (!valid) {
			throw new Error(`This transition only works on elements with a single text node child`);
		}

		const text = node.textContent;
		const duration = text.length * speed;

		return {
			duration,
			tick: t => {
				const i = ~~(text.length * t);
				node.textContent = text.slice(0, i);
			}
		};
	}
</script>

<label>
	<input type="checkbox" bind:checked={visible}>
	visible
</label>

{#if visible}
	<p in:typewriter>
		The quick brown fox jumps over the lazy dog
	</p>
{/if}
```

## 过渡事件

了解转换何时开始和结束会很有用。 Svelte 分派您可以像任何其他 DOM 事件一样侦听的事件：

``` html
<script>
	import { fly } from 'svelte/transition';

	let visible = true;
	let status = 'waiting...';
</script>

<p>status: {status}</p>

<label>
	<input type="checkbox" bind:checked={visible}>
	visible
</label>

{#if visible}
	<p
		transition:fly="{{ y: 200, duration: 2000 }}"
		on:introstart="{() => status = 'intro started'}"
		on:outrostart="{() => status = 'outro started'}"
		on:introend="{() => status = 'intro ended'}"
		on:outroend="{() => status = 'outro ended'}"
	>
		Flies in and out
	</p>
{/if}
```

## 本地过渡

通常，当添加或销毁任何容器块时，过渡将在元素上播放。在此处的示例中，切换整个列表的可见性也将转换应用于单个列表元素。

相反，我们希望仅在添加和删除单个项目时播放过渡 - 换句话说，当用户拖动滑块时。

我们可以通过局部转换来实现这一点，它仅在添加或删除直接父块时播放：

``` html
<script>
	import { slide } from 'svelte/transition';

	let showItems = true;
	let i = 5;
	let items = ['one', 'two', 'three', 'four', 'five', 'six', 'seven', 'eight', 'nine', 'ten'];
</script>

<label>
	<input type="checkbox" bind:checked={showItems}>
	show list
</label>

<label>
	<input type="range" bind:value={i} max=10>

</label>

{#if showItems}
	{#each items.slice(0, i) as item}
		<div transition:slide|local>
			{item}
		</div>
	{/each}
{/if}

<style>
	div {
		padding: 0.5em 0;
		border-top: 1px solid #eee;
	}
</style>
```