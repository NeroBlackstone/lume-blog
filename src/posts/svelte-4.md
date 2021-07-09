---
title: Svelte学习笔记（五）
description: ''
date: 2021-07-09
img: https://res.cloudinary.com/neroblackstone/image/upload/v1625101749/svelte_y2yhr6.png
tags:
- Svelte

---
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