---
title: Appwrite学习笔记(四)
description: ''
date: 2021-07-26
img: https://res.cloudinary.com/neroblackstone/image/upload/v1624671830/appwrite_i2voda.webp
tags:
- Appwrite

---
## Appwrite存储 API

每个应用程序不仅需要一个数据库，还需要存储。 Appwrite 捆绑了一个广泛的存储 API，允许您管理项目的文件。Appwrite 的存储服务为我们提供了一个时尚的 API 来上传、下载、预览和操作图像。

## 存储是如何实现的

截至目前，Appwrite 使用宿主机的存储挂载一个 Docker 卷来提供存储服务。因此，它使用本地文件系统来存储您上传到 Appwrite 的所有文件。我们正在努力增加对 AWS S3、DigitalOcean Spaces 或其他类似服务等外部对象存储的支持。您可以在我们的 [utopia-php/storage](https://github.com/utopia-php/storage) 库中查看这方面的进展。

## 使用 Appwrite 控制台管理存储

Appwrite 的控制台支持轻松管理存储中的文件。从这里您可以创建文件、更新文件的元数据、查看或下载文件以及删除文件。

![](https://res.cloudinary.com/neroblackstone/image/upload/v1627292596/Appwrite_Console_storage_urfjlg.png)

您可以通过单击右下角的圆形 + 按钮轻松创建新文件。

![](https://res.cloudinary.com/neroblackstone/image/upload/v1627292687/appwrite_choose_file_roofgw.png)

服务中的每个文件都被授予读写权限，以管理谁有权查看或编辑它。这些权限的工作方式与数据库权限的工作方式相同。

对于存储中的现有文件，您可以直接从控制台查看预览和更新权限。您也可以在新窗口中查看原始文件或下载文件。

![](https://res.cloudinary.com/neroblackstone/image/upload/v1627292788/appwrite_update_file_h5i4sc.png)

## 服务

Storage API 为我们提供了几个不同的端点来操作我们的文件。

### 创建文件

我们可以发出 **POST** `<ENDPOINT>/storage/files` 请求来上传文件。在 SDK 中，此端点公开为 `storage.createFile()`。它需要三个参数：二进制`file`、定义`read`取权限的字符串数组，以及相同的`write`权限。

我们可以使用以下代码从我们的 Web SDK 创建文件。

``` js
let sdk = new Appwrite();

sdk
    .setEndpoint('https://demo.appwrite.io/v1') // Your API Endpoint
    .setProject('5df5acd0d48c2') // Your project ID
;

let promise = sdk.storage
    .createFile(document.getElementById('uploader').files[0], ['*'], ['role:member']);

promise.then(function (response) {
    console.log(response); // Success
}, function (error) {
    console.log(error); // Failure
});
```

或者在 Flutter 中我们可以使用

``` dart
import 'package:appwrite/appwrite.dart';

void main() { // Init SDK
  Client client = Client();
  Storage storage = Storage(client);

  client
    .setEndpoint('https://demo.appwrite.io/v1') // Your API Endpoint
    .setProject('5df5acd0d48c2') // Your project ID
  ;
  Future result = storage.createFile(
    file: await MultipartFile.fromFile('./path-to-files/image.jpg', 'image.jpg'),
    read: ['*'],
    write: ['role:member'],
  );

  result
    .then((response) {
      print(response);
    }).catchError((error) {
      print(error.response);
  });
}
```

### 列出文件

我们可以发出 **GET** `<ENDPOINT>/storage/files` 请求以列出文件。在 SDK 中，这公开为 `storage.listFiles()`。我们还可以使用`search`参数来过滤结果。您还可以包含 `limit`、`offset` 和 `orderType` 参数以进一步自定义返回的结果。

使用 Web SDK：

``` js
let sdk = new Appwrite();

sdk
    .setEndpoint('https://demo.appwrite.io/v1') // Your API Endpoint
    .setProject('5df5acd0d48c2') // Your project ID
;

let promise = sdk.storage.listFiles();

promise.then(function (response) {
    console.log(response); // Success
}, function (error) {
    console.log(error); // Failure
});
```

使用Flutter：

``` dart
import 'package:appwrite/appwrite.dart';

void main() { // Init SDK
  Client client = Client();
  Storage storage = Storage(client);

  client
    .setEndpoint('https://demo.appwrite.io/v1') // Your API Endpoint
    .setProject('5df5acd0d48c2') // Your project ID
  ;
  Future result = storage.listFiles();

  result
    .then((response) {
      print(response);
    }).catchError((error) {
      print(error.response);
  });
}
```

### 获取文件

我们可以通过其 id 向单个文件发出 **GET** `<ENDPOINT>/storage/files/{fileid}` 请求。它返回一个带有文件元数据的 JSON 对象。在 SDK 中，此端点公开为 `storage.getFiles()`，它需要 `fileId` 参数。

使用 Web SDK：

``` js
let sdk = new Appwrite();

sdk
    .setEndpoint('https://[HOSTNAME_OR_IP]/v1') // Your API Endpoint
    .setProject('5df5acd0d48c2') // Your project ID
;

let promise = sdk.storage.getFile('[FILE_ID]');

promise.then(function (response) {
    console.log(response); // Success
}, function (error) {
    console.log(error); // Failure
});
```

使用Flutter：

``` dart
import 'package:appwrite/appwrite.dart';

void main() { // Init SDK
  Client client = Client();
  Storage storage = Storage(client);

  client
    .setEndpoint('https://[HOSTNAME_OR_IP]/v1') // Your API Endpoint
    .setProject('5df5acd0d48c2') // Your project ID
  ;
  Future result = storage.getFile(
    fileId: '[FILE_ID]',
  );

  result
    .then((response) {
      print(response);
    }).catchError((error) {
      print(error.response);
  });
}
```

### 文件预览

预览端点允许您为文件生成预览图像。使用预览端点，您还可以操作生成的图像，使其在尺寸、文件大小和样式方面完全适合您的应用程序。此外，预览端点允许您更改生成的图像文件格式以实现更好的压缩或更改图像质量以更好地通过网络交付。

我们可以发出 **GET** `<ENDPOINT>/storage/files/{fileId}/preview` 请求来获取图像文件的预览。它支持`width`, `height`, `quality`, `background` 和 `output`格式等参数来操作预览图像。此方法公开为 `storage.getFilePreview()` 并需要一个 `fileId`。

> 我们为预览端点提供了更多出色的功能，例如边框、边框半径和不透明度支持。

使用 Web SDK：

``` js
let sdk = new Appwrite();

sdk
    .setEndpoint('https://[HOSTNAME_OR_IP]/v1') // Your API Endpoint
    .setProject('5df5acd0d48c2') // Your project ID
;

let result = sdk.storage.getFilePreview('[FILE_ID]', 100, 100); //crops the image into 100x100

console.log(result); // Resource URL
```

使用Flutter：

``` dart
import 'package:appwrite/appwrite.dart';

void main() { // Init SDK
  Client client = Client();
  Storage storage = Storage(client);

  client
    .setEndpoint('https://[HOSTNAME_OR_IP]/v1') // Your API Endpoint
    .setProject('5df5acd0d48c2') // Your project ID
  ;
}

//displaying image
FutureBuilder(
  future: storage.getFilePreview(
    fileId: '[FILE_ID]',
    width: 100
    height: 100
  ),
  builder: (context, snapshot) {
    return snapshot.hasData && snapshot.data != null
      ? Image.memory(
          snapshot.data.data,
        )
      : CircularProgressIndicator();
  },
);
```

### 文件下载

我们可以发出 **GET** `<ENDPOINT>/storage/files/{fileId}/download` 请求以通过其唯一 ID 获取文件的内容。端点响应包含一个 `Content-Disposition:` `attachment`标头，它告诉浏览器开始将文件下载到用户下载目录。此方法公开为 `storage.getFileDownload()`，并且需要一个 `fileId`。

使用 Web SDK：

``` js
let sdk = new Appwrite();

sdk
    .setEndpoint('https://[HOSTNAME_OR_IP]/v1') // Your API Endpoint
    .setProject('5df5acd0d48c2') // Your project ID
;

let result = sdk.storage.getFileDownload('[FILE_ID]');

console.log(result); // Resource URL
```

使用Flutter：

``` dart
import 'package:appwrite/appwrite.dart';

void main() { // Init SDK
  Client client = Client();
  Storage storage = Storage(client);

  client
    .setEndpoint('https://[HOSTNAME_OR_IP]/v1') // Your API Endpoint
    .setProject('5df5acd0d48c2') // Your project ID
  ;
}

//displaying image
FutureBuilder(
  future: storage.getFileDownload(
    fileId: '[FILE_ID]',
  ),
  builder: (context, snapshot) {
    return snapshot.hasData && snapshot.data != null
      ? Image.memory(
          snapshot.data.data,
        )
      : CircularProgressIndicator();
  },
);
```

### 文件查看

我们可以发出 **GET** `<ENDPOINT>/storage/files/{fileId}/view` 请求以通过其唯一 ID 获取文件的内容。此端点类似于下载方法，但返回时没有 `Content-Disposition:attachment`标头。

使用 Web SDK：

``` js
let sdk = new Appwrite();

sdk
    .setEndpoint('https://[HOSTNAME_OR_IP]/v1') // Your API Endpoint
    .setProject('5df5acd0d48c2') // Your project ID
;

let result = sdk.storage.getFileView('[FILE_ID]');

console.log(result); // Resource URL
```

使用Flutter：

``` dart
import 'package:appwrite/appwrite.dart';

void main() { // Init SDK
  Client client = Client();
  Storage storage = Storage(client);

  client
    .setEndpoint('https://[HOSTNAME_OR_IP]/v1') // Your API Endpoint
    .setProject('5df5acd0d48c2') // Your project ID
  ;
}

//displaying image
FutureBuilder(
  future: storage.getFileView(
    fileId: '[FILE_ID]',
  ),
  builder: (context, snapshot) {
    return snapshot.hasData && snapshot.data != null
      ? Image.memory(
          snapshot.data.data,
        )
      : CircularProgressIndicator();
  },
);
```

有关存储服务的更多详细信息，请参见我们的[文档](https://appwrite.io/docs/client/storage)。