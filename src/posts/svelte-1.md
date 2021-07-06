---
title: Svelte学习笔记（二）
description: 绑定组件实例，生命周期
date: 2021-07-06
img: https://res.cloudinary.com/neroblackstone/image/upload/v1625101749/svelte_y2yhr6.png
tags:
- Svelte

---
## 绑定元素实例

正如您可以绑定到 DOM 元素一样，您也可以绑定到组件实例本身。例如，我们可以将 `<InputField>` 的实例绑定到一个名为 `field` 的 prop，就像绑定 DOM Elements 时所做的那样:

``` html
<InputField bind:this={field} />
```

现在我们可以使用 `field` 以编程方式与这个组件交互。

``` html
<button on:click="{() => field.focus()}">
	Focus field
</button>
```

> 请注意，我们不能执行 `{field.focus}` ，因为在第一次呈现按钮并引发错误时 field 是未定义的。

## 生命周期

### onMount

每个组件都有一个_生命周期_，从创建时开始，到销毁时结束。有一些函数允许您在生命周期的关键时刻运行代码。

您最常使用的是 `onMount`，它在组件第一次渲染到 DOM 之后运行。当我们需要在渲染后与 `<canvas>` 元素进行交互时，我们之前曾短暂地遇到过它。

我们将添加一个 `onMount` 处理程序，通过网络加载一些数据：

``` html
<script>
	import { onMount } from 'svelte';

	let photos = [];

	onMount(async () => {
		const res = await fetch(`https://jsonplaceholder.typicode.com/photos?_limit=20`);
		photos = await res.json();
	});
</script>
```

> 由于服务器端渲染 (SSR)，建议将 `fetch` 放在 `onMount` 而不是 `<script>` 的顶层。除了 `onDestroy` 之外，生命周期函数不会在 SSR 期间运行，这意味着我们可以避免在组件挂载到 DOM 后获取应该延迟加载的数据。

生命周期函数必须在组件初始化时调用，以便回调绑定到组件实例——而不是（比如）在 `setTimeout` 中。

如果 `onMount` 回调返回一个函数，则该函数将在组件销毁时调用。

### onDestroy

要在组件销毁时运行代码，请使用 `onDestroy`。

例如，我们可以在组件初始化时添加一个 `setInterval` 函数，并在它不再相关时将其清理干净。这样做可以防止内存泄漏。

``` html
<script>
	import { onDestroy } from 'svelte';

	let counter = 0;
	const interval = setInterval(() => counter += 1, 1000);

	onDestroy(() => clearInterval(interval));
</script>
```

虽然在组件初始化期间调用生命周期函数很重要，但从哪里调用它们并不重要。因此，如果我们愿意，我们可以将区间逻辑抽象为 `utils.js` 中的辅助函数...

``` js
import { onDestroy } from 'svelte';

export function onInterval(callback, milliseconds) {
	const interval = setInterval(callback, milliseconds);

	onDestroy(() => {
		clearInterval(interval);
	});
}
```

...并将其导入到我们的组件中：

``` html
<script>
	import { onInterval } from './utils.js';

	let counter = 0;
	onInterval(() => counter += 1, 1000);
</script>
```

打开和关闭计时器几次，并确保计数器保持滴答作响并且 CPU 负载增加。这是由于内存泄漏，因为之前的计时器没有被删除。在解决示例之前不要忘记刷新页面。