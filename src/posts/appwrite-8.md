---
title: Appwrite学习笔记(九)
description: 应用程序团队
date: 2021-07-22
img: https://res.cloudinary.com/neroblackstone/image/upload/v1624671830/appwrite_i2voda.webp
tags:
- Appwrite

---
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