---
title: Svelte学习笔记（十二）
description: ''
date: 2021-07-11
img: https://res.cloudinary.com/neroblackstone/image/upload/v1625101749/svelte_y2yhr6.png
tags:
- Svelte

---
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