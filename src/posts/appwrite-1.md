---
title: Appwrite学习笔记(二)
description: 了解Dashboard并发送第一个请求
date: 2021-06-26
img: https://res.cloudinary.com/neroblackstone/image/upload/v1624671830/appwrite_i2voda.webp
tags:
- Appwrite

---
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