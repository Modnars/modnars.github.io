---
title: Docker 使用过程中遇到的一些问题
date: 2024-06-11 11:23:36
abstract:
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