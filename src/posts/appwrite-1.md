---
title: Appwrite学习笔记（二）
description: Appwrite进阶内容
date: 2021-07-24
img: https://res.cloudinary.com/neroblackstone/image/upload/v1624671830/appwrite_i2voda.webp
tags:
- Appwrite

---
## 数据库设计

### 规划数据结构

我们将使用 Appwrite Database 提供的 **Collection** and **Rules** 功能来规划我们的应用程序所需的数据结构。首先，让我们放下我们的应用程序的要求。

#### Post

帖子是指任何经过身份验证的用户都可以发布的内容。任何人都可以注册并创建帖子。我们的帖子将包含**标题**、**封面**图片、**文本**、**已发布**（表示帖子是草稿还是已发布）、**标签**、**创建日期**和创建者的 **ID**。现在我们将使用 Appwrite 数据库提供的**规则**来计划这个。

首先，我们将创建一个集合，并将其命名为 **Posts**。然后我们将从控制台添加以下规则并更新集合。

* Title
  * label: Title
  * Key: title
  * Rule Type: Text
  * Required: true
  * Array: false
* Cover image
  * label: Cover image
  * Key: cover
  * Rule Type: Text
  * Required: false
  * Array: false
* Text
  * label: Text
  * Key: text
  * Rule Type: Markdown
  * Required: true
  * Array: false
* Published
  * label: Published
  * Key: published
  * Rule Type: Boolean
  * Required: true
  * Array: false
* Tags
  * label: Tags
  * Key: tags
  * Rule Type: Text
  * Required: false
  * Array: true
* Created Date
  * label: Created At
  * Key: created_at
  * Rule Type: Numeric
  * Required: true
  * Array: false
