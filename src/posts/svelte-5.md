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
		increment: () => {},
		decrement: () => {},
		reset: () => {}
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

