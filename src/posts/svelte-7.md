---
title: Svelte学习笔记(八)
description: ''
date: 2021-07-10
img: https://res.cloudinary.com/neroblackstone/image/upload/v1625101749/svelte_y2yhr6.png
tags:
- Svelte

---
## svelte:component

一个组件可以用 `<svelte:component>` 完全改变它的类别。而不是一系列的 `if` 块...

``` html
{#if selected.color === 'red'}
	<RedThing/>
{:else if selected.color === 'green'}
	<GreenThing/>
{:else if selected.color === 'blue'}
	<BlueThing/>
{/if}
```

...我们可以有一个动态组件：

``` html
<!-- App.svelte -->
<script>
	import RedThing from './RedThing.svelte';
	import GreenThing from './GreenThing.svelte';
	import BlueThing from './BlueThing.svelte';

	const options = [
		{ color: 'red',   component: RedThing   },
		{ color: 'green', component: GreenThing },
		{ color: 'blue',  component: BlueThing  },
	];

	let selected = options[0];
</script>

<select bind:value={selected}>
	{#each options as option}
		<option value={option}>{option.color}</option>
	{/each}
</select>

<svelte:component this={selected.component}/>
```

`this` 值可以是任何组件构造函数，也可以是一个假值——如果它是假的，则不渲染任何组件。

``` html
<!-- BlueThings.svelte -->
<strong>Blue thing</strong>

<style>
	strong {
		color: blue;
	}
</style>
```

``` html
<!-- GreenThings.svelte -->
<strong>Green thing</strong>

<style>
	strong {
		color: green;
	}
</style>
```

``` html
<!-- RedThings.svelte -->
<strong>Red thing</strong>

<style>
	strong {
		color: red;
	}
</style>
```

## svelte:window

正如您可以向任何 DOM 元素添加事件侦听器一样，您可以使用 `<svelte:window>` 向 `window` 对象添加事件侦听器。

在第 11 行，添加 `keydown` 侦听器：

``` html
<script>
	let key;
	let keyCode;

	function handleKeydown(event) {
		key = event.key;
		keyCode = event.keyCode;
	}
</script>

<svelte:window on:keydown={handleKeydown}/>

<div style="text-align: center">
	{#if key}
		<kbd>{key === ' ' ? 'Space' : key}</kbd>
		<p>{keyCode}</p>
	{:else}
		<p>Focus this window and press any key</p>
	{/if}
</div>

<style>
	div {
		display: flex;
		height: 100%;
		align-items: center;
		justify-content: center;
		flex-direction: column;
	}

	kbd {
		background-color: #eee;
		border-radius: 4px;
		font-size: 6em;
		padding: 0.2em 0.5em;
		border-top: 5px solid rgba(255, 255, 255, 0.5);
		border-left: 5px solid rgba(255, 255, 255, 0.5);
		border-right: 5px solid rgba(0, 0, 0, 0.2);
		border-bottom: 5px solid rgba(0, 0, 0, 0.2);
		color: #555;
	}
</style>
```

> 与 DOM 元素一样，您可以添加诸如 `preventDefault` 之类的[事件修饰符](https://svelte.dev/tutorial/event-modifiers)。