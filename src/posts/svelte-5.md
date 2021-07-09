---
title: Svelte学习笔记（六）
description: ''
date: 2021-07-09
img: https://res.cloudinary.com/neroblackstone/image/upload/v1625101749/svelte_y2yhr6.png
tags:
- Svelte

---
## 插槽属性

在这个应用程序中，我们有一个 `<Hoverable>` 组件来跟踪鼠标当前是否在它上面。它需要将该数据传递回父组件，以便我们可以更新插槽内容。

为此，我们使用_slot props_。在 `Hoverable.svelte` 中，将`hovering`值传递到插槽中：

``` html
<!-- Hoverable.svelte -->
<script>
	let hovering;

	function enter() {
		hovering = true;
	}

	function leave() {
		hovering = false;
	}
</script>

<div on:mouseenter={enter} on:mouseleave={leave}>
	<slot hovering={hovering}></slot>
</div>
```

> 请记住，如果您愿意，也可以使用 `{hovering}` 速记。

然后，为了将`hovering`暴露给 `<Hoverable>` 组件的内容，我们使用 `let` 指令：

如果需要，您可以重命名变量——让我们在父组件中称其为`active`的：

``` html
<!-- App.svelte -->
<script>
	import Hoverable from './Hoverable.svelte';
</script>

<Hoverable let:hovering={active}>
	<div class:active>
		{#if active}
			<p>I am being hovered upon.</p>
		{:else}
			<p>Hover over me!</p>
		{/if}
	</div>
</Hoverable>

<Hoverable let:hovering={active}>
	<div class:active>
		{#if active}
			<p>I am being hovered upon.</p>
		{:else}
			<p>Hover over me!</p>
		{/if}
	</div>
</Hoverable>

<Hoverable let:hovering={active}>
	<div class:active>
		{#if active}
			<p>I am being hovered upon.</p>
		{:else}
			<p>Hover over me!</p>
		{/if}
	</div>
</Hoverable>

<style>
	div {
		padding: 1em;
		margin: 0 0 1em 0;
		background-color: #eee;
	}

	.active {
		background-color: #ff3e00;
		color: white;
	}
</style>
```

您可以根据需要拥有任意数量的这些组件，并且开槽的 props 将保留在声明它们的组件的本地。

> 命名插槽也可以有道具；在具有 `slot="..."` 属性的元素上使用 `let` 指令，而不是在组件本身上使用。

## Context API

上下文 API 为组件提供了一种相互“对话”的机制，无需将数据和函数作为props传递，也无需分派大量事件。这是一项高级功能，但很有用。

以这个使用 [Mapbox GL](https://docs.mapbox.com/mapbox-gl-js/overview/) 地图的示例应用为例。我们想使用 `<MapMarker>` 组件显示标记，但我们不想传递对底层 Mapbox 实例的引用作为每个组件的 prop。

上下文 API 有两部分 — `setContext` 和 `getContext`。如果组件调用 `setContext(key, context)`，则任何子组件都可以使用 `const context = getContext(key)` 检索上下文。

``` html
<!-- App.svelte -->
<script>
	import Map from './Map.svelte';
	import MapMarker from './MapMarker.svelte';
</script>

<Map lat={35} lon={-84} zoom={3.5}>
	<MapMarker lat={37.8225} lon={-122.0024} label="Svelte Body Shaping"/>
	<MapMarker lat={33.8981} lon={-118.4169} label="Svelte Barbershop & Essentials"/>
	<MapMarker lat={29.7230} lon={-95.4189} label="Svelte Waxing Studio"/>
	<MapMarker lat={28.3378} lon={-81.3966} label="Svelte 30 Nutritional Consultants"/>
	<MapMarker lat={40.6483} lon={-74.0237} label="Svelte Brands LLC"/>
	<MapMarker lat={40.6986} lon={-74.4100} label="Svelte Medical Systems"/>
</Map>
```

让我们先设置上下文。在 `Map.svelte` 中，从 `svelte` 导入 `setContext`，从 `mapbox.js` 导入 `key` 并调用 `setContext`:

``` html
<!-- Map.svelte -->
<script>
	import { onMount, setContext } from 'svelte';
	import { mapbox, key } from './mapbox.js';

	setContext(key, {
		getMap: () => map
	});

	export let lat;
	export let lon;
	export let zoom;

	let container;
	let map;

	onMount(() => {
		const link = document.createElement('link');
		link.rel = 'stylesheet';
		link.href = 'https://unpkg.com/mapbox-gl/dist/mapbox-gl.css';

		link.onload = () => {
			map = new mapbox.Map({
				container,
				style: 'mapbox://styles/mapbox/streets-v9',
				center: [lon, lat],
				zoom
			});
		};

		document.head.appendChild(link);

		return () => {
			map.remove();
			link.parentNode.removeChild(link);
		};
	});
</script>

<div bind:this={container}>
	{#if map}
		<slot></slot>
	{/if}
</div>

<style>
	div {
		width: 100%;
		height: 100%;
	}
</style>
```

另一方面，在 `MapMarker.svelte` 中，我们现在可以获得对 Mapbox 实例的引用：

``` html
<!-- MapMarker.svelte -->
<script>
	import { getContext } from 'svelte';
	import { mapbox, key } from './mapbox.js';

	const { getMap } = getContext(key);
	const map = getMap();

	export let lat;
	export let lon;
	export let label;

	const popup = new mapbox.Popup({ offset: 25 })
		.setText(label);

	const marker = new mapbox.Marker()
		.setLngLat([lon, lat])
		.setPopup(popup)
		.addTo(map);
</script>
```

> `<MapMarker>` 的更完整版本也将处理删除和道具更改，但我们在这里仅演示上下文。

## Context keys

在 `mapbox.js` 中，你会看到这一行：

``` js
//mapbox.js
import mapbox from 'mapbox-gl';

// https://docs.mapbox.com/help/glossary/access-token/
mapbox.accessToken = MAPBOX_ACCESS_TOKEN;

const key = {};

export { mapbox, key };
```

我们可以使用任何东西作为键——例如，我们可以使用 `setContext('mapbox', ...)` 。使用字符串的缺点是不同的组件库可能会意外使用同一个；使用对象字面量意味着保证键在任何情况下都不会发生冲突（因为对象只对自身具有引用相等性，即 `{} !== {}` 而 `"x" === "x"`），即使您有多个不同的上下文在多个组件层上运行。

## Contexts vs. stores

上下文和商店看起来很相似。它们的不同之处在于应用程序的任何部分都可以使用商店，而上下文仅适用于组件及其后代。如果您想使用一个组件的多个实例而不干扰其他组件的状态，这会很有帮助。

事实上，您可以将两者结合使用。由于上下文不是反应性的，随时间变化的值应表示为stores：

``` js
const { these, are, stores } = getContext(...);
```