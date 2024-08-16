---
title: Docker 踩坑记录
date: 2024-06-11 11:23:36
abstract: "记录使用 Docker 过程中遇到的一些问题"
tags:
    - Docker
categories:
    - Blog
---

在使用 Docker 安装了最新的 CentOS 镜像后，发现使用 yum 安装包的时候会报错：

```text
Error: Failed to download metadata for repo 'appstream': Cannot prepare internal mirrorlist: No URLs in mirrorlist
```

参考了 [stackoverflow 上的解决方案](https://stackoverflow.com/questions/70963985/error-failed-to-download-metadata-for-repo-appstream-cannot-prepare-internal)，解决了这个问题。之前有关注到由于 CentOS 停止更新维护，所以 CentOS 8 的一些内容已经不再支持了，所以这里使用这个方案来暂时解决问题，后续实际生产环境不应当考虑这样的环境。

## 多阶段构建时 ARG 参数不生效

这是因为 Dockerfile 按序构建时，在前面阶段指定的 ARG，会在后续阶段失效，而失效的表现和 shell 中环境变量未定义的表现是类似的——都是一个空字符串，如果不留意这一块，就会出现一些出乎意料的表现。

在我的构建脚本中，我使用了两个阶段来进行构建。第一个阶段，通过一些缓存的文件（比如使用配置文件生成的相应加载的 Go 源码、使用第三方工具生成的缓存源码以及 go.mod 指定的 go 的包下载）来构建我的目标二进制文件，因为不是构建所有的二进制，所以需要一个名为 TARGET 的构建参数来指定要构建的目标；第二个阶段，将第一个阶段构建出的二进制可执行文件拷贝到部署环境，并将其整体制成一个部署镜像。

```dockerfile
# 第一阶段：构建完整的服务进程二进制文件
FROM mirrors.tencent.com/uasvr-go/build-cache:latest as builder

ARG TARGET

WORKDIR /data/home/user00
RUN mkdir -p build && chown -R user00:user00 build

COPY --chown=user00:user00 . build/

RUN mv build-cache/trpc/ build/ && \
    mv build-cache/internal/dbproxysvr/dbtable/ua_db_table build/internal/dbproxysvr/dbtable/ && \
    mkdir -p build/configs && mv build-cache/configs/excelconf build/configs/excelconf

RUN cd build/cmd/${TARGET} && go build

# 第二阶段：构建最终的服务部署镜像
FROM mirrors.tencent.com/uasvr-go/uasvr-go-init:1.0-v20240715

ARG TARGET

USER user00
WORKDIR /data/home/user00

RUN mkdir -p uasvr-go && chown -R user00:user00 uasvr-go
COPY --from=builder --chown=user00:user00 /data/home/user00/build/cmd/${TARGET}/${TARGET} uasvr-go/cmd/${TARGET}
COPY --from=builder --chown=user00:user00 /data/home/user00/build/cmd/server-boot.sh uasvr-go/server-boot.sh

ENV TRPC_LOG_TRACE=1
```

其中，如果二阶段不指定 TARGET 参数，那么下方执行拷贝的结果将会是整个 cmd 目录都拷贝到对应的 cmd 目录。

此外，如果不想为 ARG 提供一个默认参数，就按照上述直接 `ARG TARGET` 这样的形式来设置就好。如果想要设置一个默认参数值，可以这样来定义——`ARG TARGET="default param val"`。

## 使用 Dockerfile 构建镜像时，发现存在很多 Exited 的容器

这是因为 Dockerfile 中每条构建的命令，实际上都是启动了一个临时的容器，而这些构建命令执行失败时，就会残留相应的临时容器，也可以看到相应的容器的命令，都是构建的命令。此时只需要将这些“悬垂”的容器清除掉就好，docker 为这些悬垂的资源提供了两个实用的命令：

- 清理“悬垂”容器

```bash
docker container prune
```

- 清理“悬垂”镜像

```bash
docker image prune
```

## 使用多阶段构建时，需要注意路径问题

同样，在上面的例子中，在最后的构建部署镜像时，执行命令

```dockerfile
COPY --from=builder --chown=user00:user00 /data/home/user00/build/cmd/${TARGET}/${TARGET} uasvr-go/cmd/${TARGET}
COPY --from=builder --chown=user00:user00 /data/home/user00/build/cmd/server-boot.sh uasvr-go/server-boot.sh
```

如果源忽略了 /data/home/user00，就会出现报错——目标地址找不到，所以在使用多阶段构建时，务必需要注意这一点，确保执行命令的路径符合预期。

## .dockerignore 的一些说明

在使用 docker 构建镜像时，通过命令 `docker build -t target:version -f /path/to/Dockerfile docker-context/` 来构建镜像，其中 docker-context 中的内容均会被发送给 docker 守护进程用以作为源文件构建，为了避免 docker-context 中过多的干扰文件发送给守护进程导致文件过多、冗余等等，可以通过在 .dockerignore 中指定对应的文件来避免相关文件被发送给守护进程，正确理解这一步发送文件才能搞清楚 .dockerignore 的使用方式及生效场景。

值得一提的是，如果在 .dockerignore 中指定了 Dockerfile 所在的路径，比如 `/build/image.dockerfile`，那么通过 -f 选项依然可以指定 `image.dockerfile` 作为镜像构建文件，但 `/build` 目录即使被 .dockerignore 指定也依然会被发送给守护进程。因此在构建时，实际是存在这个 build 目录的。

此外，如果 docker-context 中包含了诸如软链接的目录或文件，构建时会报错 `Forbidden path outside the build context`。docker 构建上下文是在运行 docker build 命令时指定的目录，docker 只能访问这个目录及其子目录中的文件。

## Error response from daemon: No build stage in current context

这往往是因为 Dockerfile 第一行没有指定 FROM。一个 docker 镜像的构建，必须基于 FROM 来构建，不然就会出现这个报错。

## ENV 使用时的细节问题

ENV 用于在 Dockerfile 中指定环境变量，它与上面的 `ARG` 的显著差异是 ENV 本身会被保存到容器运行时而 ARG 不会。举个例子，在基础镜像中如果定义了 `ENV USERHOME="/data/home/user00"`，那么对于后续的基于此基础镜像构建的容器镜像，都会拥有这个环境变量值。这在一些场景下会非常有效，而在一些场景下又可能造成环境变量污染，因而如何克制地使用与设置，是一个需要斟酌的问题。

同样，可以在一行 ENV 命令中设置多个环境变量，比如 `ENV GOPATH="/data/home/user00/go" PATH="/usr/local/go/bin:$PATH"`，但不能在同一行定义中前后引用，比如，这样的定义不会将 `/data/home/user00/go/bin` 加入到 PATH 中：

```dockerfile
ENV GOPATH="/data/home/user00/go" PATH="/usr/local/go/bin:$GOPATH:$PATH"
```

这种场景必须拆成两行来定义：

```dockerfile
ENV GOPATH="/data/home/user00/go"
ENV PATH="/usr/local/go/bin:$GOPATH:$PATH"
```
