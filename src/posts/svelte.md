---
title: Svelte学习笔记（一）
description: Svelte入门
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