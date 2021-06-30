---
title: Appwrite学习笔记(四)
description: 帐户和用户 API
date: 2021-06-30
img: https://res.cloudinary.com/neroblackstone/image/upload/v1624671830/appwrite_i2voda.webp
tags:
- Appwrite

---
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