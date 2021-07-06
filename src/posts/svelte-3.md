---
title: Svelte学习笔记（四）
description: ''
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

并非所有应用程序状态都属于应用程序的组件层次结构。有时，您的值需要由多个不相关的组件或常规 JavaScript 模块访问。