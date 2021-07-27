---
title: Appwrite学习笔记(六)
description: ''
date: 2021-07-27
img: https://res.cloudinary.com/neroblackstone/image/upload/v1624671830/appwrite_i2voda.webp
tags:
- Appwrite

---
## Appwrite头像API

今天我们将看一看 Appwrite 的 Avatars API 并查看它在引擎盖下的一些简洁功能！Avatars API 主要允许您为各种用例生成图标和头像。让我们深入了解一下它提供了什么。

### 信用卡图标

您可以轻松获取最受欢迎的信用卡公司的信用卡图标，例如 AmEx、Discover、JCB、Mastercard、Visa、Maestro 等。

此[端点](https://appwrite.io/docs/client/avatars#avatarsGetCreditCard)还允许您在请求时自定义图标大小和质量。您可以在[此处](https://github.com/appwrite/appwrite/tree/master/app/config/avatars/credit-cards)找到支持的信用卡的完整列表。

### 浏览器图标

该[端点](https://appwrite.io/docs/client/avatars?sdk=web#avatarsGetBrowser)可以方便您获取一些常用浏览器的图标。如果您还没有看到，我们在 Appwrite 控制台中使用此端点来显示有关用户会话的信息。

![](https://res.cloudinary.com/neroblackstone/image/upload/v1627373284/appwrite_browser_icon_ce7mnr.png)

您可以在[此处](https://github.com/appwrite/appwrite/tree/master/app/config/avatars/browsers)找到支持的浏览器图标的完整列表。

### 国旗

与浏览器图标端点类似，此[端点](https://appwrite.io/docs/client/avatars#avatarsGetFlag)允许您获取国家/地区标志的图标。您可以在上面的 Appwrite 控制台屏幕截图中看到它正在使用中。您可以在[此处](https://github.com/appwrite/appwrite/tree/master/app/config/avatars/flags)找到所有国家/地区代码和旗帜的完整列表。

### 来自 URL 的图像

如果您需要在应用程序中裁剪和显示远程图像，或者您想确保使用 TLS 协议正确提供 3rd 方图像，则[此端点](https://appwrite.io/docs/client/avatars#avatarsGetImage)非常有用。

### 获取网站图标

收藏夹图标是与网站、网页或 Web 应用程序相关联的小图标或图标集合。它显示在浏览器选项卡和书签栏中。[此端点](https://appwrite.io/docs/client/avatars#avatarsGetFavicon)允许您获取任何远程 URL 的图标。![](https://res.cloudinary.com/neroblackstone/image/upload/v1627373520/appwrite_favicon_zwr3iv.png)

### 二维码

此[端点](https://appwrite.io/docs/client/avatars#avatarsGetQR)允许您为任何字符串生成二维码。您如何使用它仅受您的创造力的限制，因为它可用于共享 URL、电话号码，甚至 base64 编码的图像。我们将利用此功能在我们的演示应用程序中添加社交共享功能。

### 获取用户姓名首字母

[此端点](https://appwrite.io/docs/client/avatars?sdk=web#avatarsGetInitials)提供了一种非常方便的方法来根据用户的姓名首字母为您的用户获取头像。您可以将其用作占位符，直到用户上传个人资料图片。您还可以使用此端点为任何字符串（不一定是名称）生成头像。此外，如果您对默认设置不满意，您可以调整图像大小、文本颜色和背景颜色。

## 让我们写一些代码

在我们的演示应用程序中，我们将添加一个分享文章功能。这将允许用户将文章分享到各种社交媒体平台，甚至为当前 URL 生成二维码，然后您可以与朋友分享。

第一步是在 `src/appwrite.js` 中添加一个新函数来调用 Avatars 服务：

``` js
export const api = {
    ...
    getQRcode: text => sdk.avatars.getQR(text)
    ...
}
```

现在网络层已准备就绪，让我们前往 `src/routes/Post.svelte` 组件，我们将在其中创建用于共享的按钮。将以下标记复制到 HTML 的最后一部分：

``` html
<!-- Share -->
<section>
  <div class="share-buttons-container">
    <div class="share-list">
      <!-- FACEBOOK -->
      <a class="fb-h" on:click="{fbs_click}" target="_blank">
        <img
          src="https://img.icons8.com/material-rounded/96/000000/facebook-f.png"
        />
      </a>

      <!-- TWITTER -->
      <a class="tw-h" on:click="{tbs_click}" target="_blank">
        <img
          src="https://img.icons8.com/material-rounded/96/000000/twitter-squared.png"
        />
      </a>

      <!-- LINKEDIN -->
      <a class="li-h" on:click="{lbs_click}" target="_blank">
        <img
          src="https://img.icons8.com/material-rounded/96/000000/linkedin.png"
        />
      </a>

      <!-- REDDIT -->
      <a class="re-h" on:click="{rbs_click}" target="_blank">
        <img src="https://img.icons8.com/ios-glyphs/90/000000/reddit.png" />
      </a>

      <!-- PINTEREST -->
      <a
        data-pin-do="buttonPin"
        data-pin-config="none"
        class="pi-h"
        on:click="{pbs_click}"
        target="_blank"
      >
        <img src="https://img.icons8.com/ios-glyphs/90/000000/pinterest.png" />
      </a>

      <!-- QR Code -->
      <a class="pi-h" on:click="{qrcode_click}" target="_blank">
        <img
          src="https://img.icons8.com/ios-glyphs/60/000000/qr-code--v1.png"
        />
      </a>
    </div>
  </div>
  {#if qrCode}
  <img src="{qrCode}" alt="No QR Code" />
  {/if}
</section>
```

我们还需要为此添加一些样式。我建议从[这里](https://github.com/christyjacob4/30-days-of-appwrite/blob/add-qr-code-share/src/routes/Post.svelte#L157-L209)复制所有样式。

现在是时候添加一些 Javascript 将它们拼接在一起了。在 `src/routes/Post.svelte` 的 `<script>` 部分添加以下代码：

``` js
let qrCode = null;
var pageLink = window.location.href;
var pageTitle = String(document.title).replace(/\&/g, "%26");
const fbs_click = () => {
  window.open(
    `http://www.facebook.com/sharer.php?u=${pageLink}&quote=${pageTitle}`,
    "sharer",
    "toolbar=0,status=0,width=626,height=436"
  );
  return false;
};
const tbs_click = () => {
  window.open(
    `https://twitter.com/intent/tweet?text=${pageTitle}&url=${pageLink}`,
    "sharer",
    "toolbar=0,status=0,width=626,height=436"
  );
  return false;
};
const lbs_click = () => {
  window.open(
    `https://www.linkedin.com/sharing/share-offsite/?url=${pageLink}`,
    "sharer",
    "toolbar=0,status=0,width=626,height=436"
  );
  return false;
};
const rbs_click = () => {
  window.open(
    `https://www.reddit.com/submit?url=${pageLink}`,
    "sharer",
    "toolbar=0,status=0,width=626,height=436"
  );
  return false;
};
const pbs_click = () => {
  window.open(
    `https://www.pinterest.com/pin/create/button/?&text=${pageTitle}&url=${pageLink}&description=${pageTitle}`,
    "sharer",
    "toolbar=0,status=0,width=626,height=436"
  );
  return false;
};
let qrcode_click = async () => {
  qrCode = await api.getQRcode(pageLink);
};
```

真的是这样！您现在可以一键将您的文章分享到社交媒体平台，还可以分享带有文章链接的二维码。如果您想查看此功能中的确切文件更改，可以查看[此 PR](https://github.com/christyjacob4/30-days-of-appwrite/pull/6/files)。