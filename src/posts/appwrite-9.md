---
title: Appwrite学习笔记(十)
description: 使用团队邀请
date: 2021-07-22
img: https://res.cloudinary.com/neroblackstone/image/upload/v1624671830/appwrite_i2voda.webp
tags:
- Appwrite

---
我们将在本文的演示应用程序中实现以下功能。

1. [创建团队](https://appwrite.io/docs/client/teams?sdk=web#teamsCreate)
2. [列出用户的团队](https://appwrite.io/docs/client/teams?sdk=web#teamsList)
3. [删除团队](https://appwrite.io/docs/client/teams?sdk=web#teamsDelete)
4. [通过 ID 获取团队](https://appwrite.io/docs/client/teams?sdk=web#teamsGet)
5. [获取团队成员](https://appwrite.io/docs/client/teams#teamsGetMemberships)
6. [添加新的团队成员](https://appwrite.io/docs/client/teams#teamsCreateMembership)
7. [更新会员状态](https://appwrite.io/docs/client/teams#teamsUpdateMembershipStatus)
8. [从团队中删除用户](https://appwrite.io/docs/client/teams#teamsDeleteMembership)

我们将在我们的项目中创建三个新路由。

1. 一个 `/profile/:id/teams` 路由，允许用户查看他们所属的所有团队并创建新团队。这条路线将实现功能 \[1,2,3\]
2. `/team/:id` 路由将显示特定团队 ID 的详细信息，并允许用户管理团队成员。此路线将实现功能 \[3,4,5,6,8\]
3. `/acceptMembership` 路由，使新团队成员能够接受团队邀请。此路由将实现功能 \[7\]

## 设置

让我们开始吧。在 `src/App.svelte` 中创建三个新路由。

``` js
import Team from "./routes/Team.svelte";
import Teams from "./routes/Teams.svelte";
import AcceptMembership from "./routes/AcceptMembership.svelte";
const routes = {
    ...
    "/profile/:id/teams" : Teams,
    "/team/:id" : Team,
    "/acceptMembership": AcceptMembership,
    ...
};
```

转到 `src/appwrite.js` 并添加以下函数：

``` js
...
fetchUserTeams: () => sdk.teams.list(),
createTeam: name => sdk.teams.create(name),
deleteTeam: id => sdk.teams.delete(id),
getTeam: id => sdk.teams.get(id),
getMemberships: teamId => sdk.teams.getMemberships(teamId),
createMembership: (teamId, email, roles, url, name) =>
    sdk.teams.createMembership(teamId, email, roles, url, name),
updateMembership: (teamId, inviteId, userId, secret) =>
    sdk.teams.updateMembershipStatus(teamId, inviteId, userId, secret),
deleteMembership: (teamId, inviteId) =>
    sdk.teams.deleteMembership(teamId, inviteId)
...
```

在 `src/lib/Navigation.svelte` 中，我们将创建一个指向主 `/profile/:id/teams` 路由的链接。

``` html
...
{#if $state.user}
    <a href={`/profile/${$state.user.$id}`} use:link>{$state.user.name}</a>
    <a href={`/profile/${$state.user.$id}/teams`} use:link>My Teams</a>
    <a href="/logout" use:link>Logout</a>
{:else}
...
```

## 创建一个页面来显示所有用户的团队

创建文件 `src/routes/Teams.svelte`。我们将在此处显示用户的所有团队并允许用户创建更多团队。在 `<script>` 部分中添加以下代码。

``` html
<script>
  import { link } from "svelte-spa-router";
  import Avatar from "../lib/Avatar.svelte";
  import Loading from "../lib/Loading.svelte";
  import { api } from "../appwrite";
  export let params = {};
  let name;
  const fetchUser = () => api.fetchUser(params.id);
  const getAvatar = (name) => api.getAvatar(name);
  const fetchTeams = () => api.fetchUserTeams().then((r) => r.teams);
  const createTeam = (name) => api.createTeam(name);
  const deleteTeam = (id) => api.deleteTeam(id);
  let all = Promise.all([fetchUser(), fetchTeams()]);
</script>
```

现在让我们写一些基本的标记：

``` html
<section>
    {#await all}
        <Loading />
    {:then [author, teams]}
        <section class="author">
            <Avatar src={getAvatar(author.name)} />
            <h3>{author.name}</h3>
        </section>
        <section>
            <h1>My Teams</h1>
            <ul>
                {#each teams as team}
                    <li>
                        <a href={`/team/${team.$id}`} use:link>{team.name}</a>
                        <button
                            on:click={async () => {
                                await deleteTeam(team["$id"]);
                                all = Promise.all([
                                    author,
                                    fetchTeams(),
                                ]);
                                console.log("Deleted team", team["$id"]);
                            }}>❌</button>
                    </li>
                {/each}
            </ul>
        </section>

        <section>
            <h1>Create Team</h1>
            <div>
                <label for="team" />
                <input
                    type="text"
                    name="team"
                    placeholder="Enter Team Name"
                    bind:value={name} />
                <button
                    on:click={async () => {
                        await createTeam(name);
                        all = Promise.all([author, fetchTeams()]);
                        console.log("team created");
                    }}>Create Team</button>
            </div>
        </section>
    {:catch error}
        {error}
        <p>
            Public profile not found
            <a href="/profile/create" use:link>Create Public Profile</a>
        </p>
    {/await}
</section>
```

上述标记执行以下操作。

* 显示用户所属的团队列表
* 定义删除团队的按钮。
* 定义用于创建新团队的按钮。

接下来，让我们创建一个页面来显示由上面标记中的 <a> 标记定义的每个团队的详细信息。