---
title: Svelte学习笔记（三）
description: 进阶内容
date: 2021-07-09
img: https://res.cloudinary.com/neroblackstone/image/upload/v1625101749/svelte_y2yhr6.png
tags:
- Svelte

---
## Classes

与任何其他属性一样，您可以使用 JavaScript 属性指定类，如下所示：

``` html
<button
	class="{current === 'foo' ? 'selected' : ''}"
	on:click="{() => current = 'foo'}"
>foo</button>
```

这是 UI 开发中的一种常见模式，以至于 Svelte 包含一个特殊指令来简化它：

每当表达式的值为真时，`selected`的类就会添加到元素中，而当它为假时，则将其移除。

``` html
<script>
	let current = 'foo';
</script>

<button
	class:selected="{current === 'foo'}"
	on:click="{() => current = 'foo'}"
>foo</button>

<button
	class:selected="{current === 'bar'}"
	on:click="{() => current = 'bar'}"
>bar</button>

<button
	class:selected="{current === 'baz'}"
	on:click="{() => current = 'baz'}"
>baz</button>

<style>
	button {
		display: block;
	}

	.selected {
		background-color: #ff3e00;
		color: white;
	}
</style>
```

## 类指令缩写

通常，类的名称与其依赖的值的名称相同：

``` html
<div class:big={big}>
	<!-- ... -->
</div>
```

在这些情况下，我们可以使用速记形式：

``` html
<script>
	let big = false;
</script>

<label>
	<input type=checkbox bind:checked={big}>
	big
</label>

<div class:big={big}>
	some {big ? 'big' : 'small'} text
</div>

<style>
	.big {
		font-size: 4em;
	}
</style>
```

## 组件组成

就像元素可以有孩子一样......

``` html
<div>
	<p>I'm a child of the div</p>
</div>
```

...组件也可以。但是，在组件可以接受子组件之前，它需要知道将它们放在哪里。我们使用 `<slot>` 元素来做到这一点。把它放在 `Box.svelte` 里面：
  
``` html
<!-- Box.svelte -->
<div class="box">
	<slot></slot>
</div>

<style>
	.box {
		width: 300px;
		border: 1px solid #aaa;
		border-radius: 2px;
		box-shadow: 2px 2px 8px rgba(0,0,0,0.1);
		padding: 1em;
		margin: 0 0 1em 0;
	}
</style>
```

你现在可以把东西放进盒子里：

``` html
<!-- App.svelte -->
<script>
	import Box from './Box.svelte';
</script>

<Box>
	<h2>Hello!</h2>
	<p>This is a box. It can contain anything.</p>
</Box>
```

## 插槽回退

通过将内容放入 `<slot>` 元素中，组件可以为任何留空的插槽指定回退：
  
``` html
<div class="box">
	<slot>
		<em>no content was provided</em>
	</slot>
</div>
```

我们现在可以创建没有任何孩子的 <Box> 实例：
  
``` html
<Box/>
```
  
## 具名插槽
  
前面的示例包含一个默认插槽，它呈现组件的直接子级。有时您需要对位置进行更多控制，例如使用此 `<ContactCard>`。在这些情况下，我们可以使用具名插槽。

在 `ContactCard.svelte` 中，为每个插槽添加一个 name 属性：
  
``` html
<!-- ContactCard.svelte -->
<article class="contact-card">
	<h2>
		<slot name="name">
			<span class="missing">Unknown name</span>
		</slot>
	</h2>

	<div class="address">
		<slot name="address">
			<span class="missing">Unknown address</span>
		</slot>
	</div>

	<div class="email">
		<slot name="email">
			<span class="missing">Unknown email</span>
		</slot>
	</div>
</article>

<style>
	.contact-card {
		width: 300px;
		border: 1px solid #aaa;
		border-radius: 2px;
		box-shadow: 2px 2px 8px rgba(0,0,0,0.1);
		padding: 1em;
	}

	h2 {
		padding: 0 0 0.2em 0;
		margin: 0 0 1em 0;
		border-bottom: 1px solid #ff3e00
	}

	.address, .email {
		padding: 0 0 0 1.5em;
		background:  0 0 no-repeat;
		background-size: 20px 20px;
		margin: 0 0 0.5em 0;
		line-height: 1.2;
	}

	.address {
		background-image: url(tutorial/icons/map-marker.svg);
	}
	.email {
		background-image: url(tutorial/icons/email.svg);
	}
	.missing {
		color: #999;
	}
</style>
```
  
