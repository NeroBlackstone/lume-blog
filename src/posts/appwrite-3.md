---
title: Appwrite学习笔记(四)
description: ''
date: 2021-07-31
img: https://res.cloudinary.com/neroblackstone/image/upload/v1624671830/appwrite_i2voda.webp
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

如果您使用 SSH 来管理您的系统，请不要忘记在您的防火墙中保持打开状态！ SSH 被认为是一种私有服务，这意味着它应该可以公开访问，但仅限于授权帐户。最佳实践是使用诸如 SSH 密钥之类的加密工具而不是密码，因为它们更难以伪造。

#### 更多阅读

以下是一些其他资源，可以更详细地了解最佳实践：

* [Docker 安全](https://docs.docker.com/engine/security/)
* [DigitalOcean 推荐的安全措施](https://www.digitalocean.com/community/tutorials/recommended-security-measures-to-protect-your-servers)

### 保护Appwrite

现在，让我们讨论为生产设置 Appwrite。

#### 环境变量

您可以使用它提供的许多环境变量轻松地为生产配置 Appwrite。为生产部署时，应在 `appwrite` 安装目录中隐藏的 `.env` 文件中设置以下变量：

* `_APP_ENV`：更改为`production`。
* `_APP_OPTIONS_ABUSE`：为 API 启用滥用检查和速率限制。设置为`enabled`。
* `_APP_OPTIONS_FORCE_HTTPS`：强制连接使用 HTTPS 进行安全数据传输。设置为`enabled`
* `_APP_OPENSSL_KEY_V1`：这是用于加密会话和密码等秘密的秘密。将此更改为安全和随机的内容，并**确保其安全并进行备份**。
* `_APP_DOMAIN`：将此设置为[您的域名](https://appwrite.io/docs/custom-domains)，以便 Appwrite 自动生成 SSL 证书。

#### 限制控制台访问

三个环境变量可用于限制对 Appwrite 控制台的访问：

* `_APP_CONSOLE_WHITELIST_EMAILS`
* `_APP_CONSOLE_WHITELIST_IPS`
* `_APP_CONSOLE_WHITELIST_ROOT`

如果您只希望单个帐户可以访问控制台，请将 `_ROOT` 变量设置为`enabled`。对于多个用户，您可以使用各自的环境变量限制对特定电子邮件和 IP 地址的访问。

#### 杀毒软件

对于生产，您可以为任何已知的恶意对象启用对上传文件的 `clamav` 扫描。将 `_APP_STORAGE_ANTIVIRUS` 设置为`enabled`并[取消注释](https://github.com/appwrite/appwrite/blob/master/docker-compose.yml#L417-L423) `docker-compose.yml` 中的服务以使用此功能。不要忘记在主 `appwrite` 服务的[depends_on 部分](https://github.com/appwrite/appwrite/blob/master/docker-compose.yml#L74)取消注释 `clamav`。

#### Functions

可以自定义 Cloud Functions 以满足您的生产系统的需求，主要用于控制可用于 Function 执行的资源：

* `_APP_FUNCTIONS_CPUS`: Cloud Functions 可以使用的最大 CPU 内核数。
* `_APP_FUNCTIONS_MEMORY`: Cloud Functions 可用的最大内存（以兆字节为单位）。
* `_APP_FUNCTIONS_CONTAINERS`: Appwrite 保持活动的最大容器数，默认为 10。增加此数字以增加热函数的数量。
* `_APP_FUNCTIONS_RUNTIMES`: 新 Cloud Functions 的可用运行时列表。

> 所有 Appwrite 环境变量都可以在我们的[文档](https://appwrite.io/docs/environment-variables)中找到。

## Docker Swarm 集成

如您所知，Appwrite 被设计为一组无状态微服务，可扩展性是我们的首要任务之一！虽然有很多方法可以通过大量编排服务实现可扩展性，但我们将看一看最直观的方法之一。今天，我们将讨论使用 Docker Swarm 水平扩展 Appwrite。

### 什么是 Docker Swarm？

Docker Swarm 是一种内置于 Docker CLI 的容器编排工具，它允许我们将 Docker 服务部署到主机集群，而不仅仅是 [Docker Compose](https://docs.docker.com/compose/) 允许的主机集群。这称为 Swarm 模式，不要与[不再作为独立产品开发](https://github.com/docker/classicswarm)的经典 Docker Swarm 混淆。