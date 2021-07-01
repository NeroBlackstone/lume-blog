---
title: Svelte学习笔记（四）
description: 属性默认值，传播属性, if和else块
date: 2021-07-01
img: https://res.cloudinary.com/neroblackstone/image/upload/v1625101749/svelte_y2yhr6.png
tags:
- Svelte

---
## 默认值

我们可以轻松地在 `Nested.svelte` 中为 props 指定默认值：

``` html
<script>
	export let answer = 'a mystery';
</script>
```

如果我们现在添加第二个没有 answer 属性的组件，它将回退到默认值：

``` html
<Nested answer={42}/>
<Nested/>
```

## 传播属性

如果你有一个属性对象，你可以将它们“传播”到一个组件上，而不是指定每一个属性：

``` html
<!-- App.svelte -->
<script>
	import Info from './Info.svelte';

	const pkg = {
		name: 'svelte',
		version: 3,
		speed: 'blazing',
		website: 'https://svelte.dev'
	};
</script>

<Info name={pkg.name} version={pkg.version} speed={pkg.speed} website={pkg.website}/>
```

``` html
<!-- Info.svelte -->
<script>
	export let name;
	export let version;
	export let speed;
	export let website;
</script>

<p>
	The <code>{name}</code> package is {speed} fast.
	Download version {version} from <a href="https://www.npmjs.com/package/{name}">npm</a>
	and <a href={website}>learn more here</a>
</p>
```

``` html
<Info {...pkg}/>
```

> 相反，如果您需要引用传递给组件的所有 props，包括那些没有使用 `export` 声明的 props，您可以通过直接访问 `$$props` 来实现。通常不建议这样做，因为 Svelte 很难优化，但它在极少数情况下很有用。

## If块

HTML 没有表达逻辑的方法，如条件和循环。Svelte有。

``` html
<script>
	let user = { loggedIn: false };

	function toggle() {
		user.loggedIn = !user.loggedIn;
	}
</script>

<button on:click={toggle}>
	Log out
</button>

<button on:click={toggle}>
	Log in
</button>
```

为了有条件地渲染一些标记，我们将它包装在一个 if 块中：

``` html
{#if user.loggedIn}
	<button on:click={toggle}>
		Log out
	</button>
{/if}

{#if !user.loggedIn}
	<button on:click={toggle}>
		Log in
	</button>
{/if}
```

试试吧——更新组件，然后点击按钮。

## else块

由于这两个条件 - `if user.loggedIn` 和 `if !user.loggedIn` - 是互斥的，我们可以通过使用 `else` 块稍微简化这个组件：

``` html
{#if user.loggedIn}
	<button on:click={toggle}>
		Log out
	</button>
{:else}
	<button on:click={toggle}>
		Log in
	</button>
{/if}
```

> `#` 字符始终表示块开始标记。 `/` 字符始终表示块结束标记。 `:` 字符，如 `{:else}` 中，表示块继续标记。别担心——您已经学会了 Svelte 添加到 HTML 的几乎所有语法。