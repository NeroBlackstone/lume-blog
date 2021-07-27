---
title: Appwrite学习笔记(五)
description: ''
date: 2021-07-27
img: https://res.cloudinary.com/neroblackstone/image/upload/v1624671830/appwrite_i2voda.webp
tags:
- Appwrite

---
## 文件上传和下载

使用 Appwrite 的 Storage API 上传文件、预览图像和下载文件非常容易。正如我们之前了解到的，Storage API 还提供了操作图像和显示预览的选项。

在今天的会议中，我们将研究如何利用这一点将封面图片添加到我们的博客文章中。根据我们是查看帖子的详细信息还是仅列出帖子，我们可以将图像缩略图裁剪为合适的大小以显示为预览。让我们学习如何。

## 新帖子组件

首先我们将修改新的 post 组件 `src/routes/Create.svelte` 以在 title 字段上方添加一个 File 输入字段。

``` html
<label for="cover">Cover</label>
<input type="file" bind:files />
```

然后我们将修改同一文件的脚本部分中的提交函数来处理文件输入。

``` js
let cover;

const submit = async () => {
    //...code before
    if (files && files.length == 1) {
        let file = await api.uploadFile(files[0], $state.user.$id);
        cover = file.$id;
    }
    //... code after
}
```

现在我们需要在 `src/appwrite.js` 中创建一个 `uploadFile()` 函数。

``` js
export const api = {
    //...existing code
    uploadFile: (file, userId) =>
        sdk.storage.createFile(file, ["*"], [`user:${userId}`]),
}
```

现在，在编辑帖子时，必须处理文件。如果用户上传新的封面图片，我们删除旧的并上传新的。所以首先让我们在 `src/appwrite.js` 中添加 `deleteFile()` 函数。

``` js
export const api = {
    //...existing codes
    deleteFile: (id) => sdk.storage.deleteFile(id),
}
```

另外，修改 `src/routes/Create.svelte` 中的提交函数来处理编辑时的删除。

``` js
const submit = async () => {
    //...code before
    if (files && files.length == 1) {
        if(params.slug) {
            await api.deleteFile(cover);
        }
        let file = await api.uploadFile(files[0], $state.user.$id);
        cover = file.$id;
    }
    //... code after
}
```

## 图像预览

Appwrite 的存储 API 提供了一种使用其简单的 rest API 生成预览缩略图的简单方法。让我们在 `src/appwrite.js` 中创建一个名为 `getThumbnail` 的方法，它将为我们的博客文章生成适当大小的缩略图。

``` js
getThumbnail: (id, width = 1000, height = 600) =>
    sdk.storage.getFilePreview(id, width, height),
```

上述函数将生成具有提供的宽度和高度的图像缩略图。

现在我们将使用该函数在我们的索引组件、帖子预览组件和用户自己的帖子预览组件中显示图像。

首先，在 `src/routes/Index.svelte` 下的promotion and features 部分，如果有封面图片，我们添加图片预览。

Promoted post

``` html
<div class="promoted">
    {#if promoted.cover}
        <img src={api.getThumbnail(promoted.cover)} alt="" />
    {/if}
    <h2>{promoted.title}</h2>
    <Author user={promoted.user_id} />
    <p>
        {@html md(promoted.text)}
    </p>
    <Action href={`#/post/${promoted.$id}`}>Read more</Action>
</div>
```

Featured post

``` html
<a class="card" href={`#/post/${feature.$id}`}>
    {#if feature.cover}
        <img
            src={api.getThumbnail(feature.cover, 600, 400)}
            alt="" />
    {/if}
    <h2>{feature.title}</h2>
</a>
```

让我们也修改 `src/lib/Preview.svelte` 以显示封面图片

``` html
<a href={`#/post/${post.$id}`}>
    {#if post.cover}
        <img
            class="cover"
            src={api.getThumbnail(post.cover, 400, 250)}
            alt="" />
    {/if}
    <h2>{post.title}</h2>
</a>
```

同样，修改 `src/lib/MyPost.svelte` 也显示封面图片

``` html
<article class="card">
    {#if post.cover}
        <img
            class="cover"
            src={api.getThumbnail(post.cover, 400, 250)}
            alt="" />
    {/if}
    <h2>{post.title}</h2>
    <a href="/post/{post.$id}" use:link class="button">Preview</a>
    <a href="/post/{post.$id}/edit" use:link class="button">Edit</a>
    <a
        href="/delete"
        on:click|preventDefault={() => deletePost(post.$id)}
        class="button">Delete</a>
</article>
```

最后，同样修改 `src/routes/Post.svelte`

``` html
<h1>
    {post.title}
</h1>
<Author user={post.user_id} />
{#if post.cover}
    <img class="cover" src={api.getThumbnail(post.cover)} alt="" />
{/if}
<section class="content">
    {@html md(post.text)}
</section>
```

这就是使用 Appwrite 上传文件和显示不同大小的图像缩略图是多么容易 - 无需使用任何外部库。生成后，缩略图会缓存在服务器中，因此在运行时响应速度非常快。