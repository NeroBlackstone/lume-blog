---
title: Appwrite学习笔记(三)
description: ''
date: 2021-07-26
img: https://res.cloudinary.com/neroblackstone/image/upload/v1624671830/appwrite_i2voda.webp
tags:
- Appwrite

---
## 检索博客文章

现在是时候将我们的主要功能集成到我们的应用程序博客文章中了。

我们将使用**帖子**集合让用户创建嵌入到他们的个人资料中的帖子。

为此，我们需要在 `appwrite.js` 文件中添加几个方法：

* 获取所有帖子
* 从用户获取所有帖子
* 获取单个帖子
* 创建/编辑/删除帖子

添加的第一种方法将是获取所有帖子的方法。从技术上讲，我们希望使用 `listDocuments` 方法从服务器检索最新的 25 个帖子，从新到旧排序，这些帖子的`published`属性为 `true`。为此，我们将在 `appwrite.js` 文件中添加以下内容：

``` js
export const api = {
    //...
    fetchPosts: (limit, offset) => {
        return sdk.database.listDocuments(
            postsCollection,
            ["published=1"],
            limit,
            offset,
            "created_at",
            "DESC",
            "int"
        );
    },
    //...
}
```

为了从用户那里获取所有帖子，我们的方法看起来很相似——除了我们将通过 user_id 属性中的用户 ID 进行过滤：

``` js
export const api = {
  //...
    fetchUserPosts: userId => {
        return sdk.database.listDocuments(
            postsCollection,
            [
                `user_id=${userId}`,
                "published=1"
            ],
            100,
            0,
            "created_at",
            "DESC",
            "int"
        );
  },
  //...
}
```

要获取单个帖子，我们将使用 `getDocument` 方法来传递 ID，而不是以前使用的 `listDocuments`。

``` js
export const api = {
    //...
    fetchPost: id => sdk.database.getDocument(postsCollection, id),
    //...
}
```

对于删除帖子，我们可以使用 `deleteDocument` 方法，如下所示：

``` js
export const api = {
    //...
    deletePost: id => sdk.database.deleteDocument(postsCollection, id),
    //...
}
```

现在我们已经准备好检索博客文章的所有 API 请求，我们现在需要为其添加路由和组件。为此，我们编辑 `src/routes/Index.svelte` 文件，该文件将显示所有博客文章。

