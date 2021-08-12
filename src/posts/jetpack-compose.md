---
title: Jetpack Compose学习笔记
description: ''
date: 2021-08-12
img: https://res.cloudinary.com/neroblackstone/image/upload/v1628761044/Android_logo_2019_iukgvk.png
tags:
- Android
- Jetpack Compose
- Kotlin

---
## 开始之前

[Jetpack Compose](https://developer.android.com/jetpack/compose) 是一个现代工具包，旨在简化 UI 开发。它结合了反应式编程模型和 Kotlin 编程语言的简洁性和易用性。它是完全声明式的，这意味着您可以通过调用一系列将数据转换为 UI 层次结构的函数来描述您的 UI。当底层数据发生变化时，框架会自动调用这些函数，为您更新视图层次结构。

Compose 应用程序由可组合函数组成 - 只是标有 `@Composable` 的常规函数​​，可以调用其他可组合函数。创建新的 UI 组件只需要一个函数。注释告诉 Compose 为随着时间的推移更新和维护 UI 的功能添加特殊支持。 Compose 可让您将代码组织成小块。可组合函数通常简称为“可组合”。

通过制作小型的可重用组合 - 可以轻松构建应用程序中使用的 UI 元素库。每个人负责屏幕的一部分，可以独立编辑。

> 注意：在此 Codelab 中，术语“UI 组件”、“可组合函数”和“可组合”可互换使用以指代同一概念。

### 先决条件

* Kotlin 语法的经验，包括 lambdas

### 你会做什么