* User Id
  * label: User Id
  * Key: user_id
  * Rule Type: Text
  * Required: true
  * Array: false![](https://res.cloudinary.com/neroblackstone/image/upload/v1627117716/appwrite_post_rule_ze0nvj.png)

对于权限，读取权限应为 `[​​]`，因为任何人都应该能够阅读帖子，而此集合的写入权限应为 `[​​role:member]`，以便只有登录用户才能创建帖子。

#### Profile

我们想让我们的用户拥有一个公开名称的个人资料，以便我们可以在每篇文章中显示作者信息。我们还希望将用户的帖子作为嵌入文档添加到集合中，以便我们可以使用用户的个人资料轻松获取它。

让我们使用以下规则创建另一个名为 **Users** 的集合

* User Id
  * label: User
  * Key: user
  * Rule Type: Text
  * Required: true
  * Array: false
* Name
  * label: Name
  * Key: name
  * Rule Type: Text
  * Required: true
  * Array: false
* Posts
  * label: Posts
  * Key: posts
  * Rule Type: Document (Embedded)
  * Required: false
  * Array: true

创建**帖子**规则后，您将看到一个包含**允许集合**的部分。在这里，您需要选择**帖子**。

![](https://res.cloudinary.com/neroblackstone/image/upload/v1627118332/appwrite_profile_rule_axf8vk.png)

至于权限，读权限应该是\[*\]，因为任何人都应该可以读，写权限可以是\[role:member\]，这样任何登录的人都可以创建个人资料。

因此，这就是使用**集合**和**规则**为任何应用程序规划数据结构是多么容易——它与我们计划使用**表**和**列**的传统关系数据库非常相似。

## 创建用户

### 创建个人资料

我们将使用 **Profile** Collection 为我们应用上的用户提供一个带有公共名称的个人资料，以便我们可以在每个帖子中显示作者的信息。

为此，我们需要在 `appwrite.js` 文件中添加两种方法：一种用于获取配置文件，另一种用于创建配置文件。让我们限制用户只为一个帐户创建一个配置文件。为此，我们需要首先检查他是否已经有个人资料。因此，我们将添加一个 `fetchUser()` 函数，该函数将列出 **Profiles** 集合中的所有文档，用户字段等于我们帐户的 ID，并将文档数量限制为 1。

``` js
export const api = {
    //...
    fetchUser: async id => {
        let res = await sdk.database.listDocuments(
            profilesCollection,
            [`user=${id}`],
            1
        );
        if (res.sum > 0 && res.documents.length > 0) return res.documents[0];
        else throw Error("Not found");
    }
};
```

这里最重要的是带有 _\[user=${id}\]_ 的部分，它将通过等于传递的 `id` 的**用户**字段过滤请求的文档。

您可能已经注意到，我们在 `listDocuments` 调用中使用了一个名为 `profilesCollection` 的未知变量。为此，我们需要创建 2 个变量来表示我们集合的唯一 ID。只需在 `const api` 之前添加以下内容：

``` js
const profilesCollection = "[INSERT YOUR ID HERE]";
const postsCollection = "[INSERT YOUR ID HERE]";
```

确保使用仪表板中的 ID，并将 **Profile** 中的 ID 替换为 `profilesCollection`，并将 **Post** Collection 中的 ID 替换为 `postsCollection`。

现在我们可以检查profile是否存在，如果不存在，用户需要能够创建他们的配置文件。为此，我们将在 `appwrite.js` 中引入 `createUser` 方法：

``` js
export const api = {
    //...
    createUser: async (id, name) => {
        return sdk.database.createDocument(
            profilesCollection,
            {
                user: id,
                name: name,
            },
            ["*"],
            [`user:${id}`]
        );
    },
}
```

这将在调用时在 **Profile** 集合中创建一个文档。如您所见，第二个参数是一个遵守我们在第 16 天创建的收集规则的对象。

之后，`readwrite`权限就通过了。由于我们希望每个人都能够查看此配置文件，但只有用户自己才能对其进行编辑 - 读取权限为 *，写入权限为用户本身。

现在我们已经准备好了所有的 Appwrite 通信逻辑，我们现在需要为其添加路由和组件。为此，我们创建将显示配置文件的 `src/routes/Profile.svelte` 文件。

``` html
// src/routes/Profile.svelte
<script>
    import Loading from "../lib/Loading.svelte";

    import { api } from "../appwrite";
    import { state } from "../store";

    export let params = {};

    const fetchUser = api.fetchUser(params.id);
</script>

<section>
    {#await fetchUser}
        <Loading />
    {:then author}
        <section class="author">
            <h3>{author.name}</h3>
        </section>
        {#if $state.user.$id == params.id}
            <h1>My Posts</h1>
            <section class="my-post">
                TBD
            </section>
        {:else}
            <h1>Latest Posts</h1>
            <section class="latest">
                TBD
            </section>
        {/if}
    {:catch error}
        {error}
        <p>
            Public profile not found
            <a href="#/profile/create">Create Public Profile</a>
        </p>
    {/await}
</section>

<style>
    section.author {
        display: flex;
        align-items: center;
        gap: 1rem;
    }
    section.latest {
        display: flex;
        flex-direction: row;
        flex-wrap: wrap;
        justify-content: center;
        align-items: auto;
        align-content: start;
        gap: 1rem;
    }
    section.my-post {
        display: flex;
        flex-direction: column;
        justify-content: center;
        align-items: auto;
        align-content: start;
        gap: 0.5rem;
    }
    a {
        border: none;
        padding: 10px;
        color: white;
        font-weight: bold;
    }
    a:hover {
        text-decoration: underline;
    }
</style>
```

当我们发现错误时，我们会提示用户创建他们的个人资料并将他们导航到`#/profile/create`。由于尚未创建此路由，请创建一个名为 `src/routes/CreateProfile.svelte` 的新文件。和之前一样，我们将在 `src/App.svelte` 中将该组件引入路由器：

``` js
//src/App.svelte

import CreateProfile from "./routes/CreateProfile.svelte";  
// First import the svelte component

const routes = {
    //...
    "/profile/create": CreateProfile, // Add this component
    //...
  };
```

现在我们需要处理 `CreateProfile.svelte` 文件：

``` html
<script>
    import { state } from "../store";
    import { api } from "../appwrite";
    import { replace } from "svelte-spa-router";
    let name = $state.user.name;
    const submit = async () => {
        try {
            await api.createUser($state.user.$id, name);
            replace(`/profile/${$state.user.$id}`);
        } catch (error) {
            console.log(error.message);
        }
    };
</script>

<form on:submit|preventDefault={submit}>
    {#if $state.user}
        <label for="name">Display Name</label>
        <input type="text" name="name" bind:value={name} />
        <button class="button" type="submit">Create</button>
    {/if}
</form>

<style>
    form {
        margin: auto;
        width: 500;
        display: flex;
        flex-direction: column;
    }
</style>
```

这是一个简单的表单，用户可以在其中输入他的个人资料名称并创建它！

我们现在已经使用我们之前创建的数据库和集合向我们的应用程序添加了用户配置文件。

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