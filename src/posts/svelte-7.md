---
title: Svelte学习笔记（八）
description: 绑定
date: 2021-07-04
img: https://res.cloudinary.com/neroblackstone/image/upload/v1625101749/svelte_y2yhr6.png
tags:
- Svelte

---
## 绑定

作为一般规则，Svelte 中的数据流是_自上而下_的——父组件可以在子组件上设置 props，组件可以在元素上设置属性，但反过来不行。

有时打破这条规则很有用。以这个组件中的 `<input>` 元素为例——我们可以添加一个 `on:input` 事件处理程序，将 `name` 的值设置为 `event.target.value`，但这有点......样板。正如我们将看到的，使用其他表单元素时情况会更糟。

相反，我们可以使用 `bind:value` 指令：

``` html
<script>
	let name = 'world';
</script>

<input bind:value={name}>

<h1>Hello {name}!</h1>
```

这意味着不仅对 `name` 值的更改会更新输入值，而且对输入值的更改也会更新 `name`。

## 数据输入

在 DOM 中，一切都是字符串。当您处理数字输入时，这无济于事——`type="number"` 和 `type="range"`——因为这意味着您必须记住在使用之前强制输入 `input.value`。

使用 `bind:value，Svelte` 会为您处理：

``` html
<script>
	let a = 1;
	let b = 2;
</script>

<label>
	<input type=number bind:value={a} min=0 max=10>
	<input type=range bind:value={a} min=0 max=10>
</label>

<label>
	<input type=number bind:value={b} min=0 max=10>
	<input type=range bind:value={b} min=0 max=10>
</label>

<p>{a} + {b} = {a + b}</p>
```

## 复选框输入

复选框用于在状态之间切换。我们没有绑定到 input.value，而是绑定到 input.checked：

``` html
<script>
	let yes = false;
</script>

<label>
	<input type=checkbox bind:checked={yes}>
	Yes! Send me regular email spam
</label>

{#if yes}
	<p>Thank you. We will bombard your inbox and sell your personal details.</p>
{:else}
	<p>You must opt in to continue. If you're not paying, you're the product.</p>
{/if}

<button disabled={!yes}>
	Subscribe
</button>
```

## 输入组

如果您有多个与同一个值相关的输入，您可以将 `bind:group` 与 `value` 属性一起使用。同一组中的单选框是互斥的；同一组中的复选框输入形成选定值的数组。

``` html
<script>
	let scoops = 1;
	let flavours = ['Mint choc chip'];

	function join(flavours) {
		if (flavours.length === 1) return flavours[0];
		return `${flavours.slice(0, -1).join(', ')} and ${flavours[flavours.length - 1]}`;
	}
</script>

<h2>Size</h2>

<label>
	<input type=radio group={scoops} name="scoops" value={1}>
	One scoop
</label>

<label>
	<input type=radio group={scoops} name="scoops" value={2}>
	Two scoops
</label>

<label>
	<input type=radio group={scoops} name="scoops" value={3}>
	Three scoops
</label>

<h2>Flavours</h2>

<label>
	<input type=checkbox group={flavours} name="flavours" value="Cookies and cream">
	Cookies and cream
</label>

<label>
	<input type=checkbox group={flavours} name="flavours" value="Mint choc chip">
	Mint choc chip
</label>

<label>
	<input type=checkbox group={flavours} name="flavours" value="Raspberry ripple">
	Raspberry ripple
</label>

{#if flavours.length === 0}
	<p>Please select at least one flavour</p>
{:else if flavours.length > scoops}
	<p>Can't order more flavours than scoops!</p>
{:else}
	<p>
		You ordered {scoops} {scoops === 1 ? 'scoop' : 'scoops'}
		of {join(flavours)}
	</p>
{/if}
```

将 `bind:group` 添加到每个输入：

``` html
<input type=radio bind:group={scoops} name="scoops" value={1}>
```

在这种情况下，我们可以通过将复选框输入移动到 `each` 块中来使代码更简单。首先，在 `<script>` 块中添加一个`menu`变量...

``` js
let menu = [
	'Cookies and cream',
	'Mint choc chip',
	'Raspberry ripple'
];
```

...然后替换第二部分：

``` html
<h2>Flavours</h2>

{#each menu as flavour}
	<label>
		<input type=checkbox bind:group={flavours} name="flavours" value={flavour}>
		{flavour}
	</label>
{/each}
```

现在可以轻松地扩展我们的冰淇淋菜单。