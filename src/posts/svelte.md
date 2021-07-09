---
title: Svelte学习笔记（一）
description: Svelte基础部分
date: 2021-07-01
img: https://res.cloudinary.com/neroblackstone/image/upload/v1625101749/svelte_y2yhr6.png
tags:
- Svelte

---
## Svelte入门

### Svelte是什么

Svelte 是用于构建快速 Web 应用程序的工具。

它类似于 React 和 Vue 等 JavaScript 框架，它们的共同目标是轻松构建流畅的交互式用户界面。

但是有一个关键的区别：Svelte 在构建时将您的应用程序转换为纯 JavaScript，而不是在运行时解释您的应用程序代码。这意味着您无需牺牲框架抽象的性能成本，并且在您的应用程序首次加载时不会付出时间代价。

您可以使用 Svelte 构建整个应用程序，也可以将其逐步添加到现有代码库中。您还可以将组件作为可在任何地方工作的独立包发布，而无需依赖传统框架的开销。

### 了解组件

在 Svelte 中，应用程序由一个或多个组件组成。组件是可重用的自包含代码块，它封装了属于一起的 HTML、CSS 和 JavaScript，并写入 `.svelte`文件。代码块中的“hello world”示例是一个简单的组件。

``` html
<h1>Hello world!</h1>
```

## 添加数据

只呈现一些静态标记的组件不是很有趣。让我们添加一些数据。

首先，向您的组件添加一个脚本标签并声明一个`name`变量：

``` html
<script>
	let name = 'world';
</script>

<h1>Hello world!</h1>
```

然后，我们可以在标记中引用 name：

``` html
<h1>Hello {name}!</h1>
```

在花括号内，我们可以放置任何 JavaScript。尝试将 name 更改为 name.toUpperCase() 以获得更响亮的问候。

``` html
<h1>Hello {name.toUpperCase()}!</h1>
```

## 动态属性

就像您可以使用花括号控制文本一样，您也可以使用它们来控制元素属性。

``` html
<script>
	let src = 'tutorial/image.gif';
</script>

<img>
```

我们的图像缺少一个 `src` ——让我们添加一个：

``` html
<img src={src}>
```

这样更好。但是 Svelte 给了我们一个警告：

> A11y: <img> element should have an alt attribute

在构建 Web 应用程序时，重要的是要确保它们可供最广泛的用户群访问，包括（例如）视力或运动受损的人，或者没有强大硬件或良好互联网连接的人。可访问性（缩写为 a11y）并不总是容易正确，但是如果您编写了无法访问的标记，Svelte 会通过警告您来提供帮助。

在这种情况下，我们缺少描述图像的 alt 属性，用于使用屏幕阅读器的人，或者互联网连接缓慢或不稳定的人无法下载图像。让我们添加一个：

``` html
<img src={src} alt="A man dances.">
```

我们可以在属性中使用花括号。尝试将其更改为`“{name} 舞蹈”`。 — 记得在 `<script>` 块中声明一个 `name` 变量。

### 速记属性

具有相同名称和值的属性并不少见，例如 `src={src}`。 Svelte 为我们提供了这些情况的便捷简写：

``` html
<img {src} alt="A man dances.">
```

## 样式

就像在 HTML 中一样，您可以向组件添加 `<style>` 标签。让我们为 `<p>` 元素添加一些样式：

``` html
<p>This is a paragraph.</p>

<style>
	p {
		color: purple;
		font-family: 'Comic Sans MS', cursive;
		font-size: 2em;
	}
</style>
```

重要的是，这些规则适用于组件。您不会意外地更改应用中其他位置的 <p> 元素的样式。

## 嵌套组件

将整个应用程序放在单个组件中是不切实际的。相反，我们可以从其他文件中导入组件并包含它们，就像我们包含元素一样。

``` html
<!-- Nested.svelte -->
<p>This is another paragraph.</p>
```

添加导入 `Nested.svelte` 的 `<script>` 标签...

``` html
<script>
	import Nested from './Nested.svelte';
</script>
```

...然后将其添加到标记中：

``` html
<p>This is a paragraph.</p>
<Nested/>
```

