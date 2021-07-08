---
title: Sevlte学习笔记
description: ''
date: 2021-07-08
img: https://res.cloudinary.com/neroblackstone/image/upload/v1625101749/svelte_y2yhr6.png
tags:
- Svelte

---
## 动画

在上一章中，我们使用延迟转换来创建元素从一个待办事项列表移动到另一个列表时的运动错觉。

为了完成幻觉，我们还需要对没有过渡的元素应用运动。为此，我们使用 `animate` 指令。

首先，从 `svelte/animate` 导入 `flip` 函数——flip 代表[“First、Last、Invert、Play”](https://aerotwist.com/blog/flip-your-animations/)：

``` js
import { flip } from 'svelte/animate';
```

然后将其添加到 `<label>` 元素中：

``` html
<label
	in:receive="{{key: todo.id}}"
	out:send="{{key: todo.id}}"
	animate:flip
>
```

在这种情况下移动有点慢，所以我们可以添加一个`duration`参数：

``` html
<label
	in:receive="{{key: todo.id}}"
	out:send="{{key: todo.id}}"
	animate:flip="{{duration: 200}}"
>
```

> 持续时间也可以是 `d => milliseconds`函数，其中 `d` 是元素必须经过的像素数

请注意，所有过渡和动画都是使用 CSS 而不是 JavaScript 应用的，这意味着它们不会阻塞（或被主线程阻塞）。

## Action

Action本质上是元素级生命周期函数。它们对以下情况很有用：

* 与第三方库的接口
* 延迟加载的图像
* 工具提示
* 添加自定义事件处理程序

在这个应用程序中，我们想让橙色框“可平移”。它具有 `panstart`、`panmove` 和 `panend` 事件的事件处理程序，但这些不是原生 DOM 事件。我们必须自己派遣他们。首先，导入 `pannable` 函数...
  
...然后将它与元素一起使用:

``` html
<!-- App.Svelte -->
<script>
	import { spring } from 'svelte/motion';
	import { pannable } from './pannable.js';

	const coords = spring({ x: 0, y: 0 }, {
		stiffness: 0.2,
		damping: 0.4
	});

	function handlePanStart() {
		coords.stiffness = coords.damping = 1;
	}

	function handlePanMove(event) {
		coords.update($coords => ({
			x: $coords.x + event.detail.dx,
			y: $coords.y + event.detail.dy
		}));
	}

	function handlePanEnd(event) {
		coords.stiffness = 0.2;
		coords.damping = 0.4;
		coords.set({ x: 0, y: 0 });
	}
</script>

<style>
	.box {
		--width: 100px;
		--height: 100px;
		position: absolute;
		width: var(--width);
		height: var(--height);
		left: calc(50% - var(--width) / 2);
		top: calc(50% - var(--height) / 2);
		border-radius: 4px;
		background-color: #ff3e00;
		cursor: move;
	}
</style>

<div class="box"
	use:pannable
	on:panstart={handlePanStart}
	on:panmove={handlePanMove}
	on:panend={handlePanEnd}
	style="transform:
		translate({$coords.x}px,{$coords.y}px)
		rotate({$coords.x * 0.2}deg)"
></div>
```
  
打开 `pannable.js` 文件。与转换函数一样，动作函数接收一个`node`和一些可选参数，并返回一个动作对象。该对象可以有一个 `destroy` 函数，在卸载元素时调用该函数。

我们希望在用户将鼠标向下移动到元素上时触发 `panstart` 事件，在拖动元素时触发 `panmove` 事件（使用 `dx` 和 `dy` 属性显示鼠标移动的距离），以及在鼠标向上移动时触发 `panend` 事件。一种可能的实现如下所示：
  
``` js
// pannable.js
export function pannable(node) {
	let x;
	let y;

	function handleMousedown(event) {
		x = event.clientX;
		y = event.clientY;

		node.dispatchEvent(new CustomEvent('panstart', {
			detail: { x, y }
		}));

		window.addEventListener('mousemove', handleMousemove);
		window.addEventListener('mouseup', handleMouseup);
	}

	function handleMousemove(event) {
		const dx = event.clientX - x;
		const dy = event.clientY - y;
		x = event.clientX;
		y = event.clientY;

		node.dispatchEvent(new CustomEvent('panmove', {
			detail: { x, y, dx, dy }
		}));
	}

	function handleMouseup(event) {
		x = event.clientX;
		y = event.clientY;

		node.dispatchEvent(new CustomEvent('panend', {
			detail: { x, y }
		}));

		window.removeEventListener('mousemove', handleMousemove);
		window.removeEventListener('mouseup', handleMouseup);
	}

	node.addEventListener('mousedown', handleMousedown);

	return {
		destroy() {
			node.removeEventListener('mousedown', handleMousedown);
		}
	};
}
```
  
更新 `pannable` 函数并尝试移动框。

> 此实现用于演示目的——更完整的实现也会考虑触摸事件。