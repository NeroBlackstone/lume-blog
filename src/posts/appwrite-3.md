---
title: Appwrite学习笔记(四)
description: ''
date: 2021-07-31
img: https://res.cloudinary.com/neroblackstone/image/upload/v1625101749/svelte_y2yhr6.png
tags:
- Appwrite

---
## 生产环境的Appwrite

既然我们已经介绍了 Appwrite 的许多功能，我们应该讨论在您的应用最终为用户准备好后在生产中运行 Appwrite。

首先，良好的安全性是一个不断变化的目标。 Appwrite 提供了一套 API，可以抽象出应用程序的许多安全要求，但在线托管软件意味着将计算机暴露在互联网上。虽然我们无法涵盖所有内容，但让我们讨论在生产中运行 Appwrite 时的一些安全最佳实践。

### 服务器

在我们讨论在生产中运行 Appwrite 的步骤之前，我们需要讨论一下 Appwrite 将在哪个系统上运行。这些技巧假设您在基于 Linux 的服务器上运行 Appwrite，但这些原则适用于任何操作系统。

#### 更新

大多数安全漏洞发生在运行过时、不安全软件版本的系统上。这个问题是可以理解的 - 很难跟上系统更新。但是，按 [cron](https://man7.org/linux/man-pages/man5/crontab.5.html) 计划运行更新也不是最好的，因为最好立即安装安全更新。使用 Ubuntu 的[无人值守升级](https://help.ubuntu.com/community/AutomaticSecurityUpdates)和 Fedora 的 [dnf-automatic](https://fedoraproject.org/wiki/AutoUpdates) 软件包等工具来运行软件的最新更新。

#### 防火墙和 SSH

安全最佳实践是默认拒绝安全策略 - 我们应该只提供对我们想要的服务的显式访问。Appwrite 在其默认配置中考虑了这一点：唯一暴露给外界的服务是我们需要的 [Traefik](https://traefik.io/traefik/) 代理。因此，如果 Appwrite 是我们想要在服务器上公开公开的唯一服务，我们可以使用防火墙工具来阻止对任何其他未使用端口的访问。

如果您使用 SSH 来管理您的系统，请不要忘记在您的防火墙中保持打开状态！ SSH 被认为是一种私有服务，这意味着它应该可以公开访问，但仅限于授权帐户。