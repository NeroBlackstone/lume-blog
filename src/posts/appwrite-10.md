---
title: Appwrite学习笔记(十一)
description: 应用程序数据库
date: 2021-07-23
img: https://res.cloudinary.com/neroblackstone/image/upload/v1624671830/appwrite_i2voda.webp
tags:
- Appwrite

---
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