---
title: Svelte学习笔记（八）
description: ''
date: 2021-07-08
img: https://res.cloudinary.com/neroblackstone/image/upload/v1625101749/svelte_y2yhr6.png
tags:
- Svelte

---
## 入和出

一个元素可以有一个 `in` 或一个 `out` 指令，或者同时有两个指令，而不是 `transition` 指令。导入`fade`和`fly`...

``` js
import { fade, fly } from 'svelte/transition';
```

...然后用单独的`in`和`out`指令替换`transition`指令：

``` html
<p in:fly="{{ y: 200, duration: 2000 }}" out:fade>
	Flies in, fades out
</p>
```

在这种情况下，转换_不会_反转。

## 自定义css过渡

`svelte/transition` 模块有一些内置的过渡，但是创建自己的过渡非常容易。举例来说，这是`fade`过渡的来源：

``` js
function fade(node, {
	delay = 0,
	duration = 400
}) {
	const o = +getComputedStyle(node).opacity;

	return {
		delay,
		duration,
		css: t => `opacity: ${t * o}`
	};
}
```

该函数接受两个参数 - 应用转换的节点，以及传入的任何参数 - 并返回一个转换对象，该对象可以具有以下属性：

- `delay` — 转换开始前的毫秒数
- `duration` — 以毫秒为单位的转换长度
- `easing` — 一个 `p => t` easing 函数（参见补间章节）
- `css` — 一个 `(t, u) => css` 函数，其中 `u === 1 - t`
- `tick` — 一个`(t, u) => {...}` 对节点有影响的函数

`t` 值在开始或结束时为 `0`，在结束或开始时为 `1`。

大多数情况下，您应该返回 `css` 属性而不是 `tick` 属性，因为 CSS 动画会在主线程之外运行以尽可能防止卡顿。 Svelte 会“模拟”过渡并构建一个 CSS 动画，然后让它运行。

例如，`fade`过渡会生成类似这样的 CSS 动画：

``` css
0% { opacity: 0 }
10% { opacity: 0.1 }
20% { opacity: 0.2 }
/* ... */
100% { opacity: 1 }
```

不过，我们可以获得更多创意。让我们做一些真正无偿的事情：

``` html
<script>
	import { fade } from 'svelte/transition';
	import { elasticOut } from 'svelte/easing';

	let visible = true;

	function spin(node, { duration }) {
		return {
			duration,
			css: t => {
				const eased = elasticOut(t);

				return `
					transform: scale(${eased}) rotate(${eased * 1080}deg);
					color: hsl(
						${~~(t * 360)},
						${Math.min(100, 1000 - 1000 * t)}%,
						${Math.min(50, 500 - 500 * t)}%
					);`
			}
		};
	}
</script>

<label>
	<input type="checkbox" bind:checked={visible}>
	visible
</label>

{#if visible}
	<div class="centered" in:spin="{{duration: 8000}}" out:fade>
		<span>transitions!</span>
	</div>
{/if}

<style>
	.centered {
		position: absolute;
		left: 50%;
		top: 50%;
		transform: translate(-50%,-50%);
	}

	span {
		position: absolute;
		transform: translate(-50%,-50%);
		font-size: 4em;
	}
</style>
```

请记住：能力越大，责任越大。