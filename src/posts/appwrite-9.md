---
title: Appwrite学习笔记十)
description: ''
date: 2021-07-29
img: https://res.cloudinary.com/neroblackstone/image/upload/v1624671830/appwrite_i2voda.webp
tags:
- Appwrite

---
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