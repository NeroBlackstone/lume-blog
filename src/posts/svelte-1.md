---
title: Svelte学习笔记（二）
description: 进阶特性
date: 2021-07-06
img: https://res.cloudinary.com/neroblackstone/image/upload/v1625101749/svelte_y2yhr6.png
tags:
- Svelte

---
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

## beforeUpdate和afterUpdate

beforeUpdate 函数将工作安排在 DOM 更新之前立即发生。 afterUpdate 是它的对应物，用于在 DOM 与您的数据同步后运行代码。

它们一起用于强制执行难以以纯状态驱动的方式实现的事情，例如更新元素的滚动位置。

这个 [Eliza](https://en.wikipedia.org/wiki/ELIZA) 聊天机器人使用起来很烦人，因为您必须不断滚动聊天窗口。让我们解决这个问题。

``` html
<script>
	import Eliza from 'elizabot';
	import { beforeUpdate, afterUpdate } from 'svelte';

	let div;
	let autoscroll;

	beforeUpdate(() => {
		autoscroll = div && (div.offsetHeight + div.scrollTop) > (div.scrollHeight - 20);
	});

	afterUpdate(() => {
		if (autoscroll) div.scrollTo(0, div.scrollHeight);
	});

	const eliza = new Eliza();

	let comments = [
		{ author: 'eliza', text: eliza.getInitial() }
	];

	function handleKeydown(event) {
		if (event.key === 'Enter') {
			const text = event.target.value;
			if (!text) return;

			comments = comments.concat({
				author: 'user',
				text
			});

			event.target.value = '';

			const reply = eliza.transform(text);

			setTimeout(() => {
				comments = comments.concat({
					author: 'eliza',
					text: '...',
					placeholder: true
				});

				setTimeout(() => {
					comments = comments.filter(comment => !comment.placeholder).concat({
						author: 'eliza',
						text: reply
					});
				}, 500 + Math.random() * 500);
			}, 200 + Math.random() * 200);
		}
	}
</script>

<style>
	.chat {
		display: flex;
		flex-direction: column;
		height: 100%;
		max-width: 320px;
	}

	.scrollable {
		flex: 1 1 auto;
		border-top: 1px solid #eee;
		margin: 0 0 0.5em 0;
		overflow-y: auto;
	}

	article {
		margin: 0.5em 0;
	}

	.user {
		text-align: right;
	}

	span {
		padding: 0.5em 1em;
		display: inline-block;
	}

	.eliza span {
		background-color: #eee;
		border-radius: 1em 1em 1em 0;
	}

	.user span {
		background-color: #0074D9;
		color: white;
		border-radius: 1em 1em 0 1em;
		word-break: break-all;
	}
</style>

<div class="chat">
	<h1>Eliza</h1>

	<div class="scrollable" bind:this={div}>
		{#each comments as comment}
			<article class={comment.author}>
				<span>{comment.text}</span>
			</article>
		{/each}
	</div>

	<input on:keydown={handleKeydown}>
</div>
```

请注意，`beforeUpdate` 将在组件挂载之前首先运行，因此我们需要在读取其属性之前检查 `div` 是否存在。

## tick

`tick` 函数与其他生命周期函数不同，您可以随时调用它，而不仅仅是在组件第一次初始化时调用。它返回一个promise，一旦将任何挂起的状态更改应用于 DOM（或者立即，如果没有挂起的状态更改），该promise就会解决。

当您在 Svelte 中更新组件状态时，它不会立即更新 DOM。相反，它会等到下一个微任务来查看是否有任何其他需要应用的更改，包括其他组件中的更改。这样做可以避免不必要的工作，并允许浏览器更有效地进行批处理。

您可以在本示例中看到这种行为。选择一个文本范围，然后按 Tab 键。因为 `<textarea>` 值改变了，当前选择被清除，光标很烦人地跳到最后。我们可以通过导入`tick`来解决这个问题...

...并在我们在 `handleKeydown` 的末尾设置 `this.selectionStart` 和 `this.selectionEnd` 之前立即运行它：

``` html
<script>
	import { tick } from 'svelte';

	let text = `Select some text and hit the tab key to toggle uppercase`;

	async function handleKeydown(event) {
		if (event.key !== 'Tab') return;

		event.preventDefault();

		const { selectionStart, selectionEnd, value } = this;
		const selection = value.slice(selectionStart, selectionEnd);

		const replacement = /[a-z]/.test(selection)
			? selection.toUpperCase()
			: selection.toLowerCase();

		text = (
			value.slice(0, selectionStart) +
			replacement +
			value.slice(selectionEnd)
		);

		await tick();
		this.selectionStart = selectionStart;
		this.selectionEnd = selectionEnd;
	}
</script>

<style>
	textarea {
		width: 100%;
		height: 200px;
	}
</style>

<textarea value={text} on:keydown={handleKeydown}></textarea>

```

## Stores 

### writable store

并非所有应用程序状态都属于应用程序的组件层次结构。有时，您的值需要由多个不相关的组件或常规 JavaScript 模块访问。

在 Svelte，我们通过商店来做到这一点。 store 只是一个带有 `subscribe` 方法的对象，它允许在 store 值发生变化时通知感兴趣的各方。在 `App.svelte` 中，`count` 是一个 store，我们在 `count.subscribe` 回调中设置了 `count_value`。

``` html
<!-- App.svelte -->
<script>
	import { count } from './stores.js';
	import Incrementer from './Incrementer.svelte';
	import Decrementer from './Decrementer.svelte';
	import Resetter from './Resetter.svelte';

	let count_value;

	const unsubscribe = count.subscribe(value => {
		count_value = value;
	});
</script>

<h1>The count is {count_value}</h1>

<Incrementer/>
<Decrementer/>
<Resetter/>
```

单击`stores.js` 选项卡以查看`count` 的定义。它是一个writable store，这意味着它除了 `subscribe` 之外还有 `set` 和 `update` 方法。

``` js
//stores.js
import { writable } from 'svelte/store';

export const count = writable(0);
```

现在转到 `Incrementer.svelte` 选项卡，以便我们可以连接 `+` 按钮：

``` html
<!-- Incrementer.svelte -->
<script>
	import { count } from './stores.js';

	function increment() {
		count.update(n => n + 1);
	}
</script>

<button on:click={increment}>
	+
</button>
```

单击 `+` 按钮现在应该更新计数。对 `Decrementer.svelte` 执行逆操作。

``` html
<!-- Decrementer.svelte -->
<script>
	import { count } from './stores.js';

	function decrement() {
		count.update(n => n - 1);
	}
</script>

<button on:click={decrement}>
	-
</button>
```

最后，在`Resetter.svelte`中，实现`reset`：

``` html
<!-- Resetter.svelte -->
<script>
	import { count } from './stores.js';

	function reset() {
		count.set(0);
	}
</script>

<button on:click={reset}>
	reset
</button>
```

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
		increment: () => update(n => n + 1),
		decrement: () => update(n => n - 1),
		reset: () => set(0)
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

## store bindings

如果 store 是可写的——即它有一个 `set`方法——你可以绑定到它的值，就像你可以绑定到本地组件状态一样。

``` js
//stores.js
import { writable, derived } from 'svelte/store';

export const name = writable('world');

export const greeting = derived(
	name,
	$name => `Hello ${$name}!`
);
```

在这个例子中，我们有一个可写的store `name`和一个派生的store `greeting`。更新 `<input>` 元素：

更改输入值现在将更新 `name` 及其所有依赖项。

我们也可以直接赋值以在组件内存储值。添加一个 `<button>` 元素：

``` html
    <script>
    	import { name, greeting } from './stores.js';
    </script>
    
    <h1>{$greeting}</h1>
    <input bind:value={$name}>
    
    <button on:click="{() => $name += '!'}">
    	Add exclamation mark!
    </button>
```

`$name += '!'`赋值相当于 `name.set($name + '!')`。

## 补间

### 运动

设置值并自动观察 DOM 更新很酷。知道什么更酷吗？对这些值进行补间。 Svelte 包含的工具可帮助您构建使用动画来传达更改的流畅用户界面。

让我们首先将`progress` store更改为`tweened`值：

单击按钮会使进度条以动画方式显示其新值。尽管如此，它有点机器人和不令人满意。我们需要添加一个缓动函数：

``` html
<script>
	import { tweened } from 'svelte/motion';
	import { cubicOut } from 'svelte/easing';

	const progress = tweened(0, {
		duration: 400,
		easing: cubicOut
	});
</script>

<progress value={$progress}></progress>

<button on:click="{() => progress.set(0)}">
	0%
</button>

<button on:click="{() => progress.set(0.25)}">
	25%
</button>

<button on:click="{() => progress.set(0.5)}">
	50%
</button>

<button on:click="{() => progress.set(0.75)}">
	75%
</button>

<button on:click="{() => progress.set(1)}">
	100%
</button>

<style>
	progress {
		display: block;
		width: 100%;
	}
</style>
```

> `svelte/easing` 模块包含[ Penner 缓动方程](https://web.archive.org/web/20190805215728/http://robertpenner.com/easing/)，或者您可以提供自己的 `p => t` 函数，其中 `p` 和 `t` 都是介于 0 和 1 之间的值。

可用于`tweened`的全套选项：

* `delay` — 补间开始前的毫秒数
* `duration` — 以毫秒为单位的补间持续时间，或`(from, to) => milliseconds`函数允许您（例如）指定更长的补间以实现更大的值变化
* `easing`— 一个 `p => t` 函数
* `interpolate` — 自定义`(from, to) => t => value`函数，用于在任意值之间进行插值。默认情况下，Svelte 将在数字、日期和形状相同的数组和对象之间进行插值（只要它们只包含数字和日期或其他有效的数组和对象）。如果要插值（例如）颜色字符串或变换矩阵，请提供自定义插值器

您还可以将这些选项作为第二个参数传递给 `progress.set` 和 `progress.update`，在这种情况下，它们将覆盖默认值。 `set` 和 `update` 方法都返回一个在补间完成时解析的promise。

## spring

`spring` 函数是`tweened`的替代方法，对于经常更改的值通常更有效。

在这个例子中，我们有两个store——一个代表圆的坐标，一个代表它的大小。让我们将它们转换为soring：

两个弹簧都有默认的`stiffness`和`damping`，它们控制着弹簧的……弹性。我们可以指定我们自己的初始值：

``` html
<script>
	import { spring } from 'svelte/motion';

	let coords = spring({ x: 50, y: 50 }, {
		stiffness: 0.1,
		damping: 0.25
	});

	let size = spring(10);
</script>

<div style="position: absolute; right: 1em;">
	<label>
		<h3>stiffness ({coords.stiffness})</h3>
		<input bind:value={coords.stiffness} type="range" min="0" max="1" step="0.01">
	</label>

	<label>
		<h3>damping ({coords.damping})</h3>
		<input bind:value={coords.damping} type="range" min="0" max="1" step="0.01">
	</label>
</div>

<svg
	on:mousemove="{e => coords.set({ x: e.clientX, y: e.clientY })}"
	on:mousedown="{() => size.set(30)}"
	on:mouseup="{() => size.set(10)}"
>
	<circle cx={$coords.x} cy={$coords.y} r={$size}/>
</svg>

<style>
	svg {
		width: 100%;
		height: 100%;
	}
	circle {
		fill: #ff3e00;
	}
</style>
```

左右摆动鼠标，并尝试拖动滑块以了解它们如何影响弹簧的行为。请注意，您可以在弹簧仍在运动时调整这些值。

## 过渡

## 过渡指令

我们可以通过优雅地将元素进出 DOM 来制作更吸引人的用户界面。 Svelte 使用 `transition` 指令使这变得非常容易。

首先，从 `svelte/transition` 导入淡入淡出功能...

...然后将其添加到 `<p>` 元素：

``` html
<script>
	import { fade } from 'svelte/transition';
	let visible = true;
</script>

<label>
	<input type="checkbox" bind:checked={visible}>
	visible
</label>

{#if visible}
	<p transition:fade>
		Fades in and out
	</p>
{/if}
```

### 添加参数

转换函数可以接受参数。用`fly`替换`fade`过渡...

``` html
<script>
	import { fly } from 'svelte/transition';
	let visible = true;
</script>
```

...并将其应用到 <p> 以及一些选项：

``` html
<p transition:fly="{{ y: 200, duration: 2000 }}">
	Flies in and out
</p>
```

请注意，转换是_可逆_的——如果您在转换进行时切换复选框，它将从当前点转换，而不是从开始或结束转换。

## 入和出

一个元素可以有一个 `in` 或一个 `out` 指令，或者同时有两个指令，而不是 `transition` 指令。导入`fade`和`fly`...

``` js
import { fade, fly } from 'svelte/transition';
```

...然后用单独的`in`和`out`指令替换`transition`指令：

``` html
<p in:fly="{{ y: 200, duration: 2000 }}" out:fade>
	Flies in, fades out
</p>
```

在这种情况下，转换_不会_反转。

## 自定义css过渡

`svelte/transition` 模块有一些内置的过渡，但是创建自己的过渡非常容易。举例来说，这是`fade`过渡的来源：

``` js
function fade(node, {
	delay = 0,
	duration = 400
}) {
	const o = +getComputedStyle(node).opacity;

	return {
		delay,
		duration,
		css: t => `opacity: ${t * o}`
	};
}
```

该函数接受两个参数 - 应用转换的节点，以及传入的任何参数 - 并返回一个转换对象，该对象可以具有以下属性：

- `delay` — 转换开始前的毫秒数
- `duration` — 以毫秒为单位的转换长度
- `easing` — 一个 `p => t` easing 函数（参见补间章节）
- `css` — 一个 `(t, u) => css` 函数，其中 `u === 1 - t`
- `tick` — 一个`(t, u) => {...}` 对节点有影响的函数

`t` 值在开始或结束时为 `0`，在结束或开始时为 `1`。

大多数情况下，您应该返回 `css` 属性而不是 `tick` 属性，因为 CSS 动画会在主线程之外运行以尽可能防止卡顿。 Svelte 会“模拟”过渡并构建一个 CSS 动画，然后让它运行。

例如，`fade`过渡会生成类似这样的 CSS 动画：

``` css
0% { opacity: 0 }
10% { opacity: 0.1 }
20% { opacity: 0.2 }
/* ... */
100% { opacity: 1 }
```

不过，我们可以获得更多创意。让我们做一些真正无偿的事情：

``` html
<script>
	import { fade } from 'svelte/transition';
	import { elasticOut } from 'svelte/easing';

	let visible = true;

	function spin(node, { duration }) {
		return {
			duration,
			css: t => {
				const eased = elasticOut(t);

				return `
					transform: scale(${eased}) rotate(${eased * 1080}deg);
					color: hsl(
						${~~(t * 360)},
						${Math.min(100, 1000 - 1000 * t)}%,
						${Math.min(50, 500 - 500 * t)}%
					);`
			}
		};
	}
</script>

<label>
	<input type="checkbox" bind:checked={visible}>
	visible
</label>

{#if visible}
	<div class="centered" in:spin="{{duration: 8000}}" out:fade>
		<span>transitions!</span>
	</div>
{/if}

<style>
	.centered {
		position: absolute;
		left: 50%;
		top: 50%;
		transform: translate(-50%,-50%);
	}

	span {
		position: absolute;
		transform: translate(-50%,-50%);
		font-size: 4em;
	}
</style>
```

请记住：能力越大，责任越大。

## 自定义js变换

虽然您通常应该尽可能多地使用 CSS 进行过渡，但有些效果没有 JavaScript 就无法实现，例如打字机效果：

``` html
<script>
	let visible = false;

	function typewriter(node, { speed = 50 }) {
		const valid = (
			node.childNodes.length === 1 &&
			node.childNodes[0].nodeType === Node.TEXT_NODE
		);

		if (!valid) {
			throw new Error(`This transition only works on elements with a single text node child`);
		}

		const text = node.textContent;
		const duration = text.length * speed;

		return {
			duration,
			tick: t => {
				const i = ~~(text.length * t);
				node.textContent = text.slice(0, i);
			}
		};
	}
</script>

<label>
	<input type="checkbox" bind:checked={visible}>
	visible
</label>

{#if visible}
	<p in:typewriter>
		The quick brown fox jumps over the lazy dog
	</p>
{/if}
```

## 过渡事件

了解转换何时开始和结束会很有用。 Svelte 分派您可以像任何其他 DOM 事件一样侦听的事件：

``` html
<script>
	import { fly } from 'svelte/transition';

	let visible = true;
	let status = 'waiting...';
</script>

<p>status: {status}</p>

<label>
	<input type="checkbox" bind:checked={visible}>
	visible
</label>

{#if visible}
	<p
		transition:fly="{{ y: 200, duration: 2000 }}"
		on:introstart="{() => status = 'intro started'}"
		on:outrostart="{() => status = 'outro started'}"
		on:introend="{() => status = 'intro ended'}"
		on:outroend="{() => status = 'outro ended'}"
	>
		Flies in and out
	</p>
{/if}
```

## 本地过渡

通常，当添加或销毁任何容器块时，过渡将在元素上播放。在此处的示例中，切换整个列表的可见性也将转换应用于单个列表元素。

相反，我们希望仅在添加和删除单个项目时播放过渡 - 换句话说，当用户拖动滑块时。

我们可以通过局部转换来实现这一点，它仅在添加或删除直接父块时播放：

``` html
<script>
	import { slide } from 'svelte/transition';

	let showItems = true;
	let i = 5;
	let items = ['one', 'two', 'three', 'four', 'five', 'six', 'seven', 'eight', 'nine', 'ten'];
</script>

<label>
	<input type="checkbox" bind:checked={showItems}>
	show list
</label>

<label>
	<input type="range" bind:value={i} max=10>

</label>

{#if showItems}
	{#each items.slice(0, i) as item}
		<div transition:slide|local>
			{item}
		</div>
	{/each}
{/if}

<style>
	div {
		padding: 0.5em 0;
		border-top: 1px solid #eee;
	}
</style>
```

## 延迟过渡

Svelte 过渡引擎的一个特别强大的功能是能够延迟过渡，以便它们可以在多个元素之间进行协调。

以这对待办事项列表为例，其中切换一个待办事项会将其发送到相反的列表。在现实世界中，物体不会那样表现——它们不会在另一个地方消失和重新出现，而是通过一系列中间位置移动。使用动作可以帮助用户了解您的应用程序中发生的事情。

我们可以使用 `crossfade` 函数来实现这种效果，它创建了一对称为`send`和`receive`的过渡。当一个元素被“发送”时，它会寻找一个被“接收”的相应元素，并生成一个转换，将元素转换到其对应位置并将其淡出。当一个元素被“接收”时，会发生相反的情况。如果没有对应物，则使用`fallback`转换。

找到第 65 行的 <label> 元素，并添加发送和接收转换：

对下一个 <label> 元素执行相同的操作：

``` html
<script>
	import { quintOut } from 'svelte/easing';
	import { crossfade } from 'svelte/transition';

	const [send, receive] = crossfade({
		duration: d => Math.sqrt(d * 200),

		fallback(node, params) {
			const style = getComputedStyle(node);
			const transform = style.transform === 'none' ? '' : style.transform;

			return {
				duration: 600,
				easing: quintOut,
				css: t => `
					transform: ${transform} scale(${t});
					opacity: ${t}
				`
			};
		}
	});

	let uid = 1;

	let todos = [
		{ id: uid++, done: false, description: 'write some docs' },
		{ id: uid++, done: false, description: 'start writing blog post' },
		{ id: uid++, done: true,  description: 'buy some milk' },
		{ id: uid++, done: false, description: 'mow the lawn' },
		{ id: uid++, done: false, description: 'feed the turtle' },
		{ id: uid++, done: false, description: 'fix some bugs' },
	];

	function add(input) {
		const todo = {
			id: uid++,
			done: false,
			description: input.value
		};

		todos = [todo, ...todos];
		input.value = '';
	}

	function remove(todo) {
		todos = todos.filter(t => t !== todo);
	}

	function mark(todo, done) {
		todo.done = done;
		remove(todo);
		todos = todos.concat(todo);
	}
</script>

<div class='board'>
	<input
		placeholder="what needs to be done?"
		on:keydown={e => e.key === 'Enter' && add(e.target)}
	>

	<div class='left'>
		<h2>todo</h2>
		{#each todos.filter(t => !t.done) as todo (todo.id)}
			<label
				in:receive="{{key: todo.id}}"
				out:send="{{key: todo.id}}"
			>
				<input type=checkbox on:change={() => mark(todo, true)}>
				{todo.description}
				<button on:click="{() => remove(todo)}">remove</button>
			</label>
		{/each}
	</div>

	<div class='right'>
		<h2>done</h2>
		{#each todos.filter(t => t.done) as todo (todo.id)}
			<label
				class="done"
				in:receive="{{key: todo.id}}"
				out:send="{{key: todo.id}}"
			>
				<input type=checkbox checked on:change={() => mark(todo, false)}>
				{todo.description}
				<button on:click="{() => remove(todo)}">remove</button>
			</label>
		{/each}
	</div>
</div>

<style>
	.board {
		display: grid;
		grid-template-columns: 1fr 1fr;
		grid-gap: 1em;
		max-width: 36em;
		margin: 0 auto;
	}

	.board > input {
		font-size: 1.4em;
		grid-column: 1/3;
	}

	h2 {
		font-size: 2em;
		font-weight: 200;
		user-select: none;
		margin: 0 0 0.5em 0;
	}

	label {
		position: relative;
		line-height: 1.2;
		padding: 0.5em 2.5em 0.5em 2em;
		margin: 0 0 0.5em 0;
		border-radius: 2px;
		user-select: none;
		border: 1px solid hsl(240, 8%, 70%);
		background-color:hsl(240, 8%, 93%);
		color: #333;
	}

	input[type="checkbox"] {
		position: absolute;
		left: 0.5em;
		top: 0.6em;
		margin: 0;
	}

	.done {
		border: 1px solid hsl(240, 8%, 90%);
		background-color:hsl(240, 8%, 98%);
	}

	button {
		position: absolute;
		top: 0;
		right: 0.2em;
		width: 2em;
		height: 100%;
		background: no-repeat 50% 50% url("data:image/svg+xml,%3Csvg xmlns='http://www.w3.org/2000/svg' viewBox='0 0 24 24'%3E%3Cpath fill='%23676778' d='M12,2C17.53,2 22,6.47 22,12C22,17.53 17.53,22 12,22C6.47,22 2,17.53 2,12C2,6.47 6.47,2 12,2M17,7H14.5L13.5,6H10.5L9.5,7H7V9H17V7M9,18H15A1,1 0 0,0 16,17V10H8V17A1,1 0 0,0 9,18Z'%3E%3C/path%3E%3C/svg%3E");
		background-size: 1.4em 1.4em;
		border: none;
		opacity: 0;
		transition: opacity 0.2s;
		text-indent: -9999px;
		cursor: pointer;
	}

	label:hover button {
		opacity: 1;
	}
</style>
```

## 动画

在上一章中，我们使用延迟转换来创建元素从一个待办事项列表移动到另一个列表时的运动错觉。

为了完成幻觉，我们还需要对没有过渡的元素应用运动。为此，我们使用 `animate` 指令。

首先，从 `svelte/animate` 导入 `flip` 函数——flip 代表[“First、Last、Invert、Play”](https://aerotwist.com/blog/flip-your-animations/)：

``` js
import { flip } from 'svelte/animate';
```

然后将其添加到 `<label>` 元素中：

``` html
<label
	in:receive="{{key: todo.id}}"
	out:send="{{key: todo.id}}"
	animate:flip
>
```

在这种情况下移动有点慢，所以我们可以添加一个`duration`参数：

``` html
<label
	in:receive="{{key: todo.id}}"
	out:send="{{key: todo.id}}"
	animate:flip="{{duration: 200}}"
>
```

> 持续时间也可以是 `d => milliseconds`函数，其中 `d` 是元素必须经过的像素数

请注意，所有过渡和动画都是使用 CSS 而不是 JavaScript 应用的，这意味着它们不会阻塞（或被主线程阻塞）。

## Action

Action本质上是元素级生命周期函数。它们对以下情况很有用：

* 与第三方库的接口
* 延迟加载的图像
* 工具提示
* 添加自定义事件处理程序

在这个应用程序中，我们想让橙色框“可平移”。它具有 `panstart`、`panmove` 和 `panend` 事件的事件处理程序，但这些不是原生 DOM 事件。我们必须自己派遣他们。首先，导入 `pannable` 函数...
  
...然后将它与元素一起使用:

``` html
<!-- App.Svelte -->
<script>
	import { spring } from 'svelte/motion';
	import { pannable } from './pannable.js';

	const coords = spring({ x: 0, y: 0 }, {
		stiffness: 0.2,
		damping: 0.4
	});

	function handlePanStart() {
		coords.stiffness = coords.damping = 1;
	}

	function handlePanMove(event) {
		coords.update($coords => ({
			x: $coords.x + event.detail.dx,
			y: $coords.y + event.detail.dy
		}));
	}

	function handlePanEnd(event) {
		coords.stiffness = 0.2;
		coords.damping = 0.4;
		coords.set({ x: 0, y: 0 });
	}
</script>

<style>
	.box {
		--width: 100px;
		--height: 100px;
		position: absolute;
		width: var(--width);
		height: var(--height);
		left: calc(50% - var(--width) / 2);
		top: calc(50% - var(--height) / 2);
		border-radius: 4px;
		background-color: #ff3e00;
		cursor: move;
	}
</style>

<div class="box"
	use:pannable
	on:panstart={handlePanStart}
	on:panmove={handlePanMove}
	on:panend={handlePanEnd}
	style="transform:
		translate({$coords.x}px,{$coords.y}px)
		rotate({$coords.x * 0.2}deg)"
></div>
```
  
打开 `pannable.js` 文件。与转换函数一样，动作函数接收一个`node`和一些可选参数，并返回一个动作对象。该对象可以有一个 `destroy` 函数，在卸载元素时调用该函数。

我们希望在用户将鼠标向下移动到元素上时触发 `panstart` 事件，在拖动元素时触发 `panmove` 事件（使用 `dx` 和 `dy` 属性显示鼠标移动的距离），以及在鼠标向上移动时触发 `panend` 事件。一种可能的实现如下所示：
  
``` js
// pannable.js
export function pannable(node) {
	let x;
	let y;

	function handleMousedown(event) {
		x = event.clientX;
		y = event.clientY;

		node.dispatchEvent(new CustomEvent('panstart', {
			detail: { x, y }
		}));

		window.addEventListener('mousemove', handleMousemove);
		window.addEventListener('mouseup', handleMouseup);
	}

	function handleMousemove(event) {
		const dx = event.clientX - x;
		const dy = event.clientY - y;
		x = event.clientX;
		y = event.clientY;

		node.dispatchEvent(new CustomEvent('panmove', {
			detail: { x, y, dx, dy }
		}));
	}

	function handleMouseup(event) {
		x = event.clientX;
		y = event.clientY;

		node.dispatchEvent(new CustomEvent('panend', {
			detail: { x, y }
		}));

		window.removeEventListener('mousemove', handleMousemove);
		window.removeEventListener('mouseup', handleMouseup);
	}

	node.addEventListener('mousedown', handleMousedown);

	return {
		destroy() {
			node.removeEventListener('mousedown', handleMousedown);
		}
	};
}
```
  
更新 `pannable` 函数并尝试移动框。

> 此实现用于演示目的——更完整的实现也会考虑触摸事件。

## 添加参数

像过渡和动画一样，一个动作可以接受一个参数，动作函数将与它所属的元素一起被调用。

在这里，我们使用了一个`longpress`操作，只要用户按下并按住按钮一段给定的持续时间，它就会触发一个同名的事件。现在，如果您切换到 `longpress.js` 文件，您会看到它被硬编码为 500 毫秒。

我们可以更改 action 函数以接受持续时间作为第二个参数，并将该`duration`传递给 `setTimeout` 调用：

回到 `App.svelte`，我们可以将持续时间值传递给动作：

``` html
<!-- App.svelte -->
<script>
	import { longpress } from './longpress.js';

	let pressed = false;
	let duration = 2000;
</script>

<label>
	<input type=range bind:value={duration} max={2000} step={100}>
	{duration}ms
</label>

<button use:longpress={duration}
	on:longpress="{() => pressed = true}"
	on:mouseenter="{() => pressed = false}"
>press and hold</button>

{#if pressed}
	<p>congratulations, you pressed and held for {duration}ms</p>
{/if}
```

这几乎有效——该事件现在仅在 2 秒后触发。但是如果你将持续时间向下滑动，它仍然需要两秒钟。

为了改变这一点，我们可以在 `longpress.js` 中添加一个`update`方法。这将在参数更改时调用：

``` js
// longpress.js
export function longpress(node, duration) {
	let timer;
	
	const handleMousedown = () => {
		timer = setTimeout(() => {
			node.dispatchEvent(
				new CustomEvent('longpress')
			);
		}, duration);
	};
	
	const handleMouseup = () => {
		clearTimeout(timer)
	};

	node.addEventListener('mousedown', handleMousedown);
	node.addEventListener('mouseup', handleMouseup);

	return {
		update(newDuration) {
			duration = newDuration;
		},
		destroy() {
			node.removeEventListener('mousedown', handleMousedown);
			node.removeEventListener('mouseup', handleMouseup);
		}
	};
}
```

> 如果您需要将多个参数传递给一个动作，请将它们组合成一个对象，如使用：`longpress={{duration, spiciness}}` 。
