---
title: Appwrite学习笔记(八)
description: 找回密码
date: 2021-07-22
img: https://res.cloudinary.com/neroblackstone/image/upload/v1624671830/appwrite_i2voda.webp
tags:
- Appwrite

---
既然用户可以验证他们的帐户，我们还需要让他们能够在丢失密码的情况下恢复他们的帐户。此流程与用于验证帐户的流程非常相似。

## 初始化密码恢复

第一步是使用 `createRecovery(email, url)` 方法向用户发送一封电子邮件，其中包含用于在 URL 中重置密码的临时密钥。

``` js
let promise = sdk.account.createRecovery('email@example.com', 'http://myappdomain/resetPassword');

promise.then(function (response) {
    console.log(response); // Success
}, function (error) {
    console.log(error); // Failure
});
```

如果此调用成功，则会向用户发送一封电子邮件，其中提供一个 URL，该 URL 具有附加到 createRecovery(email, url) 中传递的 URL 的机密和用户 ID 值。

## 完整的密码恢复

恢复页面应提示用户输入新密码两次，并在提交时调用 `updateRecovery(userId, secret, password, passwordAgain)`。就像前面的场景一样，我们从 URL 中获取 `userId` 和 `secret` 值：

``` js
const urlParams = new URLSearchParams(window.location.search);
const userId = urlParams.get('userId');
const secret = urlParams.get('secret');
```

有了这些值，我们可以使用 updateRecovery(userId, secret, password, passwordAgain) 完成密码恢复：

``` js
const sdk = new Appwrite();

sdk
    .setEndpoint('https://[HOSTNAME_OR_IP]/v1') // Your API Endpoint
    .setProject('5df5acd0d48c2') // Your project ID
;
let password; // Assign the new password choosen by the user
let passwordAgain;
let promise = sdk.account.updateRecovery(userId, secret, password, paswordAgain);

promise.then(function (response) {
    console.log(response); // Success
}, function (error) {
    console.log(error); // Failure
});
```

现在我们已经成功重置了用户的密码！

## 完整示例

太好了，是时候写代码了！在 `src/routes/Login.svelte` 中创建一个按钮以允许用户恢复他们的密码

``` html
...

<button class="button" on:click|preventDefault={forgotPassword}>Forgot Password?</button>

....
```

在 `<script>` 部分添加一个 `forgotPassword` 函数。

``` js
import { api } from "../appwrite";

const forgotPassword = async () => {
    const url = `${window.location.origin}/#/resetPassword`
    console.log(url);
    try {
        await api.forgotPassword(mail, url)
        alert("Recovery Email Sent")
    } catch (error) {
        alert(error.message);
    }
}
```

在 `src/appwrite.js` 创建以下函数

``` js
....

forgotPassword: async (email, url) => { 
    await sdk.account.createRecovery(email, url) 
},
completePasswordRecovery: async (userId, secret, pass, confirmPass) => { 
    await sdk.account.updateRecovery(userId, secret, pass, confirmPass) 
},

...
```

我们需要在我们的应用程序中创建一个新页面来接收来自密码恢复电子邮件的重定向。创建一个新文件 `src/routes/ResetPassword.svelte` 并将以下代码添加到其中。

不要不知所措。它只是从 URL 参数中获取 `userId` 和 `secret`，要求用户输入新密码，请求 `updateRecovery` 并在成功时将用户重定向到登录页面。

``` html
<script>
    import { api } from "../appwrite";
    let urlSearchParams = new URLSearchParams(window.location.search);
    let secret = urlSearchParams.get("secret");
    let userId = urlSearchParams.get("userId");
    let password = "",
        confirmPassword = "";
    const submit = async () => {
        try {
            await api.completePasswordRecovery(
                userId,
                secret,
                password,
                confirmPassword
            );
            window.location.href = "/#/login";
        } catch (error) {
            alert(error.message);
        }
    };
</script>

<div>
    <h1>Reset your password</h1>

    <form on:submit|preventDefault={submit}>
        <label for="password"><b>New Password</b></label>
        <input
            bind:value={password}
            type="password"
            placeholder="Enter New Password"
            name="password"
            required />

        <label for="confirmPassword"><b>Confirm Password</b></label>
        <input
            bind:value={confirmPassword}
            type="password"
            placeholder="Confirm Password"
            name="confirmPassword"
            required />

        <button class="button" type="submit">Reset Password</button>
    </form>
</div>

<style>
    div {
        margin-left: auto;
        margin-right: auto;
        width: 400px;
    }
    form {
        display: flex;
        flex-direction: column;
    }
</style>
```

不要忘记在 `src/App.svelte` 中为此页面创建路由😊

``` js
import ResetPassword from "./routes/ResetPassword.svelte";

...
const routes = {
    "/": Index,
    "/login": Login,
    "/register": Register,
    "/resetPassword": ResetPassword,
    "/verifyEmail": VerifyEmail,
    "*": NotFound,
};
...
```

惊人的！如果一切顺利，您的用户现在将能够重置他们的密码！！