然后，在 `<ContactCard>` 组件中添加具有相应 `slot="..."` 属性的元素：

``` html
<!-- App.svelte -->
<ContactCard>
	<span slot="name">
		P. Sherman
	</span>

	<span slot="address">
		42 Wallaby Way<br>
		Sydney
	</span>
</ContactCard>  
```

## 检查插槽内容

在某些情况下，您可能希望根据父级是否为某个插槽传入内容来控制组件的某些部分。也许你在那个插槽周围有一个包装器，如果插槽是空的，你不想渲染它。或者，您可能只想在插槽存在时才应用类。您可以通过检查特殊的 `$$slots` 变量的属性来做到这一点。

`$$slots` 是一个对象，其键是父组件传入的插槽名称。如果父级将插槽留空，则 `$$slots` 将没有该插槽的条目。

请注意，本示例中 `<Project>` 的两个实例都为评论和通知点呈现了一个容器，即使只有一个有评论。我们想使用 `$$slots` 来确保我们只在父 `<App>` 为`comments`槽传递内容时才呈现这些元素。

在 `Project.svelte` 中，更新 `<article>` 上的 `class:has-discussion` 指令：

接下来，在检查 `$$slots` 的 `if` 块中包装`comments`槽及其包装 `<div>` ：

``` html
<!-- Project.svelte -->
<script>
	export let title;
	export let tasksCompleted = 0;
	export let totalTasks = 0;
</script>

<article class:has-discussion={$$slots.comments}>
	<div>
		<h2>{title}</h2>
		<p>{tasksCompleted}/{totalTasks} tasks completed</p>
	</div>
	{#if $$slots.comments}
		<div class="discussion">
			<h3>Comments</h3>
			<slot name="comments"></slot>
		</div>
	{/if}
</article>

<style>
	article {
		border: 1px #ccc solid;
		border-radius: 4px;
		position: relative;
	}

	article > div {
		padding: 1.25rem;
	}

	article.has-discussion::after {
		content: '';
		background-color: #ff3e00;
		border-radius: 10px;
		box-shadow: 0 2px 4px rgba(0,0,0,0.2);
		height: 20px;
		position: absolute;
		right: -10px;
		top: -10px;
		width: 20px;
	}

	h2,
	h3 {
		margin: 0 0 0.5rem;
	}

	h3 {
		font-size: 0.875rem;
		font-weight: 500;
		letter-spacing: 0.08em;
		text-transform: uppercase;
	}

	p {
		color: #777;
		margin: 0;
	}

	.discussion {
		background-color: #eee;
		border-top: 1px #ccc solid;
	}
</style>
```

现在，当 `<App>` 将`comments`槽留空时，评论容器和通知点将不会呈现。

``` html
<!-- Comment.svelte -->
<script>
	export let name;
	export let postedAt;

	$: avatar = `https://ui-avatars.com/api/?name=${name.replace(/ /g, '+')}&rounded=true&background=ff3e00&color=fff&bold=true`;
</script>

<article>
	<div class="header">
		<img src={avatar} alt="" height="32" width="32">
		<div class="details">
			<h4>{name}</h4>
			<time datetime={postedAt.toISOString()}>{postedAt.toLocaleDateString()}</time>
		</div>
	</div>
	<div class="body">
		<slot></slot>
	</div>
</article>

<style>
	article {
		background-color: #fff;
		border: 1px #ccc solid;
		border-radius: 4px;
		padding: 1rem;
	}

	.header {
		align-items: center;
		display: flex;
	}

	.details {
		flex: 1 1 auto;
		margin-left: 0.5rem
	}

	h4 {
		margin: 0;
	}

	time {
		color: #777;
		font-size: 0.75rem;
		text-decoration: underline;
	}

	.body {
		margin-top: 0.5rem;
	}

	.body :global(p) {
		margin: 0;
	}
