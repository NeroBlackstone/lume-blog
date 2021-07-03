---
title: Svelte学习笔记（五）
description: else-if块
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