---
title: 解决yarn set version berry无用问题
description: 还是官方discord提问靠谱
date: 2021-08-03
img: https://res.cloudinary.com/neroblackstone/image/upload/v1627974352/download_deimx9.jpg
tags:
- Yarn

---
Yarn pkg最近更新了3.0 rc，我也打算试用一下。但是按照官方教程初始化无用。yarn版本始终是1.22

![](https://res.cloudinary.com/neroblackstone/image/upload/v1627974465/Screenshot_from_2021-08-03_14-14-28_gat3uo.png)

这是因为全局的yarn设定覆盖了局部的设定。打开`\~/.yarnrc(.yml)`并且清除`yarnPath`字段即可。

![](https://res.cloudinary.com/neroblackstone/image/upload/v1627974575/Screenshot_from_2021-08-03_15-03-43_kwu5e3.png)