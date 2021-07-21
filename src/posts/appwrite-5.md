---
title: Appwrite学习笔记(六)
description: OAuth Provider和SMTP
date: 2021-07-21
img: https://res.cloudinary.com/neroblackstone/image/upload/v1624671830/appwrite_i2voda.webp
tags:
- Appwrite

---
## OAuth提供者

使用 Appwrite，设置许多 OAuth2 提供程序以提供您的用户使用他们的社交帐户进行身份验证非常容易。 Appwrite 支持大量 OAuth2 提供商，包括 Google、Facebook、Twitter、GitHub 等等。最好的部分：大多数 OAuth2 提供程序来自开源社区的拉取请求！

我们将研究如何通过 Appwrite 驱动的应用程序设置和使用 Google 登录。

### 设置 Google OAuth2 设置

在 Appwrite 控制台中，访问 `Users->OAuth2 Providers` 页面。在那里，从列表中找到 Google 并打开开关。为了完成此操作，您将需要可以从 Google API 控制台轻松设置的 `App ID` 和 `App Secret`。查看[他们的官方文档](https://support.google.com/googleapi/answer/6158849)以获取更多详细信息。

![](https://res.cloudinary.com/neroblackstone/image/upload/v1626856697/Appwrite_set_google_oauth2_b6x2aq.png)

获取并填写 Google `App ID` 和 `App Secret` 后，您需要将对话框中显示的回调 URL 提供给 Google OAuth2 的授权重定向 URI。

![](https://res.cloudinary.com/neroblackstone/image/upload/v1626856872/Appwrite_google_client_oath2_i0vrpt.png)

### 使用 OAuth2 提供程序登录

现在我们已经设置了 OAuth2 登录，我们可以请求登录我们的用户。首先，在 `src/routes/Login.svelte` 中添加一个 `Login with Google` 按钮。

``` html
<button on:click|preventDefault={loginWithGoogle}>Login With Google</button>
```

现在我们需要在我们的 Login 组件的脚本中添加一个 `loginWithGoogle` 方法：

``` js
const loginWithGoogle = async () => {
    try {
      await api.loginWithGoogle();
    } catch(error) {
      console.log(error.message);
    }
}
```

最后，在 `src/appwrite.js` 中我们需要添加 `loginWithGoogle`：

``` js
const loginWithGoogle = async () => {
    try {
      await api.loginWithGoogle();
    } catch(error) {
      console.log(error.message);
    }
}
```

最后，在 `src/appwrite.js` 中我们需要添加 `loginWithGoogle`：

``` js
export const api = {
    //....existing code
    loginWithGoogle: async () => {
        try {
            await sdk.account.createOAuth2Session('google', 'http://localhost:3000', 'http://localhost:3000/#/login');
        } catch (error) {
            throw error;
        }
    },
}
```

在这里，我们调用 `sdk.account.createOAuth2Session` 并传入几个参数。第一个参数是**提供者**。对我们来说，它是`google`。第二个参数是登录成功后要重定向到的 URL。第三个参数是登录失败时重定向到的 URL。这里我们提供 localhost，因为我们正在本地测试应用程序。一旦我们在线托管我们的应用程序，我们需要为成功和失败的登录提供更新的 URL。

现在，如果您点击使用 Google 登录，您应该会被带到 Google OAuth 同意屏幕。选择有效的 Google 帐户后，您应该能够登录并重定向回 `http://localhost:3000`。您可以查看我们应用程序的导航栏，查看新帐户名称是否已成功加载。

## SMTP 入门

**SMTP** 代表**简单邮件传输协议**。与任何其他协议一样，它定义了网络上所有计算机需要遵守的一些步骤和准则。SMTP 是 TCP/IP 堆栈中的应用层协议，与称为**邮件传输代理 (MTA)** 的东西密切合作，将您的通信发送到正确的计算机和电子邮件收件箱。

为了在 Appwrite 中启用电子邮件功能，您需要设置正确的 SMTP 配置。由于电子邮件的可传递性可能既棘手又困难，因此将这一责任委托给第三方 SMTP 提供商（如 [MailGun](https://www.mailgun.com/) 或 [SendGrid](https://sendgrid.com/)）通常更容易。这些提供程序通过为您执行大量高级配置和验证来帮助您抽象传递垃圾邮件过滤器的复杂性。

随意向您选择的任何提供商注册并跳到**Configuration**部分，否则继续学习如何从 Sendgrid 获取 SMTP 凭据。

### 设置SendGrid

1. 在此处创建一个 [SendGrid](https://signup.sendgrid.com/) 帐户。
2. 验证用作发件人的单个电子邮件地址的所有权。说明可以在[这里](https://sendgrid.com/docs/ui/sending-email/sender-verification/)找到。
3. 在[电子邮件 API -> 集成指南](https://app.sendgrid.com/guide/integrate)下设置 SMTP 中继并创建 API 密钥。
4. 在下方，您应该会看到在下一步中使用 Appwrite 设置 SendGrid 所需的所有凭据。

### 配置

Appwrite 提供多个[环境变量](https://appwrite.io/docs/environment-variables#smtp)来根据您的需要自定义您的服务器设置。为了启用 SMTP，您需要更改 Appwrite 容器的环境变量。以下对我们很重要：

| Name | Description |
| --- | --- |
| _APP_SMTP_HOST | SMTP 服务器主机名地址。使用空字符串禁用从服务器发送的所有邮件。此变量的默认值是一个空字符串 |
| _APP_SMTP_PORT | SMTP 服务器 TCP 端口。默认为空。 |
| _APP_SMTP_SECURE | SMTP 安全连接协议。默认为空，如果在安全连接上运行，则更改为 'tls'。 |
| _APP_SMTP_USERNAME | SMTP 服务器用户名。默认为空。 |
| _APP_SMTP_PASSWORD | SMTP 服务器用户密码。默认为空。 |

要根据您的需要更改这些变量，请导航到安装 Appwrite 的 `appwrite` 目录并编辑隐藏的 `.env` 文件。

    _APP_SMTP_HOST=smtp.sendgrid.net
    _APP_SMTP_PORT=587
    _APP_SMTP_SECURE=tls
    _APP_SMTP_USERNAME=YOUR-SMTP-USERNAME
    _APP_SMTP_PASSWORD=YOUR-SMTP-PASSWORD

完成更新后，您需要从终端使用以下命令重新启动 Appwrite 堆栈：

``` shell
docker-compose up -d --remove-orphans --build --force-recreate
```

### 就是这样！

转到您的 Appwrite 控制台，从您的帐户注销并尝试通过导航到**忘记密码？**来恢复您的密码。如果您已经使用 SendGrid 设置了 SMTP 服务器 - 这也应该验证您的集成。

如果一切顺利，您应该会收到一封包含重置密码说明的电子邮件。显然，这不是必需的，只是用于检查 SMTP 服务器是否正常工作的测试。