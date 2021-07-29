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