</style>
```

``` html
<!-- App.svelte -->
<script>
	import Project from './Project.svelte'
	import Comment from './Comment.svelte'
</script>

<h1>
	Projects
</h1>

<ul>
	<li>
		<Project
			title="Add Typescript support"
			tasksCompleted={25}
			totalTasks={57}
		>
			<div slot="comments">
				<Comment name="Ecma Script" postedAt={new Date('2020-08-17T14:12:23')}>
					<p>Those interface tests are now passing.</p>
				</Comment>
			</div>
		</Project>
	</li>
	<li>
		<Project
			title="Update documentation"
			tasksCompleted={18}
			totalTasks={21}
		/>
	</li>
</ul>

<style>
	h1 {
		font-weight: 300;
		margin: 0 1rem;
	}

	ul {
		list-style: none;
		padding: 0;
		margin: 0.5rem;
		display: flex;
	}

	@media (max-width: 600px) {
		ul {
			flex-direction: column;
		}
	}

	li {
		padding: 0.5rem;
		flex: 1 1 50%;
		min-width: 200px;
	}
</style>
```

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

## 特殊元素

Svelte 提供了多种内置元素。第一个，`<svelte:self>`，允许组件递归地包含自身。

这对于文件夹树视图之类的东西很有用，其中文件夹可以包含其他文件夹。在 `Folder.svelte` 中，我们希望能够做到这一点......

``` html
{#if file.files}
	<Folder {...file}/>
{:else}
	<File {...file}/>
{/if}
```

...但这是不可能的，因为模块不能导入自身。相反，我们使用 <svelte:self>：

``` html
<!-- Folder.svelte -->
<script>
	import File from './File.svelte';

	export let expanded = false;
	export let name;
	export let files;

	function toggle() {
		expanded = !expanded;
	}
</script>

<span class:expanded on:click={toggle}>{name}</span>

{#if expanded}
	<ul>
		{#each files as file}
			<li>
				{#if file.files}
					<svelte:self {...file}/>
				{:else}
					<File {...file}/>
				{/if}
			</li>
		{/each}
	</ul>
{/if}

<style>
	span {
		padding: 0 0 0 1.5em;
		background: url(tutorial/icons/folder.svg) 0 0.1em no-repeat;
		background-size: 1em 1em;
		font-weight: bold;
		cursor: pointer;
	}

	.expanded {
		background-image: url(tutorial/icons/folder-open.svg);
	}

	ul {
		padding: 0.2em 0 0 0.5em;
		margin: 0 0 0 0.5em;
		list-style: none;
		border-left: 1px solid #eee;
	}

	li {
		padding: 0.2em 0;
	}
</style>
```

``` html
<!-- File.svelte -->
<script>
	export let name;
	$: type = name.slice(name.lastIndexOf('.') + 1);
</script>

<span style="background-image: url(tutorial/icons/{type}.svg)">{name}</span>

<style>
	span {
		padding: 0 0 0 1.5em;
		background: 0 0.1em no-repeat;
		background-size: 1em 1em;
	}
</style>
```

``` html
<!-- App.svelte -->
<script>
	import Folder from './Folder.svelte';

	let root = [
		{
			name: 'Important work stuff',
			files: [
				{ name: 'quarterly-results.xlsx' }
			]
		},
		{
			name: 'Animal GIFs',
			files: [
				{
					name: 'Dogs',
					files: [
						{ name: 'treadmill.gif' },
						{ name: 'rope-jumping.gif' }
					]
				},
				{
					name: 'Goats',
					files: [
						{ name: 'parkour.gif' },
						{ name: 'rampage.gif' }
					]
				},
				{ name: 'cat-roomba.gif' },
				{ name: 'duck-shuffle.gif' },
				{ name: 'monkey-on-a-pig.gif' }
			]
		},
		{ name: 'TODO.md' }
	];
</script>

<Folder name="Home" files={root} expanded/>
```

## svelte:component

一个组件可以用 `<svelte:component>` 完全改变它的类别。而不是一系列的 `if` 块...

``` html
{#if selected.color === 'red'}
	<RedThing/>
{:else if selected.color === 'green'}
	<GreenThing/>
{:else if selected.color === 'blue'}
	<BlueThing/>
{/if}
```

...我们可以有一个动态组件：

``` html
<!-- App.svelte -->
<script>
	import RedThing from './RedThing.svelte';
	import GreenThing from './GreenThing.svelte';
	import BlueThing from './BlueThing.svelte';

	const options = [
		{ color: 'red',   component: RedThing   },
		{ color: 'green', component: GreenThing },
		{ color: 'blue',  component: BlueThing  },
	];

	let selected = options[0];
</script>

<select bind:value={selected}>
	{#each options as option}
		<option value={option}>{option.color}</option>
	{/each}
</select>

<svelte:component this={selected.component}/>
```

`this` 值可以是任何组件构造函数，也可以是一个假值——如果它是假的，则不渲染任何组件。

``` html
<!-- BlueThings.svelte -->
<strong>Blue thing</strong>

<style>
	strong {
		color: blue;
	}
</style>
```

``` html
<!-- GreenThings.svelte -->
<strong>Green thing</strong>

<style>
	strong {
		color: green;
	}
</style>
```

``` html
<!-- RedThings.svelte -->
<strong>Red thing</strong>

<style>
	strong {
		color: red;
	}
</style>
```

## svelte:window

正如您可以向任何 DOM 元素添加事件侦听器一样，您可以使用 `<svelte:window>` 向 `window` 对象添加事件侦听器。

在第 11 行，添加 `keydown` 侦听器：

``` html
<script>
	let key;
	let keyCode;

	function handleKeydown(event) {
		key = event.key;
		keyCode = event.keyCode;
	}
</script>

<svelte:window on:keydown={handleKeydown}/>

<div style="text-align: center">
	{#if key}
		<kbd>{key === ' ' ? 'Space' : key}</kbd>
		<p>{keyCode}</p>
	{:else}
		<p>Focus this window and press any key</p>
	{/if}
</div>

<style>
	div {
		display: flex;
		height: 100%;
		align-items: center;
		justify-content: center;
		flex-direction: column;
	}

	kbd {
		background-color: #eee;
		border-radius: 4px;
		font-size: 6em;
		padding: 0.2em 0.5em;
		border-top: 5px solid rgba(255, 255, 255, 0.5);
		border-left: 5px solid rgba(255, 255, 255, 0.5);
		border-right: 5px solid rgba(0, 0, 0, 0.2);
		border-bottom: 5px solid rgba(0, 0, 0, 0.2);
		color: #555;
	}
</style>
```

> 与 DOM 元素一样，您可以添加诸如 `preventDefault` 之类的[事件修饰符](https://svelte.dev/tutorial/event-modifiers)。

## svelte:window绑定

``` html
<script>
	const layers = [0, 1, 2, 3, 4, 5, 6, 7, 8];

	let y;
</script>

<svelte:window bind:scrollY={y}/>

<a class="parallax-container" href="https://www.firewatchgame.com">
	{#each layers as layer}
		<img
			style="transform: translate(0,{-y * layer / (layers.length - 1)}px)"
			src="https://www.firewatchgame.com/images/parallax/parallax{layer}.png"
			alt="parallax layer {layer}"
		>
	{/each}
</a>

<div class="text">
	<span style="opacity: {1 - Math.max(0, y / 40)}">
		scroll down
	</span>

	<div class="foreground">
		You have scrolled {y} pixels
	</div>
</div>

<style>
	.parallax-container {
		position: fixed;
		width: 2400px;
		height: 712px;
		left: 50%;
		transform: translate(-50%,0);
	}

	.parallax-container img {
		position: absolute;
		top: 0;
		left: 0;
		width: 100%;
		will-change: transform;
	}

	.parallax-container img:last-child::after {
		content: '';
		position: absolute;
		width: 100%;
		height: 100%;
		background: rgb(45,10,13);
	}

	.text {
		position: relative;
		width: 100%;
		height: 300vh;
		color: rgb(220,113,43);
		text-align: center;
		padding: 4em 0.5em 0.5em 0.5em;
		box-sizing: border-box;
		pointer-events: none;
	}

	span {
		display: block;
		font-size: 1em;
		text-transform: uppercase;
		will-change: transform, opacity;
	}

	.foreground {
		position: absolute;
		top: 711px;
		left: 0;
		width: 100%;
		height: calc(100% - 712px);
		background-color: rgb(32,0,1);
		color: white;
		padding: 50vh 0 0 0;
	}

	:global(body) {
		margin: 0;
		padding: 0;
		background-color: rgb(253, 174, 51);
	}
</style>
```

您可以绑定到的属性列表如下：

* `innerWidth`
* `innerHeight`
* `outerWidth`
* `outerHeight`
* `scrollX`
* `scrollY`
* `online` — an alias for `window.navigator.onLine`

除了 `scrollX` 和 `scrollY` 之外的所有内容都是只读的。

## svelte:body

与 `<svelte:window>` 类似，`<svelte:body>` 元素允许您侦听在 `document.body` 上触发的事件。这对于不会在窗口上触发的 `mouseenter` 和 `mouseleave` 事件很有用。

将 `mouseenter` 和 `mouseleave` 处理程序添加到 `<svelte:body>` 标签：

``` html
<script>
	let hereKitty = false;

	const handleMouseenter = () => hereKitty = true;
	const handleMouseleave = () => hereKitty = false;
</script>

<svelte:body
	on:mouseenter={handleMouseenter}
	on:mouseleave={handleMouseleave}
/>

<!-- creative commons BY-NC http://www.pngall.com/kitten-png/download/7247 -->
<img
	class:curious={hereKitty}
	alt="Kitten wants to know what's going on"
	src="tutorial/kitten.png"
>

<style>
	img {
		position: absolute;
		left: 0;
		bottom: -60px;
		transform: translate(-80%, 0) rotate(-30deg);
		transform-origin: 100% 100%;
		transition: transform 0.4s;
	}

	.curious {
		transform: translate(-15%, 0) rotate(0deg);
	}

	:global(body) {
		overflow: hidden;
	}
</style>
```

## svelte:head

`<svelte:head>` 元素允许您在文档的 `<head>` 内插入元素：

``` html
<svelte:head>
	<link rel="stylesheet" href="tutorial/dark-theme.css">
</svelte:head>

<h1>Hello world!</h1>
```

> 在服务器端呈现 (SSR) 模式下， [svelte:head](svelte:head) 的内容与 HTML 的其余部分分开返回。

## svelte:options

[svelte:options](svelte:options) 元素允许您指定编译器选项。

我们将使用`immutable`选项作为示例。在这个应用程序中，`<Todo>` 组件在收到新数据时闪烁。单击其中一项会通过创建更新的 `todos` 数组来切换其`done`状态。这会导致其他 `<Todo>` 项目闪烁，即使它们最终没有对 DOM 进行任何更改。

``` js
// flash.js
export default function flash(element) {
	requestAnimationFrame(() => {
		element.style.transition = 'none';
		element.style.color = 'rgba(255,62,0,1)';
		element.style.backgroundColor = 'rgba(255,62,0,0.2)';

		setTimeout(() => {
			element.style.transition = 'color 1s, background 1s';
			element.style.color = '';
			element.style.backgroundColor = '';
		});
	});
}
```

我们可以通过告诉 `<Todo>` 组件期待不可变数据来优化它。这意味着我们承诺永远不会改变 `todo` prop，而是会在情况发生变化时创建新的 todo 对象。

``` html
<!-- App.svelte -->
<script>
	import Todo from './Todo.svelte';

	let todos = [
		{ id: 1, done: true, text: 'wash the car' },
		{ id: 2, done: false, text: 'take the dog for a walk' },
		{ id: 3, done: false, text: 'mow the lawn' }
	];

	function toggle(toggled) {
		todos = todos.map(todo => {
			if (todo === toggled) {
				// return a new object
				return {
					id: todo.id,
					text: todo.text,
					done: !todo.done
				};
			}

			// return the same object
			return todo;
		});
	}
</script>

<h2>Todos</h2>
{#each todos as todo}
	<Todo {todo} on:click={() => toggle(todo)}/>
{/each}
```

将此添加到 `Todo.svelte` 文件的顶部：

``` html
<!-- Todo.svelte -->
<svelte:options immutable={true}/>

<script>
	import { afterUpdate } from 'svelte';
	import flash from './flash.js';

	export let todo;

	let div;

	afterUpdate(() => {
		flash(div);
	});
</script>

<!-- the text will flash red whenever
     the `todo` object changes -->
<div bind:this={div} on:click>
	{todo.done ? '👍': ''} {todo.text}
</div>

<style>
	div {
		cursor: pointer;
		line-height: 1.5;
	}
</style>
```

> 如果您愿意，可以将其缩短为 `<svelte:options immutable/>`。

现在，当您通过单击来切换待办事项时，只有更新的组件会闪烁。

可以在此处设置的选项有：

* `immutable={true}` — 您从不使用可变数据，因此编译器可以进行简单的引用相等性检查以确定值是否已更改
* `immutable={false}` — 默认值。 Svelte 对于可变对象是否发生变化会更加保守
* `accessors={true}` — 为组件的 props 添加 getter 和 setter
* `accessors={false}` — 默认值
* `namespace="..."` — 将使用此组件的命名空间，最常见的是“svg”
* `tag="..."` — 将此组件编译为自定义元素时使用的名称

有关这些选项的更多信息，请参阅[ API 参考](https://svelte.dev/docs)。

## svelte:fragment

`<svelte:fragment>`元素允许您将内容放置在命名槽中，而无需将其包装在容器 DOM 元素中。这可以保持文档的流布局完整无缺。

在示例中，请注意我们如何将间隙为 `1em` 的 flex 布局应用到框上。

``` html
<!-- Box.svelte -->
<div class="box">
	<slot name="header">No header was provided</slot>
	<p>Some content between header and footer</p>
	<slot name="footer"></slot>
</div>

<style>
	.box {
		width: 300px;
		border: 1px solid #aaa;
		border-radius: 2px;
		box-shadow: 2px 2px 8px rgba(0, 0, 0, 0.1);
		padding: 1em;
		margin: 0 0 1em 0;
		
		display: flex;
		flex-direction: column;
		gap: 1em;
	}
</style>
```

但是，页脚中的内容并没有按照这种节奏间隔开，因为将其包裹在一个 div 中创建了一个新的流布局。

我们可以通过改变 App 组件中的 `<div slot="footer">` 来解决这个问题。将 `<div>` 替换为 `<svelte:fragment>`：

``` html
<!-- App.svelte -->
<script>
	import Box from './Box.svelte'
</script>

<Box>
	<svelte:fragment slot="footer">
		<p>All rights reserved.</p>
		<p>Copyright (c) 2019 Svelte Industries</p>
	</svelte:fragment>
</Box>
```

## 模块上下文

在我们目前看到的所有示例中， `<script>` 块包含在每个组件实例初始化时运行的代码。对于绝大多数组件，这就是您所需要的。

偶尔，您需要在单个组件实例之外运行一些代码。例如，您可以同时播放所有五个音频播放器；如果玩一个停止所有其他人会更好。

我们可以通过声明一个 `<script context="module">` 块来做到这一点。包含在其中的代码将在模块首次评估时运行一次，而不是在实例化组件时运行。将它放在 `AudioPlayer.svelte` 的顶部：

现在组件可以在没有任何状态管理的情况下相互“交谈”：

``` html
<!-- AudioPlayer.svelte -->
<script context="module">
	let current;
</script>

<script>
	export let src;
	export let title;
	export let composer;
	export let performer;

	let audio;
	let paused = true;

	function stopOthers() {
		if (current && current !== audio) current.pause();
		current = audio;
	}
</script>

<article class:playing={!paused}>
	<h2>{title}</h2>
	<p><strong>{composer}</strong> / performed by {performer}</p>

	<audio
		bind:this={audio}
		bind:paused
		on:play={stopOthers}
		controls
		{src}
	></audio>
</article>

<style>
	article {
		margin: 0 0 1em 0; max-width: 800px;
	}
	h2, p {
		margin: 0 0 0.3em 0;
	}
	audio {
		width: 100%; margin: 0.5em 0 1em 0;
	}
	.playing {
		color: #ff3e00;
	}
</style>
```

``` html
<!-- App.svelte -->
<script>
	import AudioPlayer from './AudioPlayer.svelte';
</script>

<!-- https://musopen.org/music/9862-the-blue-danube-op-314/ -->
<AudioPlayer
	src="https://sveltejs.github.io/assets/music/strauss.mp3"
	title="The Blue Danube Waltz"
	composer="Johann Strauss"
	performer="European Archive"
/>

<!-- https://musopen.org/music/43775-the-planets-op-32/ -->
<AudioPlayer
	src="https://sveltejs.github.io/assets/music/holst.mp3"
	title="Mars, the Bringer of War"
	composer="Gustav Holst"
	performer="USAF Heritage of America Band"
/>

<!-- https://musopen.org/music/8010-3-gymnopedies/ -->
<AudioPlayer
	src="https://sveltejs.github.io/assets/music/satie.mp3"
	title="Gymnopédie no. 1"
	composer="Erik Satie"
	performer="Prodigal Procrastinator"
/>

<!-- https://musopen.org/music/2567-symphony-no-5-in-c-minor-op-67/ -->
<AudioPlayer
	src="https://sveltejs.github.io/assets/music/beethoven.mp3"
	title="Symphony no. 5 in Cm, Op. 67 - I. Allegro con brio"
	composer="Ludwig van Beethoven"
	performer="European Archive"
/>

<!-- https://musopen.org/music/43683-requiem-in-d-minor-k-626/ -->
<AudioPlayer
	src="https://sveltejs.github.io/assets/music/mozart.mp3"
	title="Requiem in D minor, K. 626 - III. Sequence - Lacrymosa"
	composer="Wolfgang Amadeus Mozart"
	performer="Markus Staab"
/>
```

## 导出

从 `context="module"` 脚本块导出的任何内容都将成为模块本身的导出。如果我们从 `AudioPlayer.svelte` 导出一个 `stopAll` 函数...

``` html
<!-- AudioPlayer.svelte -->
<script context="module">
	const elements = new Set();

	export function stopAll() {
		elements.forEach(element => {
			element.pause();
		});
	}
</script>

<script>
onMount(() => {
		elements.add(audio);
		return () => elements.delete(audio);
	});
  
function stopOthers() {
		elements.forEach(element => {
			if (element !== audio) element.pause();
		});
	}
</script>
```

...然后我们可以从 `App.svelte` 导入它...

..并在事件处理程序中使用它：

``` html
<!-- App.svelte -->
<script>
	import AudioPlayer, { stopAll } from './AudioPlayer.svelte';
</script>

<button on:click={stopAll}>
	stop all audio
</button>
```

> 您不能有默认导出，因为组件是默认导出。

## 调试

有时，在数据流经您的应用程序时检查它会很有用。

一种方法是在标记中使用 `console.log(...)` 。但是，如果您想暂停执行，您可以使用 `{@debug ...}` 标签和您要检查的以逗号分隔的值列表：

``` html
<script>
	let user = {
		firstname: 'Ada',
		lastname: 'Lovelace'
	};
</script>

<input bind:value={user.firstname}>
<input bind:value={user.lastname}>

{@debug user}

<h1>Hello {user.firstname}!</h1>
```

如果您现在打开开发工具并开始与 `<input>` 元素交互，您将在`user`值更改时触发调试器。