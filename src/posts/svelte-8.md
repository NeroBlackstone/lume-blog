---
title: Svelte学习笔记（九）
description: 可编辑内容绑定，each块绑定，媒体元素
date: 2021-07-05
img: https://res.cloudinary.com/neroblackstone/image/upload/v1625101749/svelte_y2yhr6.png
tags:
- Svelte

---
## 可编辑内容绑定

具有 `contenteditable="true"` 属性的元素支持 `exContent`和 `innerHTML` 绑定：

``` html
<script>
	let html = '<p>Write some text!</p>';
</script>

<div
	contenteditable="true"
	bind:innerHTML={html}
></div>

<pre>{html}</pre>

<style>
	[contenteditable] {
		padding: 0.5em;
		border: 1px solid #eee;
		border-radius: 4px;
	}
</style>
```

## Each块绑定

您甚至可以绑定到 `each` 块内的属性。

``` html
<script>
	let todos = [
		{ done: false, text: 'finish Svelte tutorial' },
		{ done: false, text: 'build an app' },
		{ done: false, text: 'world domination' }
	];

	function add() {
		todos = todos.concat({ done: false, text: '' });
	}

	function clear() {
		todos = todos.filter(t => !t.done);
	}

	$: remaining = todos.filter(t => !t.done).length;
</script>

<h1>Todos</h1>

{#each todos as todo}
	<div class:done={todo.done}>
		<input
			type=checkbox
			bind:checked={todo.done}
		>

		<input
			placeholder="What needs to be done?"
			bind:value={todo.text}
		>
	</div>
{/each}

<p>{remaining} remaining</p>

<button on:click={add}>
	Add new
</button>

<button on:click={clear}>
	Clear completed
</button>

<style>
	.done {
		opacity: 0.4;
	}
</style>
```

> 请注意，与这些 `<input>` 元素交互将改变数组。如果您更喜欢使用不可变数据，则应避免使用这些绑定并改用事件处理程序。

## 媒体元素

`<audio>` 和 `<video>` 元素有几个可以绑定的属性。这个例子演示了其中的一些。

添加 `currentTime={time}`、`duration` 和 `paused` 绑定：

``` html
<script>
	// These values are bound to properties of the video
	let time = 0;
	let duration;
	let paused = true;

	let showControls = true;
	let showControlsTimeout;

	function handleMousemove(e) {
		// Make the controls visible, but fade out after
		// 2.5 seconds of inactivity
		clearTimeout(showControlsTimeout);
		showControlsTimeout = setTimeout(() => showControls = false, 2500);
		showControls = true;

		if (!(e.buttons & 1)) return; // mouse not down
		if (!duration) return; // video not loaded yet

		const { left, right } = this.getBoundingClientRect();
		time = duration * (e.clientX - left) / (right - left);
	}

	function handleMousedown(e) {
		// we can't rely on the built-in click event, because it fires
		// after a drag — we have to listen for clicks ourselves

		function handleMouseup() {
			if (paused) e.target.play();
			else e.target.pause();
			cancel();
		}

		function cancel() {
			e.target.removeEventListener('mouseup', handleMouseup);
		}

		e.target.addEventListener('mouseup', handleMouseup);

		setTimeout(cancel, 200);
	}

	function format(seconds) {
		if (isNaN(seconds)) return '...';

		const minutes = Math.floor(seconds / 60);
		seconds = Math.floor(seconds % 60);
		if (seconds < 10) seconds = '0' + seconds;

		return `${minutes}:${seconds}`;
	}
</script>

<h1>Caminandes: Llamigos</h1>
<p>From <a href="https://cloud.blender.org/open-projects">Blender Open Projects</a>. CC-BY</p>

<div>
	<video
		poster="https://sveltejs.github.io/assets/caminandes-llamigos.jpg"
		src="https://sveltejs.github.io/assets/caminandes-llamigos.mp4"
		on:mousemove={handleMousemove}
		on:mousedown={handleMousedown}
		bind:currentTime={time}
		bind:duration
		bind:paused>
		<track kind="captions">
	</video>

	<div class="controls" style="opacity: {duration && showControls ? 1 : 0}">
		<progress value="{(time / duration) || 0}"/>

		<div class="info">
			<span class="time">{format(time)}</span>
			<span>click anywhere to {paused ? 'play' : 'pause'} / drag to seek</span>
			<span class="time">{format(duration)}</span>
		</div>
	</div>
</div>

<style>
	div {
		position: relative;
	}

	.controls {
		position: absolute;
		top: 0;
		width: 100%;
		transition: opacity 1s;
	}

	.info {
		display: flex;
		width: 100%;
		justify-content: space-between;
	}

	span {
		padding: 0.2em 0.5em;
		color: white;
		text-shadow: 0 0 8px black;
		font-size: 1.4em;
		opacity: 0.7;
	}

	.time {
		width: 3em;
	}

	.time:last-child { text-align: right }

	progress {
		display: block;
		width: 100%;
		height: 10px;
		-webkit-appearance: none;
		appearance: none;
	}

	progress::-webkit-progress-bar {
		background-color: rgba(0,0,0,0.2);
	}

	progress::-webkit-progress-value {
		background-color: rgba(255,255,255,0.6);
	}

	video {
		width: 100%;
	}
</style>
```

> `bind:duration` is equivalent to `bind:duration={duration}`

现在，当您单击视频时，它将根据需要更新`time`, `duration`和 `paused`。这意味着我们可以使用它们来构建自定义控件。

> 通常在网络上，您可以通过侦听 `timeupdate` 事件来跟踪 `currentTime`。但是这些事件触发的频率太低，导致 UI 不稳定。 Svelte 做得更好——它使用 `requestAnimationFrame` 检查 `currentTime`。

`<audio>` 和 `<video>` 的完整绑定集如下——六个_只读_绑定...

* `duration` (readonly) — 视频的总持续时间，以秒为单位
* `buffered` (readonly) — `{start, end}`对象的数组
* `seekable` (readonly) — 同上
* `played` (readonly) — 同上
* `seeking` (readonly) — 布尔值
* `ended` (readonly) — 布尔值

...和五个_双向_绑定：

* `currentTime` — 视频中的当前点，以秒为单位
* `playbackRate`— 播放视频的速度，其中`1` 是“正常”
- `paused` — 这应该是不言自明的
- `volume`— 0 到 1 之间的值
- `muted` — 一个布尔值，其中 true 被静音

视频还具有只读 `videoWidth` 和 `videoHeight` 绑定。