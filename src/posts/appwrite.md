---
title: Appwrite学习笔记(一)
description: Appwrite基础入门
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

* **缓存**：Appwrite 使用 Redis 内存缓存来更快地获取数据库查询。
* **Pub/Sub**：Appwrite 使用带有 Resque 的 Redis 作为 pub/sub 机制在 Appwrite API 和不同的 worker 之间传输消息。
* **计划任务**：Appwrite 使用 Redis 来存储和触发使用调度容器的未来任务。

### Appwrite的workers

Appwrite 中有很多异步任务发生，一个很好的例子是记录 Appwrite API 的使用统计信息。

我们使用内部pub/sub系统， [Resque](https://github.com/resque/php-resque) ， 来累积所有这些任务。各个workers使用这些任务并独立执行。我们有八个消息队列和八个与之配对的workers。

* Audits worker

[Audits worker](https://github.com/appwrite/appwrite/blob/master/app/workers/audits.php)消费来自 `v1-audits` 队列的消息。 Appwrite 定义了一组[系统事件](https://appwrite.io/docs/webhooks#events)。当这些事件发生时，Audits worker会将它们记录到`mariadb`。Audits worker使用 [utopia-php/audit](https://github.com/utopia-php/audit) 库。

* Certificates Worker

[Certificates worker](https://github.com/appwrite/appwrite/blob/master/app/workers/certificates.php) 消费来自 `v1-certificates` 队列的消息。Certificates worker使用 Let's Encrypt 中的 `certbot` 来创建和定期更新 SSL 证书。

- Deletes Worker

[Deletes Worker](https://github.com/appwrite/appwrite/blob/master/app/workers/deletes.php) 消费来自 `v1-deletes` 队列的消息。顾名思义，它在 Appwrite 数据库中执行删除操作。对文档、用户、项目、函数等的删除请求由 Deletes Worker 处理。在目前的状态下，deletes worker 是由某些 API 请求触发的，也由Maintenance worker 触发。

- Functions Worker

[Functions worker](https://github.com/appwrite/appwrite/blob/master/app/workers/functions.php) 消费来自 `v1-functions` 队列的消息并处理与 Appwrite 的云函数相关的所有任务。

- Appwrite 中的云函数可以通过 3 种方式触发：
	- [异步使用事件](https://appwrite.io/docs/webhooks#events)
    - [使用 CRON 计划](https://en.wikipedia.org/wiki/Cron)
    - [使用 Appwrite HTTP API](https://appwrite.io/docs/client/functions?sdk=web#functionsCreateExecution)
    
Functions worker 完成启动和运行云函数所需的所有繁重工作。从在启动时为各个环境拉取 Docker 镜像，到管理和运行容器，再到响应错误，functions worker会处理这一切！

- Mails Worker

[Mails worker](https://github.com/appwrite/appwrite/blob/master/app/workers/mails.php) 消费来自 `v1-mails` 队列的消息并且只负责一个功能：发送电子邮件！它只是收集信息并使用 [PHPMailer](https://github.com/PHPMailer/PHPMailer) 发送它们。

- Tasks Worker

[Tasks worker](https://github.com/appwrite/appwrite/blob/master/app/workers/mails.php) 消费来自 `v1-tasks` 队列的消息。 Appwrite 的 Tasks API 允许您安排您的应用程序可能需要在后台运行的任何重复任务。每个任务都是通过定义 CRON 计划和目标 HTTP 端点来创建的。

每个任务都可以使用任何 HTTP 方法、标头或基本 HTTP 身份验证定义任何 HTTP 端点。在 Appwrite 控制台中，您可以查看所有任务、它们的当前状态、上一次和下一次运行时间以及用于查看先前执行结果的响应日志。

- Usage Worker

[Usage worker](https://github.com/appwrite/appwrite/blob/master/app/workers/usage.php) 消费来自 `v1-usage` 队列的消息，并使用 `statsd` 通过 UDP 连接将消息发送到 `Telegraf`。然后将使用统计信息记录在 `influxDB` 中，包括函数执行统计信息、请求总数、存储统计信息等。

- Webhooks Worker

[Webhooks worker](https://github.com/appwrite/appwrite/blob/master/app/workers/webhooks.php) 消费来自 `v1-webhooks` 队列的消息并触发在 Appwrite 控制台中注册的 webhooks。worker检查发生的事件并通过发出 CURL 请求来触发相应的 webhook。Webhooks 允许您构建或设置订阅 Appwrite 上某些[事件](https://appwrite.io/docs/webhooks#events)的集成。当这些事件之一被触发时，我们会向 webhook 的配置 URL 发送一个 HTTP POST 负载。Webhooks 可用于从 CDN 中清除缓存、计算数据或发送 Slack 通知。

此外，我们还有两个workers负责将任务委派给其他workers。

- Maintenance Worker

Maintenance Worker对应 docker-compose 文件中的`appwrite-maintenance`服务。[Maintenance worker](https://github.com/appwrite/appwrite/blob/master/app/tasks/maintenance.php)在这里执行一些内务管理任务，因此您的 Appwrite 服务器不会随着时间的推移而崩溃！在当前状态下，maintenance worker将删除任务委托给 `appwrite-worker-deletes`，然后执行实际删除。我们使用Maintenance worker来调度三种删除：

	- 清理Abuse logs
	- 清理Audit Logs
	- 清理Execution Logs

- Schedules Worker

Schedules worker 对应于 docker-compose 文件中的 `appwrite-schedule` 服务。 Schedules worker 在底层使用 [Resque Scheduler](https://github.com/resque/resque) 并处理跨 Appwrite 的 CRON 作业的调度。这包括来自 Tasks API、Webhooks API 和functions API 的 CRON 作业。

### Mariadb

Appwrite 使用 MariaDB 作为项目集合、文档和所有其他元数据的默认数据库。Appwrite 与您在后台使用的数据库无关，并且目前正在积极开发对 Postgres、CockroachDB、MySQL 和 MongoDB 等更多数据库的支持！

### ClamAV

ClamAV 是一个 TCP 防病毒服务器，负责扫描所有用户上传到 Appwrite 存储。 ClamAV 微服务是可选的，可以使用 Appwrite 环境变量禁用。从 Appwrite 0.8 版开始，默认情况下禁用此功能以节省较小设置的内存。如果遇到内存使用率过高的问题，您可以在[此处](https://dev.to/appwrite/learn-how-to-disable-clamav-in-your-appwrite-stack-and-reduce-memory-usage-2e37)了解如何禁用它。

### Influxdb

Appwrite 使用 InfluxDB 来存储您项目的 API 使用指标和统计信息。这是用于生成 API 使用图和处理时间序列数据的引擎。

### Telegraf

Telegraf 是一个插件驱动的服务器代理，用于从多个来源收集指标和事件并将其发送到多个目的地。 Telegraf 通过在将数据发送到数据库之前聚合数据来保护 InfluxDB。Telegraf 在 UDP 协议上运行，这使得数据传输速度极快！

## Appwrite Dashboard

登录到Appwrite 控制台后就能看到开始页面。可以在此处创建第一个项目。

![](https://res.cloudinary.com/neroblackstone/image/upload/v1624763348/appwrite_dashboard_ooddyh.webp)

创建新项目或选择项目后，可以进入项目主页

### 主页

在主页上，您会看到一些漂亮的图表，这些图表突出显示了您项目的使用统计数据。在这里，可以找到 Appwrite 处理的请求总数、消耗的总带宽、集合中的文档总数、项目中的用户总数等。

![](https://res.cloudinary.com/neroblackstone/image/upload/v1624763513/appwrite_home_pze4h0.webp)

在此下方，可以找到**Platforms**一栏。在这里，您可以向 Appwrite 添加 Web 或 Flutter 项目。添加平台很重要，因为它告诉 Appwrite 受信任的域并限制来自此处未列出的域的请求。这也解决了那些讨厌的 CORS 问题。

让我们从侧栏上的第一项开始，**Database**。

## Database

数据库部分允许您查看、创建和编辑您的集合。它还允许您查看项目中的所有文档。可以在主屏幕创建集合。![](https://res.cloudinary.com/neroblackstone/image/upload/v1624763837/appwrite_database_vnbexd.png)

创建集合后，您可以单击它以进一步配置它。在**设置**选项卡下，您将找到更改集合名称、向集合添加规则、更改集合的读写权限、删除集合等的选项。

![](https://res.cloudinary.com/neroblackstone/image/upload/v1624764027/appwrite_database_settings_mzsmue.png)

将规则视为**关系数据库中的一列**。单击**添加规则**按钮，您可以创建您的第一个规则。你可以给你的规则（列）取一个名字，并给它一个类型。 这里有一个有趣的规则是**文档**规则类型。**文档**规则类型允许您的集合具有来自其他集合的嵌套文档。

![](https://res.cloudinary.com/neroblackstone/image/upload/v1624764225/appwrite_add_rule_odmrz8.jpg)

规则部分下方是权限部分。您可以在此处设置集合级别的权限。此权限控制谁可以读取、创建或更新此集合中的文档。

![](https://res.cloudinary.com/neroblackstone/image/upload/v1624764328/appwrite_dashboard_permissions_bwwbr9.png)

### 存储

侧栏中的第二个选项是**存储**。您可以在此处管理上传到服务器的所有文件。您可以使用右下角的**“+”按钮**上传文件。此对话框还允许您设置文件的读写权限。

您也可以使用 SDK 上传文件。这只是执行此操作的图形界面方式。

![](https://res.cloudinary.com/neroblackstone/image/upload/v1624764665/appwrite_dashboard_storage_jnlbnz.webp)

完成文件上传后，您可以随时更新其权限或将其删除。

![](https://res.cloudinary.com/neroblackstone/image/upload/v1624764764/appwrite_update_storage_zzsdkt.png)

### Users

这是侧边栏中的第三部分，您可以在其中管理所有项目的用户。您可以使用此部分来创建、删除甚至禁用某些用户。

![](https://res.cloudinary.com/neroblackstone/image/upload/v1624765077/appwrite_dashboard_users_mvxcbd.webp)

您也可以在此处创建和管理团队并启用 OAuth providers。我们共有 24 个不同的 OAuth providers，其中大部分是由社区贡献的。因此，如果仍然缺少providers，请随时查看[如何帮助添加新的 OAuth Provider](https://github.com/appwrite/appwrite/blob/master/docs/tutorials/add-oauth2-provider.md)。

![](https://res.cloudinary.com/neroblackstone/image/upload/v1624765115/appwrite_OAuth_Provider_mtc7md.png)

### Functions

侧边栏上的第四个选项是 Cloud Functions - Appwrite 最强大的功能之一！顾名思义，Cloud Functions 允许您和您的用户执行serverless函数。支持 13 种不同的语言环境，包括 Node、PHP、Python、Ruby、Deno、Dart 和 .NET。您可以在[此处](https://appwrite.io/docs/functions)找到有关 Cloud Functions 的更多信息。

![](https://res.cloudinary.com/neroblackstone/image/upload/v1624783477/appwrite_dashboard_functions_j8gfaa.png)

一旦你创建了一个函数，你就可以配置它的行为。你可以添加函数的新代码版本，我们将其称为**标签（tag）**。每个标签都有自己的 ID 和用于执行其代码的专用容器。

在 **Monitors** 一栏，您将找到高亮执行、CPU 利用率和故障的可视化界面。

在 **Logs** 一栏下，您将找到函数每次执行的执行日志。

最后，在**Settings**一栏下，您可以设置执行函数的权限或设置触发函数的 CRON 计划。您还可以设置侦听器来执行由系统事件触发的功能。当服务器上发生某些操作时会发出[系统事件](https://appwrite.io/docs/webhooks#events)，例如创建用户、创建集合、文档等。侦听器可用于触发您的云函数。举个例子：当用户注册您的应用程序时，就会发送一封欢迎电子邮件。我们已经在 [dev.to](https://dev.to/appwrite) 上深入[讨论了这个用例](https://dev.to/appwrite/sending-a-custom-welcome-email-using-appwrite-functions-and-mailgun-225a)。

最后，在 Cloud Function 的 Settings 一栏，您还可以添加此函数可能需要的环境变量。这可能包括您可能正在使用的第三方 API 的 API 密钥。

![](https://res.cloudinary.com/neroblackstone/image/upload/v1624784077/appwrite_functions_settings_ndxbie.png)

### Tasks

侧边栏中的下一个选项是**Tasks（任务**）。可以将任务视为 Cloud Functions 的前身。 Tasks 允许您将 HTTP 请求调度到您可能拥有的任何外部端点，而不是在 Appwrite 服务器上运行您的函数。这可能是您的其他一些后端服务器，您可能在其中设置了一些额外的业务逻辑。![](https://res.cloudinary.com/neroblackstone/image/upload/v1624784594/appwrite_dashboard_Webhooks_b6gif4.png)

### Webhooks

我们列表中的下一项是 **Webhooks**。当[服务器事件](https://appwrite.io/docs/webhooks#events)发生在 Appwrite 内部时，Webhooks 允许您访问第三方端点。它们类似于任务，因为它们可用于访问外部 HTTP 端点，但它们的触发方式与任务（使用 CRON 计划）不同。

![](https://res.cloudinary.com/neroblackstone/image/upload/v1624784295/appwrite_dashboard_task_npfo6e.png)

### API Keys

我们列表中的下一部分是 **API Keys（API 密钥**）。从服务器 SDK 与 Appwrite 交互需要 API 密钥。每个 API 密钥的范围仅限于仅访问所选功能并防止滥用。要创建 API 密钥，只需选择**添加 API 密钥**，选择所需的范围，为您的密钥命名并单击**创建**。您现在可以在服务器端集成中使用此 API 密钥。

![](https://res.cloudinary.com/neroblackstone/image/upload/v1624784799/appwrite_dashboard_api_keys_xtjoyn.webp)

### Settings

在**Settings（设置）**一栏中，您可以找到管理项目的选项。还有邀请成员加入您的项目、设置自定义域、更改项目名称、删除项目等的选项。以及黑暗模式：

![](https://res.cloudinary.com/neroblackstone/image/upload/v1624784990/appwrite_dashboard_dark_trigger_t40u3p.png)

![](https://res.cloudinary.com/neroblackstone/image/upload/v1624785034/appwrite_dark_lwxgjm.jpg)

## 发送你的第一个请求

### 向项目添加平台

向项目添加平台有助于我们验证来自客户的请求。我们根据控制台中项目中添加的平台验证请求的来源。任何与可用平台不匹配的来源都将被拒绝。

### 添加平台

在控制台主页上，您可以找到平台列表和添加平台按钮。要添加新平台，您只需点击“添加平台”按钮并选择可用选项之一。

![](https://res.cloudinary.com/neroblackstone/image/upload/v1624785385/appwrite_adding_platform_w18gcr.png)

### Web

要添加网页平台，您只需要一个**Name（名称）和**（host）主机。 Name 可以是任何你想给你的平台命名的东西，**Host** 可以是你的 web 项目运行所在的域。为了在本地构建和测试 Web 项目，主机可以是 `http://localhost`。

![](https://res.cloudinary.com/neroblackstone/image/upload/v1624785641/appwrite_new_web_mhtbw4.png)

#### Flutter

添加 Flutter 平台时，您有两个选择：Android 和 iOS。您可以根据您正在构建的平台添加 Android 或 iOS 或两者都添加。

一旦您选择了 Android 或 iOS，您只需要一个**name(名称)和**application id(应用程序 ID)。**名称**可以是您想命名平台的任何内容，**应用程序 ID** 可以在您的项目中找到。对于 Android，它位于 Flutter 项目的 `android/app/build.gradle` 文件中。对于 iOS，可以通过在 XCode 中打开您的 iOS 应用程序来找到它。通常，默认情况下，如果您没有更改，则对于 Android 和 iOS 都是相同的。

对于 Flutter web，您将平台添加为 Web，如上所述。

### 向 Appwrite 服务器发出请求

虽然我们可以通过直接的 REST API 调用向 Appwrite 服务器发出请求，但使用 SDK 会使它更加结构化和容易。因此，我们将研究如何使用 Appwrite 的官方 SDK 向 Appwrite 服务器发出请求。

### Web

Appwrite 的 [Web SDK](https://github.com/appwrite/sdk-for-web) 使用起来非常简单。您可以使用 [NPM](https://www.npmjs.com/) 或 [Yarn](https://yarnpkg.com/) 等包管理器将其添加到您的项目中。以下命令将 Appwrite Web SDK 添加到您的项目。

``` shell
    npm install appwrite
```

或者您可以直接从 CDN导入：

``` html
<script src="https://cdn.jsdelivr.net/npm/appwrite@3.1.0"></script>
```

添加依赖项后，您现在可以在项目中使用 Appwrite SDK。为了发出请求，我们首先需要使用端点和项目详细信息初始化 SDK，如下所示：

``` js
var appwrite = new Appwrite();

appwrite
    .setEndpoint('http://localhost/v1') // Set your endpoint
    .setProject('455x34dfkj') // Your Appwrite Project UID
;
```

现在让我们使用我们的 SDK 发出请求。

``` js
// Register User
appwrite
    .account.create('me@example.com', 'password', 'Jane Doe')
        .then(function (response) {
            console.log(response);
        }, function (error) {
            console.log(error);
        });
```

更多信息可以在[Web 入门指南](https://appwrite.io/docs/getting-started-for-web)中找到。

#### Flutter

Appwrite 的 Flutter SDK 非常容易上手。首先，您需要在 `pubspec.yaml` 文件中添加 Appwrite 的 SDK 作为依赖项：

``` yaml
dependencies:
    appwrite: ^0.5.0-dev.1 #use the latest version available
```

添加依赖项并运行 `flutter pub get` 后，您可以在项目中导入和使用 Appwrite SDK。在发出请求之前，您必须首先使用所需的端点和项目 ID 初始化 SDK。

``` dart
import 'package:appwrite/appwrite.dart';

Client client = Client();

client
    .setEndpoint('https://localhost/v1') // Your Appwrite Endpoint
    .setProject('5e8cf4f46b5e8') // Your project ID
    .setSelfSigned() // Remove in production
;
```

现在您可以轻松地发出请求和处理响应：

    // Register User
    Account account = Account(client);
    
    Response user = await account
        .create(
            email: 'me@appwrite.io',
            password: 'password',
            name: 'My Name'
        );

更多信息可以在我们的 [Flutter 入门指南](https://appwrite.io/docs/getting-started-for-flutter)中找到。

## SSL证书

### Appwrite 中的 SSL 证书

#### 什么是 SSL？

SSL 是一种安全协议，它以加密方式为 Internet 上相互通信的计算机提供身份验证，几年前经过改进，后来被 [TLS](https://developer.mozilla.org/en-US/docs/Glossary/TLS) 取代。尽管 TLS 取代了 SSL，但这两个名称通常用于指代相同的过程：使用由[Certificate Authority](https://developer.mozilla.org/en-US/docs/Glossary/Certificate_authority)（证书颁发机构）（简称 CA）签署的证书密钥对（花式文本文件）来保护 HTTP 会话。

#### 信任证书

TLS 协议提供加密唯一的密钥对，不仅提供加密，而且在证书中包括域、主机和组织信息。但是，由于 TLS 技术是开源的，任何人都可以作为 CA 操作并签署证书。为了保证用户的安全，计算机和浏览器附带了预先审查的 CA 列表以自动信任\[1\]。使用这些可信来源颁发的证书的网站会在 URL 栏中的域旁边获得非常重要的锁🔒。然而，没有它们的网站面临着可怕的警告：`潜在的安全风险` 。

> 这里是 Mozilla 的 Firefox[ 可信来源](https://wiki.mozilla.org/CA/Included_Certificates)列表。

![](https://res.cloudinary.com/neroblackstone/image/upload/v1624875085/trusted_sources_f7s2ju.png)

成为这些名单上普遍信任的 CA 的过程可能[代价高昂](https://en.wikipedia.org/wiki/Key_ceremony)，这就是 IdenTrust 和 DigiCert 等组织对其服务收费的原因。这些公司拥有资源和知识来提供一系列安全保障，保护金融机构、政府、军队等。不过，我假设您没有开办银行，也没有资金获得商业 SSL 证书。如何免费成为CA呢？

#### Let's Encrypt

Let's Encrypt 是一个免费、自动化且值得信赖的证书颁发机构，旨在提供更安全、更安全的互联网。Appwrite 在后台使用他们的 [certbot](https://certbot.eff.org/) 工具来自动处理证书生成和更新，因此您可以专注于构建您的应用程序。

### 使用 HTTPS 保护 Appwrite

假设我已经在 VPS 上安装了 Appwrite，并为我的下一个由 Appwrite 驱动的项目购买了域名 `example.com`。在 `example.com`上运行我的应用程序需要哪些步骤？

#### 域记录

您的域名注册商最终可以控制您的域（我们的 `example.com`），因此我们需要从那里开始将域指向您的 VPS 的 IP 地址。为此，我们可以使用 DNS A 记录。向您的域添加 DNS 记录的过程因注册商而异，因此请查看关于[自定义域](https://appwrite.io/docs/custom-domains)的文档，以获取大量有用的链接和更具体的说明。

#### 开发环境中的 SSL 证书

如前所述，生成您自己的 SSL 证书所需的所有技术都是开源的，但它只是不受浏览器的全球信任。这对开发来说完全没问题（假设你相信自己）。Appwrite 提供开箱即用的自签名证书（通过 Traefik 代理），因此您的工作会立即加密。为此，我们需要让 Appwrite 知道我们正在尝试使用自签名证书。Appwrite的 SDK 都可以接受 client.setSelfSigned() 方法来处理这个问题。这是使用我们的 [Web SDK](https://appwrite.io/docs/getting-started-for-web) 的示例：

``` js
import * as Appwrite from "appwrite";

let client = new Appwrite();

client
    .setEndpoint('https://example.com/v1')
    .setProject('5df5acd0d48c2')
    .setSelfSigned()
;
```

#### 生产环境中的 SSL 证书

现在，假设您已经过了 `example.com` 的开发阶段，并且您已准备好进入生产阶段。 Appwrite 需要以下条件来颁发生产就绪的 SSL 证书（带锁🔒）：

* 通过 `_APP_ENV=production` 为 Appwrite开启 `production` 模式
* 通过 `_APP_SYSTEM_SECURITY_EMAIL_ADDRESS` 设置的有效电子邮件
* 通过 `_APP_DOMAIN` 设置的面向公众的域
* Traefik（代理网络服务器）侦听端口 80
* 删除对 client.setSelfSigned 的引用

> 在更改 Appwrite 的配置时，我们关于[ Appwrite 环境变量](https://appwrite.io/docs/environment-variables)的文档是一个很好的参考。

要应用新的环境变量，请运行：

    docker-compose up -d

这是 Appwrite Certificates worker 处理的阶段，调用 `certbot` 生成由 Let's Encrypt 签名的证书。然后，worker 将证书存储在 Docker 卷中以进行持久化，并定期检查证书更新（Let's Encrypt 证书的有效期默认为 90 天）。

### 调试

查找任何证书问题的第一个位置是 Certificates worker。您可以使用以下命令检查服务日志：

    docker-compose logs appwrite-worker-certificates

如果您在 Appwrite 服务器启动后配置了您的域，您可以通过重新启动 Appwrite 来重新触发 Certificates worker：

    docker-compose restart appwrite

## 服务器端 SDK

Appwrite 的愿景强调了这样一个事实，即后端即服务不应该只为前端开发人员设计。基于这一愿景，Appwrite 被设计为与平台无关，并与客户端和服务器端应用程序无缝集成。因为 Appwrite 是自托管的，所以它可以在您现有的防火墙后面使用，并与您现有的后端服务一起使用。Appwrite 的目标不是替换您的后端，而是与其一起工作。

Appwrite 支持 6 个服务器端 SDK。所有的 SDK 都是根据 API 的 Swagger 规范自动生成的。这允许Appwrite的小开发团队总共维护 8 个（客户端 + 服务端）SDK。如果您想帮助Appwrite团队用您喜欢的语言创建 SDK，请随时查看 [SDK Generator](https://github.com/appwrite/sdk-generator)。

### 服务端SDK与客户端SDK有何不同？

#### 身份验证

客户端和服务器端 SDK 之间的主要区别在于身份验证机制。服务器端 SDK 使用作用域 API 密钥来访问 Appwrite API，而客户端 SDK 则依赖安全 cookie 进行身份验证。

#### 范围

第二个主要区别是允许客户端和服务器端 SDK 访问的范围。范围限制了您可以使用 SDK 完成的任务类型。服务器端 SDK 提供了更多功能和灵活性，并允许您控制 Appwrite 的更多方面。使用您的 API 密钥，您可以使用您选择的 SDK 访问 Appwrite 服务。

要创建新的 API 密钥，请使用 Appwrite 控制台转到项目设置中的 API 密钥选项卡，然后单击“添加 API 密钥”按钮。添加新的 API 密钥时，您可以选择要授予应用程序的范围。最佳做法是仅允许满足项目目标所需的权限。如果您需要替换 API 密钥，请创建一个新密钥，更新您的应用凭据，并在准备好后删除旧密钥。

在服务器端使用带有 API 密钥的 Appwrite API 时，您将自动以`admin mode`运行。管理员模式禁用默认的[用户权限访问控制](https://appwrite.io/docs/permissions)限制，并允许您访问项目中的所有服务器资源（文档、用户、集合、文件、团队），无论读写权限如何。当您想要操作用户的数据（如文件和文档）或者甚至想要获取用户列表时，这非常有用。

> 不建议从客户端运行管理模式，因为这会导致巨大的隐私和安全风险。查看管理模式文档以了解更多信息。

下表很好地展示了您可以使用客户端和服务器端 SDK 做什么和不能做什么，并且很好地总结了我们所涵盖的内容。

| 名称 | 描述 | 服务端 | 客户端 |
| --- | --- | --- | --- |
| account | 代表当前登录用户的读写权限 | ❌ | ✅ |
| users.read | 有权读取您的项目的用户 | ✅ | ❌ |
| users.write | 拥有创建、更新和删除项目用户的权限 | ✅ | ❌ |
| teams.read | 有权读取您的项目团队 | ✅ | ✅ |
| teams.write | 拥有创建、更新和删除项目团队的权限 | ✅ | ✅ |
| collections.read | 有权读取项目的数据库集合 | ✅ | ❌ |
| collections.write | 有权创建、更新和删除项目的数据库集合 | ✅ | ❌ |
| documents.read | 有权读取您项目的数据库文档 | ✅ | ✅ |
| documents.write | 有权创建、更新和删除项目的数据库文档 | ✅ | ✅ |
| files.read | 有权读取项目的存储文件和预览图像 | ✅ | ✅ |
| files.write | 有权创建、更新和删除项目的存储文件 | ✅ | ✅ |
| functions.read | 有权读取项目的函数和代码标签 | ✅ | ❌ |
| functions.write | 有权创建、更新和删除项目的函数和代码标签 | ✅ | ❌ |
| execution.read | 有权读取项目的执行日志 | ✅ | ✅ |
| execution.write | 有权执行您的项目功能 | ✅ | ✅ |
| locale.read | 有权访问您项目的 Locale 服务 | ✅ | ✅ |
| avatars.read | 有权访问您项目的 Avatars 服务 | ✅ | ✅ |
| health.read | 有权读取您的项目的健康状态 | ✅ | ❌ |

### 入门

开始使用服务器端 SDK 并发出第一个请求非常简单。对于本示例，我们将选择 Deno SDK - 同样的原则也适用于所有其他 SDK。

前往您的 Appwrite Dashboard 并**创建**一个新项目。为您的项目命名，然后单击“创建”开始。创建项目后，转到 **API 密钥**部分并创建一个具有所需范围的密钥（确保它具有 `users.read` 和 `users.write` 范围，因为示例依赖于此）。复制此密钥，因为我们将在下一步中需要它。还要记下您的**项目 ID** 和 **API Endpoint**，它们可以在 Appwrite 仪表板的“**设置**”部分下找到。

![](https://res.cloudinary.com/neroblackstone/image/upload/v1624879340/appwrite_project_settings_evshwq.png)

是时候初始化您的 SDK 并发出您的第一个请求了。填写您在上一步中复制的所有值。然后我们将尝试使用 Appwrite SDK 创建一个用户。

``` ts
import * as sdk from "https://deno.land/x/appwrite/mod.ts";

let client = new sdk.Client();

client
    .setEndpoint('https://[HOSTNAME_OR_IP]/v1') // Your API Endpoint
    .setProject('5df5acd0d48c2') // Your project ID
    .setKey('919c2d18fb5d4...a2ae413da83346ad2') // Your secret API key
    .setSelfSigned() // Use only on dev mode with a self-signed SSL cert
;
let users = new sdk.Users(client);

let promise = users.create('email@example.com', 'password');

promise.then(function (response) {
    console.log(response);
}, function (error) {
    console.log(error);
});
```

这是您使用 Appwrite 的服务器端 SDK 的第一个请求！如果您想以我们支持的其他语言查看此示例，可以在[此处](https://appwrite.io/docs/getting-started-for-server)查看。

如果您喜欢冒险并且想使用您最喜欢的 HTTP 请求库来使用 Appwrite API，那么[本指南](https://dev.to/eldadfux/learn-how-you-can-take-advantage-of-the-appwrite-api-without-using-any-sdk-a41)就是为此而编写的！

## Accounts和Users API

### 主要区别

| Users API | Accounts API |
| --- | --- |
| 服务器端 API | 客户端 API |
| 使用 API 密钥访问 | 使用 Cookie（或 JWT）访问 |
| 在Admin范围内操作 | 在当前登录的用户范围内操作 |
| 对应用内所有用户执行 CRUD 操作 | 对当前登录的用户执行 CRUD 操作 |

Users API 是服务器端 SDK 规范的一部分，在Admin范围内运行（即使用 API 密钥），可以访问您的所有项目用户。Users API 允许您执行创建、更新、删除和列出应用用户、创建、更新和删除他们的首选项等操作。[Users API 的完整文档](https://appwrite.io/docs/server/users)。

Accounts API 在当前登录用户的范围内运行（使用 cookie 或 JWT）并且通常与客户端集成。Accounts API 允许您创建帐户、使用用户名和密码以及 OAuth2 创建用户会话、更新帐户的电子邮件和密码、启动密码恢复、启动电子邮件验证等。可以在此处找到[ Accounts API 的完整文档](https://appwrite.io/docs/client/account)。

### 深入研究 Accounts API

让我们试着更好地理解 Accounts API。 Accounts API 最常用的方法是 [createSession()](https://appwrite.io/docs/client/account?sdk=web#accountCreateSession) 和 [createOAuth2Session()](https://appwrite.io/docs/client/account?sdk=web#accountCreateOAuth2Session) 方法。如果成功，他们的响应包含一个 `set-cookie` 标头，然后告诉浏览器保存并在每个后续请求中包含此 cookie。在 Flutter（和即将推出的 Android）SDK 中，我们使用 [Cookie Jar / Cookie Store](https://developer.android.com/reference/java/net/CookieStore) 来实现类似的功能。

**Appwrite 0.8** 增加了对**匿名用户**的支持。当您开发应用程序时，有时您可能希望让用户在登录之前与应用程序的某些部分进行交互。这也提高了用户的转化率，因为注册的门槛非常高。如果匿名用户决定注册您的应用程序，他们可以稍后使用他们的电子邮件和密码或 OAuth 方法转换他们的帐户。

在 0.8 版本中，我们的仪表板也得到了更新，让您可以更好地控制项目允许的登录方法。

![](https://res.cloudinary.com/neroblackstone/image/upload/v1625051119/appwrite_login_control_piokue.png)

让我们使用 Accounts API 发出第一个请求。要在完整的应用程序中看到这个例子，请查看[演示应用程序](https://github.com/appwrite?q=todo&type=&language=&sort=)的源代码。

我们将在本教程中使用 JavaScript 示例。无论您使用的是框架还是普通 JS，都非常容易上手 - 我们的 [Web 入门教程](https://appwrite.io/docs/getting-started-for-web)解释了如何使用。安装并初始化 SDK 后，您可以继续操作。

#### create()

如果您想在您的应用程序中实现**注册**功能，则可以使用此方法。请注意，这只会创建一个新用户。您**仍然需要使用**相同的电子邮件和密码调用 `createSession()` 方法来实际为此用户创建新会话。

``` javascript
let promise = sdk.account.create("email@example.com", "password", "name");

promise.then(
  function (response) {
    console.log(response); // Success
  },
  function (error) {
    console.log(error); // Failure
  }
);
```

### createSession()

如果您想在您的应用中实现**登录**功能，这就是您需要的方法。此方法为现有用户创建会话。因此，请确保您在此之前已经调用了 `create()` 。

``` js
// Using the promise syntax
let promise = sdk.account.createSession("email@example.com", "password");
promise.then(
  function (response) {
    console.log(response); // Success
  },
  function (error) {
    console.log(error); // Failure
  }
);

// Or using async/await
const login = async () => {
  try {
    let response = await sdk.account.createSession(
      "email@example.com",
      "password"
    );
    console.log(response);
  } catch (e) {
    console.log(e);
  }
};
login();
```

如果您检查来自 `createSession()` 的响应，您会发现以下标头。

    set-cookie:
    a_session_6062f9c2c09ce_legacy=eyJpZCI6IjYwNmI3Y....NmVhMzQ2In0=; expires=Wed, 27-Apr-2022 14:17:29 GMT; path=/; domain=.demo.appwrite.io; secure; httponly
    
    set-cookie:
    a_session_6062f9c2c09ce=eyJpZCI6IjYwNmI3Y....NmVhMzQ2In0=; expires=Wed, 27-Apr-2022 14:17:29 GMT; path=/; domain=.demo.appwrite.io; secure; httponly; samesite=None
    
    x-fallback-cookies
    {"a_session_6062f9c2c09ce":"eyJpZCI6IjYwNmI3Y....NmVhMzQ2In0="}

Appwrite 会话 cookie 使用以下语法：`a_session_<PROJECT-ID>`、`a_session_<PROJECT-ID>_legacy`。由于许多浏览器禁用 3rd 方 cookie，我们使用 x-fallback-cookies 标头将 cookie 存储在本地存储中，如果尚未设置 cookie，则在后续请求中使用它。

### deleteSession()

为了实现**注销**功能，您需要使用会话 ID 删除会话。您可以通过传递 `current` 代替 `SESSION_ID` 来删除当前会话。

``` js
let promise = sdk.account.deleteSession("[SESSION_ID]");

promise.then(
  function (response) {
    console.log(response); // Success
  },
  function (error) {
    console.log(error); // Failure
  }
);
```

我们只介绍了几个基本方法来传达 API 的工作原理。完整的功能列表可以在[这里](https://appwrite.io/docs/client/account)找到。

### 深入了解Users API

我们也可以使用Users API 实现我们上面讨论的所有功能。但是，您将使用 API 密钥执行所有操作。

#### create()

create 方法可用于创建新用户。请注意，这与使用 Accounts API 创建会话不同。这里不涉及cookie。将此视为管理员，代表他的一个用户创建一个帐户。要创建会话，用户将使用这些凭据从客户端应用程序登录。

``` js
let promise = users.create("email@example.com", "password");

promise.then(
  function (response) {
    console.log(response);
  },
  function (error) {
    console.log(error);
  }
);
```

#### deleteSession()

假设您有一个 Cloud Functions 函数，它可以监控帐户登录并就来自不同位置或 IP 的可疑登录向用户发出警报。在这种情况下，作为预防措施，您可能希望删除会话或完全阻止帐户，直到真正的用户采取行动。在这种情况下，deleteSession() 方法就派上用场了。

``` js
let promise = users.deleteSession("[USER_ID]", "[SESSION_ID]");

promise.then(
  function (response) {
    console.log(response);
  },
  function (error) {
    console.log(error);
  }
);
```

因此，请在构建客户端应用程序时使用 **Accounts API**，在构建服务器端应用程序时使用 **Users API**。

## 登录和注册

对用户进行身份验证和授权以验证他们的请求或保存他们的数据是现代应用程序开发生命周期不可或缺的一部分。这是我们在每个应用程序中都会做的事情。Appwrite 通过抽象出所有关于用户管理的功能，使这个过程变得非常简单，为我们提供了一个漂亮的 API。Appwrite 的身份验证功能支持基于电子邮件和密码的身份验证以及各种 OAuth2 提供商，例如 **Google、Facebook、GitHub、Twitter** 等。您可以在 **Appwrite Console -> Users -> Providers** 中找到所有支持的提供程序的列表。[即将推出的 Appwrite 版本](https://dev.to/appwrite/appwrite-0-8-is-coming-soon-and-this-is-what-you-can-expect-35li)还将添加对匿名登录的支持。

默认情况下，身份验证、用户管理、团队和角色都由 Appwrite 处理。在今天的会议中，我们将继续上次会议的演示并添加身份验证功能。我们将实现注册和登录。让我们开始吧。

### 开始

我们已经创建了一个基础项目，我们将开始在我们的应用程序中使用 Appwrite 实现更多功能。要继续以下部分，您需要克隆或下载我们的基础项目。可以在[这里](https://github.com/christyjacob4/30-days-of-appwrite/tree/starter)找到我们的基础项目。从这里开始，我们将建立在这个初始项目的基础上。所以请克隆或下载它，以便您可以跟上。

### 创建注册和登录组件

首先，我们将创建我们的 Login 和 Signup 组件并设置到这些组件的路由。

#### 注册组件

让我们添加一个注册组件 `src/routes/Register.svelte`。

``` html
<!-- Signup page -->
<script>
  import { api } from "../appwrite";

  let name,
    mail,
    pass = "";

  const submit = async () => {
    try {
      await api.register(mail, pass, name);
    } catch (error) {
      console.log(error.message);
    }
  };
</script>

<div>
  <h1>Register</h1>
  <form on:submit|preventDefault={submit}>
    <label for="mail"><b>Name</b></label>
    <input
      bind:value={name}
      type="text"
      placeholder="Enter Name"
      name="name"
      required
    />

    <label for="mail"><b>E-Mail</b></label>
    <input
      bind:value={mail}
      type="email"
      placeholder="Enter E-mail"
      name="mail"
      required
    />

    <label for="pass"><b>Password</b></label>
    <input
      bind:value={pass}
      type="password"
      placeholder="Enter Password"
      name="pass"
      required
    />

    <button type="submit">Register</button>
  </form>
</div>

<style>
    div {
     margin-left: auto;
     margin-right: auto;
     width: 400px;
   }
    form {
        display: flex;
        flex-direction: column;
    }
</style>
```

如您所见，我们这里的`submit`函数调用了 `api.register(name, mail, pass)`。所以我们需要实现注册功能。

#### 处理注册

让我们创建 `src/appwrite.js` 文件，我们将在其中使用 `Appwrite` SDK 来处理`register`功能。

``` js
import Appwrite from "appwrite"; //importing from Appwrite's SDK
import { state } from "./store"; // saving user data to svelte store

const sdk = new Appwrite();
sdk
  .setEndpoint("https://demo.appwrite.io/v1") //set your own endpoint
  .setProject("607dd16494c6b"); //set your own project id

export const api = {
    register: async (name, mail, pass) => {
        try {
            await sdk.account.create(mail, pass, name);
            await api.login(mail, pass)
        } catch (error) {
            throw error;
        }
    },
}
```

在 `register` 函数中，我们调用了 `sdk.account.create` 方法。我们正在传递要创建的帐户的`mail`, `password`和`name`。

#### 登录组件

让我们添加一个新组件 `scr/routes/Login.svelte`。

``` html
<!-- Login page -->
<script>
  import { api } from "../appwrite";

  let mail,
    pass = "";

  const submit = async () => {
    try {
      await api.login(mail, pass);
    } catch (error) {
      console.log(error.message);
    }
  };
</script>
<div>
  <h1>Login</h1>
  <form on:submit|preventDefault={submit}>
    <label for="mail"><b>E-Mail</b></label>
    <input
      bind:value={mail}
      type="email"
      placeholder="Enter E-mail"
      name="mail"
      required
    />

    <label for="pass"><b>Password</b></label>
    <input
      bind:value={pass}
      type="password"
      placeholder="Enter Password"
      name="pass"
      required
    />

    <button type="submit">Login</button>
  </form>
</div>

<style>
   div {
     margin-left: auto;
     margin-right: auto;
     width: 400px;
   }
    form {
        display: flex;
        flex-direction: column;
    }
</style>
```

如果您查看`submit`函数，我们正在调用 `api.login(mail, pass)` 并且我们的 `api` 是从 `../appwrite` 导入的。所以让我们创建我们的登录函数，它将与 Appwrite 通信以执行登录。

#### 处理登录

再次在 `src/appwrite.js` 中，我们将创建登录功能。

``` js
export const api = {
    //..register code
    login: async (mail, pass) => {
        try {
            await sdk.account.createSession(mail, pass);
            const user = await api.getAccount();
            state.update(n => {
                n.user = user;
                return n;
            });
        } catch (error) {
            state.update(n => {
                n.user = null;
                return n;
            });
            throw error;
        }
    },
}
```

因此，我们首先导入 Appwrite SDK 并使用 `const sdk = new Appwrite();` 创建一个实例。之后，我们需要定义 API 端点和项目 ID。您可以在 Appwrite 仪表板的“设置”页面中找到这些内容。

在我们的登录函数中，我们使用 `sdk.account.createSession(mail, pass)`。如果凭据有效，则来自 `Account` 服务的 `createSession` 方法负责创建用户的会话。最后我们调用 `api.getAccount()` - 让我们也实现这个函数。

``` js
export const api = {
  //...login and register code
  getAccount: async () => sdk.account.get(),
}
```

对于 `getAccount`，我们只是调用 `sdk.account.get()`。来自 `Account` 服务的 `get` 方法将获取具有活动会话的用户的用户配置文件。如果不存在会话，它将抛出错误。

## OAuth提供者

使用 Appwrite，设置许多 OAuth2 提供程序以提供您的用户使用他们的社交帐户进行身份验证非常容易。 Appwrite 支持大量 OAuth2 提供商，包括 Google、Facebook、Twitter、GitHub 等等。最好的部分：大多数 OAuth2 提供程序来自开源社区的拉取请求！

我们将研究如何通过 Appwrite 驱动的应用程序设置和使用 Google 登录。

### 设置 Google OAuth2 设置

在 Appwrite 控制台中，访问 `Users->OAuth2 Providers` 页面。在那里，从列表中找到 Google 并打开开关。为了完成此操作，您将需要可以从 Google API 控制台轻松设置的 `App ID` 和 `App Secret`。查看[他们的官方文档](https://support.google.com/googleapi/answer/6158849)以获取更多详细信息。

![](https://res.cloudinary.com/neroblackstone/image/upload/v1626856697/Appwrite_set_google_oauth2_b6x2aq.png)

获取并填写 Google `App ID` 和 `App Secret` 后，您需要将对话框中显示的回调 URL 提供给 Google OAuth2 的授权重定向 URI。

![](https://res.cloudinary.com/neroblackstone/image/upload/v1626856872/Appwrite_google_client_oath2_i0vrpt.png)

### 使用 OAuth2 提供程序登录

现在我们已经设置了 OAuth2 登录，我们可以请求登录我们的用户。首先，在 `src/routes/Login.svelte` 中添加一个 `Login with Google` 按钮。

``` html
<button on:click|preventDefault={loginWithGoogle}>Login With Google</button>
```

现在我们需要在我们的 Login 组件的脚本中添加一个 `loginWithGoogle` 方法：

``` js
const loginWithGoogle = async () => {
    try {
      await api.loginWithGoogle();
    } catch(error) {
      console.log(error.message);
    }
}
```

最后，在 `src/appwrite.js` 中我们需要添加 `loginWithGoogle`：

``` js
const loginWithGoogle = async () => {
    try {
      await api.loginWithGoogle();
    } catch(error) {
      console.log(error.message);
    }
}
```

最后，在 `src/appwrite.js` 中我们需要添加 `loginWithGoogle`：

``` js
export const api = {
    //....existing code
    loginWithGoogle: async () => {
        try {
            await sdk.account.createOAuth2Session('google', 'http://localhost:3000', 'http://localhost:3000/#/login');
        } catch (error) {
            throw error;
        }
    },
}
```

在这里，我们调用 `sdk.account.createOAuth2Session` 并传入几个参数。第一个参数是**提供者**。对我们来说，它是`google`。第二个参数是登录成功后要重定向到的 URL。第三个参数是登录失败时重定向到的 URL。这里我们提供 localhost，因为我们正在本地测试应用程序。一旦我们在线托管我们的应用程序，我们需要为成功和失败的登录提供更新的 URL。

现在，如果您点击使用 Google 登录，您应该会被带到 Google OAuth 同意屏幕。选择有效的 Google 帐户后，您应该能够登录并重定向回 `http://localhost:3000`。您可以查看我们应用程序的导航栏，查看新帐户名称是否已成功加载。

## SMTP 入门

**SMTP** 代表**简单邮件传输协议**。与任何其他协议一样，它定义了网络上所有计算机需要遵守的一些步骤和准则。SMTP 是 TCP/IP 堆栈中的应用层协议，与称为**邮件传输代理 (MTA)** 的东西密切合作，将您的通信发送到正确的计算机和电子邮件收件箱。

为了在 Appwrite 中启用电子邮件功能，您需要设置正确的 SMTP 配置。由于电子邮件的可传递性可能既棘手又困难，因此将这一责任委托给第三方 SMTP 提供商（如 [MailGun](https://www.mailgun.com/) 或 [SendGrid](https://sendgrid.com/)）通常更容易。这些提供程序通过为您执行大量高级配置和验证来帮助您抽象传递垃圾邮件过滤器的复杂性。

随意向您选择的任何提供商注册并跳到**Configuration**部分，否则继续学习如何从 Sendgrid 获取 SMTP 凭据。

### 设置SendGrid

1. 在此处创建一个 [SendGrid](https://signup.sendgrid.com/) 帐户。
2. 验证用作发件人的单个电子邮件地址的所有权。说明可以在[这里](https://sendgrid.com/docs/ui/sending-email/sender-verification/)找到。
3. 在[电子邮件 API -> 集成指南](https://app.sendgrid.com/guide/integrate)下设置 SMTP 中继并创建 API 密钥。
4. 在下方，您应该会看到在下一步中使用 Appwrite 设置 SendGrid 所需的所有凭据。

### 配置

Appwrite 提供多个[环境变量](https://appwrite.io/docs/environment-variables#smtp)来根据您的需要自定义您的服务器设置。为了启用 SMTP，您需要更改 Appwrite 容器的环境变量。以下对我们很重要：

| Name | Description |
| --- | --- |
| _APP_SMTP_HOST | SMTP 服务器主机名地址。使用空字符串禁用从服务器发送的所有邮件。此变量的默认值是一个空字符串 |
| _APP_SMTP_PORT | SMTP 服务器 TCP 端口。默认为空。 |
| _APP_SMTP_SECURE | SMTP 安全连接协议。默认为空，如果在安全连接上运行，则更改为 'tls'。 |
| _APP_SMTP_USERNAME | SMTP 服务器用户名。默认为空。 |
| _APP_SMTP_PASSWORD | SMTP 服务器用户密码。默认为空。 |

要根据您的需要更改这些变量，请导航到安装 Appwrite 的 `appwrite` 目录并编辑隐藏的 `.env` 文件。

    _APP_SMTP_HOST=smtp.sendgrid.net
    _APP_SMTP_PORT=587
    _APP_SMTP_SECURE=tls
    _APP_SMTP_USERNAME=YOUR-SMTP-USERNAME
    _APP_SMTP_PASSWORD=YOUR-SMTP-PASSWORD

完成更新后，您需要从终端使用以下命令重新启动 Appwrite 堆栈：

``` shell
docker-compose up -d --remove-orphans --build --force-recreate
```

### 就是这样！

转到您的 Appwrite 控制台，从您的帐户注销并尝试通过导航到**忘记密码？**来恢复您的密码。如果您已经使用 SendGrid 设置了 SMTP 服务器 - 这也应该验证您的集成。

如果一切顺利，您应该会收到一封包含重置密码说明的电子邮件。显然，这不是必需的，只是用于检查 SMTP 服务器是否正常工作的测试。

## 电子邮件验证

电子邮件验证和密码恢复是任何应用程序的两个关键功能！让我们学习如何使用 Appwrite 实现这两个功能。

> 这要求您在 Appwrite 上进行 SMTP 设置。

让我们从电子邮件验证开始。 OAuth2 登录不需要电子邮件验证，因为在这种情况下，电子邮件地址已由登录提供商验证。

## 初始化验证

要验证帐户，您必须在为用户创建会话后调用 `createVerification(url)` 方法。此方法会向您用户的电子邮件地址发送一条验证消息，以确认他们是该地址的有效所有者。提供的 URL 应将用户重定向回您的应用程序，并允许您通过验证已提供给用户的 `userId` 和 `secret` 参数来完成验证过程。

``` js
let promise = sdk.account.createVerification('http://myappdomain/verifyEmail');

promise.then(function (response) {
    console.log(response); // Success
}, function (error) {
    console.log(error); // Failure
});
```

URL 获取下一步所需的这两个参数，作为查询参数附加到 URL。例如，如果您将 `http://myappdomain/verify` 传递给该方法，则通过电子邮件提供给用户的 URL 将如下所示：

`http://localhost/verifyEmail?userId=XXXX&secret=YYYY`

## 完整的电子邮件验证

既然用户能够初始化其帐户的验证过程，我们需要通过处理来自电子邮件中提供的 URL 的重定向来完成它。

第一步是检索 URL 中提供的 `userId` 和 `secret` 值。在 JavaScript 中，我们可以像这样得到这些：

``` js
const urlParams = new URLSearchParams(window.location.search);
const userId = urlParams.get('userId');
const secret = urlParams.get('secret');
```

有了这些值，我们现在可以使用 `updateVerification(userId, secret)` 完成电子邮件验证：

``` js
const sdk = new Appwrite();

sdk
    .setEndpoint('https://[HOSTNAME_OR_IP]/v1') // Your API Endpoint
    .setProject('5df5acd0d48c2') // Your project ID
;

let promise = sdk.account.updateVerification(userId, secret);

promise.then(function (response) {
    console.log(response); // Success
}, function (error) {
    console.log(error); // Failure
});
```

现在我们已经成功验证了一个用户！

> 请确保仅在电子邮件中提供的 URL 上执行此方法。在我们的示例中，这将是 http://myappdomain/verifyEmail。

## 完整示例

太好了，现在让我们在演示应用程序中实现上述功能。在 `src/lib/Navigation.svelte` 中创建一个按钮来显示用户的验证状态

``` html
...
{#if !$state.user.emailVerification}
    <button on:click={startEmailVerification}>Not Verified ❌</button>
{:else}
    <p>Verified ✅</p>
{/if}
....
```

在 `<script>` 部分添加一个 `startEmailVerification` 函数。

``` js
... 
import { api } from "../appwrite";

const startEmailVerification = async () => {
    try {
        const url = `${window.location.origin}/#/verifyEmail`
        await api.createVerification(url)
        alert("Verification Email sent")
    } catch (error) {
        alert(error.message)
    }
}
...
```

在 `src/appwrite.js` 创建以下函数

``` js
...
createVerification: async (url) => {
    await sdk.account.createVerification(url);
},
completeEmailVerification: async(userId, secret) => {
    await sdk.account.updateVerification(userId, secret);
},
...
```

我们需要在我们的应用程序中创建一个新页面来接收来自验证电子邮件的重定向并完成验证。

创建一个新文件 `src/routes/VerifyEmail.svelte` 并添加以下内容

``` html
<script>
    import { api } from "../appwrite";
    let urlSearchParams = new URLSearchParams(window.location.search);
    let secret = urlSearchParams.get("secret");
    let userId = urlSearchParams.get("userId");
    console.log(userId,secret);
    const completeEmailVerification = async () => {
        await api.completeEmailVerification(userId, secret)
        window.location.href = "/"
    }
    completeEmailVerification()
</script> 
```

不要忘记在 `src/App.svelte` 中为此页面创建路由😊

``` js
import VerifyEmail from "./routes/VerifyEmail.svelte";

...
const routes = {
    "/": Index,
    "/login": Login,
    "/register": Register,
    "/verifyEmail": VerifyEmail,
    "*": NotFound,
};
...
```

做得好！您刚刚为您的用户启用了`Email Verification`，而没有编写一行后端代码！

## 找回密码

既然用户可以验证他们的帐户，我们还需要让他们能够在丢失密码的情况下恢复他们的帐户。此流程与用于验证帐户的流程非常相似。

## 初始化密码恢复

第一步是使用 `createRecovery(email, url)` 方法向用户发送一封电子邮件，其中包含用于在 URL 中重置密码的临时密钥。

``` js
let promise = sdk.account.createRecovery('email@example.com', 'http://myappdomain/resetPassword');

promise.then(function (response) {
    console.log(response); // Success
}, function (error) {
    console.log(error); // Failure
});
```

如果此调用成功，则会向用户发送一封电子邮件，其中提供一个 URL，该 URL 具有附加到 createRecovery(email, url) 中传递的 URL 的机密和用户 ID 值。

## 完整的密码恢复

恢复页面应提示用户输入新密码两次，并在提交时调用 `updateRecovery(userId, secret, password, passwordAgain)`。就像前面的场景一样，我们从 URL 中获取 `userId` 和 `secret` 值：

``` js
const urlParams = new URLSearchParams(window.location.search);
const userId = urlParams.get('userId');
const secret = urlParams.get('secret');
```

有了这些值，我们可以使用 updateRecovery(userId, secret, password, passwordAgain) 完成密码恢复：

``` js
const sdk = new Appwrite();

sdk
    .setEndpoint('https://[HOSTNAME_OR_IP]/v1') // Your API Endpoint
    .setProject('5df5acd0d48c2') // Your project ID
;
let password; // Assign the new password choosen by the user
let passwordAgain;
let promise = sdk.account.updateRecovery(userId, secret, password, paswordAgain);

promise.then(function (response) {
    console.log(response); // Success
}, function (error) {
    console.log(error); // Failure
});
```

现在我们已经成功重置了用户的密码！

## 完整示例

太好了，是时候写代码了！在 `src/routes/Login.svelte` 中创建一个按钮以允许用户恢复他们的密码

``` html
...

<button class="button" on:click|preventDefault={forgotPassword}>Forgot Password?</button>

....
```

在 `<script>` 部分添加一个 `forgotPassword` 函数。

``` js
import { api } from "../appwrite";

const forgotPassword = async () => {
    const url = `${window.location.origin}/#/resetPassword`
    console.log(url);
    try {
        await api.forgotPassword(mail, url)
        alert("Recovery Email Sent")
    } catch (error) {
        alert(error.message);
    }
}
```

在 `src/appwrite.js` 创建以下函数

``` js
....

forgotPassword: async (email, url) => { 
    await sdk.account.createRecovery(email, url) 
},
completePasswordRecovery: async (userId, secret, pass, confirmPass) => { 
    await sdk.account.updateRecovery(userId, secret, pass, confirmPass) 
},

...
```

我们需要在我们的应用程序中创建一个新页面来接收来自密码恢复电子邮件的重定向。创建一个新文件 `src/routes/ResetPassword.svelte` 并将以下代码添加到其中。

不要不知所措。它只是从 URL 参数中获取 `userId` 和 `secret`，要求用户输入新密码，请求 `updateRecovery` 并在成功时将用户重定向到登录页面。

``` html
<script>
    import { api } from "../appwrite";
    let urlSearchParams = new URLSearchParams(window.location.search);
    let secret = urlSearchParams.get("secret");
    let userId = urlSearchParams.get("userId");
    let password = "",
        confirmPassword = "";
    const submit = async () => {
        try {
            await api.completePasswordRecovery(
                userId,
                secret,
                password,
                confirmPassword
            );
            window.location.href = "/#/login";
        } catch (error) {
            alert(error.message);
        }
    };
</script>

<div>
    <h1>Reset your password</h1>

    <form on:submit|preventDefault={submit}>
        <label for="password"><b>New Password</b></label>
        <input
            bind:value={password}
            type="password"
            placeholder="Enter New Password"
            name="password"
            required />

        <label for="confirmPassword"><b>Confirm Password</b></label>
        <input
            bind:value={confirmPassword}
            type="password"
            placeholder="Confirm Password"
            name="confirmPassword"
            required />

        <button class="button" type="submit">Reset Password</button>
    </form>
</div>

<style>
    div {
        margin-left: auto;
        margin-right: auto;
        width: 400px;
    }
    form {
        display: flex;
        flex-direction: column;
    }
</style>
```

不要忘记在 `src/App.svelte` 中为此页面创建路由😊

``` js
import ResetPassword from "./routes/ResetPassword.svelte";

...
const routes = {
    "/": Index,
    "/login": Login,
    "/register": Register,
    "/resetPassword": ResetPassword,
    "/verifyEmail": VerifyEmail,
    "*": NotFound,
};
...
```

惊人的！如果一切顺利，您的用户现在将能够重置他们的密码！！

## 团队

今天我们将通过 Teams API 并了解它如何让我们轻松管理用户组的权限。 Teams API 的主要目的是创建用户组并以简单的方式授予批量权限。然后，这些权限可用于控制对 Appwrite 资源（如存储中的文档和文件）的访问。

假设您有一个想要与一群朋友共享的文本文件。在这种情况下，您可以创建一个团队并为团队成员分配不同的角色。 （查看、编辑、评论、所有者等）

Appwrite 中的团队权限使用以下语法之一

* **team:\[TEAM_ID\]**

  此权限授予特定团队的任何成员访问权限。要获得此权限，用户必须是团队创建者（所有者），或者收到并接受加入此团队的邀请。
* **member:\[MEMBER_ID\]**

  此权限授予团队特定成员的访问权限。只有当用户仍然是特定团队的活跃成员时，此权限才有效。要查看用户的成员 ID，请使用 [Get Team Memberships](https://appwrite.io/docs/client/teams?sdk=web#teamsGetMemberships) 端点获取团队成员列表。
* **team:\[TEAM_ID\]/\[ROLE\]**

  此权限授予在团队中拥有特定角色的任何成员的访问权限。要获得此权限，用户必须是特定团队的成员并具有分配给他或她的给定角​​色。邀请用户成为团队成员时，可以分配团队角色。 `ROLE` 可以是任何字符串。但是，当从客户端 SDK 创建新团队时，会自动创建`owner`角色。

让我们举几个例子来更清楚地说明这一点

| Permission | Description |
| --- | --- |
| team:abcd | 访问团队 abcd 的所有成员 |
| team:abc | 访问团队 abc 的所有成员 |
| member:abc | 访问具有membershipId abc 的用户 |
| team:abcd/owner | 访问具有角色owner的 abcd 团队成员。默认情况下，只有团队的创建者具有此角色。 |
| team:abcd/viewer | 访问具有角色viewer的 abcd 团队成员。 |

可以从客户端和服务器 SDK 访问团队 API。我们将介绍如何使用客户端和服务器 SDK 创建这些团队并分配角色😊。

## 陷阱

从客户端 SDK 创建团队时与使用服务器 SDK 创建团队时存在一些显着差异。

当用户使用客户端 SDK 创建团队时，他们将成为团队的所有者并被自动分配 `team:[TEAM_ID]/owner` 角色。

当使用 API 密钥使用服务器 SDK 创建团队时，由于 API 密钥在[管理员模式](https://appwrite.io/docs/admin)下运行，因此没有团队的逻辑所有者。在这种情况下，Server SDK 还应创建团队的第一个成员，并明确分配所有者权限。我们将用一个例子来介绍这些。

## 客户端SDK

您可以在此处找到 [Client Teams API](https://appwrite.io/docs/client/teams) 的文档。创建一个团队真的很简单——你需要做的就是想一个很酷的名字。

``` js
let promise = sdk.teams.create('Really Cool Name');
promise.then(function (response) {
    console.log(response); // Success
}, function (error) {
    console.log(error); // Failure
});
```

这将创建一个以当前用户为`owner`的团队。您可以通过前往 **Appwrite 控制台 > 用户 > 团队 > 非常酷**的名称来验证这一点

![](https://res.cloudinary.com/neroblackstone/image/upload/v1626944992/appwrite_team_name_b5g0cn.png)

要向该团队添加新成员，您可以使用 `createMembership()`函数。只有团队的`owners`才能向团队添加新成员。包含加入团队链接的电子邮件将发送到新成员的电子邮件地址。如果项目中不存在该成员，则会自动创建该成员。

> 在此之前，请确保您在 Appwrite Server 上设置了 SMTP。

假设您想邀请一名新成员 ( `email@example.com` ) 加入您的团队，并授予他们团队中的两个新角色，即：`viewer`和`editor`。您可以使用以下代码段执行此操作。使用“URL”参数将用户从邀请电子邮件重定向回您的应用。当用户被重定向时，使用[更新团队成员状态](https://appwrite.io/docs/client/teams?sdk=web#teamsUpdateMembershipStatus)端点来允许用户接受团队邀请。

``` js
let promise = sdk.teams.createMembership('[TEAM_ID]', 'email@example.com', '', ['viewer', 'editor'], 'https://example.com/acceptTeamInvite');

promise.then(function (response) {
    console.log(response); // Success
}, function (error) {
    console.log(error); // Failure
});
```

当用户从他们的收件箱点击团队邀请电子邮件时，他们将被重定向到 `https://example.com/acceptTeamInvite?teamId=xxx&inviteId=yyy&userId=zzz&secret=xyz` 。然后可以从查询字符串中提取四个参数，并且可以调用 `updateMembershipStatus()` 方法来确认团队成员身份。

``` js
let promise = sdk.teams.updateMembershipStatus('[TEAM_ID]', '[INVITE_ID]', '[USER_ID]', '[SECRET]');

promise.then(function (response) {
    console.log(response); // Success
}, function (error) {
    console.log(error); // Failure
});
```

我们将在我们的博客应用程序中添加支持以邀请用户加入团队！

## 服务器 SDK

该函数的服务器版本看起来与客户端版本非常相似，但这里的主要区别在于使用具有 `team.read` 和 `team.write` 范围的 API 密钥。此函数创建一个团队，但与 Client SDK 不同的是，该团队还没有成员。

``` js
const sdk = require('node-appwrite');

// Init SDK
let client = new sdk.Client();

let teams = new sdk.Teams(client);

client
    .setEndpoint('https://[HOSTNAME_OR_IP]/v1') // Your API Endpoint
    .setProject('5df5acd0d48c2') // Your project ID
    .setKey('919c2d18fb5d4...a2ae413da83346ad2') // Your secret API key
;

let promise = teams.create('Really Cool Team');

promise.then(function (response) {
    console.log(response);
}, function (error) {
    console.log(error);
});
```

我们需要使用 `createMembership()` 的服务器版本明确地向这个团队添加成员。这里的参数和Client版本完全一样。

``` js
const sdk = require('node-appwrite');

// Init SDK
let client = new sdk.Client();

let teams = new sdk.Teams(client);

client
    .setEndpoint('https://[HOSTNAME_OR_IP]/v1') // Your API Endpoint
    .setProject('5df5acd0d48c2') // Your project ID
    .setKey('919c2d18fb5d4...a2ae413da83346ad2') // Your secret API key
;

let promise = teams.createMembership('[TEAM_ID]', 'email@example.com', '', ['owner'], 'https://example.com/acceptTeamInvite');

promise.then(function (response) {
    console.log(response);
}, function (error) {
    console.log(error);
});
```

当新成员从服务器添加到团队时，不需要电子邮件验证，因此在这种情况下不会发送电子邮件。

您现在知道如何从客户端和服务器为您的团队创建和新成员。

## 团队邀请

我们将在本文的演示应用程序中实现以下功能。

1. [创建团队](https://appwrite.io/docs/client/teams?sdk=web#teamsCreate)
2. [列出用户的团队](https://appwrite.io/docs/client/teams?sdk=web#teamsList)
3. [删除团队](https://appwrite.io/docs/client/teams?sdk=web#teamsDelete)
4. [通过 ID 获取团队](https://appwrite.io/docs/client/teams?sdk=web#teamsGet)
5. [获取团队成员](https://appwrite.io/docs/client/teams#teamsGetMemberships)
6. [添加新的团队成员](https://appwrite.io/docs/client/teams#teamsCreateMembership)
7. [更新会员状态](https://appwrite.io/docs/client/teams#teamsUpdateMembershipStatus)
8. [从团队中删除用户](https://appwrite.io/docs/client/teams#teamsDeleteMembership)

我们将在我们的项目中创建三个新路由。

1. 一个 `/profile/:id/teams` 路由，允许用户查看他们所属的所有团队并创建新团队。这条路线将实现功能 \[1,2,3\]
2. `/team/:id` 路由将显示特定团队 ID 的详细信息，并允许用户管理团队成员。此路线将实现功能 \[3,4,5,6,8\]
3. `/acceptMembership` 路由，使新团队成员能够接受团队邀请。此路由将实现功能 \[7\]

## 设置

让我们开始吧。在 `src/App.svelte` 中创建三个新路由。

``` js
import Team from "./routes/Team.svelte";
import Teams from "./routes/Teams.svelte";
import AcceptMembership from "./routes/AcceptMembership.svelte";
const routes = {
    ...
    "/profile/:id/teams" : Teams,
    "/team/:id" : Team,
    "/acceptMembership": AcceptMembership,
    ...
};
```

转到 `src/appwrite.js` 并添加以下函数：

``` js
...
fetchUserTeams: () => sdk.teams.list(),
createTeam: name => sdk.teams.create(name),
deleteTeam: id => sdk.teams.delete(id),
getTeam: id => sdk.teams.get(id),
getMemberships: teamId => sdk.teams.getMemberships(teamId),
createMembership: (teamId, email, roles, url, name) =>
    sdk.teams.createMembership(teamId, email, roles, url, name),
updateMembership: (teamId, inviteId, userId, secret) =>
    sdk.teams.updateMembershipStatus(teamId, inviteId, userId, secret),
deleteMembership: (teamId, inviteId) =>
    sdk.teams.deleteMembership(teamId, inviteId)
...
```

在 `src/lib/Navigation.svelte` 中，我们将创建一个指向主 `/profile/:id/teams` 路由的链接。

``` html
...
{#if $state.user}
    <a href={`/profile/${$state.user.$id}`} use:link>{$state.user.name}</a>
    <a href={`/profile/${$state.user.$id}/teams`} use:link>My Teams</a>
    <a href="/logout" use:link>Logout</a>
{:else}
...
```

## 创建一个页面来显示所有用户的团队

创建文件 `src/routes/Teams.svelte`。我们将在此处显示用户的所有团队并允许用户创建更多团队。在 `<script>` 部分中添加以下代码。

``` html
<script>
  import { link } from "svelte-spa-router";
  import Avatar from "../lib/Avatar.svelte";
  import Loading from "../lib/Loading.svelte";
  import { api } from "../appwrite";
  export let params = {};
  let name;
  const fetchUser = () => api.fetchUser(params.id);
  const getAvatar = (name) => api.getAvatar(name);
  const fetchTeams = () => api.fetchUserTeams().then((r) => r.teams);
  const createTeam = (name) => api.createTeam(name);
  const deleteTeam = (id) => api.deleteTeam(id);
  let all = Promise.all([fetchUser(), fetchTeams()]);
</script>
```

现在让我们写一些基本的标记：

``` html
<section>
    {#await all}
        <Loading />
    {:then [author, teams]}
        <section class="author">
            <Avatar src={getAvatar(author.name)} />
            <h3>{author.name}</h3>
        </section>
        <section>
            <h1>My Teams</h1>
            <ul>
                {#each teams as team}
                    <li>
                        <a href={`/team/${team.$id}`} use:link>{team.name}</a>
                        <button
                            on:click={async () => {
                                await deleteTeam(team["$id"]);
                                all = Promise.all([
                                    author,
                                    fetchTeams(),
                                ]);
                                console.log("Deleted team", team["$id"]);
                            }}>❌</button>
                    </li>
                {/each}
            </ul>
        </section>

        <section>
            <h1>Create Team</h1>
            <div>
                <label for="team" />
                <input
                    type="text"
                    name="team"
                    placeholder="Enter Team Name"
                    bind:value={name} />
                <button
                    on:click={async () => {
                        await createTeam(name);
                        all = Promise.all([author, fetchTeams()]);
                        console.log("team created");
                    }}>Create Team</button>
            </div>
        </section>
    {:catch error}
        {error}
        <p>
            Public profile not found
            <a href="/profile/create" use:link>Create Public Profile</a>
        </p>
    {/await}
</section>
```

上述标记执行以下操作。

* 显示用户所属的团队列表
* 定义删除团队的按钮。
* 定义用于创建新团队的按钮。

接下来，让我们创建一个页面来显示由上面标记中的 <a> 标记定义的每个团队的详细信息。

## 创建一个页面来显示特定团队的详细信息

创建一个新文件 `src/routes/Team.svelte`。 在 `<script>` 标签下添加以下内容：

``` html
<script>
    import { link } from "svelte-spa-router";
    import Loading from "../lib/Loading.svelte";
    import { api } from "../appwrite";
    import { state } from "../store";
    export let params = {};
    let name = "",
        email = "";
    const fetchTeam = () => api.getTeam(params.id);
    const fetchMemberships = () =>
        api.getMemberships(params.id).then(r => r.memberships);
    const createMembership = (email, name) =>
        api.createMembership(
            params.id,
            email,
            ["member"],
            `${window.origin}/#/acceptMembership`,
            name
        );
    const deleteMembership = async (teamId, membershipId) => {
        try {
            await api.deleteMembership(teamId, membershipId);
            all = Promise.all([fetchTeam(), fetchMemberships()]);
        } catch (error) {
            alert(error.message);
        }
    };
    let all = Promise.all([fetchTeam(), fetchMemberships()]);
</script>
```

让我们添加一些标记来定义布局

``` html
<section>
    {#await all}
        <Loading />
    {:then [team, memberships]}
        <section>
            <div class="header">
                <h1>{team.name}</h1>
                <button
                    on:click={async () => {
                        api.deleteTeam(params.id).then(() => {
                            window.history.go(-1);
                        });
                    }}>❌ Delete Team</button>
            </div>
            <div>
                <label for="email" />
                <input
                    type="text"
                    name="email"
                    placeholder="Enter Email Address"
                    bind:value={email} />
                <label for="name" />
                <input
                    type="text"
                    name="name"
                    placeholder="Enter Name"
                    bind:value={name} />
                <button
                    on:click={async () => {
                        await createMembership(email, name);
                        all = Promise.all([fetchTeam(), fetchMemberships()]);
                        console.log("membership created");
                    }}>➕ Add Member</button>
            </div>
            <h3>Members</h3>
            <ul>
                {#each memberships as member}
                    <li>
                        <div>
                            <div>
                                <p>Name : {member.name}</p>
                                {#if member.userId != $state.user.$id}
                                <button on:click={() => deleteMembership(params.id, member.$id)}
                                    >❌ Delete Member</button>
                                {/if}
                            </div>

                            <p>Email: {member.email}</p>
                            <p>
                                Invited on : {new Date(member.invited * 1000)}
                            </p>
                            <p>Joined on : {new Date(member.joined * 1000)}</p>
                            <p>Confirmed : {member.confirm}</p>
                            <p>Roles : {member.roles}</p>
                        </div>
                    </li>
                {/each}
            </ul>
        </section>
    {:catch error}
        {error}
        <p>
            Team not found
            <a href="/" use:link>Go Home</a>
        </p>
    {/await}
</section>
```

我们将忽略这里的样式。有关样式的更多详细信息，您可以查看[项目的 repo](https://github.com/christyjacob4/30-days-of-appwrite)。

上面的标记做了几件事

* 显示特定团队中的成员列表。
* 允许用户向团队添加新成员
* 允许用户从团队中删除成员。
* 允许用户删除团队。

## 创建一个页面来接受团队成员资格

当我们单击 `➕ 添加成员`按钮时，会向受邀者发送一封带有邀请链接的电子邮件。该链接应重定向回您的应用程序，您需要在其中调用 [Update Membership status ](https://appwrite.io/docs/client/teams#teamsUpdateMembershipStatus)方法以确认会员资格。在我们的例子中，该链接会将用户带到 `https://<your domain>/#/acceptMembership`。对于已经在您的应用中拥有帐户的用户，它只需将他们添加到团队中即可。对于新用户，除了将他们添加到团队之外，它还为他们创建一个新帐户。

创建一个新文件 `src/routes/AcceptMembership.svelte` 并在 `<script>` 部分添加以下代码：

``` html
<script>
    import { api } from "../appwrite";
    let urlSearchParams = new URLSearchParams(window.location.search);
    let inviteId = urlSearchParams.get("inviteId");
    let secret = urlSearchParams.get("secret");
    let teamId = urlSearchParams.get("teamId");
    let userId = urlSearchParams.get("userId");
    api.updateMembership(teamId, inviteId, userId, secret).then(() => {
        window.location = "/"
    });
</script> 
```

就像那样，您现在可以在您的应用程序中创建和管理团队！

## 数据库

Appwrite 提供了一个易于使用的、基于文档的数据库 API，用于存储应用程序的数据。我们在 MariaDB 的基础上构建了我们的 NoSQL 接口，其灵感来自做同样事情的 [Wix](https://www.wix.engineering/post/scaling-to-100m-mysql-is-a-better-nosql)。MariaDB 提供久经考验的稳定性和性能，您可以使用现有的、熟悉的数据库工具（如 MySQLWorkbench、phpMyAdmin 等）管理 Appwrite。集合、文档、规则和权限都可以使用 Appwrite 控制台或我们的任何一款 [SDK](https://appwrite.io/docs/sdks) 进行管理。有很多东西要介绍，所以让我们深入研究。

> 正在进行数据库重构以提高整体性能并增加对各种数据库的支持，包括 PostgreSQL、MongoDB 等！

### 词汇表

每个数据库都有自己的一套技术术语 - 在我们走得太远之前，让我们回顾一下我们的术语。

* **集合**：一组**文档**。每个**集合**都有定义其**文档**结构和_读写_**权限**的**规则**。
* **文档**：_键_和_值_的结构化 JSON 对象，属于一个**集合**。_键_及其类型在**集合规则**中定义。
* **规则**：每个**文档**属性的定义。它有一个_标签、键和规则类型_。将它们视为传统关系数据库中的列。列有名称、值和类型
* **权限**：定义对存储中的**文档、集合**和文件的访问控制的字符串数组。

现在，让我们更详细地回顾一下。

### 集合和文件

简而言之：`集合`保存`文档`。如果您是 SQL 老手，您可能更了解这些`表`和`行`（在内部，这在技术上是正确的）。每个集合都获得一个唯一的随机`集合 ID` 并保存文档（换句话说，原始数据）。 Appwrite 接受的数据类型由为集合定义的**属性规则**控制。

### 规则

简而言之，**规则**概述了您的文档应该是什么样子。通过这种方法，Appwrite 的规则验证器确保进入数据库的数据是您期望的确切格式。因此，对于我们文档的每个键值对，我们提供：

* 一个_标签_，仅用于展示
* _键_的名称
* 规则_类型_
* _默认_值
* 键是否_必须_
* 值是否是一个_数组_

以下是可用于_规则类型_的验证器：

| Rule Type | Description |
| --- | --- |
| text | 任何字符串值。 |
| numeric | 任何整数或浮点值。 |
| boolean | 任何布尔值。 |
| wildcard | 任何值。 |
| url | 任何有效的 URL。 |
| email | 任何有效的电子邮件地址 |
| ip | 任何有效的 IPv4 或 IPv6 地址 |

### 权限

为了控制对资源的访问，Appwrite 为开发人员提供了一个灵活的权限系统，该系统了解 Appwrite 用户和团队。让我们介绍最常用的权限：

| Permission | Description |
| --- | --- |
| * | 通配符权限。授予任何人读或写访问权限。 |
| user:\\\[userID\\\] | 通过用户 ID 向特定用户授予访问权限。 |
| team:\\\[teamID\\\] | 向特定团队的任何成员授予访问权限。注意：用户必须是团队所有者或已接受团队邀请才能授予此访问权限。 |
| team:\\\[teamID\\\]/\\\[role\\\] | 授予在团队中拥有特定角色的任何成员的访问权限。可以在受邀时分配角色。 |
| member:\\\[memberID\\\] | 仅当团队的特定成员仍然是团队成员时才授予他们访问权限。 |
| role:guest | 向任何未登录的访客用户授予访问权限。 |
| role:member | 授予任何登录用户（具有有效会话的用户）的访问权限。登录用户无权访问 role:guest 资源。 |

> 注意：文档不会从其父集合继承权限。

### 过滤器

`listDocuments()` 接受一组过滤器字符串，以便您可以从 Appwrite 数据库中提取所需的文档。过滤器由以下部分组成：

* 要_过滤_的属性
* 比较运算符：`=、!=、>、<、<=、>=`之一
* 目标_值_

以下是使用我们的 **Books** 集合的一些示例：

* `'title=The Hobbit'`
* `'director!=Woody Allen'`
* `'published>=2000'`

### 把这一切放在一起

举个例子，让我们在 Appwrite 中创建一个书籍集合。虽然一些项目需要以编程方式创建集合，但其他项目使用 Appwrite 控制台更容易创建。

![](https://res.cloudinary.com/neroblackstone/image/upload/v1627024386/appwrite_new_collection_ffmov1.png)

一本书有_书名、作者_和出版年份。让我们添加这些，从使用_文本_规则类型的_标题_开始：![](https://res.cloudinary.com/neroblackstone/image/upload/v1627024488/appwrite_add_rule_dbva5p.png)

如果您看到，默认情况下新规则不是_必须_的。让我们让_标题_必须：

![](https://res.cloudinary.com/neroblackstone/image/upload/v1627024645/appwrite_rule_requied_ube74h.png)

现在，我们可以使用出版年份的_数字_规则类型对_作者_和_出版_做同样的事情，所以我们现在有：

![](https://res.cloudinary.com/neroblackstone/image/upload/v1627024750/appwrite_rules_all_a6afws.png)

### 权限，举例

现在我们的 Books 集合具有必要的规则，我们可以根据需要创建文档并限制访问。查看以下代码：

``` js
let sdk = new Appwrite();
sdk
    .setEndpoint('https://[HOSTNAME_OR_IP]/v1') // Your API Endpoint
    .setProject('5df5acd0d48c2') // Your project ID
;

let promise = sdk.database.createDocument(
    '609bdea2f0f99', // collectionID for Books
    {'title': 'The Great Gatsby', 'author': 'F. Scott Fitzgerald', 'published': 1925},
    ['role:member'],
    ['team:5c1f88b87435e/owner', 'user:6095f2933a96f']);
```

在此示例中，任何登录用户都可以读取来自 `createDocument` 的新书，但只有 Team **5c1f88b87435e** 的**所有者**和用户 **6095f2933a96f** 具有写入（或更新）的权限。

### 嵌套文档

还有一种直到现在我才提到的规则类型：嵌套`document`，当您需要将一个文档存储在另一个文档中时。这最好通过示例来说明，所以让我们使用我们的 **Books** 集合。

假设我们还有一个 **Authors** 集合来保持我们的数据井井有条。使用嵌套`document`规则，我们可以在创建新书时交叉引用 **Authors** 集合。

``` js
let promise = database.createDocument(
'609bdea2f0f99', // collectionID for Books
{
  'title': 'The Great Gatsby',
  'published': 1925,
  'author': {
      '$collection': '', // The actors collection unique ID
      '$permissions': {'read': ['role:member'], 'write': ['team:5c1f88b87435e/owner', 'user:6095f2933a96f']}, // Set document permissions
      'name': 'F. Scott Fitzgerald'
    }
  ]
},
['*'], // Read permissions
);
```