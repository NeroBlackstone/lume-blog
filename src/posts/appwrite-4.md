---
title: Appwrite学习笔记(五)
description: 登录和注册
date: 2021-06-30
img: https://res.cloudinary.com/neroblackstone/image/upload/v1624671830/appwrite_i2voda.webp
tags:
- Appwrite

---
## 登录和注册

对用户进行身份验证和授权以验证他们的请求或保存他们的数据是现代应用程序开发生命周期不可或缺的一部分。这是我们在每个应用程序中都会做的事情。Appwrite 通过抽象出所有关于用户管理的功能，使这个过程变得非常简单，为我们提供了一个漂亮的 API。Appwrite 的身份验证功能支持基于电子邮件和密码的身份验证以及各种 OAuth2 提供商，例如 **Google、Facebook、GitHub、Twitter** 等。您可以在 **Appwrite Console -> Users -> Providers** 中找到所有支持的提供程序的列表。[即将推出的 Appwrite 版本](https://dev.to/appwrite/appwrite-0-8-is-coming-soon-and-this-is-what-you-can-expect-35li)还将添加对匿名登录的支持。

默认情况下，身份验证、用户管理、团队和角色都由 Appwrite 处理。在今天的会议中，我们将继续上次会议的演示并添加身份验证功能。我们将实现注册和登录。让我们开始吧。

### 开始

我们已经创建了一个基础项目，我们将开始在我们的应用程序中使用 Appwrite 实现更多功能。要继续以下部分，您需要克隆或下载我们的基础项目。可以在[这里](https://github.com/christyjacob4/30-days-of-appwrite/tree/starter)找到我们的基础项目。从这里开始，我们将建立在这个初始项目的基础上。所以请克隆或下载它，以便您可以跟上。

### 创建注册和登录组件

首先，我们将创建我们的 Login 和 Signup 组件并设置到这些组件的路由。

#### 注册组件

让我们添加一个注册组件 `src/routes/Register.svelte`。

``` html
<!-- Signup page -->
<script>
  import { api } from "../appwrite";

  let name,
    mail,
    pass = "";

  const submit = async () => {
    try {
      await api.register(mail, pass, name);
    } catch (error) {
      console.log(error.message);
    }
  };
</script>

<div>
  <h1>Register</h1>
  <form on:submit|preventDefault={submit}>
    <label for="mail"><b>Name</b></label>
    <input
      bind:value={name}
      type="text"
      placeholder="Enter Name"
      name="name"
      required
    />

    <label for="mail"><b>E-Mail</b></label>
    <input
      bind:value={mail}
      type="email"
      placeholder="Enter E-mail"
      name="mail"
      required
    />

    <label for="pass"><b>Password</b></label>
    <input
      bind:value={pass}
      type="password"
      placeholder="Enter Password"
      name="pass"
      required
    />

    <button type="submit">Register</button>
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

如您所见，我们这里的`submit`函数调用了 `api.register(name, mail, pass)`。所以我们需要实现注册功能。

#### 处理注册

让我们创建 `src/appwrite.js` 文件，我们将在其中使用 `Appwrite` SDK 来处理`register`功能。

``` js
import Appwrite from "appwrite"; //importing from Appwrite's SDK
import { state } from "./store"; // saving user data to svelte store

const sdk = new Appwrite();
sdk
  .setEndpoint("https://demo.appwrite.io/v1") //set your own endpoint
  .setProject("607dd16494c6b"); //set your own project id

export const api = {
    register: async (name, mail, pass) => {
        try {
            await sdk.account.create(mail, pass, name);
            await api.login(mail, pass)
        } catch (error) {
            throw error;
        }
    },
}
```

在 `register` 函数中，我们调用了 `sdk.account.create` 方法。我们正在传递要创建的帐户的`mail`, `password`和`name`。

#### 登录组件

让我们添加一个新组件 `scr/routes/Login.svelte`。

``` html
<!-- Login page -->
<script>
  import { api } from "../appwrite";

  let mail,
    pass = "";

  const submit = async () => {
    try {
      await api.login(mail, pass);
    } catch (error) {
      console.log(error.message);
    }
  };
</script>
<div>
  <h1>Login</h1>
  <form on:submit|preventDefault={submit}>
    <label for="mail"><b>E-Mail</b></label>
    <input
      bind:value={mail}
      type="email"
      placeholder="Enter E-mail"
      name="mail"
      required
    />

    <label for="pass"><b>Password</b></label>
    <input
      bind:value={pass}
      type="password"
      placeholder="Enter Password"
      name="pass"
      required
    />

    <button type="submit">Login</button>
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

如果您查看`submit`函数，我们正在调用 `api.login(mail, pass)` 并且我们的 `api` 是从 `../appwrite` 导入的。所以让我们创建我们的登录函数，它将与 Appwrite 通信以执行登录。

#### 处理登录

再次在 `src/appwrite.js` 中，我们将创建登录功能。

``` js
export const api = {
    //..register code
    login: async (mail, pass) => {
        try {
            await sdk.account.createSession(mail, pass);
            const user = await api.getAccount();
            state.update(n => {
                n.user = user;
                return n;
            });
        } catch (error) {
            state.update(n => {
                n.user = null;
                return n;
            });
            throw error;
        }
    },
}
```

因此，我们首先导入 Appwrite SDK 并使用 `const sdk = new Appwrite();` 创建一个实例。之后，我们需要定义 API 端点和项目 ID。您可以在 Appwrite 仪表板的“设置”页面中找到这些内容。

在我们的登录函数中，我们使用 `sdk.account.createSession(mail, pass)`。如果凭据有效，则来自 `Account` 服务的 `createSession` 方法负责创建用户的会话。最后我们调用 `api.getAccount()` - 让我们也实现这个函数。

``` js
export const api = {
  //...login and register code
  getAccount: async () => sdk.account.get(),
}
```

对于 `getAccount`，我们只是调用 `sdk.account.get()`。来自 `Account` 服务的 `get` 方法将获取具有活动会话的用户的用户配置文件。如果不存在会话，它将抛出错误。