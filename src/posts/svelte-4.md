---
title: Svelte学习笔记（五）
description: 自动订阅
date: 2021-07-07
img: https://res.cloudinary.com/neroblackstone/image/upload/v1625101749/svelte_y2yhr6.png
tags:
- Svelte

---
## 自动订阅

上一个示例中的应用程序可以工作，但有一个微妙的错误 - 商店已订阅，但从未取消订阅。如果组件被多次实例化和销毁，这将导致_内存泄漏_。

首先在 `App.svelte` 中声明`unsubscribe`:

您现在声明了`unsubscribe`，但它仍然需要被调用，例如通过 `onDestroy` [生命周期钩子](https://svelte.dev/tutorial/ondestroy)：

``` html
<script>
	import { onDestroy } from 'svelte';
	import { count } from './stores.js';
	import Incrementer from './Incrementer.svelte';
	import Decrementer from './Decrementer.svelte';
	import Resetter from './Resetter.svelte';

	let count_value;

	const unsubscribe = count.subscribe(value => {
		count_value = value;
	});

	onDestroy(unsubscribe);
</script>

<h1>The count is {count_value}</h1>
```

但是它开始变得有点样板，特别是如果您的组件订阅了多个商店。相反，Svelte 有一个技巧——您可以通过在商店名称前加上 `$` 来引用store值：

``` html
<script>
	import { count } from './stores.js';
	import Incrementer from './Incrementer.svelte';
	import Decrementer from './Decrementer.svelte';
	import Resetter from './Resetter.svelte';
</script>

<h1>The count is {$count}</h1>
```

> 自动订阅仅适用于在组件的顶级范围内声明（或导入）的store变量。

您也不仅限于在标记中使用 `$count` — 您也可以在 `<script>` 的任何位置使用它，例如在事件处理程序或响应式声明中。

任何以 `$` 开头的名称都被假定为引用一个存储值。它实际上是一个保留字符——Svelte 将阻止您使用 `$` 前缀声明自己的变量。

## Readable store

并非所有商店都应该由引用它们的人写入。例如，您可能有一个表示鼠标位置或用户地理位置的存储，并且能够从“外部”设置这些值是没有意义的。对于这些情况，我们有readable stores。

单击到 `stores.js` 选项卡。`readable`的第一个参数是一个初始值，如果你还没有，它可以是 `null` 或 `undefined`。第二个参数是一个`start`函数，它接受一个`set`回调并返回一个`stop`函数。当商店获得第一个订阅者时调用 `start` 函数；当最后一个订阅者取消订阅时调用`stop`。

``` js
//stores.js
import { readable } from 'svelte/store';

export const time = readable(new Date(), function start(set) {
	const interval = setInterval(() => {
		set(new Date());
	}, 1000);

	return function stop() {
		clearInterval(interval);
	};
});
```

``` html
<!-- App.svelte -->
<script>
	import { time } from './stores.js';

	const formatter = new Intl.DateTimeFormat('en', {
		hour12: true,
		hour: 'numeric',
		minute: '2-digit',
		second: '2-digit'
	});
</script>

<h1>The time is {formatter.format($time)}</h1>
```

## Derived stores

您可以创建一个store，其值基于一个或多个其他store的值和`derived`。在我们之前的示例的基础上，我们可以创建一个store来获取页面打开的时间：

``` js
// stores.js
const start = new Date();

export const elapsed = derived(
	time,
	$time => Math.round(($time - start) / 1000)
);
```

``` html
<!-- App.svelte -->
<p>
	This page has been open for
	{$elapsed} {$elapsed === 1 ? 'second' : 'seconds'}
</p>
```

> 可以从多个输入派生一个store，并显式`set`一个值而不是返回它（这对于异步派生值很有用）。有关更多信息，请参阅[ API 参考](https://svelte.dev/docs#derived)。