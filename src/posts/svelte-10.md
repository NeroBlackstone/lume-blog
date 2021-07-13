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

## 模块上下文

在我们目前看到的所有示例中， `<script>` 块包含在每个组件实例初始化时运行的代码。对于绝大多数组件，这就是您所需要的。

偶尔，您需要在单个组件实例之外运行一些代码。例如，您可以同时播放所有五个音频播放器；如果玩一个停止所有其他人会更好。

我们可以通过声明一个 `<script context="module">` 块来做到这一点。包含在其中的代码将在模块首次评估时运行一次，而不是在实例化组件时运行。将它放在 `AudioPlayer.svelte` 的顶部：

现在组件可以在没有任何状态管理的情况下相互“交谈”：

``` html
<!-- AudioPlayer.svelte -->
<script context="module">
	let current;
</script>

<script>
	export let src;
	export let title;
	export let composer;
	export let performer;

	let audio;
	let paused = true;

	function stopOthers() {
		if (current && current !== audio) current.pause();
		current = audio;
	}
</script>

<article class:playing={!paused}>
	<h2>{title}</h2>
	<p><strong>{composer}</strong> / performed by {performer}</p>

	<audio
		bind:this={audio}
		bind:paused
		on:play={stopOthers}
		controls
		{src}
	></audio>
</article>

<style>
	article {
		margin: 0 0 1em 0; max-width: 800px;
	}
	h2, p {
		margin: 0 0 0.3em 0;
	}
	audio {
		width: 100%; margin: 0.5em 0 1em 0;
	}
	.playing {
		color: #ff3e00;
	}
</style>
```

``` html
<!-- App.svelte -->
<script>
	import AudioPlayer from './AudioPlayer.svelte';
</script>

<!-- https://musopen.org/music/9862-the-blue-danube-op-314/ -->
<AudioPlayer
	src="https://sveltejs.github.io/assets/music/strauss.mp3"
	title="The Blue Danube Waltz"
	composer="Johann Strauss"
	performer="European Archive"
/>

<!-- https://musopen.org/music/43775-the-planets-op-32/ -->
<AudioPlayer
	src="https://sveltejs.github.io/assets/music/holst.mp3"
	title="Mars, the Bringer of War"
	composer="Gustav Holst"
	performer="USAF Heritage of America Band"
/>

<!-- https://musopen.org/music/8010-3-gymnopedies/ -->
<AudioPlayer
	src="https://sveltejs.github.io/assets/music/satie.mp3"
	title="Gymnopédie no. 1"
	composer="Erik Satie"
	performer="Prodigal Procrastinator"
/>

<!-- https://musopen.org/music/2567-symphony-no-5-in-c-minor-op-67/ -->
<AudioPlayer
	src="https://sveltejs.github.io/assets/music/beethoven.mp3"
	title="Symphony no. 5 in Cm, Op. 67 - I. Allegro con brio"
	composer="Ludwig van Beethoven"
	performer="European Archive"
/>

<!-- https://musopen.org/music/43683-requiem-in-d-minor-k-626/ -->
<AudioPlayer
	src="https://sveltejs.github.io/assets/music/mozart.mp3"
	title="Requiem in D minor, K. 626 - III. Sequence - Lacrymosa"
	composer="Wolfgang Amadeus Mozart"
	performer="Markus Staab"
/>
```

