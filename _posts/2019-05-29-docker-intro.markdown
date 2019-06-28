---
layout: post
title: 够用就好的 Docker 教程
date: 2019-05-29 15:50:01.000000000 +09:00
---
## 核心概念

* Docker images （镜像） ：不可变的主模板，用于抽出完全相同的容器。镜像包含应用程序需要运行的Dockerfile、库和代码。

* Dockerfile ：一个文件，其中包含Docker应如何构建映像的说明。

* Docker Container （容器）：Docker 镜像加上命令 `docker run image_name` 将从镜像创建并启动容器。

* Container Registry ：存储 Docker 镜像的远程位置。将镜像推送到 Registry 并从 Registry 中提取镜像。 可以托管自己的 Registry 或使用提供商的 Registry，例如 AWS 和 Google Cloud 都有 Registry。

* Engine  ：client-server app ([CE](https://docs.docker.com/install/) or [Enterprise](https://www.docker.com/products/docker-enterprise))

* Client  ：处理 Docker CLI 来和 Daemon 进行交互

![Docker Client](http://www.caotouchan.tech/wp-content/uploads/2019/05/image-1559101276532.png)

* Daemon （守护进程）  ：侦听 Docker API 请求的 Docker 服务器，用来管理图像、容器、网络和卷。

* Volumes  （卷）：存储应用程序使用和创建的持久数据的最佳方式。

* Docker Hub  ： 默认的 Docker Registry

* Repository  ：具有相同名称和不同标记的 Docker 镜像的集合。 标签是镜像的标识符。

* Networking  ：连接容器。连接的 Docker 容器可以位于同一主机或多个主机上。 有关Docker网络的更多信息，请参阅 [此文章](https://www.oreilly.com/learning/what-is-docker-networking)。

![file](http://www.caotouchan.tech/wp-content/uploads/2019/05/image-1559101765535.png)

* Compose  ：方便运行需要多个Docker容器的应用程序的工具。 Docker Compose 允许您将命令移动到 `docker-compose.yml` 文件中以供重用。

* Swarm  ：一个协调容器部署的产品。 参考 [官方Docker教程](https://docs.docker.com/get-started/#recap-and-cheat-sheet) 。 推文作者建议不要花时间在Docker Swarm上，除非你有令人信服的理由这样做。

* Services  ：分布式应用程序的不同部分。Services 只运行一个映像，但它编码了映像的运行方式 ：端口、容器运行副本个数等等。Services 允许您跨多个 Docker Daemon 扩展容器，并使 Docker Swarms 成为可能。

* Kubernetes ：Kubernetes 自动化部署、扩展和管理容器化的应用程序。使用 Kubernetes 扩展具有多个 Docker 容器的项目。 （ Kubernetes 不是 Docker 的官方部分; 它更像是 Docker 的 BFF 。）

## Dockerfile 指令

Dockerfile 指引 Docker 构建将用于制作容器的镜像。

当调用 `docker build` 以创建镜像时，会假定 Dockerfile 位于当前工作目录中。 可以使用文件标志（`-f`）指定其他位置。

容器从一系列的 layer 中构建，**除了最后一层都是只读**。

Dockerfile 指示添加哪些层和添加的顺序。每层实际上只是一个包含自上一层以来的更改的文件。基础层（ The base image ） 包含提供初始层。

将映像从远程存储库提取到本地计算机时，仅下载本地计算机上尚未存在的层。

Dockerfile指令是一行的开头的大写单词，后跟其参数。

示例 Dockerfile ：

```
FROM python:3.7.2-alpine3.8
LABEL maintainer="jeffmshale@gmail.com"
ENV ADMIN="jeff"
RUN apk update && apk upgrade && apk add bash
COPY . ./app
ADD https://raw.githubusercontent.com/discdiver/pachy-vid/master/sample_vids/vid1.mp4 \
/my_app_directory
RUN ["mkdir", "/a_directory"]
CMD ["python", "./my_script.py"]
```

* `FROM` : 指定 base (parent) image.

```
FROM ubuntu:18.04
```

其中 `ubuntu` 是镜像 repository ， `18.04` 是标签（tag）。没有标签则拉取最新版本。

![file](http://www.caotouchan.tech/wp-content/uploads/2019/05/image-1559108982583.png)

创建容器时，可以在只读层的顶部添加可写层。当图像运行时，如果某个层需要由容器修改，则该文件将被复制到顶部可写层中。 了解有关 `copy-on-write` 的更多信息，点击 [链接](https://docs.docker.com/v17.09/engine/userguide/storagedriver/imagesandcontainers/)。

* `LABEL` : 提供元数据 （ metadata ）。更多信息点击 [链接](https://docs.docker.com/config/labels-custom-metadata/)。

* `ENV` : 设置持久化环境变量。

* `RUN` : 运行命令，创建镜像层。用于安装包到容器中。

* `COPY` : 拷贝文件和目录到容器中。目标目录不存在的话会自行创建。

* `ADD` : 将文件和目录复制到容器。

和 `COPY` 不同的地方：
(1) 可用于将文件从远程URL移动到容器 
(2) 可以 upack 本地 的 .tar 文件。

`ADD` 指令包含` \ ` 实现分解多行的长指令，使用它来提高可读性。

Docker文档不建议以这种方式使用远程URL，因为您无法删除文件。

* `CMD` : 给运行时容器提供命令和参数。

只有一个 `CMD`。

`CMD` 可以包含可执行文件。 如果 `CMD` 不存在可执行文件，则必须存在 `ENTRYPOINT` 指令。 在这种情况下，`CMD` 和 `ENTRYPOINT` 指令都应采用JSON格式。

docker 的命令行参数运行**覆盖** Dockerfile 中提供给 `CMD` 的参数。

以下是另一个示例 Dockerfile：

```
FROM python:3.7.2-alpine3.8
LABEL maintainer="jeffmshale@gmail.com"
## Install dependencies
RUN apk add --update git
## Set current working directory
WORKDIR /usr/src/my_app_directory
## Copy code from your local context to the image working directory
COPY . .
## Set default value for a variable
ARG my_var=my_default_value
## Set code to run at container run time
ENTRYPOINT ["python", "./app/my_script.py", "my_var"]
## Expose our port to the world
EXPOSE 8000
## Create a volume for data storage
VOLUME /my_volume
```

有几种方法可以使用 `RUN` 安装软件包：

1. 使用`apk`在 Alpine Docker 映像中安装软件包。 具有基本 Ubuntu 映像的 Dockerfile 中的包可以像这样更新和安装：`RUN apt-get update && apt-get install my_package`。
2. 通过`pip`，`wheel`和`conda`安装 Python 包。其他语言可以使用各种安装程序。（请确保在尝试使用包管理器之前安装包管理器）请使用 ` \ ` 拆分多行。
3. 在文件中列出包需求，通常将文件命名为 **requirements.txt**。下文将分享一个推荐的模式使用 requirements.txt 来利用构建时缓存。

* `WORKDIR` : 设置后续指令的工作目录。

1. 最好使用 `WORKDIR` 设置绝对路径，而不是使用Dockerfile中的cd命令浏览文件系统。

2. 如果目录不存在， `WORKDIR` 会自动创建目录。

3. 可以使用多个 `WORKDIR` 指令。 如果提供了相对路径，则每个 `WORKDIR` 指令都会更改当前工作目录。

* `ARG` : 定义一个在构建时传递给 Docker 的变量。

与 `ENV` 变量不同，`ARG` 变量不可用于运行容器。 但是，在构建映像时，可以使用 `ARG` 值从命令行设置 `ENV` 变量的默认值。 然后，`ENV` 变量在容器运行时间内持续存在。 在 [此处](https://vsupalov.com/docker-build-time-env-values/) 详细了解此技术。

* `ENTRYPOINT` : 在容器启动时提供默认命令和参数。

类似于`CMD`，但如果使用命令行参数运行容器，则不会覆盖`ENTRYPOINT`参数。相反，**传递给 `docker run my_image_name` 的命令行参数会附加到 `ENTRYPOINT` 指令的参数中**。例如，`docker run my_image bash` 将参数`bash` 添加到`ENTRYPOINT` 指令的现有参数的末尾。

Dockerfile至少应有一条`CMD`或`ENTRYPOINT`指令。`CMD` 和`ENTRYPOINT`之间进行选择的建议：

1. 每次需要运行相同的命令时，请关注`ENTRYPOINT`。
2. 当容器将用作可执行程序时，请关注`ENTRYPOINT`。
3. 当您需要提供可以从命令行覆盖的额外默认参数时，请支持`CMD`。

* `EXPOSE` : 显示要发布的端口，以提供对正在运行的容器的访问。

`EXPOSE` 实际上并不发布端口。 相反，它充当构建镜像的人和运行容器的人之间的文档。

使用带有`-p` 标志的 `docker run` 可以在运行时发布和映射一个或多个端口。 大写的 `-P` 标志将发布所有暴露的端口。

* `VOLUME` : 创建目录挂载点以访问和存储持久化数据。

另外，关于 Dockerfile 的小抄，请点击 [此链接](https://kapeli.com/cheat_sheets/Dockerfile.docset/Contents/Resources/Documents/index)。

## 压缩 Docker 镜像

### 缓存

在检查每条指令时，Docker在其缓存中查找现有的中间镜像，它可以重复使用而不是创建新的（重复的）中间镜像。

大多数新指令只是与中间镜像中的指令进行比较。如果匹配，则使用缓存副本。

一些实用缓存的建议：

* 通过使用 `docker build` 传递 ` --no-cache = True` 可以关闭缓存。
* 将可能更改的指令放在Dockerfile中尽可能低的位置。
* 链式的 `RUN apt-get update` 和 `apt-get install` 命令可避免缓存缺失问题。
* 如果使用包装安装程序（如`pip`和 requirements.txt 文件），请按照下面的模型进行操作，以确保没有收到带有 requirements.txt 中列出的旧包的旧版本中间镜像。

```
COPY requirements.txt /tmp/
RUN pip install -r /tmp/requirements.txt
COPY . /tmp/
```

### Size Reduction

Alpine基础映像是一个完整的Linux发行版，没有太多其他内容。 下载通常不到5mb，但它要求您花更多的时间编写构建工作应用程序所需的依赖项的代码。

如果你的容器中需要Python，那么Python Alpine构建是一个很好的折衷方案。

### 多阶段构建

多级构建使用多个 `FROM` 指令，每个 `FROM` 指令：

1. begins a new stage of the build.
2. leaves behind any state created in prior stages.
3. can use a different base.

多阶段构建示例：

```
FROM golang:1.7.3 AS build
WORKDIR /go/src/github.com/alexellis/href-counter/
RUN go get -d -v golang.org/x/net/html  
COPY app.go .
RUN CGO_ENABLED=0 GOOS=linux go build -a -installsuffix cgo -o app .

FROM alpine:latest  
RUN apk --no-cache add ca-certificates
WORKDIR /root/
COPY --from=build /go/src/github.com/alexellis/href-counter/app .
CMD ["./app"]
```

通过在 `FROM` 指令中附加名称来命名第一个阶段。 然后在 Dockerfile 中的 `COPY --from =` 指令中引用指定的阶段。

有时多级构建会增加更多复杂性，使图像难以维护，因此您可能不会在大多数构建中使用它们。 请参阅进一步讨论：[权衡](https://blog.realkinetic.com/building-minimal-docker-containers-for-python-applications-37d0272c52f3) 和 [高阶模式](https://medium.com/@tonistiigi/advanced-multi-stage-build-patterns-6f741b852fae) 。

### .dockerignore

.dockerignore类似于.gitignore，其中包含Docker与文件名匹配的模式列表，并在制作镜像时排除。

使用 Go 的 [filepath.Match 规则](https://golang.org/pkg/path/filepath/#Match)  和一些 [Docker 自身的规则](https://docs.docker.com/v17.09/engine/reference/builder/#dockerignore-file)。

### Size Inspection

* 查看正在运行的容器大小： `docker container ls -s`

* 显示镜像的大小： `docker image ls`

* 查看中间镜像的大小：`docker image history my_image：my_tag`

* 运行 `docker image inspect my_image`：tag将显示有关镜像的许多内容，包括每个 layer 的大小。layer 和中间镜像的复杂性，查看Nigel Brown的 [这篇文章](https://windsock.io/explaining-docker-image-ids/)。

* 安装和使用 [dive](https://github.com/wagoodman/dive) 可以轻松查看 layer 的内容。

### 几个实用的建议

1. 尽可能使用官方的 base image。
2. 尽可能使用 Alpine 镜像的变体来保持镜像的量级。
3. 如果使用 `apt`，在同一指令中将 `RUN apt-get update`与 `apt-get install` 结合使用（减少了构建的层数），如：
```
RUN apt-get update && apt-get install -y \
    package-one \
    package-two 
 && rm -rf /var/lib/apt/lists/*
```

4. 在 `RUN` 指令的末尾包含 `&& rm -rf / var / lib / apt / lists / *` 以清理 `apt` 缓存，使其不存储在层中。

5. 通过在 Dockerfile 中放置可能更改的指令，明智地使用缓存。

6. 使用 .dockerignore 文件。

7. 使用 [dive](https://github.com/wagoodman/dive) 检查 Docker 镜像层方便压缩规模。

8. 不要安装不需要的包。（- ，- b）


## 常用命令

### Containers
Use `docker container my_command`

`create` — Create a container from an image. 

`start `— Start an existing container. 

`run `— Create a new container and start it. 

`ls` — List running containers. 

`inspect `— See lots of info about a container.

`logs `— Print logs. 

`stop` — Gracefully stop running container. 

`kill` —Stop main process in container abruptly. 

`rm`— Delete a stopped container.

### Images
Use `docker image my_command`

`build `— Build an image.

`push `— Push an image to a remote registry.

`ls `— List images. 

`history` — See intermediate image info.

`inspect `— See lots of info about an image, including the layers. 

`rm `— Delete an image.

### Misc

`docker version` — List info about your Docker Client and Server versions.

`docker login `— Log in to a Docker registry.

`docker system prune `— Delete all unused containers, unused networks, and dangling images.

## 卷（Volume）：Docker 中的数据

### 临时数据

通过两种方式临时保存文件：

1. 默认情况下，容器内应用程序创建的文件存储在容器的可写层中。当容器不再存在时，数据也将消失。

2. 保存临时数据同时具有更好的性能：`tmpfs mount` 使用主机内存进行临时挂载，具有更快的读写操作的优点。

### 持久化数据

两种持久化数据的方式：

1. 将文件系统挂载到容器。使用绑定装载，Docker外部的进程也可以修改数据。绑定挂载很难备份，迁移或与其他Container共享。

![file](http://www.caotouchan.tech/wp-content/uploads/2019/05/image-1559117952332.png)

2. 使用卷（Volume）保存数据是更好的方法。

### 卷

卷是一个文件系统，它位于任何容器之外的主机上。 卷由Docker创建和管理。 卷的特性：

* 持久化

* 自由浮动的文件系统，与任何一个容器分开

* 与其他容器共用

* 高效的输入和输出

* 能够托管在远程云提供商上

* 可加密

* 可命名

* 能够通过容器预先填充其内容

* 方便测试

### 创建卷

通过 Dockerfile 或 API 请求创建卷。

`VOLUME / my_volume`

使用Dockerfile创建卷，则仍需要在运行时声明卷的安装点。

还可以使用JSON数组格式在 Dockerfile 中创建卷。

卷也可以在运行时从命令行实例化。

### Volume CLI 命令

常用的命令：

*  `docker volume create —-name my_volume` 

*  `docker volume ls` 

*  `docker volume inspect my_volume` 

*  `docker volume rm my_volume` 

*  `docker volume prune` 

docker run中的--mount标志的常用选项 -  mount my_options my_image：

*  `type=volume` 

*  `source=volume_name` 

*  `destination=/path/in/container` 

*  `readonly` 

## 参考资料

[1]. [Learn Enough Docker to be Useful](https://medium.com/search?q=Learn%20Enough%20Docker%20to%20be%20Useful)