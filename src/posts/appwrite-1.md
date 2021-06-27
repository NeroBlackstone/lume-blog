---
title: Appwrite学习笔记(二)
description: 了解Appwrite Dashboard
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