``` html
<script>
    import md from "snarkdown";
    import Loading from "../lib/Loading.svelte";
    import Action from "../lib/Action.svelte";
    import Author from "../lib/Author.svelte";
    import Preview from "../lib/Preview.svelte";
    import { api } from "../appwrite";
    const data = api
        .fetchPosts(25, 0)
        .then(r => r.documents)
        .then(posts => {
            return {
                promoted: posts[0],
                featured: posts.slice(1, 5),
                latest: posts.slice(5),
            };
        });
</script>

{#await data}
    <Loading />
{:then { promoted, featured, latest }}
    <section class="top">
        <div class="promoted">
            {#if promoted.cover}
                <img src={promoted.cover} alt={promoted.title} />
            {/if}
            <h2>{promoted.title}</h2>
            <Author user={promoted.user_id} />
            <p>
                {@html md(promoted.text)}
            </p>
            <Action href={`#/post/${promoted.$id}`}>Read more</Action>
        </div>
        <div class="cards">
            {#each featured as feature}
                <a class="card" href={`#/post/${feature.$id}`}>
                    {#if feature.cover}
                        <img
                            src={feature.cover}
                            alt={feature.title} />
                    {/if}
                    <h2>{feature.title}</h2>
                </a>
            {/each}
        </div>
    </section>
    <h1>Latest</h1>
    <section class="latest">
        {#each latest as post}
            <Preview {post} />
        {/each}
    </section>
{/await}

<style>
    section.top {
        display: flex;
        justify-content: space-evenly;
        gap: 1rem;
    }
    section.latest {
        display: flex;
        flex-wrap: wrap;
        flex-direction: row;
        justify-content: center;
        align-items: auto;
        align-content: start;
        gap: 1rem;
    }
    img {
        width: 100%;
    }
    .promoted img {
        border-radius: 0.5rem;
    }
    .cards {
        display: flex;
        flex-direction: column;
        gap: 3rem;
    }
    .cards .card {
        font-size: 0.75rem;
        display: flex;
        border-radius: 0.5rem;
        align-items: center;
        gap: 0.5rem;
        background-color: white;
        transition: all 0.2s;
    }
    .cards .card:hover {
        background-color: #f02e65;
        color: white;
        transform: scale(1.05);
    }
    .card img {
        width: 50%;
        height: 100%;
        border-radius: 0.5rem;
        object-fit: cover;
    }
</style>
```

在这个例子中，`fetchPosts()` 方法从我们的数据库中检索最新的 25 个帖子，并将它们拆分为以下对象结构：

* **推广** - 最新帖子
* **精选** - 在**推广**之后的接下来的 4 个帖子
* **最新** - 所有剩余的帖子

我们创建了一个个人资料页面，但还没有帖子。要添加此功能，我们将重新访问 `src/routes/Profile.svelte` 并更新以下代码

``` html
<script>
    import Preview from "../lib/Preview.svelte";
    import MyPost from "../lib/MyPost.svelte";
    //...
    const fetchUser = () => api.fetchUser(params.id);
    const fetchPosts = () => api.fetchUserPosts(params.id).then(r => r.documents);
    let all = Promise.all([fetchUser(), fetchPosts()]);
</script>

<section>
    {#await all}
        <Loading />
    {:then [author, posts]}
        <section class="author">
            <h3>{author.name}</h3>
        </section>
        {#if $state.user.$id == params.id}
            <h1>My Posts</h1>
            <p><a class="button" href="/create" use:link>Create</a></p>
            <section class="my-post">
                {#each posts as post}
                    <MyPost on:deleted={() => {all = Promise.all([fetchUser(), fetchPosts()]); console.log("deleted")} } {post} />
                {/each}
            </section>
        {:else}
            <h1>Latest Posts</h1>
            <section class="latest">
                {#each posts as post}
                    <Preview {post} />
                {/each}
            </section>
        {/if}
    {:catch error}
        {error}
        <p>
            Public profile not found
            <a href="/profile/create" use:link>Create Public Profile</a>
        </p>
    {/await}
</section>
```

我们在这里使用了两个尚未创建的组件。`MyPost` 是一个可编辑的组件，只会向帖子的所有者显示，并允许他们编辑和删除他们的帖子。

另一方面，`Preview`组件是只读组件，仅用于显示博客文章的预览。我们将在 `Index` 路由中重用这个组件。

**src/lib/Preview.svelte**

``` html
<script>
    export let post;
</script>

<a href={`#/post/${post.$id}`}>
    {#if post.cover}
        <img
            class="cover"
            src={post.cover}
            alt={post.title} />
    {/if}
    <h2>{post.title}</h2>
</a>

<style>
    img.cover {
        width: 100%;
        border-radius: 0.5rem;
    }
    a {
        display: flex;
        flex-direction: column;
        justify-content: flex-start;
        align-items: center;
        border-radius: 0.5rem;
        background-color: white;
        max-width: 18rem;
        font-size: 1.1rem;
        line-height: 2rem;
        transition: all 0.2s;
    }
    a:hover {
        background-color: #f02e65;
        color: white;
        transform: scale(1.05);
    }
    h2 {
        font-size: 1.1rem;
        margin: 0.5rem;
        text-align: center;
    }
</style>
```

**src/lib/MyPost.svelte**

``` html
<script>
    import { createEventDispatcher } from "svelte";
    import { link } from "svelte-spa-router";
    import { api } from "../appwrite";
    export let post;
    const dispatch = createEventDispatcher()
    const deletePost = async id => {
        if (confirm("are you sure you want to delete?")) {
            await api.deletePost(id);
            dispatch('deleted');
        }
    };
</script>

<article class="card">
    {#if post.cover}
        <img
            class="cover"
            src={post.cover}
            alt={post.title} />
    {/if}
    <h2>{post.title}</h2>
    <a href="/post/{post.$id}" use:link class="button">Preview</a>
    <a href="/post/{post.$id}/edit" use:link class="button">Edit</a>
    <a
        href="/delete"
        on:click|preventDefault={() => deletePost(post.$id)}
        class="button">Delete</a>
</article>

<style>
    article.card {
        background-color: white;
        display: flex;
        align-items: center;
        gap: 0.5rem;
        border-radius: 0.5rem;
    }
    img.cover {
        width: 8rem;
        border-top-left-radius: 0.5rem;
        border-bottom-left-radius: 0.5rem;
    }
    h2 {
        font-size: 1.1rem;
        margin: 0.5rem;
        text-align: center;
    }
</style>
```

现在只剩下显示单个博客文章的组件了。为此，我们将使用以下内容创建 **src/routes/Post.svelte**：

``` js
<script>
    import md from "snarkdown";
    import Loading from "../lib/Loading.svelte";
    import Author from "../lib/Author.svelte";
    import { api } from "../appwrite";

    export let params = {};

    let postFetch = api.fetchPost(params.slug);
</script>

{#await postFetch}
    <Loading />
{:then post}
    <h1>
        {post.title}
    </h1>
    <Author user={post.user_id} />
    {#if post.cover}
        <img class="cover" src={post.cover} alt={post.title} />
    {/if}
    <section class="content">
        {@html md(post.text)}
    </section>
    <h2>Comments</h2>
{/await}

<style>
    img.cover {
        width: 100%;
        border-radius: 0.5rem;
    }
    section.content {
        font-size: 1.1rem;
        line-height: 2rem;
    }
</style>
```

现在可以阅读所有博客文章，不幸的是我们无法验证这一点，因为我们的用户缺乏创建文章的能力。我们将在下一节中解决这个问题。

## 创建博客文章

现在我们要添加第一个组件，它将数据写入我们的 Appwrite 数据库。为此，我们将添加 `src/routes/Create.svelte` 文件并填充以下内容：

``` html
<script>
    import EasyMDE from "easymde";
    import { api } from "../appwrite";
    import { state } from "../store";
    import { onMount } from "svelte";
    import { replace } from 'svelte-spa-router';
    import "../../node_modules/easymde/dist/easymde.min.css";
    import Loading from "../lib/Loading.svelte";
    export let params = {};
    let published = false,
        title = "",
        easyMDE,
        message = "",
        loading = false,
        cover,
        post,
        content = "";
    let postFetch = async () => {
        post = await api.fetchPost(params.slug);
        title = post.title;
        easyMDE.value(post.text);
        cover = post.cover;
    };
    onMount(() => {
        if (params.slug) {
            postFetch();
        }
        easyMDE = new EasyMDE({ element: document.getElementById("content"), renderingConfig: {
            singleLineBreaks: true,
        } });
    });
    const submit = async () => {
        message = "";
        loading = true;
        let content = easyMDE.value();
        if (title.trim() == "" || content.trim() == "") {
            message = "Title and content are both required";
            console.log("title and content are both required");
            loading = false;
            return;
        }
        console.log({
            title: title,
            text: content,
            published: published,
            user: $state.user.$id,
            profile: $state.profile.$id,
        });
        try {
            let data = {
                    title: title,
                    text: content,
                    published: published,
                    user_id: $state.user.$id,
                    created_at: params.slug ? post.created_at :  new Date().getTime(),
                };
            if(params.slug) {
                //update
                await api.updatePost(params.slug,data,$state.user.$id)
                replace('/profile/'+$state.user.$id);
            } else {
                await api.createPost(
                    data,
                    $state.user.$id,
                    $state.profile.$id
                );
                easyMDE.value("");
                title = "";
                content = "";
                console.log("post created successfully");
                message = "Post created successfully";
            }
        } catch (error) {
            console.log(error);
            message = error;
        } finally {
            loading = false;
        }
    };
</script>

<section>
    {#if params.slug}
        <h2>Edit Post</h2>
    {:else}
        <h2>Create Post</h2>
    {/if}
    {#if message}
        <div class="alert">{message}</div>
    {/if}
    <form on:submit|preventDefault={submit}>
        <label for="title">Title</label>
        <input
            required
            type="text"
            placeholder="Enter title"
            bind:value={title} />
        <label for="content">Content</label>
        <textarea
            bind:value={content}
            name="content"
            id="content"
            cols="30"
            rows="10"
            placeholder="Enter content" />
        <label for="status">Status</label>
        <select name="status" id="status" bind:value={published}>
            <option value={false}>Draft</option>
            <option value={true}>Published</option>
        </select>
        <button disabled={loading ? true : false} class="button" type="submit"
            >{ params.slug ? 'Save' : 'Create'}</button>
    </form>
</section>

<style>
    form {
        display: flex;
        flex-direction: column;
    }
    label {
        margin-top: 1rem;
    }
    .alert {
        background-color: #ff000066;
        padding: 1rem;
    }
</style>
```

这允许用户创建和编辑他们的帖子。现在最后一步是在 `src/App.svelte` 上将所有组件添加到我们的路由器。

``` html
<script>
    //...
     import Post from "./routes/Post.svelte";
     import Create from "./routes/Create.svelte";
    //..    
    const routes = {
        //...
        "/create": Create,
        "/post/:slug": Post,
        "/post/:slug/edit": Create
    };
</script>
```