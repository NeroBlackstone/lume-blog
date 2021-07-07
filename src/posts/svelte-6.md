---
title: Svelte学习笔记（七）
description: ''
date: 2021-07-07
img: https://res.cloudinary.com/neroblackstone/image/upload/v1625101749/svelte_y2yhr6.png
tags:
- Svelte

---
## spring

`spring` 函数是`tweened`的替代方法，对于经常更改的值通常更有效。

在这个例子中，我们有两个store——一个代表圆的坐标，一个代表它的大小。让我们将它们转换为soring：

两个弹簧都有默认的`stiffness`和`damping`，它们控制着弹簧的……弹性。我们可以指定我们自己的初始值：

``` html
<script>
	import { spring } from 'svelte/motion';

	let coords = spring({ x: 50, y: 50 }, {
		stiffness: 0.1,
		damping: 0.25
	});

	let size = spring(10);
</script>

<div style="position: absolute; right: 1em;">
	<label>
		<h3>stiffness ({coords.stiffness})</h3>
		<input bind:value={coords.stiffness} type="range" min="0" max="1" step="0.01">
	</label>

	<label>
		<h3>damping ({coords.damping})</h3>
		<input bind:value={coords.damping} type="range" min="0" max="1" step="0.01">
	</label>
</div>

<svg
	on:mousemove="{e => coords.set({ x: e.clientX, y: e.clientY })}"
	on:mousedown="{() => size.set(30)}"
	on:mouseup="{() => size.set(10)}"
>
	<circle cx={$coords.x} cy={$coords.y} r={$size}/>
</svg>

<style>
	svg {
		width: 100%;
		height: 100%;
	}
	circle {
		fill: #ff3e00;
	}
</style>
```

左右摆动鼠标，并尝试拖动滑块以了解它们如何影响弹簧的行为。请注意，您可以在弹簧仍在运动时调整这些值。

## 过渡

## 过渡指令

我们可以通过优雅地将元素进出 DOM 来制作更吸引人的用户界面。 Svelte 使用 `transition` 指令使这变得非常容易。

首先，从 `svelte/transition` 导入淡入淡出功能...

...然后将其添加到 `<p>` 元素：

``` html
<script>
	import { fade } from 'svelte/transition';
	let visible = true;
</script>

<label>
	<input type="checkbox" bind:checked={visible}>
	visible
</label>

{#if visible}
	<p transition:fade>
		Fades in and out
	</p>
{/if}
```

### 添加参数

转换函数可以接受参数。用`fly`替换`fade`过渡...

``` html
<script>
	import { fly } from 'svelte/transition';
	let visible = true;
</script>
```

...并将其应用到 <p> 以及一些选项：

``` html
<p transition:fly="{{ y: 200, duration: 2000 }}">
	Flies in and out
</p>
```

请注意，转换是_可逆_的——如果您在转换进行时切换复选框，它将从当前点转换，而不是从开始或结束转换。