---
title: Appwrite学习笔记(七)
description: 电子邮件验证
date: 2021-07-22
img: https://res.cloudinary.com/neroblackstone/image/upload/v1624671830/appwrite_i2voda.webp
tags:
- Appwrite

---
电子邮件验证和密码恢复是任何应用程序的两个关键功能！让我们学习如何使用 Appwrite 实现这两个功能。

> 这要求您在 Appwrite 上进行 SMTP 设置。

让我们从电子邮件验证开始。 OAuth2 登录不需要电子邮件验证，因为在这种情况下，电子邮件地址已由登录提供商验证。

## 初始化验证

要验证帐户，您必须在为用户创建会话后调用 `createVerification(url)` 方法。此方法会向您用户的电子邮件地址发送一条验证消息，以确认他们是该地址的有效所有者。提供的 URL 应将用户重定向回您的应用程序，并允许您通过验证已提供给用户的 `userId` 和 `secret` 参数来完成验证过程。

``` js
let promise = sdk.account.createVerification('http://myappdomain/verifyEmail');

promise.then(function (response) {
    console.log(response); // Success
}, function (error) {
    console.log(error); // Failure
});
```

URL 获取下一步所需的这两个参数，作为查询参数附加到 URL。例如，如果您将 `http://myappdomain/verify` 传递给该方法，则通过电子邮件提供给用户的 URL 将如下所示：

`http://localhost/verifyEmail?userId=XXXX&secret=YYYY`

## 完整的电子邮件验证

既然用户能够初始化其帐户的验证过程，我们需要通过处理来自电子邮件中提供的 URL 的重定向来完成它。

第一步是检索 URL 中提供的 `userId` 和 `secret` 值。在 JavaScript 中，我们可以像这样得到这些：

``` js
const urlParams = new URLSearchParams(window.location.search);
const userId = urlParams.get('userId');
const secret = urlParams.get('secret');
```

有了这些值，我们现在可以使用 `updateVerification(userId, secret)` 完成电子邮件验证：

``` js
const sdk = new Appwrite();

sdk
    .setEndpoint('https://[HOSTNAME_OR_IP]/v1') // Your API Endpoint
    .setProject('5df5acd0d48c2') // Your project ID
;

let promise = sdk.account.updateVerification(userId, secret);

promise.then(function (response) {
    console.log(response); // Success
}, function (error) {
    console.log(error); // Failure
});
```

现在我们已经成功验证了一个用户！

> 请确保仅在电子邮件中提供的 URL 上执行此方法。在我们的示例中，这将是 http://myappdomain/verifyEmail。

## 完整示例

太好了，现在让我们在演示应用程序中实现上述功能。在 `src/lib/Navigation.svelte` 中创建一个按钮来显示用户的验证状态

``` html
...
{#if !$state.user.emailVerification}
    <button on:click={startEmailVerification}>Not Verified ❌</button>
{:else}
    <p>Verified ✅</p>
{/if}
....
```

在 `<script>` 部分添加一个 `startEmailVerification` 函数。

``` js
... 
import { api } from "../appwrite";

const startEmailVerification = async () => {
    try {
        const url = `${window.location.origin}/#/verifyEmail`
        await api.createVerification(url)
        alert("Verification Email sent")
    } catch (error) {
        alert(error.message)
    }
}
...
```

在 `src/appwrite.js` 创建以下函数

``` js
...
createVerification: async (url) => {
    await sdk.account.createVerification(url);
},
completeEmailVerification: async(userId, secret) => {
    await sdk.account.updateVerification(userId, secret);
},
...
```

我们需要在我们的应用程序中创建一个新页面来接收来自验证电子邮件的重定向并完成验证。

创建一个新文件 `src/routes/VerifyEmail.svelte` 并添加以下内容

``` html
<script>
    import { api } from "../appwrite";
    let urlSearchParams = new URLSearchParams(window.location.search);
    let secret = urlSearchParams.get("secret");
    let userId = urlSearchParams.get("userId");
    console.log(userId,secret);
    const completeEmailVerification = async () => {
        await api.completeEmailVerification(userId, secret)
        window.location.href = "/"
    }
    completeEmailVerification()
</script> 
```

不要忘记在 `src/App.svelte` 中为此页面创建路由😊

``` js
import VerifyEmail from "./routes/VerifyEmail.svelte";

...
const routes = {
    "/": Index,
    "/login": Login,
    "/register": Register,
    "/verifyEmail": VerifyEmail,
    "*": NotFound,
};
...
```

做得好！您刚刚为您的用户启用了`Email Verification`，而没有编写一行后端代码！