---
title: Appwrite学习笔记(八)
description: ''
date: 2021-07-28
img: https://res.cloudinary.com/neroblackstone/image/upload/v1624671830/appwrite_i2voda.webp
tags:
- Appwrite

---
## Appwrite 云函数

如果您熟悉无服务器世界，您可能已经知道 **Cloud Functions** 是什么。对于那些不知道的人，将云函数视为一段无状态的代码，可以独立执行而无需您管理服务器。如果您使用过 **AWS Lambdas** 或类似的产品，那么您会对 **Appwrite Cloud Functions** 感到宾至如归。 Appwrite 支持超过 **13 种不同的运行时语言**，如 python、deno、dotnet，还有更多！

今天，我们将引导您浏览 Appwrite 控制台中的函数仪表板，并了解如何创建和部署函数。

Appwrite 中的 Cloud Functions 可以通过 3 种方式触发