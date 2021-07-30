---
title: Appwrite学习笔记(十一)
description: ''
date: 2021-07-30
img: https://res.cloudinary.com/neroblackstone/image/upload/v1624671830/appwrite_i2voda.webp
tags:
- Appwrite

---
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