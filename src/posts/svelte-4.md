---
title: Svelte学习笔记（五）
description: else-if块，each块与键
date: 2021-07-03
img: https://res.cloudinary.com/neroblackstone/image/upload/v1625101749/svelte_y2yhr6.png
tags:
- Svelte

---
## Else-if块

多个条件可以用`else-if` “链接”：

``` html
<script>
	let x = 7;
</script>

{#if x > 10}
	<p>{x} is greater than 10</p>
{:else if 5 > x}
	<p>{x} is less than 5</p>
{:else}
	<p>{x} is between 5 and 10</p>
{/if}
```

## Each块

如果您需要遍历数据列表，请使用 each 块：

``` html
<script>
	let cats = [
		{ id: 'J---aiyznGQ', name: 'Keyboard Cat' },
		{ id: 'z_AbfPXTKms', name: 'Maru' },
		{ id: 'OUtn3pvWmpg', name: 'Henri The Existential Cat' }
	];
</script>

<h1>The Famous Cats of YouTube</h1>

<ul>
	{#each cats as cat}
		<li><a target="_blank" href="https://www.youtube.com/watch?v={cat.id}">
			{cat.name}
		</a></li>
	{/each}
</ul>
```

> 表达式（在本例中为`cats`）可以是任何数组或类似数组的对象（即它具有`length`属性）。您可以使用`each[...iterable]` 循环遍历通用可迭代对象。

您可以获取当前索引作为第二个参数，如下所示：

``` html
{#each cats as cat, i}
	<li><a target="_blank" href="https://www.youtube.com/watch?v={cat.id}">
		{i + 1}: {cat.name}
	</a></li>
{/each}
```

如果你愿意，你可以使用解构——`each cats as { id, name }` —— 并用 `id` 和 `name` 替换 cat.id 和 cat.name。

## 键

默认情况下，当您修改 `each` 块的值时，它将在块的末尾添加和删除项目，并更新任何已更改的值。那可能不是您想要的。

``` html
<!-- Thing.svelte -->
<script>
	const emojis = {
        apple: "🍎",
        banana: "🍌",
        carrot: "🥕",
        doughnut: "🍩",
        egg: "🥚"
	}

	// the name is updated whenever the prop value changes...
	export let name;

	// ...but the "emoji" variable is fixed upon initialisation of the component
	const emoji = emojis[name];
</script>

<p>
	<span>The emoji for { name } is { emoji }</span>
</p>

<style>
	p {
		margin: 0.8em 0;
	}
	span {
		display: inline-block;
		padding: 0.2em 1em 0.3em;
		text-align: center;
		border-radius: 0.2em;
		background-color: #FFDFD3;
	}
</style>
```

``` html
<!-- App.svelte -->
<script>
	import Thing from './Thing.svelte';

	let things = [
		{ id: 1, name: 'apple' },
		{ id: 2, name: 'banana' },
		{ id: 3, name: 'carrot' },
		{ id: 4, name: 'doughnut' },
		{ id: 5, name: 'egg' },
	];

	function handleClick() {
		things = things.slice(1);
	}
</script>

<button on:click={handleClick}>
	Remove first thing
</button>

{#each things as thing}
	<Thing name={thing.name}/>
{/each}
```

说明原因比解释更容易。单击“删除第一件事”按钮几次，注意会发生什么：它删除了第一个 `<Thing>` 组件，但删除了_最后_一个 DOM 节点。然后它更新其余 DOM 节点中的`name`值，但不更新表情符号。

相反，我们只想删除第一个 `<Thing>` 组件及其 DOM 节点，而让其他组件不受影响。

为此，我们为`每个`块指定一个唯一标识符（或者说“键”）：

``` html
{#each things as thing (thing.id)}
	<Thing name={thing.name}/>
{/each}
```

这里，`(thing.id)` 是_关键_，它告诉 Svelte 在组件更新时如何找出要更改的 DOM 节点。

> 您可以使用任何对象作为键，因为 Svelte 在内部使用 Map — 换句话说，您可以执行 (thing) 而不是 (thing.id)。然而，使用字符串或数字通常更安全，因为这意味着身份在没有引用相等的情况下持续存在，例如在使用来自 API 服务器的新数据进行更新时。