请注意，即使 `Nested.svelte` 有一个 `<p>` 元素，来自 `App.svelte` 的样式也不会泄漏。

另请注意，组件名称 `Nested` 是大写的。已采用此约定使我们能够区分用户定义的组件和常规 HTML 标记。

## HTML tags

通常，字符串作为纯文本插入，这意味着 `<` 和 `>` 之类的字符没有特殊含义。

``` html
<script>
	let string = `this string contains some <strong>HTML!!!</strong>`;
</script>

<p>{string}</p>
```

但有时您需要将 HTML 直接渲染到组件中。例如，您现在正在阅读的单词存在于 Markdown 文件中，该文件作为 HTML 块包含在此页面中。

在 Svelte 中，您可以使用特殊的 `{@html ...}` 标签执行此操作：

``` html
<p>{@html string}</p>
```

> 在将 {@html ...} 中的表达式插入到 DOM 之前，Svelte 不会对其进行任何清理。换句话说，如果您使用此功能，请务必手动转义来自您不信任的来源的 HTML，否则您将面临将用户暴露于 XSS 攻击的风险。

## Making an app

某些时候，您会希望在自己的文本编辑器中舒适地开始编写组件。

在Deno中使用svelte可以使用[snel](https://github.com/crewdevio/Snel)。

如果您使用的是 VS Code，请安装 [Svelte 扩展](https://marketplace.visualstudio.com/items?itemName=svelte.svelte-vscode)，否则请按照[本指南](https://svelte.dev/blog/setting-up-your-editor)配置您的文本编辑器，以将 `.svelte` 文件与 `.html` 相同，以便语法突出显示。

然后，一旦您设置好项目，就可以轻松使用 Svelte 组件。编译器将每个组件转换为一个常规的 JavaScript 类——只需导入它并使用 `new` 实例化：

``` js
import App from './App.svelte';

const app = new App({
	target: document.body,
	props: {
		// we'll learn about props later
		answer: 42
	}
});
```

然后，如果需要，您可以使用[组件 API](https://svelte.dev/docs#Client-side_component_API) 与`app`交互。

## 响应式

Svelte 的核心是一个强大的响应性系统，用于使 DOM 与您的应用程序状态保持同步——例如，响应一个事件。

``` html
<script>
	let count = 0;

	function handleClick() {
		// event handler code goes here
	}
</script>

<button>
	Clicked {count} {count === 1 ? 'time' : 'times'}
</button>
```

为了演示它，我们首先需要连接一个事件处理程序。将第 9 行替换为：

``` html
<button on:click={handleClick}>
```

在 handleClick 函数中，我们需要做的就是改变 count 的值：

``` js
function handleClick() {
	count += 1;
}
```

Svelte 用一些代码“检测”了这个分配，告诉它需要更新 DOM。

## 声明响应式变量

当你的组件状态改变时，Svelte 会自动更新 DOM。通常，组件状态的某些部分需要根据其他部分（例如从`名字`和`姓氏`派生的`全名`）进行计算，并在它们发生变化时重新计算。

对于这些需求，我们有反应式声明。它们看起来像这样：

``` js
let count = 0;
$: doubled = count * 2;
```

> 如果这看起来有点陌生，请不要担心。它是有效的（如果是非常规的）JavaScript，Svelte 将其解释为“只要引用的任何值发生变化就重新运行此代码”。你会习惯的。

让我们在标记中使用 `doubled`:

``` html
<p>{count} doubled is {doubled}</p>
```

当然，您可以只在标记中写入 {count * 2} - 您不必使用反应值。当您需要多次引用它们，或者您的值依赖于其他反应值时，反应值变得特别有价值。

## 状态

我们不仅限于声明反应值——我们还可以反应性地运行任意语句。例如，我们可以在 count 发生变化时记录它的值：

``` js
$: console.log(`the count is ${count}`);
```

您可以轻松地将语句与块组合在一起：

``` js
$: {
	console.log(`the count is ${count}`);
	alert(`I SAID THE COUNT IS ${count}`);
}
```

你甚至可以把 $: 放在 if 块之类的东西前面：

``` js
$: if (count >= 10) {
	alert(`count is dangerously high!`);
	count = 9;
}
```

## 更新数组和对象

因为 Svelte 的反应性是由赋值触发的，所以使用 `push` 和 `splice` 等数组方法不会自动导致更新。例如，单击按钮不会执行任何操作。

解决这个问题的一种方法是添加一个原本多余的赋值：

``` js
function addNumber() {
	numbers.push(numbers.length + 1);
	numbers = numbers;
}
```

但有一个更惯用的解决方案：

    function addNumber() {
    	numbers = [...numbers, numbers.length + 1];
    }

您可以使用类似的模式来替换 `pop`、`shift`、`unshift` 和 `splice`。

分配给数组和对象的属性——例如`obj.foo += 1` 或 `array[i] = x` — 与赋值本身的工作方式相同。

``` js
function addNumber() {
	numbers[numbers.length] = numbers.length + 1;
}
```

一个简单的经验法则：更新变量的名称必须出现在赋值的左侧。例如这个...

``` js
const foo = obj.foo;
foo.bar = 'baz';
```

...不会更新对 obj.foo.bar 的引用，除非你再加上 obj = obj 。

## 声明属性

到目前为止，我们只处理内部状态——也就是说，这些值只能在给定的组件内访问。

``` html
<!-- App.svelte -->
<script>
	import Nested from './Nested.svelte';
</script>

<Nested answer={42}/>
```

``` html
<!-- Nested.svelte -->
<script>
	let answer;
</script>

<p>The answer is {answer}</p>
```

在任何实际应用程序中，您都需要将数据从一个组件向下传递到其子组件。为此，我们需要声明属性，通常缩写为“props”。在 Svelte 中，我们使用 `export` 关键字来做到这一点。编辑 `Nested.svelte` 组件：

``` html
<script>
	export let answer;
</script>
```

> 就像 `$:` 一样，一开始可能会觉得有点奇怪。这不是`export`在 JavaScript 模块中的正常工作方式！你会习惯的。

## 默认值

我们可以轻松地在 `Nested.svelte` 中为 props 指定默认值：

``` html
<script>
	export let answer = 'a mystery';
</script>
```

如果我们现在添加第二个没有 answer 属性的组件，它将回退到默认值：

``` html
<Nested answer={42}/>
<Nested/>
```

## 传播属性

如果你有一个属性对象，你可以将它们“传播”到一个组件上，而不是指定每一个属性：

``` html
<!-- App.svelte -->
<script>
	import Info from './Info.svelte';

	const pkg = {
		name: 'svelte',
		version: 3,
		speed: 'blazing',
		website: 'https://svelte.dev'
	};
</script>

<Info name={pkg.name} version={pkg.version} speed={pkg.speed} website={pkg.website}/>
```

``` html
<!-- Info.svelte -->
<script>
	export let name;
	export let version;
	export let speed;
	export let website;
</script>

<p>
	The <code>{name}</code> package is {speed} fast.
	Download version {version} from <a href="https://www.npmjs.com/package/{name}">npm</a>
	and <a href={website}>learn more here</a>
</p>
```

``` html
<Info {...pkg}/>
```

> 相反，如果您需要引用传递给组件的所有 props，包括那些没有使用 `export` 声明的 props，您可以通过直接访问 `$$props` 来实现。通常不建议这样做，因为 Svelte 很难优化，但它在极少数情况下很有用。

## If块

HTML 没有表达逻辑的方法，如条件和循环。Svelte有。

``` html
<script>
	let user = { loggedIn: false };

	function toggle() {
		user.loggedIn = !user.loggedIn;
	}
</script>

<button on:click={toggle}>
	Log out
</button>

<button on:click={toggle}>
	Log in
</button>
```

为了有条件地渲染一些标记，我们将它包装在一个 if 块中：

``` html
{#if user.loggedIn}
	<button on:click={toggle}>
		Log out
	</button>
{/if}

{#if !user.loggedIn}
	<button on:click={toggle}>
		Log in
	</button>
{/if}
```

试试吧——更新组件，然后点击按钮。

## else块

由于这两个条件 - `if user.loggedIn` 和 `if !user.loggedIn` - 是互斥的，我们可以通过使用 `else` 块稍微简化这个组件：

``` html
{#if user.loggedIn}
	<button on:click={toggle}>
		Log out
	</button>
{:else}
	<button on:click={toggle}>
		Log in
	</button>
{/if}
```

> `#` 字符始终表示块开始标记。 `/` 字符始终表示块结束标记。 `:` 字符，如 `{:else}` 中，表示块继续标记。别担心——您已经学会了 Svelte 添加到 HTML 的几乎所有语法。

## Else-if块

多个条件可以用`else-if` “链接”：

``` html
<script>
	let x = 7;
</script>

{#if x > 10}
	<p>{x} is greater than 10</p>
{:else if 5 > x}
	<p>{x} is less than 5</p>
{:else}
	<p>{x} is between 5 and 10</p>
{/if}
```

## Each块

如果您需要遍历数据列表，请使用 each 块：

``` html
<script>
	let cats = [
		{ id: 'J---aiyznGQ', name: 'Keyboard Cat' },
		{ id: 'z_AbfPXTKms', name: 'Maru' },
		{ id: 'OUtn3pvWmpg', name: 'Henri The Existential Cat' }
	];
</script>

<h1>The Famous Cats of YouTube</h1>

<ul>
	{#each cats as cat}
		<li><a target="_blank" href="https://www.youtube.com/watch?v={cat.id}">
			{cat.name}
		</a></li>
	{/each}
</ul>
```

> 表达式（在本例中为`cats`）可以是任何数组或类似数组的对象（即它具有`length`属性）。您可以使用`each[...iterable]` 循环遍历通用可迭代对象。

您可以获取当前索引作为第二个参数，如下所示：

``` html
{#each cats as cat, i}
	<li><a target="_blank" href="https://www.youtube.com/watch?v={cat.id}">
		{i + 1}: {cat.name}
	</a></li>
{/each}
```

如果你愿意，你可以使用解构——`each cats as { id, name }` —— 并用 `id` 和 `name` 替换 cat.id 和 cat.name。

## 键

默认情况下，当您修改 `each` 块的值时，它将在块的末尾添加和删除项目，并更新任何已更改的值。那可能不是您想要的。

``` html
<!-- Thing.svelte -->
<script>
	const emojis = {
        apple: "🍎",
        banana: "🍌",
        carrot: "🥕",
        doughnut: "🍩",
        egg: "🥚"
	}

	// the name is updated whenever the prop value changes...
	export let name;

	// ...but the "emoji" variable is fixed upon initialisation of the component
	const emoji = emojis[name];
</script>

<p>
	<span>The emoji for { name } is { emoji }</span>
</p>

<style>
	p {
		margin: 0.8em 0;
	}
	span {
		display: inline-block;
		padding: 0.2em 1em 0.3em;
		text-align: center;
		border-radius: 0.2em;
		background-color: #FFDFD3;
	}
</style>
```

``` html
<!-- App.svelte -->
<script>
	import Thing from './Thing.svelte';

	let things = [
		{ id: 1, name: 'apple' },
		{ id: 2, name: 'banana' },
		{ id: 3, name: 'carrot' },
		{ id: 4, name: 'doughnut' },
		{ id: 5, name: 'egg' },
	];

	function handleClick() {
		things = things.slice(1);
	}
</script>

<button on:click={handleClick}>
	Remove first thing
</button>

{#each things as thing}
	<Thing name={thing.name}/>
{/each}
```

说明原因比解释更容易。单击“删除第一件事”按钮几次，注意会发生什么：它删除了第一个 `<Thing>` 组件，但删除了_最后_一个 DOM 节点。然后它更新其余 DOM 节点中的`name`值，但不更新表情符号。

相反，我们只想删除第一个 `<Thing>` 组件及其 DOM 节点，而让其他组件不受影响。

为此，我们为`每个`块指定一个唯一标识符（或者说“键”）：

``` html
{#each things as thing (thing.id)}
	<Thing name={thing.name}/>
{/each}
```

这里，`(thing.id)` 是_关键_，它告诉 Svelte 在组件更新时如何找出要更改的 DOM 节点。

> 您可以使用任何对象作为键，因为 Svelte 在内部使用 Map — 换句话说，您可以执行 (thing) 而不是 (thing.id)。然而，使用字符串或数字通常更安全，因为这意味着身份在没有引用相等的情况下持续存在，例如在使用来自 API 服务器的新数据进行更新时。

## await块

大多数 Web 应用程序必须在某些时候处理异步数据。 Svelte 可以很容易地直接在你的标记中_await_ [promises](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Guide/Using_promises)的值：

``` html
<script>
	async function getRandomNumber() {
		const res = await fetch(`tutorial/random-number`);
		const text = await res.text();

		if (res.ok) {
			return text;
		} else {
			throw new Error(text);
		}
	}

	let promise = getRandomNumber();

	function handleClick() {
		promise = getRandomNumber();
	}
</script>

<button on:click={handleClick}>
	generate random number
</button>

{#await promise}
	<p>...waiting</p>
{:then number}
	<p>The number is {number}</p>
{:catch error}
	<p style="color: red">{error.message}</p>
{/await}
```

> 仅考虑最近的 `promise`，这意味着您无需担心竞争条件。

如果你知道你的promise不能reject，你可以省略 `catch`块。如果在 promise 解决之前不想显示任何内容，也可以省略第一个块：

``` html
{#await promise then value}
	<p>the value is {value}</p>
{/await}
```

## 事件

正如我们已经简要看到的，您可以使用 `on:` 指令侦听元素上的任何事件：

``` html
<script>
	let m = { x: 0, y: 0 };

	function handleMousemove(event) {
		m.x = event.clientX;
		m.y = event.clientY;
	}
</script>

<div on:mousemove={handleMousemove}>
	The mouse position is {m.x} x {m.y}
</div>

<style>
	div { width: 100%; height: 100%; }
</style>
```

## 内联控制器

您还可以内联声明事件处理程序：

``` html
<div on:mousemove="{e => m = { x: e.clientX, y: e.clientY }}">
	The mouse position is {m.x} x {m.y}
</div>
```

> 在某些框架中，您可能会看到出于性能原因避免使用内联事件处理程序的建议，尤其是在循环内部。该建议不适用于 Svelte — 编译器将始终做正确的事情，无论您选择哪种形式。

## 事件修饰符

DOM 事件处理程序可以具有改变其行为的修饰符。例如，带有 `once` 修饰符的处理程序只会运行一次：

``` html
<script>
	function handleClick() {
		alert('no more alerts')
	}
</script>

<button on:click|once={handleClick}>
	Click me
</button>
```

完整的修饰符列表：

* `preventDefault` - 在运行处理程序之前调用 `event.preventDefault ()`。例如，对于客户端表单处理很有用。
* `stopPropagation` — 调用 event.stopPropagation()，阻止事件到达下一个元素
* `passive` — 提高触摸/滚轮事件的滚动性能（Svelte 会在安全的地方自动添加它）
* `nonpassive` — 显式设置passive：false
* `capture` — 在capture阶段而不是冒泡阶段触发处理程序（[MDN 文档](https://developer.mozilla.org/en-US/docs/Learn/JavaScript/Building_blocks/Events#Event_bubbling_and_capture)）
* `once` - 在第一次运行后删除处理程序
- `self` — 如果 event.target 是元素本身，则仅触发处理程序

您可以将修饰符链接在一起，例如`on:click|once|capture={...}`。

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

## 绑定

作为一般规则，Svelte 中的数据流是_自上而下_的——父组件可以在子组件上设置 props，组件可以在元素上设置属性，但反过来不行。

有时打破这条规则很有用。以这个组件中的 `<input>` 元素为例——我们可以添加一个 `on:input` 事件处理程序，将 `name` 的值设置为 `event.target.value`，但这有点......样板。正如我们将看到的，使用其他表单元素时情况会更糟。

相反，我们可以使用 `bind:value` 指令：

``` html
<script>
	let name = 'world';
</script>

<input bind:value={name}>

<h1>Hello {name}!</h1>
```

这意味着不仅对 `name` 值的更改会更新输入值，而且对输入值的更改也会更新 `name`。

## 数据输入

在 DOM 中，一切都是字符串。当您处理数字输入时，这无济于事——`type="number"` 和 `type="range"`——因为这意味着您必须记住在使用之前强制输入 `input.value`。

使用 `bind:value，Svelte` 会为您处理：

``` html
<script>
	let a = 1;
	let b = 2;
</script>

<label>
	<input type=number bind:value={a} min=0 max=10>
	<input type=range bind:value={a} min=0 max=10>
</label>

<label>
	<input type=number bind:value={b} min=0 max=10>
	<input type=range bind:value={b} min=0 max=10>
</label>

<p>{a} + {b} = {a + b}</p>
```

## 复选框输入

复选框用于在状态之间切换。我们没有绑定到 input.value，而是绑定到 input.checked：

``` html
<script>
	let yes = false;
</script>

<label>
	<input type=checkbox bind:checked={yes}>
	Yes! Send me regular email spam
</label>

{#if yes}
	<p>Thank you. We will bombard your inbox and sell your personal details.</p>
{:else}
	<p>You must opt in to continue. If you're not paying, you're the product.</p>
{/if}

<button disabled={!yes}>
	Subscribe
</button>
```

## 输入组

如果您有多个与同一个值相关的输入，您可以将 `bind:group` 与 `value` 属性一起使用。同一组中的单选框是互斥的；同一组中的复选框输入形成选定值的数组。

``` html
<script>
	let scoops = 1;
	let flavours = ['Mint choc chip'];

	function join(flavours) {
		if (flavours.length === 1) return flavours[0];
		return `${flavours.slice(0, -1).join(', ')} and ${flavours[flavours.length - 1]}`;
	}
</script>

<h2>Size</h2>

<label>
	<input type=radio group={scoops} name="scoops" value={1}>
	One scoop
</label>

<label>
	<input type=radio group={scoops} name="scoops" value={2}>
	Two scoops
</label>

<label>
	<input type=radio group={scoops} name="scoops" value={3}>
	Three scoops
</label>

<h2>Flavours</h2>

<label>
	<input type=checkbox group={flavours} name="flavours" value="Cookies and cream">
	Cookies and cream
</label>

<label>
	<input type=checkbox group={flavours} name="flavours" value="Mint choc chip">
	Mint choc chip
</label>

<label>
	<input type=checkbox group={flavours} name="flavours" value="Raspberry ripple">
	Raspberry ripple
</label>

{#if flavours.length === 0}
	<p>Please select at least one flavour</p>
{:else if flavours.length > scoops}
	<p>Can't order more flavours than scoops!</p>
{:else}
	<p>
		You ordered {scoops} {scoops === 1 ? 'scoop' : 'scoops'}
		of {join(flavours)}
	</p>
{/if}
```

将 `bind:group` 添加到每个输入：

``` html
<input type=radio bind:group={scoops} name="scoops" value={1}>
```

在这种情况下，我们可以通过将复选框输入移动到 `each` 块中来使代码更简单。首先，在 `<script>` 块中添加一个`menu`变量...

``` js
let menu = [
	'Cookies and cream',
	'Mint choc chip',
	'Raspberry ripple'
];
```

...然后替换第二部分：

``` html
<h2>Flavours</h2>

{#each menu as flavour}
	<label>
		<input type=checkbox bind:group={flavours} name="flavours" value={flavour}>
		{flavour}
	</label>
{/each}
```

现在可以轻松地扩展我们的冰淇淋菜单。

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

## 维度

每个块级元素都有 `clientWidth`、`clientHeight`、`offsetWidth` 和 `offsetHeight` 绑定：

``` html
<script>
	let w;
	let h;
	let size = 42;
	let text = 'edit me';
</script>

<input type=range bind:value={size}>
<input bind:value={text}>

<p>size: {w}px x {h}px</p>

<div bind:clientWidth={w} bind:clientHeight={h}>
	<span style="font-size: {size}px">{text}</span>
</div>

<style>
	input { display: block; }
	div { display: inline-block; }
	span { word-break: break-all; }
</style>
```

这些绑定是只读的——改变 `w` 和 `h` 的值不会有任何影响。

> 使用[与此类似](http://www.backalleycoder.com/2013/03/18/cross-browser-event-based-element-resize-detection/)的技术测量元素。涉及一些开销，因此不建议将其用于大量元素。
>
> `display: inline` 元素不能用这种方法测量；不能包含其他元素（例如 `<canvas>`）的元素也不能。在这些情况下，您需要改为测量包装元素。

## This

只读 `this` 绑定适用于每个元素（和组件），并允许您获取对渲染元素的引用。例如，我们可以获得对 `<canvas>` 元素的引用：

``` html
<script>
	import { onMount } from 'svelte';

	let canvas;

	onMount(() => {
		const ctx = canvas.getContext('2d');
		let frame = requestAnimationFrame(loop);

		function loop(t) {
			frame = requestAnimationFrame(loop);

			const imageData = ctx.getImageData(0, 0, canvas.width, canvas.height);

			for (let p = 0; p < imageData.data.length; p += 4) {
				const i = p / 4;
				const x = i % canvas.width;
				const y = i / canvas.height >>> 0;

				const r = 64 + (128 * x / canvas.width) + (64 * Math.sin(t / 1000));
				const g = 64 + (128 * y / canvas.height) + (64 * Math.cos(t / 1000));
				const b = 128;

				imageData.data[p + 0] = r;
				imageData.data[p + 1] = g;
				imageData.data[p + 2] = b;
				imageData.data[p + 3] = 255;
			}

			ctx.putImageData(imageData, 0, 0);
		}

		return () => {
			cancelAnimationFrame(frame);
		};
	});
</script>

<canvas
	bind:this={canvas}
	width={32}
	height={32}
></canvas>

<style>
	canvas {
		width: 100%;
		height: 100%;
		background-color: #666;
		-webkit-mask: url(svelte-logo-mask.svg) 50% 50% no-repeat;
		mask: url(svelte-logo-mask.svg) 50% 50% no-repeat;
	}
</style>
```

请注意，在组件挂载之前，`canvas` 的值将是`undefined`的，因此我们将逻辑放在 `onMount` [生命周期函数](https://svelte.dev/tutorial/onmount)中。

## 组件绑定

正如您可以绑定到 DOM 元素的属性一样，您也可以绑定到组件 props。例如，我们可以绑定到这个 `<Keypad>` 组件的 `value` 属性，就好像它是一个表单元素：

``` html
<!-- Keypad.svelte -->
<script>
	import { createEventDispatcher } from 'svelte';

	export let value = '';

	const dispatch = createEventDispatcher();

	const select = num => () => value += num;
	const clear  = () => value = '';
	const submit = () => dispatch('submit');
</script>

<div class="keypad">
	<button on:click={select(1)}>1</button>
	<button on:click={select(2)}>2</button>
	<button on:click={select(3)}>3</button>
	<button on:click={select(4)}>4</button>
	<button on:click={select(5)}>5</button>
	<button on:click={select(6)}>6</button>
	<button on:click={select(7)}>7</button>
	<button on:click={select(8)}>8</button>
	<button on:click={select(9)}>9</button>

	<button disabled={!value} on:click={clear}>clear</button>
	<button on:click={select(0)}>0</button>
	<button disabled={!value} on:click={submit}>submit</button>
</div>

<style>
	.keypad {
		display: grid;
		grid-template-columns: repeat(3, 5em);
		grid-template-rows: repeat(4, 3em);
		grid-gap: 0.5em
	}

	button {
		margin: 0
	}
</style>
```

``` html
<!-- App.svelte -->
<script>
	import Keypad from './Keypad.svelte';

	let pin;
	$: view = pin ? pin.replace(/\d(?!$)/g, '•') : 'enter your pin';

	function handleSubmit() {
		alert(`submitted ${pin}`);
	}
</script>

<h1 style="color: {pin ? '#333' : '#ccc'}">{view}</h1>

<Keypad bind:value={pin} on:submit={handleSubmit}/>
```

现在，当用户与键盘交互时，父组件中的 `pin` 值会立即更新。

> 谨慎使用组件绑定。如果您的应用程序中的数据流过多，则很难跟踪应用程序周围的数据流，尤其是在没有“单一事实来源”的情况下。

### 绑定元素实例

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