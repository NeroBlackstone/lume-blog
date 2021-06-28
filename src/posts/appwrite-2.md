---
title: Appwrite学习笔记(三)
description: 了解SSL证书和服务端SDK
date: 2021-06-28
img: https://res.cloudinary.com/neroblackstone/image/upload/v1624671830/appwrite_i2voda.webp
tags:
- Appwrite

---
## SSL证书

### Appwrite 中的 SSL 证书

#### 什么是 SSL？

SSL 是一种安全协议，它以加密方式为 Internet 上相互通信的计算机提供身份验证，几年前经过改进，后来被 [TLS](https://developer.mozilla.org/en-US/docs/Glossary/TLS) 取代。尽管 TLS 取代了 SSL，但这两个名称通常用于指代相同的过程：使用由[Certificate Authority](https://developer.mozilla.org/en-US/docs/Glossary/Certificate_authority)（证书颁发机构）（简称 CA）签署的证书密钥对（花式文本文件）来保护 HTTP 会话。

#### 信任证书

TLS 协议提供加密唯一的密钥对，不仅提供加密，而且在证书中包括域、主机和组织信息。但是，由于 TLS 技术是开源的，任何人都可以作为 CA 操作并签署证书。为了保证用户的安全，计算机和浏览器附带了预先审查的 CA 列表以自动信任\[1\]。使用这些可信来源颁发的证书的网站会在 URL 栏中的域旁边获得非常重要的锁🔒。然而，没有它们的网站面临着可怕的警告：`潜在的安全风险` 。

> 这里是 Mozilla 的 Firefox[ 可信来源](https://wiki.mozilla.org/CA/Included_Certificates)列表。

![](https://res.cloudinary.com/neroblackstone/image/upload/v1624875085/trusted_sources_f7s2ju.png)

成为这些名单上普遍信任的 CA 的过程可能[代价高昂](https://en.wikipedia.org/wiki/Key_ceremony)，这就是 IdenTrust 和 DigiCert 等组织对其服务收费的原因。这些公司拥有资源和知识来提供一系列安全保障，保护金融机构、政府、军队等。不过，我假设您没有开办银行，也没有资金获得商业 SSL 证书。如何免费成为CA呢？

#### Let's Encrypt

Let's Encrypt 是一个免费、自动化且值得信赖的证书颁发机构，旨在提供更安全、更安全的互联网。Appwrite 在后台使用他们的 [certbot](https://certbot.eff.org/) 工具来自动处理证书生成和更新，因此您可以专注于构建您的应用程序。

### 使用 HTTPS 保护 Appwrite

假设我已经在 VPS 上安装了 Appwrite，并为我的下一个由 Appwrite 驱动的项目购买了域名 `example.com`。在 `example.com`上运行我的应用程序需要哪些步骤？

#### 域记录

您的域名注册商最终可以控制您的域（我们的 `example.com`），因此我们需要从那里开始将域指向您的 VPS 的 IP 地址。为此，我们可以使用 DNS A 记录。向您的域添加 DNS 记录的过程因注册商而异，因此请查看关于[自定义域](https://appwrite.io/docs/custom-domains)的文档，以获取大量有用的链接和更具体的说明。

#### 开发环境中的 SSL 证书

如前所述，生成您自己的 SSL 证书所需的所有技术都是开源的，但它只是不受浏览器的全球信任。这对开发来说完全没问题（假设你相信自己）。Appwrite 提供开箱即用的自签名证书（通过 Traefik 代理），因此您的工作会立即加密。为此，我们需要让 Appwrite 知道我们正在尝试使用自签名证书。Appwrite的 SDK 都可以接受 client.setSelfSigned() 方法来处理这个问题。这是使用我们的 [Web SDK](https://appwrite.io/docs/getting-started-for-web) 的示例：

``` js
import * as Appwrite from "appwrite";

let client = new Appwrite();

client
    .setEndpoint('https://example.com/v1')
    .setProject('5df5acd0d48c2')
    .setSelfSigned()
;
```

#### 生产环境中的 SSL 证书

现在，假设您已经过了 `example.com` 的开发阶段，并且您已准备好进入生产阶段。 Appwrite 需要以下条件来颁发生产就绪的 SSL 证书（带锁🔒）：

* 通过 `_APP_ENV=production` 为 Appwrite开启 `production` 模式
* 通过 `_APP_SYSTEM_SECURITY_EMAIL_ADDRESS` 设置的有效电子邮件
* 通过 `_APP_DOMAIN` 设置的面向公众的域
* Traefik（代理网络服务器）侦听端口 80
* 删除对 client.setSelfSigned 的引用

> 在更改 Appwrite 的配置时，我们关于[ Appwrite 环境变量](https://appwrite.io/docs/environment-variables)的文档是一个很好的参考。

要应用新的环境变量，请运行：

    docker-compose up -d

这是 Appwrite Certificates worker 处理的阶段，调用 `certbot` 生成由 Let's Encrypt 签名的证书。然后，worker 将证书存储在 Docker 卷中以进行持久化，并定期检查证书更新（Let's Encrypt 证书的有效期默认为 90 天）。

### 调试

查找任何证书问题的第一个位置是 Certificates worker。您可以使用以下命令检查服务日志：

    docker-compose logs appwrite-worker-certificates

如果您在 Appwrite 服务器启动后配置了您的域，您可以通过重新启动 Appwrite 来重新触发 Certificates worker：

    docker-compose restart appwrite

## 服务器端 SDK

Appwrite 的愿景强调了这样一个事实，即后端即服务不应该只为前端开发人员设计。基于这一愿景，Appwrite 被设计为与平台无关，并与客户端和服务器端应用程序无缝集成。因为 Appwrite 是自托管的，所以它可以在您现有的防火墙后面使用，并与您现有的后端服务一起使用。Appwrite 的目标不是替换您的后端，而是与其一起工作。

Appwrite 支持 6 个服务器端 SDK。所有的 SDK 都是根据 API 的 Swagger 规范自动生成的。这允许Appwrite的小开发团队总共维护 8 个（客户端 + 服务端）SDK。如果您想帮助Appwrite团队用您喜欢的语言创建 SDK，请随时查看 [SDK Generator](https://github.com/appwrite/sdk-generator)。

### 服务端SDK与客户端SDK有何不同？

#### 身份验证

客户端和服务器端 SDK 之间的主要区别在于身份验证机制。服务器端 SDK 使用作用域 API 密钥来访问 Appwrite API，而客户端 SDK 则依赖安全 cookie 进行身份验证。

#### 范围

第二个主要区别是允许客户端和服务器端 SDK 访问的范围。范围限制了您可以使用 SDK 完成的任务类型。服务器端 SDK 提供了更多功能和灵活性，并允许您控制 Appwrite 的更多方面。使用您的 API 密钥，您可以使用您选择的 SDK 访问 Appwrite 服务。

要创建新的 API 密钥，请使用 Appwrite 控制台转到项目设置中的 API 密钥选项卡，然后单击“添加 API 密钥”按钮。添加新的 API 密钥时，您可以选择要授予应用程序的范围。最佳做法是仅允许满足项目目标所需的权限。如果您需要替换 API 密钥，请创建一个新密钥，更新您的应用凭据，并在准备好后删除旧密钥。

在服务器端使用带有 API 密钥的 Appwrite API 时，您将自动以`admin mode`运行。管理员模式禁用默认的[用户权限访问控制](https://appwrite.io/docs/permissions)限制，并允许您访问项目中的所有服务器资源（文档、用户、集合、文件、团队），无论读写权限如何。当您想要操作用户的数据（如文件和文档）或者甚至想要获取用户列表时，这非常有用。

> 不建议从客户端运行管理模式，因为这会导致巨大的隐私和安全风险。查看管理模式文档以了解更多信息。

下表很好地展示了您可以使用客户端和服务器端 SDK 做什么和不能做什么，并且很好地总结了我们所涵盖的内容。

| 名称 | 描述 | 服务端 | 客户端 |
| --- | --- | --- | --- |
| account | 代表当前登录用户的读写权限 | ❌ | ✅ |
| users.read | 有权读取您的项目的用户 | ✅ | ❌ |
| users.write | 拥有创建、更新和删除项目用户的权限 | ✅ | ❌ |
| teams.read | 有权读取您的项目团队 | ✅ | ✅ |
| teams.write | 拥有创建、更新和删除项目团队的权限 | ✅ | ✅ |
| collections.read | 有权读取项目的数据库集合 | ✅ | ❌ |
| collections.write | 有权创建、更新和删除项目的数据库集合 | ✅ | ❌ |
| documents.read | 有权读取您项目的数据库文档 | ✅ | ✅ |
| documents.write | 有权创建、更新和删除项目的数据库文档 | ✅ | ✅ |
| files.read | 有权读取项目的存储文件和预览图像 | ✅ | ✅ |
| files.write | 有权创建、更新和删除项目的存储文件 | ✅ | ✅ |
| functions.read | 有权读取项目的函数和代码标签 | ✅ | ❌ |
| functions.write | 有权创建、更新和删除项目的函数和代码标签 | ✅ | ❌ |
| execution.read | 有权读取项目的执行日志 | ✅ | ✅ |
| execution.write | 有权执行您的项目功能 | ✅ | ✅ |
| locale.read | 有权访问您项目的 Locale 服务 | ✅ | ✅ |
| avatars.read | 有权访问您项目的 Avatars 服务 | ✅ | ✅ |
| health.read | 有权读取您的项目的健康状态 | ✅ | ❌ |

### 入门

开始使用服务器端 SDK 并发出第一个请求非常简单。对于本示例，我们将选择 Deno SDK - 同样的原则也适用于所有其他 SDK。

前往您的 Appwrite Dashboard 并**创建**一个新项目。为您的项目命名，然后单击“创建”开始。创建项目后，转到 **API 密钥**部分并创建一个具有所需范围的密钥（确保它具有 `users.read` 和 `users.write` 范围，因为示例依赖于此）。复制此密钥，因为我们将在下一步中需要它。还要记下您的**项目 ID** 和 **API Endpoint**，它们可以在 Appwrite 仪表板的“**设置**”部分下找到。

![](https://res.cloudinary.com/neroblackstone/image/upload/v1624879340/appwrite_project_settings_evshwq.png)

是时候初始化您的 SDK 并发出您的第一个请求了。填写您在上一步中复制的所有值。然后我们将尝试使用 Appwrite SDK 创建一个用户。