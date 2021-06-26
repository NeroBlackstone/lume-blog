---
title: Appwrite学习笔记(一)
description: Appwrite的安装与其结构
date: 2021-06-25
img: https://res.cloudinary.com/neroblackstone/image/upload/v1624671830/appwrite_i2voda.webp
tags:
- Appwrite

---
## 为什么要使用Appwrite

构建现代应用程序很难——构建安全、高性能和可扩展的应用程序更难。Appwrite 使应用程序开发更轻松、更有趣。

Appwrite 是一个开源，自托管Backend-as-a-Service (BaaS)，可为开发人员提供快速构建安全、现代应用程序所需的一切。Appwrite 打包为一组 Docker 微服务，为开发人员提供了一套易于使用的 API、SDK、工具和管理 UI 来构建应用功能，而不是编写样板。本地开发就像 `docker run` 一样简单。

Appwrite 设计为跨平台且与技术无关，这意味着它可以在任何操作系统、编码语言、框架或平台上运行。可在任何需要的地方使用 Appwrite， Appwrite提供多种编程语言的 SDK，所有 SDK 均由相同的源代码生成。

## 安装Appwrite

Appwrite 使用 Docker 来安装和运行其服务，因此可以在任何支持 Docker 的系统上安装 Appwrite，包括 MacOS、Linux 和 Windows。要在机器上进行本地开发，需要先安装 Docker。如果还没有安装 Docker，请访问 [Docker安装文档](https://docs.docker.com/engine/install/) 。

在任何平台上安装 Appwrite 就像在终端/命令提示符中运行单个命令一样简单。只需要为我们的平台选择正确的命令。确保您使用[此处](https://appwrite.io/docs/installation)的最新 appwrite 版本：

    docker run -it --rm \
        --volume /var/run/docker.sock:/var/run/docker.sock \
        --volume "$(pwd)"/appwrite:/usr/src/code/appwrite:rw \
        --entrypoint="install" \
        appwrite/appwrite:0.7.2

### 升级Appwrite

如果已经有一个 Appwrite 服务器在运行并且想要升级到新版本，Appwrite 迁移工具可以提供帮助。为了升级 Appwrite，请确保您位于之前运行 Appwrite 安装脚本的同一文件夹中。这个文件夹应该包含一个文件夹 `appwrite`，里面应该有你的 `docker-compose.yml` 和 `.env` 文件。从包含 `appwrite` 文件夹的文件夹运行下一个版本的安装。成功安装新版本后，即可运行迁移工具。 **确保在运行迁移之前备份您的数据、docker-compose.yml 文件和 .env 文件和设置。** 您可以从包含 `docker-compose.yml` 文件的文件夹中使用以下命令运行迁移工具。

    docker-compose exec appwrite migrate

可以在[官方文档](https://appwrite.io/docs/upgrade)中找到有关升级和迁移的更多信息。

### 在云服务上安装 Appwrite

在云服务上安装 Appwrite 与在本地安装没有太大区别。我们可以在任何云服务上部署 Linux 虚拟机，然后按照 UNIX 安装代码安装 Appwrite。

### 检查 Appwrite 安装的健康状况

成功安装 Appwrite 后，可以检查 Appwrite 服务器的运行状况和状态。为了运行初始检查，您可以使用 Appwrite 的`doctor`命令，通过 `docker-compose exec appwrite doctor` 执行。这个命令应该从你安装 Appwrite 的目录运行，即你的 docker-compose.yml 文件所在的目录。

### 检查日志

如果查看 `docker-compose.yml` 文件，可以看到各种不同服务的列表。每当遇到问题时，可以查看每个服务的日志或整个堆栈的日志。从包含 `docker-compose.yml` 文件的同一目录中，可以运行以下命令来查看日志:

* `docker-compose logs` : 查看所有服务的日志
* `docker-compose logs <name_of_the_service>` : 查看单个服务的日志。

## Appwrite的微服务

Appwrite被设计为在云原生环境中工作，为了保持这种设计，Appwrite 被方便地打包为一组 docker 容器（准确地说是 17 个）。

![](https://res.cloudinary.com/neroblackstone/image/upload/v1624677003/appwrite_microservice_kf5amm.webp)

这是 Appwrite 架构的总览，我们将在接下来的部分中讨论其中的大部分组件。 Appwrite 中的每个容器都单独处理一个微服务。由于它们已被容器化，因此每个服务都可以独立于任何其他微服务进行扩展。

目前，所有 Appwrite 微服务都通过[专用网络](https://docs.docker.com/network/bridge/)，使用TCP 协议进行通信。应该注意不要向面向公众的网络公开任何服务，除了公共端口 80 和 443（默认情况下用于公开 Appwrite HTTP API）。

### Appwrite

这是主要的 [Appwrite 容器](https://gist.github.com/eldadfux/977869ff6bdd7312adfd4e629ee15cc5#file-docker-compose-yml-L29), 这个容器是由[Dockerfile](https://github.com/appwrite/appwrite/blob/master/Dockerfile) 建立的。 主 Appwrite 容器实现 Appwrite API 协议，处理身份验证、授权和速率限制。此微服务是完全无状态的，并且可以轻松复制以实现可扩展性。

### Traefik

Traefik 是用 Go 编写的现代反向代理和负载均衡器，可以轻松部署微服务。 Traefik 与您现有的基础设施组件集成，并自动和动态地进行自我配置。我们使用 Traefik 作为不同 Appwrite API 的主要入口点。 Traefik 还负责提供 Appwrite 自动生成的 SSL 证书。这个微服务是完全无状态的。

### Redis

Appwrite 使用 Redis 来提供三个主要功能。

- **缓存**：Appwrite 使用 Redis 内存缓存来更快地获取数据库查询。
- **Pub/Sub**：Appwrite 使用带有 Resque 的 Redis 作为 pub/sub 机制在 Appwrite API 和不同的 worker 之间传输消息。
- **计划任务**：Appwrite 使用 Redis 来存储和触发使用调度容器的未来任务。

### Appwrite的workers

Appwrite 中有很多异步任务发生，一个很好的例子是记录 Appwrite API 的使用统计信息。

我们使用内部pub/sub系统， [Resque](https://github.com/resque/php-resque) ， 来累积所有这些任务。各个workers使用这些任务并独立执行。我们有八个消息队列和八个与之配对的workers。

- Audits worker : [Audits worker](https://github.com/appwrite/appwrite/blob/master/app/workers/audits.php)使用来自 `v1-audits` 队列的消息。 Appwrite 定义了一组[系统事件](https://appwrite.io/docs/webhooks#events)。当这些事件发生时，Audits worker会将它们记录到`mariadb`。Audits worker使用 [utopia-php/audit](https://github.com/utopia-php/audit) 库。