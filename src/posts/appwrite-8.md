---
title: Appwrite学习笔记(九)
description: ''
date: 2021-07-29
img: https://res.cloudinary.com/neroblackstone/image/upload/v1624671830/appwrite_i2voda.webp
tags:
- Appwrite

---
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