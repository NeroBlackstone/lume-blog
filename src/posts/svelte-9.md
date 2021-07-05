---
title: Svelte学习笔记（十）
description: 维度
date: 2021-07-05
img: https://res.cloudinary.com/neroblackstone/image/upload/v1625101749/svelte_y2yhr6.png
tags:
- Svelte

---
## 维度

每个块级元素都有 `clientWidth`、`clientHeight`、`offsetWidth` 和 `offsetHeight` 绑定：

``` html
<script>
	let w;
	let h;
	let size = 42;
	let text = 'edit me';
</script>

<input type=range bind:value={size}>
<input bind:value={text}>

<p>size: {w}px x {h}px</p>

<div bind:clientWidth={w} bind:clientHeight={h}>
	<span style="font-size: {size}px">{text}</span>
</div>

<style>
	input { display: block; }
	div { display: inline-block; }
	span { word-break: break-all; }
</style>
```

这些绑定是只读的——改变 `w` 和 `h` 的值不会有任何影响。

> 使用[与此类似](http://www.backalleycoder.com/2013/03/18/cross-browser-event-based-element-resize-detection/)的技术测量元素。涉及一些开销，因此不建议将其用于大量元素。
>
> `display: inline` 元素不能用这种方法测量；不能包含其他元素（例如 `<canvas>`）的元素也不能。在这些情况下，您需要改为测量包装元素。

## This

只读 `this` 绑定适用于每个元素（和组件），并允许您获取对渲染元素的引用。例如，我们可以获得对 `<canvas>` 元素的引用：

``` html
<script>
	import { onMount } from 'svelte';

	let canvas;

	onMount(() => {
		const ctx = canvas.getContext('2d');
		let frame = requestAnimationFrame(loop);

		function loop(t) {
			frame = requestAnimationFrame(loop);

			const imageData = ctx.getImageData(0, 0, canvas.width, canvas.height);

			for (let p = 0; p < imageData.data.length; p += 4) {
				const i = p / 4;
				const x = i % canvas.width;
				const y = i / canvas.height >>> 0;

				const r = 64 + (128 * x / canvas.width) + (64 * Math.sin(t / 1000));
				const g = 64 + (128 * y / canvas.height) + (64 * Math.cos(t / 1000));
				const b = 128;

				imageData.data[p + 0] = r;
				imageData.data[p + 1] = g;
				imageData.data[p + 2] = b;
				imageData.data[p + 3] = 255;
			}

			ctx.putImageData(imageData, 0, 0);
		}

		return () => {
			cancelAnimationFrame(frame);
		};
	});
</script>

<canvas
	bind:this={canvas}
	width={32}
	height={32}
></canvas>

<style>
	canvas {
		width: 100%;
		height: 100%;
		background-color: #666;
		-webkit-mask: url(svelte-logo-mask.svg) 50% 50% no-repeat;
		mask: url(svelte-logo-mask.svg) 50% 50% no-repeat;
	}
</style>
```

请注意，在组件挂载之前，`canvas` 的值将是`undefined`的，因此我们将逻辑放在 `onMount` [生命周期函数](https://svelte.dev/tutorial/onmount)中。

## 组件绑定

正如您可以绑定到 DOM 元素的属性一样，您也可以绑定到组件 props。例如，我们可以绑定到这个 `<Keypad>` 组件的 `value` 属性，就好像它是一个表单元素：

``` html
<!-- Keypad.svelte -->
<script>
	import { createEventDispatcher } from 'svelte';

	export let value = '';

	const dispatch = createEventDispatcher();

	const select = num => () => value += num;
	const clear  = () => value = '';
	const submit = () => dispatch('submit');
</script>

<div class="keypad">
	<button on:click={select(1)}>1</button>
	<button on:click={select(2)}>2</button>
	<button on:click={select(3)}>3</button>
	<button on:click={select(4)}>4</button>
	<button on:click={select(5)}>5</button>
	<button on:click={select(6)}>6</button>
	<button on:click={select(7)}>7</button>
	<button on:click={select(8)}>8</button>
	<button on:click={select(9)}>9</button>

	<button disabled={!value} on:click={clear}>clear</button>
	<button on:click={select(0)}>0</button>
	<button disabled={!value} on:click={submit}>submit</button>
</div>

<style>
	.keypad {
		display: grid;
		grid-template-columns: repeat(3, 5em);
		grid-template-rows: repeat(4, 3em);
		grid-gap: 0.5em
	}

	button {
		margin: 0
	}
</style>
```

``` html
<!-- App.svelte -->
<script>
	import Keypad from './Keypad.svelte';

	let pin;
	$: view = pin ? pin.replace(/\d(?!$)/g, '•') : 'enter your pin';

	function handleSubmit() {
		alert(`submitted ${pin}`);
	}
</script>

<h1 style="color: {pin ? '#333' : '#ccc'}">{view}</h1>

<Keypad bind:value={pin} on:submit={handleSubmit}/>
```

现在，当用户与键盘交互时，父组件中的 `pin` 值会立即更新。

> 谨慎使用组件绑定。如果您的应用程序中的数据流过多，则很难跟踪应用程序周围的数据流，尤其是在没有“单一事实来源”的情况下。