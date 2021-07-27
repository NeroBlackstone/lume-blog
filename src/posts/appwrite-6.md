---
title: Appwrite学习笔记(七)
description: ''
date: 2021-07-27
img: https://res.cloudinary.com/neroblackstone/image/upload/v1624671830/appwrite_i2voda.webp
tags:
- Appwrite

---
## Appwrite CLI

很长一段时间，我们发现自己必须设置一个 SDK 来快速测试一些新功能，所以我们决定自己构建一个 CLI！我们希望对所使用的技术保持不可知论，因此我们决定采用 Docker 方法。Appwrite CLI 被打包成一个 Docker 容器，所以你唯一需要的依赖是 Docker 😊。 CLI 是使用我们的 Swagger 规范和我们自己的[ SDK 生成器](https://github.com/appwrite/sdk-generator)自动生成的。

Appwrite CLI 具有服务器端 SDK 的所有强大功能，方便您使用终端。您甚至可以使用它来自动化 CI 管道上的任务。这是[ Torsten 的一篇关于如何使用 CLI 在 CI 中自动化功能部署的文章](https://dev.to/appwrite/automate-appwrite-functions-deployment-with-github-actions-ci-5bef)。

## 安装

如果你已经安装了 Docker，你可以跟着做。否则，您需要先按照[此处](https://docs.docker.com/engine/install/)的说明为您的平台安装 Docker。安装 Docker 后，安装 CLI 应该是轻而易举的 🌬

* MacOS 和 Linux

``` shell
$ curl -sL https://appwrite.io/cli/install.sh | bash
```

您应该会看到一个光滑的安装盾牌，如果一切顺利，您应该会看到 `Obi-Wan Kenobi` 本人的消息。

![](https://res.cloudinary.com/neroblackstone/image/upload/v1627374344/appwrite_cli_install_gfgq7z.png)

太好了，然后您可以使用验证您的安装

``` shell
$ appwrite version 
CLI Version : 0.0.1
Server Version : 0.8.0
```

要与您的 Appwrite 服务器通信，您需要首先使用 `appwrite init` 命令初始化您的 CLI。您将必须输入您的端点、项目 ID、API 密钥和区域设置，因此请妥善保管这些内容。我们的 API 密钥需要以下范围来运行以下示例命令：

* users.read
* users.write
* locale.read

![](https://res.cloudinary.com/neroblackstone/image/upload/v1627374494/appwrite_cli_init_k7lrtb.png)

如果您计划在 CI 管道中使用 CLI，您可以像这样使用 init 命令的非交互式版本

``` shell
$ appwrite init --project="PROJECT_ID" \ 
                --endpoint="http://localhost/v1" \ 
                --key="PROJECT_KEY" \
                --locale="en-US"
```

在我们开始执行一些命令之前，让我们熟悉一下命令的语法。

``` shell
$ appwrite [SERVICE] [COMMAND] --[OPTIONS]
```

您可以使用 `appwrite help` 命令查看 Appwrite 提供的所有服务的列表

![](https://res.cloudinary.com/neroblackstone/image/upload/v1627374628/appwrite_cli_help_crpzre.png)

让我们向 Locale 服务发出请求

``` shell
$ appwrite locale getContinents
sum : 7
continents :
╔═══════════════╤══════╗
║ Name          │ Code ║
╟───────────────┼──────╢
║ Africa        │ AF   ║
║ Antarctica    │ AN   ║
║ Asia          │ AS   ║
║ Europe        │ EU   ║
║ North America │ NA   ║
║ Oceania       │ OC   ║
║ South America │ SA   ║
╚═══════════════╧══════╝
```

如果您尝试连接到没有有效 SSL 证书的域，您可能会遇到 SSL 错误。默认情况下，禁用对具有自签名 SSL 证书（或无证书）的域的请求。如果您信任域，则可以通过使用绕过证书验证。

``` shell
$ appwrite client setSelfSigned --value=true 
```

太好了，现在让我们尝试执行一个带有一些参数的命令。假设您想在项目中创建一个新用户。在 CLI 之前，您必须设置服务器端 SDK 才能发出此请求。在 CLI 之前，您必须设置服务器端 SDK 才能发出此请求。使用 CLI，您可以简单地使用 `appwrite users create` 命令。

``` shell
$  appwrite users create --email="chris@hemsworth.com" --password="very_strong_password" --name="Chris Hemsworth"
$id : 60a2b0c66148c
name : Chris Hemsworth
registration : 1621274822
status : 0
email : chris@hemsworth.com
emailVerification : false
prefs : {}
```

您可以使用列出您的用户

``` shell
$ appwrite users list
sum : 1
users :
╔═══════════════╤═════════════════╤══════════════╤════════╤════════════════════╤═══════════════════╤═══════╗
║ $id           │ Name            │ Registration │ Status │ Email              │ EmailVerification │ Prefs ║
╟───────────────┼─────────────────┼──────────────┼────────┼────────────────────┼───────────────────┼───────╢
║ 60a2b0c66148c │ Chris Hemsworth │ 1621274822   │ 0      │ chris@hemsworth... │ false             │ {}    ║
╚═══════════════╧═════════════════╧══════════════╧════════╧════════════════════╧═══════════════════╧═══════╝
```

如果您在使用特定命令时遇到困难，您可以随时使用 help 命令，如下所示

``` shell
$ appwrite users help
$ appwrite database help
```

![](https://res.cloudinary.com/neroblackstone/image/upload/v1627374929/appwrite_service_help_g1uium.png)

在即将举行的会议中，我们将讨论云函数并重点介绍如何使用 CLI 轻松创建、打包和部署云函数，而无需离开控制台！