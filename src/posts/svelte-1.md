---
title: Svelte学习笔记（二）
description: 嵌套组件，HTML tags，编辑器设置，响应式
date: 2021-07-01
img: https://res.cloudinary.com/neroblackstone/image/upload/v1625101749/svelte_y2yhr6.png
tags:
- Svelte

---
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