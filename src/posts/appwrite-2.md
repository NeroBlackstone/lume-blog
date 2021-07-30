---
title: Appwrite学习笔记(三)
description: Appwrite补充内容
date: 2021-07-30
img: https://res.cloudinary.com/neroblackstone/image/upload/v1624671830/appwrite_i2voda.webp
tags:
- Appwrite

---
## Appwrite头像API

今天我们将看一看 Appwrite 的 Avatars API 并查看它在引擎盖下的一些简洁功能！Avatars API 主要允许您为各种用例生成图标和头像。让我们深入了解一下它提供了什么。

### 信用卡图标

您可以轻松获取最受欢迎的信用卡公司的信用卡图标，例如 AmEx、Discover、JCB、Mastercard、Visa、Maestro 等。

此[端点](https://appwrite.io/docs/client/avatars#avatarsGetCreditCard)还允许您在请求时自定义图标大小和质量。您可以在[此处](https://github.com/appwrite/appwrite/tree/master/app/config/avatars/credit-cards)找到支持的信用卡的完整列表。

### 浏览器图标

该[端点](https://appwrite.io/docs/client/avatars?sdk=web#avatarsGetBrowser)可以方便您获取一些常用浏览器的图标。如果您还没有看到，我们在 Appwrite 控制台中使用此端点来显示有关用户会话的信息。

![](https://res.cloudinary.com/neroblackstone/image/upload/v1627373284/appwrite_browser_icon_ce7mnr.png)

您可以在[此处](https://github.com/appwrite/appwrite/tree/master/app/config/avatars/browsers)找到支持的浏览器图标的完整列表。

### 国旗

与浏览器图标端点类似，此[端点](https://appwrite.io/docs/client/avatars#avatarsGetFlag)允许您获取国家/地区标志的图标。您可以在上面的 Appwrite 控制台屏幕截图中看到它正在使用中。您可以在[此处](https://github.com/appwrite/appwrite/tree/master/app/config/avatars/flags)找到所有国家/地区代码和旗帜的完整列表。

### 来自 URL 的图像

如果您需要在应用程序中裁剪和显示远程图像，或者您想确保使用 TLS 协议正确提供 3rd 方图像，则[此端点](https://appwrite.io/docs/client/avatars#avatarsGetImage)非常有用。

### 获取网站图标

收藏夹图标是与网站、网页或 Web 应用程序相关联的小图标或图标集合。它显示在浏览器选项卡和书签栏中。[此端点](https://appwrite.io/docs/client/avatars#avatarsGetFavicon)允许您获取任何远程 URL 的图标。![](https://res.cloudinary.com/neroblackstone/image/upload/v1627373520/appwrite_favicon_zwr3iv.png)

### 二维码

此[端点](https://appwrite.io/docs/client/avatars#avatarsGetQR)允许您为任何字符串生成二维码。您如何使用它仅受您的创造力的限制，因为它可用于共享 URL、电话号码，甚至 base64 编码的图像。我们将利用此功能在我们的演示应用程序中添加社交共享功能。

### 获取用户姓名首字母

[此端点](https://appwrite.io/docs/client/avatars?sdk=web#avatarsGetInitials)提供了一种非常方便的方法来根据用户的姓名首字母为您的用户获取头像。您可以将其用作占位符，直到用户上传个人资料图片。您还可以使用此端点为任何字符串（不一定是名称）生成头像。此外，如果您对默认设置不满意，您可以调整图像大小、文本颜色和背景颜色。

## 让我们写一些代码

在我们的演示应用程序中，我们将添加一个分享文章功能。这将允许用户将文章分享到各种社交媒体平台，甚至为当前 URL 生成二维码，然后您可以与朋友分享。

第一步是在 `src/appwrite.js` 中添加一个新函数来调用 Avatars 服务：

``` js
export const api = {
    ...
    getQRcode: text => sdk.avatars.getQR(text)
    ...
}
```

现在网络层已准备就绪，让我们前往 `src/routes/Post.svelte` 组件，我们将在其中创建用于共享的按钮。将以下标记复制到 HTML 的最后一部分：

``` html
<!-- Share -->
<section>
  <div class="share-buttons-container">
    <div class="share-list">
      <!-- FACEBOOK -->
      <a class="fb-h" on:click="{fbs_click}" target="_blank">
        <img
          src="https://img.icons8.com/material-rounded/96/000000/facebook-f.png"
        />
      </a>

      <!-- TWITTER -->
      <a class="tw-h" on:click="{tbs_click}" target="_blank">
        <img
          src="https://img.icons8.com/material-rounded/96/000000/twitter-squared.png"
        />
      </a>

      <!-- LINKEDIN -->
      <a class="li-h" on:click="{lbs_click}" target="_blank">
        <img
          src="https://img.icons8.com/material-rounded/96/000000/linkedin.png"
        />
      </a>

      <!-- REDDIT -->
      <a class="re-h" on:click="{rbs_click}" target="_blank">
        <img src="https://img.icons8.com/ios-glyphs/90/000000/reddit.png" />
      </a>

      <!-- PINTEREST -->
      <a
        data-pin-do="buttonPin"
        data-pin-config="none"
        class="pi-h"
        on:click="{pbs_click}"
        target="_blank"
      >
        <img src="https://img.icons8.com/ios-glyphs/90/000000/pinterest.png" />
      </a>

      <!-- QR Code -->
      <a class="pi-h" on:click="{qrcode_click}" target="_blank">
        <img
          src="https://img.icons8.com/ios-glyphs/60/000000/qr-code--v1.png"
        />
      </a>
    </div>
  </div>
  {#if qrCode}
  <img src="{qrCode}" alt="No QR Code" />
  {/if}
</section>
```

我们还需要为此添加一些样式。我建议从[这里](https://github.com/christyjacob4/30-days-of-appwrite/blob/add-qr-code-share/src/routes/Post.svelte#L157-L209)复制所有样式。

现在是时候添加一些 Javascript 将它们拼接在一起了。在 `src/routes/Post.svelte` 的 `<script>` 部分添加以下代码：

``` js
let qrCode = null;
var pageLink = window.location.href;
var pageTitle = String(document.title).replace(/\&/g, "%26");
const fbs_click = () => {
  window.open(
    `http://www.facebook.com/sharer.php?u=${pageLink}&quote=${pageTitle}`,
    "sharer",
    "toolbar=0,status=0,width=626,height=436"
  );
  return false;
};
const tbs_click = () => {
  window.open(
    `https://twitter.com/intent/tweet?text=${pageTitle}&url=${pageLink}`,
    "sharer",
    "toolbar=0,status=0,width=626,height=436"
  );
  return false;
};
const lbs_click = () => {
  window.open(
    `https://www.linkedin.com/sharing/share-offsite/?url=${pageLink}`,
    "sharer",
    "toolbar=0,status=0,width=626,height=436"
  );
  return false;
};
const rbs_click = () => {
  window.open(
    `https://www.reddit.com/submit?url=${pageLink}`,
    "sharer",
    "toolbar=0,status=0,width=626,height=436"
  );
  return false;
};
const pbs_click = () => {
  window.open(
    `https://www.pinterest.com/pin/create/button/?&text=${pageTitle}&url=${pageLink}&description=${pageTitle}`,
    "sharer",
    "toolbar=0,status=0,width=626,height=436"
  );
  return false;
};
let qrcode_click = async () => {
  qrCode = await api.getQRcode(pageLink);
};
```

真的是这样！您现在可以一键将您的文章分享到社交媒体平台，还可以分享带有文章链接的二维码。如果您想查看此功能中的确切文件更改，可以查看[此 PR](https://github.com/christyjacob4/30-days-of-appwrite/pull/6/files)。

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

## Appwrite 云函数

如果您熟悉无服务器世界，您可能已经知道 **Cloud Functions** 是什么。对于那些不知道的人，将云函数视为一段无状态的代码，可以独立执行而无需您管理服务器。如果您使用过 **AWS Lambdas** 或类似的产品，那么您会对 **Appwrite Cloud Functions** 感到宾至如归。 Appwrite 支持超过 **13 种不同的运行时语言**，如 python、deno、dotnet，还有更多！

今天，我们将引导您浏览 Appwrite 控制台中的函数仪表板，并了解如何创建和部署函数。

Appwrite 中的 Cloud Functions 可以通过 3 种方式触发

* **REST API** - 您可以使用任何 HTTP 客户端或我们的 SDK 来创建和触发云功能。
* **事件** - Appwrite 在服务器中发生某些操作时发出事件，例如创建用户、创建文档等等。您可以配置一个函数来监听这些事件。您可以在[我们的文档](https://appwrite.io/docs/webhooks#events)中了解有关所有系统事件的更多信息
* **CRON 计划** - 您还可以配置您的函数以根据[ CRON 计划触发](https://en.wikipedia.org/wiki/Cron)。

在以下示例中，我们将介绍 **REST API** 触发器。

## 使用控制台部署函数

我们将在本演示中使用 python hello-world 函数。我们将首先使用 Appwrite Console 创建和执行函数，然后使用我们的 CLI 🤩 执行相同的步骤。我们希望您已经安装了 Appwrite。我们希望您已经安装了 Appwrite。

### 💻 创建你的函数

启动并运行 Appwrite 后，转到“函数”部分并​​单击“**添加函数**”。在以下对话框中，为您的函数命名并选择 **Python 3.9** 运行时。

![](https://res.cloudinary.com/neroblackstone/image/upload/v1627548324/appwrite_create_function_s8b899.png)

现在让我们探索刚刚为我们创建的函数运行时。

### ⚙️ 设置

![](https://res.cloudinary.com/neroblackstone/image/upload/v1627548430/appwrite_function_settings_yvgsit.png)

您可以在此处配置功能的所有方面。

| Field | Description |
| --- | --- |
| Name | 您的函数名称 |
| Execute Access | 管理谁可以使用权限执行此功能 |
| Timeout (seconds) | 限制函数的执行时间以防止滥用 |
| Events | 触发此功能的事件 |
| Schedule (CRON Syntax) | 设置一个 CRON Schedule 来执行这个函数 |
| Variables | 使用环境变量安全地存储机密和其他值 |

### 📊 监控器

在这里，您将能够找到有关函数执行的一些有用信息和一些使用指标，例如 CPU 时间、执行次数、错误等。

![](https://res.cloudinary.com/neroblackstone/image/upload/v1627548620/appwrite_function_monitors_fvqhvx.jpg)

### 📑 日志

您可以在此处查看所有执行日志。您还可以检查函数的输出到 `stdout` 和 `stderr` 。

![](https://res.cloudinary.com/neroblackstone/image/upload/v1627548696/appwrite_function_logs_ipl062.png)

现在我们已经熟悉了仪表板，让我们继续为我们刚刚创建的函数创建一个标签。

### ✍️ 创建标签

创建函数后，就可以上传新标签了。将标签视为函数的一个版本。为此，我们将使用我们的演示功能之一。

``` shell
$ git clone https://github.com/appwrite/demos-for-functions
$ cd python/hello-world
$ cat function.py

print("Hello World, I'm an Appwrite cloud function written in Python.")
```

这是一个简单的 python 函数，它只是将字符串打印到`stdout`。您可能已经注意到，在此示例中我们没有任何依赖项，因此您可以跳到下一部分。但如果这样做，则在打包函数之前还需要一个步骤。为了确保更快的函数执行时间，我们不会在运行时提取任何依赖项。依赖项需要与函数一起打包。

您需要确保您的所有依赖项都列在您的 `requirements.txt` 文件中。然后，您需要运行这个简单的命令来获取所有依赖项并将它们全部保存在本地 `.appwrite` 目录中。

``` shell
PIP_TARGET=./.appwrite pip install -r ./requirements.txt --upgrade --ignore-installed
```

我们有其他运行时的类似安装说明，您可以在[我们的文档](https://appwrite.io/docs/functions)中查看。

### 📦 打包云功能

安装依赖项（如果有）后，现在是打包函数的时候了。使用创建函数的 tar 文件

``` shell
$ cd ../
$ tar -zcvf code.tar.gz hello-world

a hello-world
a hello-world/function.py
```

现在，返回仪表板中的函数并单击 **Deploy Tag**。在随后出现的对话框中，选择**手动**选项卡。  
您将需要提供一个入口点命令，它本质上是执行您的函数的命令。在我们的例子中，这是 `python function.py`。接下来，上传我们刚刚创建的 tar 文件。仔细检查您的选择并选择**创建**。

![](https://res.cloudinary.com/neroblackstone/image/upload/v1627549282/appwrite_packaging_functions_uzoxoy.png)

### ✅ 激活和执行

创建函数后，您需要**激活**标签。太好了，标签现在已激活，您现在可以执行该功能。单击**立即执行**。在弹出的对话框中，系统会要求您输入要传递给函数的任何自定义数据。您可以将其留空并继续执行。

您现在可以转到“**日志**”选项卡并检查我们函数的输出！

![](https://res.cloudinary.com/neroblackstone/image/upload/v1627549464/appwrite_functions_logs_c391ir.png)

## 使用 CLI 部署函数

在本节中，我们将学习如何使用 Appwrite CLI 部署 `hello-world` 功能。如果您尚未安装 CLI，您可以查看[我们的指南](https://dev.to/appwrite/30daysofappwrite-appwrite-cli-2mde)以快速入门。在我们继续之前，请确保您已运行 `appwrite init` 并且您的 API 密钥具有以下范围。

* **functions.read**
* **functions.write**
* **execution.read**
* **execution.write**

### 💻 创建你的函数

您可以使用`functions`服务的 `create` 命令创建函数。

``` shell
$ appwrite functions create --name=test --execute="*" --env=python-3.9

$id : 60a285e91b4eb # This is your functionId
$permissions :
name : test
dateCreated : 1621263849
dateUpdated : 1621263849
status : disabled
env : python-3.9
tag :
vars : {}
events : {}
schedule :
scheduleNext : 0
schedulePrevious : 0
timeout : 15
```

此命令需要 3 个参数

* **name** - 函数的名称。
* **execute** - 一个字符串，用于管理执行此函数的访问权限。在我们的例子中，我们使用通配符权限 (*) 允许任何人执行此函数。您可以在[我们的文档](https://appwrite.io/docs/permissions)中阅读有关所有可用权限字符串的更多信息。
* **env** - 您的函数运行时。

### ✍️ 创建标签

下一步是让我们创建一个新标签。将标签视为功能的新版本/修订版。我们将使用`functions`服务中的 `createTag` 命令。

``` shell
$ appwrite functions createTag --code=. --functionId=60a285e91b4eb --command='python function.py'

$id : 60a2a9f9e1f1b # this is your tag id
functionId : 60a2a9ef5f63f # functionId
dateCreated : 1621273081
command : python function.py
size : 221
```

此命令接受 3 个参数

* **code** - 云函数的路径。假设您仍在 `demos-for-functions/python/hello-world` 中，您可以使用 `.`来表示当前目录。请注意，由于 docker 卷安装施加的限制，不允许引用 `../../directory` 等父路径。
* **functionId** - 函数的 id。在我们的例子中，来自上一个响应的 `$id` 属性。
* **command** - 您的入口点命令。

### ✅ 激活标签

一个函数可以有多个标签，所以在这一步我们需要激活你刚刚创建的标签。您可以使用`functions`服务中的 `updateTag` 命令执行此操作。

``` shell
$ appwrite functions updateTag --functionId=60a2a9ef5f63f --tag=60a2a9f9e1f1b

$id : 60a2a9ef5f63f
$permissions :
name : test
dateCreated : 1621273071
dateUpdated : 1621273071
status : disabled
env : python-3.9
tag : 60a2a9f9e1f1b
vars : {}
events : {}
schedule :
scheduleNext :
schedulePrevious : 0
timeout : 15
```

此命令接受 2 个参数

* **functionId** - 函数的 id。在我们的例子中，来自上一个响应的 `$id` 属性。
* **tag** - 您刚刚创建的标签的 **$id**。

### 🚀 运行你的函数

您现在可以通过使用`functions`服务的 `createExecution` 命令创建执行来运行您的函数。此命令只接受一个参数，即您已经熟悉的 `functionId`。

``` shell
$ appwrite functions createExecution --functionId=60a2a9ef5f63f

$id : 60a2aa86aa46a # executionId
functionId : 60a2a9ef5f63f
dateCreated : 1621273222
trigger : http
status : waiting
exitCode : 0
stdout :
stderr :
time : 0
```

### 🥳 获取输出

为了获得执行结果，您可以使用`functions`服务中的 `getExecution` 命令。

``` shell
$ appwrite functions getExecution --functionId=60a2a9ef5f63f --executionId=60a2aa86aa46a

$id : 60a2aa86aa46a
functionId : 60a2a9ef5f63f
dateCreated : 1621273222
trigger : http
status : completed
exitCode : 0
stdout : Hello World, I'm an Appwrite cloud function written in Python.

stderr :
time : 0.2251238822937
```

此命令接受 2 个参数

* **functionId** - 函数的 id。
* **executionId** - 上一个命令的执行 ID。

完美的！您刚刚创建并执行了您的第一个云函数！您可以浏览我们的 [demos-for-functions 存储库](https://github.com/appwrite/demos-for-functions)，以获取更酷的 Cloud Functions 示例和用例。

## 我们的第一个云函数

### 阅读时间

我们将在 Medium Clone 中实现的第一个 Cloud Function 函数将是一个函数，它将计算帖子的阅读时间。根据内容的长度，计算帖子的阅读时间可能是一项艰巨的任务。为了避免不必要地减慢您的应用程序的速度，我们将在我们的服务器上运行此过程。

我们将使用 [Infusion Media 的这篇博文](https://infusion.media/content-marketing/how-to-calculate-reading-time/)中建议的公式。

首先，我们将在我们的 **Posts** 集合中添加以下规则：

* **Label**：阅读时间
* **Key**：读书时间
* **Rule Type**：文本

![](https://res.cloudinary.com/neroblackstone/image/upload/v1627551291/appwrite_add_rule_uugxqr.jpg)

现在数据库已准备就绪，让我们从我们的云功能开始。为此，我们将使用 Node.js 运行时创建一个 Cloud 函数。在**设置选项卡**下的**功能仪表板中**，我们需要为事件 **database.documents.create** 和 **database.documents.update** 启用触发器。作为环境变量，我们将添加以下内容：

* **APPWRITE_PROJECT_ID**: 插入您的项目 ID。
* **APPWRITE_ENDPOINT**: 插入您的 appwrite 端点。
* **APPWRITE_API_KEY**: 插入具有文档写权限的 API 密钥。
* **POSTS_COLLECTION**: 插入 Posts 集合的 ID。

为了忠于我们演示项目的语言，我们将用 Node.js 编写它。

使用 npm 创建一个 Node.js 包：

``` shell
mkdir calculate-reading-time
cd calculate-reading-time
npm init -y
```

现在添加 `node-appwrite` 作为依赖项：

``` shell
npm install node-appwrite
```

创建 `index.js` 文件并放入以下内容：

``` js
const DATA = JSON.parse(process.env.APPWRITE_FUNCTION_EVENT_DATA);
const POSTS_COLLECTION = process.env.POSTS_COLLECTION;

const { $id, $collection, text, published } = DATA;

// Stop if it's not the Posts Collection or not published
if ($collection !== POSTS_COLLECTION || !published) {
    return;
}

// Initialise the client SDK
const appwrite = require("node-appwrite");
const client = new appwrite.Client();
const database = new appwrite.Database(client);

client
    .setEndpoint(process.env.APPWRITE_ENDPOINT) // Your API Endpoint
    .setProject(process.env.APPWRITE_PROJECT_ID) // Your project ID
    .setKey(process.env.APPWRITE_API_KEY) // Your secret API key
;

// Get word count
let words = text.match(
    /[A-Za-z\u00C0-\u017F]+|[\u0400-\u04FF\u0500–\u052F]+|[\u0370-\u03FF\u1F00-\u1FFF]+|[\u4E00–\u9FFF]|\d+/g
);

words = words ? words.length : 0;

let minutes = words / 200;
let seconds = (minutes % 1) * 60;
let readingTime = `${Math.floor(minutes)}m ${Math.floor(seconds)}s`;

// Don't update Post if reading time has not changed
if (readingTime === DATA.readingTime) {
    return;
}

database.updateDocument($collection, $id, {
    readingTime: readingTime
}).then(console.log).catch(console.error)
```

此函数在每次文档**写入**和**更新**事件时触发，计算读取时间并将其保存到 _readingTime_ 属性。我们也在检查读取时间是否发生变化——这是必要的，以避免创建无限循环和使用我们的云函数不必要地访问 API。

我们可以使用 Appwrite CLI 非常轻松地上传函数（或在仪表板中手动上传）：

``` shell
appwrite functions createTag --code=. --functionId=[YOUR_FUNCTION_ID] --command='node index.js'
```

> 不要忘记激活我们刚刚创建的标签！

### 测试我们的云函数

现在，当导航到 Posts 集合并编辑已发布帖子的文本时，我们可以轻松测试我们的阅读时间计算。您可以导航到 Functions Dashboard 并检查日志，或者只是刷新我们刚刚更新的文档，看看 **readingTime** 属性是如何神奇地更新的！

![](https://res.cloudinary.com/neroblackstone/image/upload/v1627552749/appwrite_test_fuctions_at7wfo.png)

唯一留给我们的是，在每篇文章的顶部添加阅读时间到我们的Medium Clone：

``` html
// src/routes/Post.svelte

//...
<i>
  {post.readingTime}
</i>
//...
```

## 使用 CRON 的云函数

### 创建统计

我们创建了一个由事件触发的云函数。当您想对来自客户端的交互做出反应时，这会派上用场。我们将创建一个将在特定时间间隔触发的云函数。我们可以通过向我们的云函数添加 CRON 计划来实现这一点。

我们将创建一个每天运行的云函数并为我们的应用程序创建统计信息。我们将把每天的个人资料和帖子的数量保存在一个集合中——这些数据允许我们创建图表和统计数据进行跟踪。

首先，我们将使用以下**规则**创建一个新的**统计**集合：

* Profiles:
  * **Label:** Profiles
  * **Key:** profiles
  * **Rule Type:** Numeric
* Posts:
  * **Label:** Posts
  * **Key:** posts
  * **Rule Type:** Numeric
* Timestamp:
  * **Label:** Timestamp
  * **Key:** timestamp
  * **Rule Type:** Numeric

**权限**将是 `*`用于_读取_，因此任何人都可以检索统计信息，我们将写入权限留空。将写入留空，将阻止任何人写入该集合 - 除非他们使用 API 密钥。![](https://res.cloudinary.com/neroblackstone/image/upload/v1627553183/appwrite_add_rules_hgvsug.png)

现在集合已准备就绪，让我们从我们的云功能开始。对于此示例，我们将创建另一个 Node.js 云函数。作为环境变量，我们将添加以下内容：

* **APPWRITE_PROJECT_ID**:插入您的项目 ID。
* **APPWRITE_ENDPOINT**: 插入您的 Appwrite 端点。
* **APPWRITE_API_KEY**: 插入具有文档读取和文档写入权限的 API 密钥。
* **STATISTICS_COLLECTION**: 插入统计集合的 ID。
* **PROFILE_COLLECTION**: 插入 Profile 集合的 ID。
* **POST_COLLECTION**:插入 Post 集合的 ID。

在此 Cloud Function 的 **Settings** 页面下，您还需要在 **Schedule (CRON Syntax)** 字段中添加一个值。对于我们的用例，我们将其设置为 `0 12 * * *`，这将在每天 12:00 执行此函数。

使用 npm 创建一个 Node.js 项目：

``` shell
mkdir create-statistics
cd create-statistics
npm init -y
```

现在添加 `node-appwrite` 作为依赖项：

``` shell
npm install node-appwrite
```

创建 `index.js` 文件并放入以下内容：

``` js
const STATISTICS_COLLECTION = process.env.STATISTICS_COLLECTION;
const PROFILE_COLLECTION = process.env.PROFILE_COLLECTION;
const POST_COLLECTION = process.env.POST_COLLECTION;

// Initialise the client SDK
const appwrite = require('node-appwrite');
const client = new appwrite.Client();
const database = new appwrite.Database(client);

client
    .setEndpoint(process.env.APPWRITE_ENDPOINT) // Your API Endpoint
    .setProject(process.env.APPWRITE_PROJECT_ID) // Your project ID
    .setKey(process.env.APPWRITE_API_KEY) // Your secret API key
;

// Get the sum of Profiles and Posts
const profiles = database.listDocuments(PROFILE_COLLECTION, [], 0).then(r => r.sum);
const posts = database.listDocuments(POST_COLLECTION, ['published=1'], 0).then(r => r.sum);

// Waiting for all promises to resolve and write into the Statistics Collection
Promise.all([profiles, posts]).then(([profiles, posts]) => {
    return database.createDocument(STATISTICS_COLLECTION, {
        posts: posts,
        profiles: profiles,
        timestamp: Date.now()
    }, ['*']);
}).then(console.log).catch(console.error);
```

我们可以使用 Appwrite CLI 非常轻松地上传函数（或在仪表板中手动上传）：

``` shell
appwrite functions createTag --code=. --functionId=[YOUR_FUNCTION_ID] --command='node index.js'
```

> 不要忘记激活我们刚刚创建的标签！

## 测试我们的云函数

我们可以通过等待 12:00 或只是在您的函数页面中手动执行来轻松测试我们的函数。如果函数执行成功，你可以在 **Statistics** Collection 中找到一个这样的文档：

![](https://res.cloudinary.com/neroblackstone/image/upload/v1627553652/appwrite_test_function_zhfctn.png)

有了这些数据，我们就可以创建图表和统计数据来监控我们应用程序的使用情况。

随意分享您将如何利用这些数据并在 Medium Clone 中实施它！

## Appwrite 中的 JWT 支持

### 什么是 JWT

JWT（**J**SON **W**eb **T**oken）是一种用于为应用程序创建访问令牌的标准。它的工作方式是：服务器生成一个令牌来证明用户身份，并将其发送给客户端。客户端将为每个后续请求将令牌发送回服务器，因此服务器知道请求来自特定身份。

格式良好的 JWT 由三个连接的 **Base64** url​​ 编码字符串组成，以点 (`.`) 分隔：

* **Header**：包含有关令牌类型和用于保护其内容的加密算法的元数据。
* **Payload**：包含可验证的安全声明，例如用户的身份和他们被允许的权限。
* **Signature**：用于验证令牌是否可信且未被篡改。

![](https://res.cloudinary.com/neroblackstone/image/upload/v1627615177/appwrite_jwt_xwdf1y.png)

事实证明，这种架构在现代 Web 应用程序中非常有效，在用户通过身份验证后，我们向 REST 或 GraphQL API 执行 API 请求。

无论如何，并不总是建议对会话使用 JWT。将常规服务器端会话与 Cookie 结合使用通常会更有效率且不易泄露数据。

#### 那么，为什么我们需要 JWT 呢？

在现代网络中，您通常会有多个实体相互通信。某些功能自然会受到限制并需要某种授权机制。在 Appwrite，我们在客户端使用 Cookie 与后端进行通信。

使用 JWT，您将能够在云功能、微服务或 SSR 中的服务器端授权用户。

### 创建一个 JWT

Appwrite 0.8 版引入了 JWT，使用 Web 或 Flutter SDK 生成它真的很容易。因为 JWT 用于身份验证和授权 - 我们只有在我们通过身份验证时才能生成它们。

#### Web

``` js
appwrite.account.createJWT().then(response => {
    console.log(response); // Success
}, error => {
    console.log(error); // Failure
});
```

#### flutter

``` dart
account.createJWT().then((response) {
    print(response);
}).catchError((error) {
    print(error.response);
});
```

`createJWT()` 方法将接收这样的对象：

``` json
{
  jwt: "eyJhbGciOiJIUzI1NiIsInR5cCI6I..."
}
```

此 JWT 有效期为 15 分钟，每个**用户帐户**每 **60 分钟**只能生成 **10 次**。

### 带有服务器 SDK 的 JWT

现在我们可以使用 JWT，我们可以使用它代表用户在服务器上执行操作，而无需登录或提供 API 密钥。

为了演示，让我们尝试使用 Node.js 脚本获取当前用户：

``` shell
mkdir appwrite-jwt-test
cd appwrite-jwt-test
npm init -y
```

现在添加 `node-appwrite` 作为依赖项：

``` shell
npm install node-appwrite
```

创建 `index.js` 文件并放入以下内容：

``` js
const appwrite = require('node-appwrite');
const client = new appwrite.Client();
const account = new appwrite.Account(client);

client
    .setEndpoint("[ENDPOINT]") // Your API Endpoint
    .setProject("[PROJECT_ID]") // Your project ID
    .setJWT("[INSERT_JWT_HERE]") // Your users JWT
;

account.get().then(r => console.log(r));
```

> 记得填写端点、项目 ID 和 JWT。请记住，JWT 仅在生成后 15 分钟内有效。

现在我们可以使用 node index.js 执行这个文件，如果一切顺利，我们应该看到我们用户的对象👏

### 具有云函数的 JWT

还记得，用户可以通过 Rest API 执行 Cloud Functions 吗？如果用户这样做，默认情况下，Cloud Function 将在 APPWRITE_FUNCTION_JWT 环境变量中为执行该函数的用户传递一个 JWT。

这样我们甚至不必从客户端创建 JWT 并将其传递给我们 🎉

### JWT 与 SSR

随着用于 Appwrite 的 Web SDK 3.0.0 版，我们已将其重构为同构。这在 JavaScript 生态系统中很重要——因为随着 SSR 的日益流行，库需要在浏览器中工作——以及在服务器端使用 Node.js。

这就是我们将服务器 SDK 中的 `setJWT(jwt)` 方法也添加到 Web SDK 的原因 - 这允许开发人员将相同的 SDK 用于客户端和服务器端操作以及 Next.js、Nuxt.js 和 Svelte Kit。