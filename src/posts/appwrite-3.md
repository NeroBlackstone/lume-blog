---
title: Appwrite学习笔记(四)
description: ''
date: 2021-07-31
img: https://res.cloudinary.com/neroblackstone/image/upload/v1624671830/appwrite_i2voda.webp
tags:
- Appwrite

---
## 生产环境的Appwrite

既然我们已经介绍了 Appwrite 的许多功能，我们应该讨论在您的应用最终为用户准备好后在生产中运行 Appwrite。

首先，良好的安全性是一个不断变化的目标。 Appwrite 提供了一套 API，可以抽象出应用程序的许多安全要求，但在线托管软件意味着将计算机暴露在互联网上。虽然我们无法涵盖所有内容，但让我们讨论在生产中运行 Appwrite 时的一些安全最佳实践。

### 服务器

在我们讨论在生产中运行 Appwrite 的步骤之前，我们需要讨论一下 Appwrite 将在哪个系统上运行。这些技巧假设您在基于 Linux 的服务器上运行 Appwrite，但这些原则适用于任何操作系统。

#### 更新

大多数安全漏洞发生在运行过时、不安全软件版本的系统上。这个问题是可以理解的 - 很难跟上系统更新。但是，按 [cron](https://man7.org/linux/man-pages/man5/crontab.5.html) 计划运行更新也不是最好的，因为最好立即安装安全更新。使用 Ubuntu 的[无人值守升级](https://help.ubuntu.com/community/AutomaticSecurityUpdates)和 Fedora 的 [dnf-automatic](https://fedoraproject.org/wiki/AutoUpdates) 软件包等工具来运行软件的最新更新。

#### 防火墙和 SSH

安全最佳实践是默认拒绝安全策略 - 我们应该只提供对我们想要的服务的显式访问。Appwrite 在其默认配置中考虑了这一点：唯一暴露给外界的服务是我们需要的 [Traefik](https://traefik.io/traefik/) 代理。因此，如果 Appwrite 是我们想要在服务器上公开公开的唯一服务，我们可以使用防火墙工具来阻止对任何其他未使用端口的访问。

如果您使用 SSH 来管理您的系统，请不要忘记在您的防火墙中保持打开状态！ SSH 被认为是一种私有服务，这意味着它应该可以公开访问，但仅限于授权帐户。最佳实践是使用诸如 SSH 密钥之类的加密工具而不是密码，因为它们更难以伪造。

#### 更多阅读

以下是一些其他资源，可以更详细地了解最佳实践：

* [Docker 安全](https://docs.docker.com/engine/security/)
* [DigitalOcean 推荐的安全措施](https://www.digitalocean.com/community/tutorials/recommended-security-measures-to-protect-your-servers)

### 保护Appwrite

现在，让我们讨论为生产设置 Appwrite。

#### 环境变量

您可以使用它提供的许多环境变量轻松地为生产配置 Appwrite。为生产部署时，应在 `appwrite` 安装目录中隐藏的 `.env` 文件中设置以下变量：

* `_APP_ENV`：更改为`production`。
* `_APP_OPTIONS_ABUSE`：为 API 启用滥用检查和速率限制。设置为`enabled`。
* `_APP_OPTIONS_FORCE_HTTPS`：强制连接使用 HTTPS 进行安全数据传输。设置为`enabled`
* `_APP_OPENSSL_KEY_V1`：这是用于加密会话和密码等秘密的秘密。将此更改为安全和随机的内容，并**确保其安全并进行备份**。
* `_APP_DOMAIN`：将此设置为[您的域名](https://appwrite.io/docs/custom-domains)，以便 Appwrite 自动生成 SSL 证书。

#### 限制控制台访问

三个环境变量可用于限制对 Appwrite 控制台的访问：

* `_APP_CONSOLE_WHITELIST_EMAILS`
* `_APP_CONSOLE_WHITELIST_IPS`
* `_APP_CONSOLE_WHITELIST_ROOT`

如果您只希望单个帐户可以访问控制台，请将 `_ROOT` 变量设置为`enabled`。对于多个用户，您可以使用各自的环境变量限制对特定电子邮件和 IP 地址的访问。

#### 杀毒软件

对于生产，您可以为任何已知的恶意对象启用对上传文件的 `clamav` 扫描。将 `_APP_STORAGE_ANTIVIRUS` 设置为`enabled`并[取消注释](https://github.com/appwrite/appwrite/blob/master/docker-compose.yml#L417-L423) `docker-compose.yml` 中的服务以使用此功能。不要忘记在主 `appwrite` 服务的[depends_on 部分](https://github.com/appwrite/appwrite/blob/master/docker-compose.yml#L74)取消注释 `clamav`。

#### Functions

可以自定义 Cloud Functions 以满足您的生产系统的需求，主要用于控制可用于 Function 执行的资源：

* `_APP_FUNCTIONS_CPUS`: Cloud Functions 可以使用的最大 CPU 内核数。
* `_APP_FUNCTIONS_MEMORY`: Cloud Functions 可用的最大内存（以兆字节为单位）。
* `_APP_FUNCTIONS_CONTAINERS`: Appwrite 保持活动的最大容器数，默认为 10。增加此数字以增加热函数的数量。
* `_APP_FUNCTIONS_RUNTIMES`: 新 Cloud Functions 的可用运行时列表。

> 所有 Appwrite 环境变量都可以在我们的[文档](https://appwrite.io/docs/environment-variables)中找到。

## Docker Swarm 集成

如您所知，Appwrite 被设计为一组无状态微服务，可扩展性是我们的首要任务之一！虽然有很多方法可以通过大量编排服务实现可扩展性，但我们将看一看最直观的方法之一。今天，我们将讨论使用 Docker Swarm 水平扩展 Appwrite。

### 什么是 Docker Swarm？

Docker Swarm 是一种内置于 Docker CLI 的容器编排工具，它允许我们将 Docker 服务部署到主机集群，而不仅仅是 [Docker Compose](https://docs.docker.com/compose/) 允许的主机集群。这称为 Swarm 模式，不要与[不再作为独立产品开发](https://github.com/docker/classicswarm)的经典 Docker Swarm 混淆。Docker Swarm 与 A​​ppwrite 配合得很好，因为它建立在[ Compose 规范](https://docs.docker.com/compose/compose-file/)之上，这意味着我们可以使用 Appwrite 的 `docker-compose` 配置来部署到一个 swarm（在这里和那里做一些更改）。它的简单性使我们可以立即开始！

### 使用 Swarm 部署 Appwrite

#### 先决条件

对于此示例，我们将需要以下内容：

* [Docker 安装](https://docs.docker.com/get-docker/)在您的每台主机上。
* 您的主机之间必须打开以下端口：
  * 用于集群管理通信的 TCP 端口 2377
  * 用于节点间通信的 TCP 和 UDP 端口 7946
  * UDP 端口 4789 用于覆盖网络流量
* “领导者”服务器有 Appwrite 的 [Compose 文件](https://gist.github.com/eldadfux/977869ff6bdd7312adfd4e629ee15cc5)。

#### 创建 Swarm

我们将在我们想要成为“领导者”的任何主机上创建集群。使用以下命令初始化群：

``` shell
$ docker swarm init
Swarm initialized: current node (7db8w7aurb7qrhvm0c0ttd4ky) is now a manager.

To add a worker to this swarm, run the following command:

    docker swarm join --token SWMTKN-1-0wagrl3qt4loflf9jcadj8gx53fj2dzmbwaato7r50vghmgiwp-cvo3jflyfh2gnu46pzjtaexv2 your.ip.addr.ess:2377

To add a manager to this swarm, run 'docker swarm join-token manager' and follow the instructions.
```

现在，让我们在其他系统上运行提供的命令 - 我们正在寻找消息 `This node join a swarm as a worker`。完成后，我们可以返回“领导者”主机并可以看到两个系统：

``` shell
$ docker node ls
ID                            HOSTNAME          STATUS    AVAILABILITY   MANAGER STATUS   ENGINE VERSION
yfl7xsy5birfbpiw040chef67     appwrite          Ready     Active                          20.10.6
op3nf4ab6f5v1lulwkpyy2a83 *   appwrite_leader   Ready     Active         Leader 
```

#### 更新 `docker-compose.yml`

现在 swarm 已准备就绪，我们需要对 `docker-compose.yml` 进行一些更改以使其与 Swarm 兼容。

默认情况下，Docker swarm 中的卷不在主机之间共享，因此我们将使用 NFS 在主机之间共享目录。共享数据可以通过多种方式实现，但这是最简单的开始方式。为此，我们将用 NFS 挂载替换所有命名卷。 DigitalOcean 有一个[很好的 NFS 配置指南](https://www.digitalocean.com/community/tutorials/how-to-set-up-an-nfs-mount-on-ubuntu-20-04)，因此请参阅该指南以获取更多详细信息。

我们将在我们的“领导者”主机上配置这些 NFS 卷，并与群中的其他主机共享这些文件夹。我们将使用以下目录来替换 Docker 卷并通过 NFS 共享：

``` shell
mkdir -p /nfs/{mariadb,redis,cache,uploads,certificates,functions,influxdb,config}
```

接下来，我们将在第二台主机上创建相应的 `/nfs` 目录（使用与上面相同的命令），我们将从“领导者”主机挂载 NFS 共享。

现在，将 `docker-compose.yml` 中的每个命名卷替换为其对应的 NFS 目录：

``` shell
# - appwrite-uploads:/storage/uploads:rw
- /nfs/uploads:/storage/uploads:rw

# - appwrite-certificates:/storage/certificates:rw
- /nfs/certificates:/storage/certificates:rw
```

然后，我们需要从 `docker-compose.yml` 中删除 `depends_on` 和 `container_name` 节，因为 Docker Swarm 不支持它们。

#### 覆盖网络

Docker 使用[覆盖网络](https://docs.docker.com/network/overlay/)将 swarm 中的每个节点连接在一起，因此无论部署在何处，容器都可以相互通信。我们可以使用 Docker CLI 创建覆盖网络，但让我们在 `docker-compose.yml` 中捕获此更改：

``` shell
networks:
  gateway:
  appwrite:
    driver: overlay
```

#### 准备部署

一切就绪后，我们将设置 [Appwrite 环境变量](https://appwrite.io/docs/environment-variables)并使用以下命令部署到 swarm：

``` shell
docker stack deploy -c <(docker-compose config) appwrite
```

如果您看到 `docker-compose config`警告，请尝试将 `docker-compose.yml` 开头的 Compose 版本升级到 `version: '3.8'` 以利用最新的 Compose 规范。

我们的微服务工作者依赖 Redis 来处理发布/订阅，因此您可能会看到它们重新启动，直到堆栈自我修复。部署完所有内容后，您可以使用以下命令检查服务的状态：

``` shell
$ docker service ls
ID             NAME                                    MODE         REPLICAS   IMAGE                     PORTS
ktfto6dap451   appwrite_appwrite                       replicated   1/1        appwrite/appwrite:0.8.0   
hazw2csk4epd   appwrite_appwrite-maintenance           replicated   1/1        appwrite/appwrite:0.8.0   
fshro0zn8iw6   appwrite_appwrite-schedule              replicated   1/1        appwrite/appwrite:0.8.0   
jep5n0gnmvy6   appwrite_appwrite-worker-audits         replicated   1/1        appwrite/appwrite:0.8.0   
oiftp636aq6v   appwrite_appwrite-worker-certificates   replicated   1/1        appwrite/appwrite:0.8.0   
tlu7yxvtrr0r   appwrite_appwrite-worker-deletes        replicated   1/1        appwrite/appwrite:0.8.0   
rda2kspenbzr   appwrite_appwrite-worker-functions      replicated   1/1        appwrite/appwrite:0.8.0   
im800v9tct4n   appwrite_appwrite-worker-mails          replicated   1/1        appwrite/appwrite:0.8.0   
ry0u3v726o8h   appwrite_appwrite-worker-tasks          replicated   1/1        appwrite/appwrite:0.8.0   
734y2mr6gzkc   appwrite_appwrite-worker-usage          replicated   1/1        appwrite/appwrite:0.8.0   
bkotuk5kwmxx   appwrite_appwrite-worker-webhooks       replicated   1/1        appwrite/appwrite:0.8.0   
ff6iicbmf5my   appwrite_influxdb                       replicated   1/1        appwrite/influxdb:1.0.0   
892923vq96on   appwrite_mariadb                        replicated   1/1        appwrite/mariadb:1.2.0    
uw3l8bkoc3sl   appwrite_redis                          replicated   1/1        redis:6.0-alpine3.12      
ulp1cy06plnv   appwrite_telegraf                       replicated   1/1        appwrite/telegraf:1.0.0   
9aswnz3qq693   appwrite_traefik                        replicated   1/1        traefik:2.3               *:80->80/tcp, *:443->443/tcp
```

> 我已将我完成的 Compose 文件包含在 GitHub gist中以供参考。

### 配置

Docker Swarm 有很多可用的配置选项，所以我们不会在这里涵盖所有内容。相反，让我们讨论一些在配置部署时最有用的节。

#### Replicas

由于 Appwrite 基本上是无状态的，因此您可以根据应用程序的需要单独扩展或缩减每个服务。例如，我们可能希望有两个 Functions workers，以便我们可以处理两倍的函数执行：

``` shell
deploy:
  replicas: 1
```

我们可以通过过滤特定服务来检查副本是否已部署：

``` shell
$ docker service ls --filter name=appwrite_appwrite-worker-functions 
ID             NAME                                 MODE         REPLICAS   IMAGE                     PORTS 
rda2kspenbzr   appwrite_appwrite-worker-functions   replicated   2/2        appwrite/appwrite:0.8.0
```

#### 节点约束

Docker Swarm 允许我们使用放置约束来控制容器在集群中的部署位置。例如，我们可以将 Traefik 或 MariaDB 配置为仅驻留在管理器节点上，并将以下内容添加到 `docker-compose.yml`：

``` shell
deploy:
  placement:
    constraints: [node.role == manager]
```

### 下一步是什么

我们只是覆盖了冰山一角。有关在 Docker Swarm 中运行 Appwrite 的进一步阅读：

* Docker 的[管理指南](https://docs.docker.com/engine/swarm/admin_guide/)有很多关于如何在集群中管理节点的额外信息，以及一些生产环境的注意事项。
* [Docker secrets](https://docs.docker.com/engine/swarm/secrets/)和[ Docker 配置](https://docs.docker.com/engine/swarm/configs/)可用于通过 swarm 更轻松地控制和分发敏感数据。

## Grafana 集成

我们喜欢 Dashboards，我们认为将 Grafana 支持添加到 Appwrite 会很棒。

Appwrite 没有随 Grafana 一起提供，原因有几个。首先，您的堆栈中可能已经拥有一套自己的监控工具，我们认为我们的堆栈应该是没有意见的，并允许您使用您觉得舒服的工具。第二个原因是我们尝试使用最少的组件来发布 Appwrite 设置，以使 Appwrite 易于启动，但仍然足够灵活以进行增长。

### 将 Grafana 添加到 Appwrite

我们将创建两个仪表板：一个用于监控 Appwrite 的使用统计信息，另一个用于监控您的服务器统计信息。

第一步是将 Grafana 服务添加到 Appwrite 的 [docker-compose.yml 文件](https://github.com/appwrite/appwrite/blob/master/docker-compose.yml#L476)中。

``` yml
grafana:
    image: grafana/grafana
    container_name: appwrite-grafana
    ports:
      - "3000:3000"
    networks:
      - appwrite
    volumes:
      - appwrite-grafana:/var/lib/grafana
```

接下来，您需要将 `appwrite-grafana` 卷[添加到所有卷](https://github.com/appwrite/appwrite/blob/master/docker-compose.yml#L532)的列表中。这将允许您的 Grafana 容器保留数据。

``` yml
volumes:
  appwrite-mariadb:
  appwrite-redis:
  appwrite-cache:
  appwrite-uploads:
  appwrite-certificates:
  appwrite-functions:
  appwrite-influxdb:
  appwrite-config:
  appwrite-grafana:
```

现在运行

``` shell
docker-compose up -d
```

### 仪表板 #1 - Appwrite 指标

对于您的第一个仪表板，我们不需要在我们的服务中进行任何额外配置。现在转到 `http://localhost:3000` 来配置 Grafana。您可以使用默认凭据登录

``` shell
username : admin
password : admin
```

系统将提示您输入新密码，强烈建议您更改密码。在[官方指南](https://grafana.com/docs/grafana/latest/manage-users/user-admin/change-your-password/)中了解有关管理用户和密码的更多信息。  
第一步是配置您的数据源。在我们的例子中，我们将使用 InfluxDB 插件。

![](https://res.cloudinary.com/neroblackstone/image/upload/v1627905558/appwrite_influxDB_z1vu7d.png)

添加 InfluxDB 数据源后，就可以对其进行配置了。在这里，您需要填写2个字段的值，

* **URL** - [http://influxdb:8086](http://influxdb:8086/)
* **Database** - telegraf

最后单击“**保存并测试**”以检查您的数据库连接。如果一切顺利，您将看到一条成功消息。

![](https://res.cloudinary.com/neroblackstone/image/upload/v1627905642/appwrite_influxDB2_w5bojl.png)

就是这样！您现在应该会看到这个精美的仪表板

![](https://res.cloudinary.com/neroblackstone/image/upload/v1627905735/appwrite_dashboard_qabiau.png)

![](https://res.cloudinary.com/neroblackstone/image/upload/v1627905748/appwrite_dashboard2_tkannn.png)

### 仪表板 #2 - 服务器指标

[下一个仪表板](https://grafana.com/grafana/dashboards/5955)将监控我们的服务器指标。这包括 CPU 使用率、磁盘 I/O、网络 I/O 等等。这个仪表板需要一些额外的信息，所以我们需要在我们的 `Telegraf` Docker 镜像中进行一些更改以提供这些信息。

### 我们将首先克隆 Appwrite 的telegraf镜像

``` shell
git clone https://github.com/appwrite/docker-telegraf.git 
cd docker-telegraf
```

我们需要为收集器提供更多指标。将以下行添加到[第 83 行](https://github.com/appwrite/docker-telegraf/blob/master/telegraf.conf#L83)

``` conf
[[inputs.cpu]]
    percpu = true
    totalcpu = true
    collect_cpu_time = false
    report_active = false
[[inputs.disk]]
    ignore_fs = ["tmpfs", "devtmpfs", "devfs"]
[[inputs.io]]
[[inputs.mem]]
[[inputs.net]]
[[inputs.system]]
[[inputs.swap]]
[[inputs.netstat]]
[[inputs.processes]]
[[inputs.kernel]]
```