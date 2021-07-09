---
title: Svelte学习笔记（四）
description: ''
date: 2021-07-09
img: https://res.cloudinary.com/neroblackstone/image/upload/v1625101749/svelte_y2yhr6.png
tags:
- Svelte

---
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