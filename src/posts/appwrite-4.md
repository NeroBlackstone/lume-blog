---
title: Appwrite学习笔记(五)
description: 登录和注册
date: 2021-06-30
img: https://res.cloudinary.com/neroblackstone/image/upload/v1624671830/appwrite_i2voda.webp
tags:
- Appwrite

---
## 登录和注册

对用户进行身份验证和授权以验证他们的请求或保存他们的数据是现代应用程序开发生命周期不可或缺的一部分。这是我们在每个应用程序中都会做的事情。Appwrite 通过抽象出所有关于用户管理的功能，使这个过程变得非常简单，为我们提供了一个漂亮的 API。Appwrite 的身份验证功能支持基于电子邮件和密码的身份验证以及各种 OAuth2 提供商，例如 **Google、Facebook、GitHub、Twitter** 等。您可以在 **Appwrite Console -> Users -> Providers** 中找到所有支持的提供程序的列表。[即将推出的 Appwrite 版本](https://dev.to/appwrite/appwrite-0-8-is-coming-soon-and-this-is-what-you-can-expect-35li)还将添加对匿名登录的支持。

默认情况下，身份验证、用户管理、团队和角色都由 Appwrite 处理。在今天的会议中，我们将继续上次会议的演示并添加身份验证功能。我们将实现注册和登录。让我们开始吧。

### 开始

我们已经创建了一个基础项目，我们将开始在我们的应用程序中使用 Appwrite 实现更多功能。要继续以下部分，您需要克隆或下载我们的基础项目。可以在[这里](https://github.com/christyjacob4/30-days-of-appwrite/tree/starter)找到我们的基础项目。从这里开始，我们将建立在这个初始项目的基础上。所以请克隆或下载它，以便您可以跟上。

### 创建注册和登录组件

首先，我们将创建我们的 Login 和 Signup 组件并设置到这些组件的路由。

#### 注册组件