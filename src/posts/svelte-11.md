---
title: Svelte学习笔记（十一）
description: 导出
date: 2021-07-20
img: https://res.cloudinary.com/neroblackstone/image/upload/v1625101749/svelte_y2yhr6.png
tags:
- Svelte

---
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