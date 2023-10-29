# Docker基本命令


<!--more-->

# Docker 基本命令

## 1. 使用镜像

```bash
docker pull [选项] [Docker Registry 地址[:端口号]/]仓库名[:标签]
```

- Docker 镜像仓库地址：地址的格式一般是 `<域名/IP>[:端口号]`。默认地址是 Docker Hub(`docker.io`)。

- 仓库名：如之前所说，这里的仓库名是两段式名称，即 `<用户名>/<软件名>`。对于 Docker Hub，如果不给出用户名，则默认为 `library`，也就是官方镜像。



## 2. 操作容器

`-it`：这是两个参数，一个是 `-i`：交互式操作，一个是 `-t` 终端。我们这里打算进入 `bash` 执行一些命令并查看返回结果，因此我们需要交互式终端。

`-d`: 让容器在后台运行

`--rm`：这个参数是说容器退出后随之将其删除。默认情况下，为了排障需求，退出的容器并不会立即删除，除非手动 `docker rm`。我们这里只是随便执行个命令，看看结果，不需要排障和保留结果，因此使用 `--rm` 可以避免浪费空间。

`ubuntu:18.04`：这是指用 `ubuntu:18.04` 镜像为基础来启动容器。

`bash`：放在镜像名后的是 **命令**，这里我们希望有个交互式 Shell，因此用的是 `bash`, 也可以使用dash或者fish。



## 3. 数据管理

**创建一个数据卷**

```bash
docker volume create my-vol
```

**删除数据卷**

```bash
docker volume rm my-vol
```

无主的数据卷可能会占据很多空间，要清理请使用以下命令:

```bash
docker volume prune
```

**查看所有数据卷**

```bash
docker volume ls
```

**在主机里使用以下命令可以查看指定 `数据卷` 的信息**

```bash
docker volume inspect my-vol
```

**查看容器详细信息**

```bash
docker inspect container_id
```

**挂载主机目录**

```bash
docker run -d -P \
    --name web \
    # -v /src/webapp:/usr/share/nginx/html \
    --mount type=bind,source=/src/webapp,target=/usr/share/nginx/html \
    nginx:alpine
```

>  注意: 本地目录的路径必须是绝对路径,使用 `-v` 参数时如果本地目录不存在 Docker 会自动为你创建一个文件夹，现在使用 `--mount` 参数时如果本地目录不存在，Docker 会报错。

**Docker 挂载主机目录的默认权限是 `读写`，用户也可以通过增加 `readonly` 指定为 `只读`**

```bash
docker run -d -P \
    --name web \
    # -v /src/webapp:/usr/share/nginx/html:ro \
    --mount type=bind,source=/src/webapp,target=/usr/share/nginx/html,readonly \
    nginx:alpine
```


---

> 作者: map[avatar:<nil> email:<nil> link:<nil> name:<nil>]  
> URL: https://blog-12x.pages.dev/docker%E5%9F%BA%E6%9C%AC%E5%91%BD%E4%BB%A4/  

