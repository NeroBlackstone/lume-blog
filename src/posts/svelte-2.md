---
title: Svelte学习笔记（三）
description: ''
date: 2021-07-09
img: https://res.cloudinary.com/neroblackstone/image/upload/v1625101749/svelte_y2yhr6.png
tags:
- Svelte

---
## 添加参数

像过渡和动画一样，一个动作可以接受一个参数，动作函数将与它所属的元素一起被调用。

在这里，我们使用了一个`longpress`操作，只要用户按下并按住按钮一段给定的持续时间，它就会触发一个同名的事件。现在，如果您切换到 `longpress.js` 文件，您会看到它被硬编码为 500 毫秒。

我们可以更改 action 函数以接受持续时间作为第二个参数，并将该`duration`传递给 `setTimeout` 调用：

回到 `App.svelte`，我们可以将持续时间值传递给动作：

``` html
<!-- App.svelte -->
<script>
	import { longpress } from './longpress.js';

	let pressed = false;
	let duration = 2000;
</script>

<label>
	<input type=range bind:value={duration} max={2000} step={100}>
	{duration}ms
</label>

<button use:longpress={duration}
	on:longpress="{() => pressed = true}"
	on:mouseenter="{() => pressed = false}"
>press and hold</button>

{#if pressed}
	<p>congratulations, you pressed and held for {duration}ms</p>
{/if}
```

这几乎有效——该事件现在仅在 2 秒后触发。但是如果你将持续时间向下滑动，它仍然需要两秒钟。

为了改变这一点，我们可以在 `longpress.js` 中添加一个`update`方法。这将在参数更改时调用：

``` js
// longpress.js
export function longpress(node, duration) {
	let timer;
	
	const handleMousedown = () => {
		timer = setTimeout(() => {
			node.dispatchEvent(
				new CustomEvent('longpress')
			);
		}, duration);
	};
	
	const handleMouseup = () => {
		clearTimeout(timer)
	};

	node.addEventListener('mousedown', handleMousedown);
	node.addEventListener('mouseup', handleMouseup);

	return {
		update(newDuration) {
			duration = newDuration;
		},
		destroy() {
			node.removeEventListener('mousedown', handleMousedown);
			node.removeEventListener('mouseup', handleMouseup);
		}
	};
}
```

> 如果您需要将多个参数传递给一个动作，请将它们组合成一个对象，如使用：`longpress={{duration, spiciness}}` 。