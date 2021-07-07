---
title: Svelte学习笔记（六）
description: 自定义store
date: 2021-07-07
img: https://res.cloudinary.com/neroblackstone/image/upload/v1625101749/svelte_y2yhr6.png
tags:
- Svelte

---
## Custom stores

只要一个对象正确地实现了 `subscribe` 方法，它就是一个store。除此之外，任何事情都会发生。因此，使用特定于域的逻辑创建自定义store非常容易。

例如，我们之前示例中的计数存储可以包括 `increment`、`decrement` 和 `reset` 方法，并避免暴露 `set` 和 `update`：

``` js
// stores.js
import { writable } from 'svelte/store';

function createCount() {
	const { subscribe, set, update } = writable(0);

	return {
		subscribe,
		increment: () => update(n => n + 1),
		decrement: () => update(n => n - 1),
		reset: () => set(0)
	};
}

export const count = createCount();
```

``` html
<!-- App.svelte -->
<script>
	import { count } from './stores.js';
</script>

<h1>The count is {$count}</h1>

<button on:click={count.increment}>+</button>
<button on:click={count.decrement}>-</button>
<button on:click={count.reset}>reset</button>
```

## store bindings

如果 store 是可写的——即它有一个 `set`方法——你可以绑定到它的值，就像你可以绑定到本地组件状态一样。

``` js
//stores.js
import { writable, derived } from 'svelte/store';

export const name = writable('world');

export const greeting = derived(
	name,
	$name => `Hello ${$name}!`
);
```

在这个例子中，我们有一个可写的store `name`和一个派生的store `greeting`。更新 `<input>` 元素：

更改输入值现在将更新 `name` 及其所有依赖项。

我们也可以直接赋值以在组件内存储值。添加一个 <button> 元素：

``` html
    <script>
    	import { name, greeting } from './stores.js';
    </script>
    
    <h1>{$greeting}</h1>
    <input bind:value={$name}>
    
    <button on:click="{() => $name += '!'}">
    	Add exclamation mark!
    </button>
```

`$name += '!'`赋值相当于 `name.set($name + '!')`。

## 补间

### 运动

设置值并自动观察 DOM 更新很酷。知道什么更酷吗？对这些值进行补间。 Svelte 包含的工具可帮助您构建使用动画来传达更改的流畅用户界面。

让我们首先将`progress` store更改为`tweened`值：

单击按钮会使进度条以动画方式显示其新值。尽管如此，它有点机器人和不令人满意。我们需要添加一个缓动函数：

``` html
<script>
	import { tweened } from 'svelte/motion';
	import { cubicOut } from 'svelte/easing';

	const progress = tweened(0, {
		duration: 400,
		easing: cubicOut
	});
</script>

<progress value={$progress}></progress>

<button on:click="{() => progress.set(0)}">
	0%
</button>

<button on:click="{() => progress.set(0.25)}">
	25%
</button>

<button on:click="{() => progress.set(0.5)}">
	50%
</button>

<button on:click="{() => progress.set(0.75)}">
	75%
</button>

<button on:click="{() => progress.set(1)}">
	100%
</button>

<style>
	progress {
		display: block;
		width: 100%;
	}
</style>
```

> `svelte/easing` 模块包含[ Penner 缓动方程](https://web.archive.org/web/20190805215728/http://robertpenner.com/easing/)，或者您可以提供自己的 `p => t` 函数，其中 `p` 和 `t` 都是介于 0 和 1 之间的值。

可用于`tweened`的全套选项：

* `delay` — 补间开始前的毫秒数
* `duration` — 以毫秒为单位的补间持续时间，或`(from, to) => milliseconds`函数允许您（例如）指定更长的补间以实现更大的值变化
* `easing`— 一个 `p => t` 函数
* `interpolate` — 自定义`(from, to) => t => value`函数，用于在任意值之间进行插值。默认情况下，Svelte 将在数字、日期和形状相同的数组和对象之间进行插值（只要它们只包含数字和日期或其他有效的数组和对象）。如果要插值（例如）颜色字符串或变换矩阵，请提供自定义插值器

您还可以将这些选项作为第二个参数传递给 `progress.set` 和 `progress.update`，在这种情况下，它们将覆盖默认值。 `set` 和 `update` 方法都返回一个在补间完成时解析的promise。