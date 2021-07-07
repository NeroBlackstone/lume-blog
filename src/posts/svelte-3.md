---
title: Svelte学习笔记（四）
description: tick，writable store
date: 2021-07-06
img: https://res.cloudinary.com/neroblackstone/image/upload/v1625101749/svelte_y2yhr6.png
tags:
- Svelte

---
## tick

`tick` 函数与其他生命周期函数不同，您可以随时调用它，而不仅仅是在组件第一次初始化时调用。它返回一个promise，一旦将任何挂起的状态更改应用于 DOM（或者立即，如果没有挂起的状态更改），该promise就会解决。

当您在 Svelte 中更新组件状态时，它不会立即更新 DOM。相反，它会等到下一个微任务来查看是否有任何其他需要应用的更改，包括其他组件中的更改。这样做可以避免不必要的工作，并允许浏览器更有效地进行批处理。

您可以在本示例中看到这种行为。选择一个文本范围，然后按 Tab 键。因为 `<textarea>` 值改变了，当前选择被清除，光标很烦人地跳到最后。我们可以通过导入`tick`来解决这个问题...

...并在我们在 `handleKeydown` 的末尾设置 `this.selectionStart` 和 `this.selectionEnd` 之前立即运行它：

``` html
<script>
	import { tick } from 'svelte';

	let text = `Select some text and hit the tab key to toggle uppercase`;

	async function handleKeydown(event) {
		if (event.key !== 'Tab') return;

		event.preventDefault();

		const { selectionStart, selectionEnd, value } = this;
		const selection = value.slice(selectionStart, selectionEnd);

		const replacement = /[a-z]/.test(selection)
			? selection.toUpperCase()
			: selection.toLowerCase();

		text = (
			value.slice(0, selectionStart) +
			replacement +
			value.slice(selectionEnd)
		);

		await tick();
		this.selectionStart = selectionStart;
		this.selectionEnd = selectionEnd;
	}
</script>

<style>
	textarea {
		width: 100%;
		height: 200px;
	}
</style>

<textarea value={text} on:keydown={handleKeydown}></textarea>

```

## Stores 

### writable store

并非所有应用程序状态都属于应用程序的组件层次结构。有时，您的值需要由多个不相关的组件或常规 JavaScript 模块访问。

在 Svelte，我们通过商店来做到这一点。 store 只是一个带有 `subscribe` 方法的对象，它允许在 store 值发生变化时通知感兴趣的各方。在 `App.svelte` 中，`count` 是一个 store，我们在 `count.subscribe` 回调中设置了 `count_value`。

``` html
<!-- App.svelte -->
<script>
	import { count } from './stores.js';
	import Incrementer from './Incrementer.svelte';
	import Decrementer from './Decrementer.svelte';
	import Resetter from './Resetter.svelte';

	let count_value;

	const unsubscribe = count.subscribe(value => {
		count_value = value;
	});
</script>

<h1>The count is {count_value}</h1>

<Incrementer/>
<Decrementer/>
<Resetter/>
```

单击`stores.js` 选项卡以查看`count` 的定义。它是一个writable store，这意味着它除了 `subscribe` 之外还有 `set` 和 `update` 方法。

``` js
//stores.js
import { writable } from 'svelte/store';

export const count = writable(0);
```

现在转到 `Incrementer.svelte` 选项卡，以便我们可以连接 `+` 按钮：

``` html
<!-- Incrementer.svelte -->
<script>
	import { count } from './stores.js';

	function increment() {
		count.update(n => n + 1);
	}
</script>

<button on:click={increment}>
	+
</button>
```

单击 `+` 按钮现在应该更新计数。对 `Decrementer.svelte` 执行逆操作。

``` html
<!-- Decrementer.svelte -->
<script>
	import { count } from './stores.js';

	function decrement() {
		count.update(n => n - 1);
	}
</script>

<button on:click={decrement}>
	-
</button>
```

最后，在`Resetter.svelte`中，实现`reset`：

``` html
<!-- Resetter.svelte -->
<script>
	import { count } from './stores.js';

	function reset() {
		count.set(0);
	}
</script>

<button on:click={reset}>
	reset
</button>
```