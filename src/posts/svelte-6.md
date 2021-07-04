---
title: Svelte学习笔记（七）
description: 组件事件，事件转发，DOM事件转发
date: 2021-07-04
img: https://res.cloudinary.com/neroblackstone/image/upload/v1625101749/svelte_y2yhr6.png
tags:
- Svelte

---
## 组件事件

组件还可以调度事件。为此，他们必须创建一个事件调度程序。更新 `Inner.svelte`:

``` html
<!-- Inner.svelte -->
<script>
	import { createEventDispatcher } from 'svelte';

	const dispatch = createEventDispatcher();

	function sayHello() {
		dispatch('message', {
			text: 'Hello!'
		});
	}
</script>

<button on:click={sayHello}>
	Click to say hello
</button>
```

``` html
<!-- App.svelte -->
<script>
	import Inner from './Inner.svelte';

	function handleMessage(event) {
		alert(event.detail.text);
	}
</script>

<Inner on:message={handleMessage}/>
```

> `createEventDispatcher` 必须在组件第一次实例化时调用——你不能稍后在内部执行它，例如一个 `setTimeout` 回调。这将连接`dispatch`到组件实例。

请注意，由于 `on:message` 指令，`App` 组件正在侦听`Inner`组件发送的消息。该指令是一个以 `on:` 为前缀的属性，后跟我们正在调度的事件名称（在本例中为`message`）。

如果没有这个属性，消息仍然会被分派，但应用程序不会对它做出反应。您可以尝试删除 `on:message` 属性并再次按下按钮。

> 您也可以尝试将事件名称更改为其他名称。例如，将 `Inner.svelte` 中的 `dispatch('message')` 更改为 `dispatch('myevent')` 并将 `App.svelte` 组件中的属性名称从 `on:message` 更改为 `on:myevent`。

## 事件转发

与 DOM 事件不同，组件事件不会冒泡。如果要监听某个深度嵌套组件上的事件，则中间组件必须_转发_该事件。

在本例中，我们有与[前一章](https://svelte.dev/tutorial/component-events)相同的 `App.svelte` 和 `Inner.svelte`，但现在有一个包含 `<Inner/>` 的 `Outer.svelte` 组件。

``` html
<!-- Outer.svelte -->
<script>
	import Inner from './Inner.svelte';
	import { createEventDispatcher } from 'svelte';

	const dispatch = createEventDispatcher();

	function forward(event) {
		dispatch('message', event.detail);
	}
</script>

<Inner on:message={forward}/>
```

我们可以解决这个问题的一种方法是将 `createEventDispatcher` 添加到 `Outer.svelte`，监听消息事件，并为其创建一个处理程序：

但要编写大量代码，因此 Svelte 为我们提供了一个等效的速记 — 没有值的 `on:message` 事件指令意味着“转发所有`message`事件”。

``` html
<script>
	import Inner from './Inner.svelte';
</script>

<Inner on:message/>
```

## DOM事件转发

事件转发也适用于 DOM 事件。

我们希望在我们的 `<CustomButton>` 上获得点击通知——为此，我们只需要在 `CustomButton.svelte` 中的 `<button>` 元素上转发`click`事件：

``` html
<!-- CustomButton.svelte-->
<button on:click>
	Click me
</button>

<style>
	button {
		background: #E2E8F0;
		color: #64748B;
		border: unset;
		border-radius: 6px;
		padding: .75rem 1.5rem;
		cursor: pointer;
	}
	button:hover {
		background: #CBD5E1;
		color: #475569;
	}
	button:focus {
		background: #94A3B8;
		color: #F1F5F9;
	}
</style>
```

``` html
<!-- App.svelte -->
<script>
	import CustomButton from './CustomButton.svelte';

	function handleClick() {
		alert('Button Clicked');
	}
</script>

<CustomButton on:click={handleClick}/>
```