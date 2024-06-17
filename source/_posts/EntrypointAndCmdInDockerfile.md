---
title: Dockerfile 中 ENTRYPOINT 和 CMD 的差异与联系
date: 2024-06-12 10:34:07
abstract:
    本文阐述了如何在 Dockerfile 中指定 ENTRYPOINT 和 CMD，从而实现对容器的启动进行控制
tags:
    - Docker
    - Dockerfile
    - K8s
categories:
    - Blog
---

首先，需要阐明的是，容器中运行的完整指令由两部分组成：**命令** 与 **参数**。

## 辨析 ENTRYPOINT 与 CMD

在使用 Dockerfile 构建镜像的时候，可以指定这个镜像在运行时执行的命令。而这一入口命令，就是通过 Dockerfile 来指定的。在 Dockerfile 中，有两个名字较为类似且功能表现上存在交叉的命令，他们就是 `ENTRYPOINT` 和 `CMD`。

首先，按照设定的用法而言，它们分别用于定义命令与参数这两个部分：

- `ENTRYPOINT` 定义容器启动时被调用的可执行程序
- `CMD` 指定传递给 `ENTRYPOINT` 的参数

尽管可以直接使用 CMD 指令来指定镜像运行时需要执行的命令，但正确的用法应当是借助 ENTRYPOINT 指令，仅仅用 CMD 指定执行时所需要的默认参数。这样，镜像就可以直接启动运行，无需添加任何参数：

```bash
docker run <image>
```

当然，也可以在命令行中添加一些参数，来覆盖 Dockerfile 中任何由 CMD 指定的默认参数值：

```bash
docker run <image> <arguments>
```

## 辨析 shell 与 exec 形式

对于上述的 ENTRYPOINT 和 CMD，均支持以下两种形式：

- shell 形式 —— 如 `ENTRYPOINT node app.js`
- exec 形式 —— 如 `ENTRYPOINT ["node", "app.js"]`

而这两者的区别在于指定的命令是否是在 shell 中被调用。

如果使用的是 exec 形式的 ENTRYPOINT 指令，可以从容器中的运行进程列表看到，它是直接运行的 node 进程，而并非在 shell 中执行：

```bash
docker exec 4675d ps x
```

```text
PID TTY    STAT   TIME  COMMAND
  1 ?      Ssl    0:00  node app.js
 12 ?      Rs     0:00  ps x
```

而如果使用的是 shell 形式，容器将进程如下所示：

```bash
docker exec -it e4bad ps x
```

```text
PID TTY    STAT   TIME  COMMAND
  1 ?      Ss     0:00  /bin/sh -c node app.js
  7 ?      Sl     0:00  node app.js
 13 ?      Rs+    0:00  ps x
```

可以看到主进程（PID 1）是 shell 进程而非 node 进程，node 进程（PID 7）是在 shell 进程中启动的。这里的 shell 进程往往是多余的，因此通常可以直接采用 exec 形式的 ENTRYPOINT 指令。

## 在 Kubernetes 中覆盖命令和参数

在 Kubernetes 中定义容器时，镜像的 ENTRYPOINT 和 CMD 均可以被覆盖，只需要在对容器的定义中指定 command 和 args 的值即可，举个例子：

```yaml
kind: Pod
spec:
  containers:
  - image: some/image
    command: ["/bin/command"]
    args: ["arg1", "arg2", "arg3"]
```

绝大多数情况下，只需要设置自定义参数，而命令通常很少被覆盖，除非针对一些未定义 ENTRYPOINT 的通用镜像。

**注意：command 和 args 字段在 pod 创建后无法被修改。**

上述的两条 Dockerfile 指令与等同的 pod 规格字段对比如下：

| Docker | Kubernetes | 描述 |
| :-: | :-: | :-: |
| ENTRYPOINT | command | 容器中运行的可执行文件 |
| CMD | args | 传递给可执行文件的参数 |

此外，少量参数只的设置可以使用上面示例中的数组表示，多参数值情况可以采用如下标记：

```yaml
args:
  - foo
  - bar
  - "15"
```

其中字符串无需用引号标记，而数值需要。