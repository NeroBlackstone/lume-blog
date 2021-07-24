---
title: Appwrite学习笔记（二）
description: ''
date: 2021-07-24
img: https://res.cloudinary.com/neroblackstone/image/upload/v1624671830/appwrite_i2voda.webp
tags:
- Appwrite

---
## 数据库设计

### 规划数据结构

我们将使用 Appwrite Database 提供的 **Collection** and **Rules** 功能来规划我们的应用程序所需的数据结构。首先，让我们放下我们的应用程序的要求。

#### Post

帖子是指任何经过身份验证的用户都可以发布的内容。任何人都可以注册并创建帖子。我们的帖子将包含**标题**、**封面**图片、**文本**、**已发布**（表示帖子是草稿还是已发布）、**标签**、**创建日期**和创建者的 **ID**。现在我们将使用 Appwrite 数据库提供的**规则**来计划这个。

首先，我们将创建一个集合，并将其命名为 **Posts**。然后我们将从控制台添加以下规则并更新集合。

* Title
  * label: Title
  * Key: title
  * Rule Type: Text
  * Required: true
  * Array: false
* Cover image
  * label: Cover image
  * Key: cover
  * Rule Type: Text
  * Required: false
  * Array: false
* Text
  * label: Text
  * Key: text
  * Rule Type: Markdown
  * Required: true
  * Array: false
* Published
  * label: Published
  * Key: published
  * Rule Type: Boolean
  * Required: true
  * Array: false
* Tags
  * label: Tags
  * Key: tags
  * Rule Type: Text
  * Required: false
  * Array: true
* Created Date
  * label: Created At
  * Key: created_at
  * Rule Type: Numeric
  * Required: true
  * Array: false
* User Id
  * label: User Id
  * Key: user_id
  * Rule Type: Text
  * Required: true
  * Array: false![](https://res.cloudinary.com/neroblackstone/image/upload/v1627117716/appwrite_post_rule_ze0nvj.png)

对于权限，读取权限应为 `[​​]`，因为任何人都应该能够阅读帖子，而此集合的写入权限应为 `[​​role:member]`，以便只有登录用户才能创建帖子。

#### Profile

我们想让我们的用户拥有一个公开名称的个人资料，以便我们可以在每篇文章中显示作者信息。我们还希望将用户的帖子作为嵌入文档添加到集合中，以便我们可以使用用户的个人资料轻松获取它。

让我们使用以下规则创建另一个名为 **Users** 的集合

* User Id
  * label: User
  * Key: user
  * Rule Type: Text
  * Required: true
  * Array: false
* Name
  * label: Name
  * Key: name
  * Rule Type: Text
  * Required: true
  * Array: false
* Posts
  * label: Posts
  * Key: posts
  * Rule Type: Document (Embedded)
  * Required: false
  * Array: true

创建**帖子**规则后，您将看到一个包含**允许集合**的部分。在这里，您需要选择**帖子**。

![](https://res.cloudinary.com/neroblackstone/image/upload/v1627118332/appwrite_profile_rule_axf8vk.png)

至于权限，读权限应该是\[*\]，因为任何人都应该可以读，写权限可以是\[role:member\]，这样任何登录的人都可以创建个人资料。

因此，这就是使用**集合**和**规则**为任何应用程序规划数据结构是多么容易——它与我们计划使用**表**和**列**的传统关系数据库非常相似。

## 创建用户

### 创建个人资料

我们将使用 **Profile** Collection 为我们应用上的用户提供一个带有公共名称的个人资料，以便我们可以在每个帖子中显示作者的信息。

为此，我们需要在 `appwrite.js` 文件中添加两种方法：一种用于获取配置文件，另一种用于创建配置文件。让我们限制用户只为一个帐户创建一个配置文件。为此，我们需要首先检查他是否已经有个人资料。因此，我们将添加一个 `fetchUser()` 函数，该函数将列出 **Profiles** 集合中的所有文档，用户字段等于我们帐户的 ID，并将文档数量限制为 1。

``` js
export const api = {
    //...
    fetchUser: async id => {
        let res = await sdk.database.listDocuments(
            profilesCollection,
            [`user=${id}`],
            1
        );
        if (res.sum > 0 && res.documents.length > 0) return res.documents[0];
        else throw Error("Not found");
    }
};
```

这里最重要的是带有 _\[user=${id}\]_ 的部分，它将通过等于传递的 `id` 的**用户**字段过滤请求的文档。

您可能已经注意到，我们在 `listDocuments` 调用中使用了一个名为 `profilesCollection` 的未知变量。为此，我们需要创建 2 个变量来表示我们集合的唯一 ID。只需在 `const api` 之前添加以下内容：

``` js
const profilesCollection = "[INSERT YOUR ID HERE]";
const postsCollection = "[INSERT YOUR ID HERE]";
```

确保使用仪表板中的 ID，并将 **Profile** 中的 ID 替换为 `profilesCollection`，并将 **Post** Collection 中的 ID 替换为 `postsCollection`。

现在我们可以检查profile是否存在，如果不存在，用户需要能够创建他们的配置文件。为此，我们将在 `appwrite.js` 中引入 `createUser` 方法：

``` js
export const api = {
    //...
    createUser: async (id, name) => {
        return sdk.database.createDocument(
            profilesCollection,
            {
                user: id,
                name: name,
            },
            ["*"],
            [`user:${id}`]
        );
    },
}
```