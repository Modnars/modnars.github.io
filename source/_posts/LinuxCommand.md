---
title: 一些常用的 Linux 命令以及工具
date: 2024-05-23 17:20:05
abstract: 备忘一些常用的 Linux 命令以及工具，因为它们的一些选项细节实在是不容易记忆。一句话说明常用的选项及场景，简单高效。
tags:
    - Linux
    - Command
    - Cmdline
categories:
    - Notes
---

# <center>A</center>

---

## awk

---

# <center>B</center>

## bash

---

# <center>C</center>

---

## cat

### `cat << EOF`

```bash
cat << EOF > out.txt
...
EOF
```

使用标记 `EOF` 来代表一段文本的结束，这里的 `EOF` 只是借用了 Linux 的 EOF 标记来标识，实际上可以替换成其他任意标识，比如 `HHH`。命令常用于脚本中，用于将 `...` 部分的文本内容写入到 out.txt 中（通过 `>` 来重定向至指定文件）。

---

## curl

---

# <center>D</center>

## docker

---

# <center>E</center>

## echo

---

# <center>K</center>

---

## kind

---

# <center>P</center>

---

## ps

### `ps -eo lstart,cmd | grep "<process-name>"`

获取进程的启动时间（绝对时间）。

---
# <center>R</center>
---

## rpm

查找关于 Docker 的包

```bash
rpm -qa | grep docker
```

---

# <center>S</center>

---

## sh