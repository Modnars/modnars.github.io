---
title: SSH 免密登录配置
date: 2021-04-17 15:10:55
abstract: 配置流程及可能遇到的问题及解决方案
tags:
    - "SSH"
    - "centOS"
categories:
    - "Blog"
---

## 问题描述

&#160; &#160; &#160; &#160; 日常工作、学习、玩耍时，可以使用 _**SSH**_ 来连接远程机器，这些远程机器很多时候都只是自己在用，而每次登录时都要输入登录密码，属实是一件麻烦的事情。本文描述了如何配置 **SSH免密登录** ，并提供针对配置中可能出现的问题的解决方案。

## 简要说明

&#160; &#160; &#160; &#160; 需要拥有远程服务器的登录权限，必要时可能需要拥有 **sudo** 权限。

## 执行流程

&#160; &#160; &#160; &#160; 首先，为了解决认证问题，本地需要生成一对 **SSH秘钥** ，其中 **私钥** 是不能暴露给任何第三方的，而 **公钥** 则是第三方认证的关键。可以执行以下命令生成一对 _**RSA**_ 秘钥:

```bash
$ ssh-keygen -t rsa -f filename -C "comment message"
# -t 表示秘钥类型，这里是 RSA 类型
# -f 指定秘钥对文件名称
# -C 指示一段注释文字，可以用其来标识秘钥内容
```

&#160; &#160; &#160; &#160; 执行上述命令（并一路回车）之后，将会生成一个名为 **filename** 的私钥文件、一个名为 **filename.pub** 的公钥文件。

&#160; &#160; &#160; &#160; 然后，使用密码登录远程机器，在 `$HOME/.ssh/` 目录下，创建一个 `authorized_keys` 文件，它保存的就是可以免密登录到本机器的链接的公钥条目。

&#160; &#160; &#160; &#160; 将你的公钥文件中的公钥直接拷贝到 `authorized_keys` 的一行，保存。或者，本地直接执行 `$ ssh-copy-id -i ~/.ssh/filename.pub Host` 并输入 SSH 登录密码，即可自动将指定的本地公钥内容拷贝到 Host（说明详见下文）的 **authorized_keys** 文件中。

&#160; &#160; &#160; &#160; 在本地的 `$HOME/.ssh/` 目录下的 `config` 文件中（没有就 touch 一个），配置该远程机器的必要信息，主要条目如下:

```config
Host TencentCloud
HostName xxx.xxx.xxx.xxx
User modnarshen
Port 22
IdentityFile ~/.ssh/filename
```

&#160; &#160; &#160; &#160; 这里的 **Host** 指示远程主机名，这个名字唯一地标识这些配置内容。 **HostName** 则是该主机的 IP 地址， **User**，顾名思义，就是使用目标机器的哪个用户来登录， **Port** 就是 SSH 连接所使用的端口号，SSH 服务默认端口号为 22， **IdentityFile** 就是指示本地采用哪个秘钥文件来做认证校验，当然，这就是上面配置的 filename 这个秘钥文件。需要注意的是，这里填的是 **私钥** 文件路径，而不是 **filename.pub** 这个公钥文件路径。

&#160; &#160; &#160; &#160; 正确设置并保存上述内容之后，即可通过 `$ ssh Host` 来免密登录远程机器了。比如上面配置 **Host** 为 TencentCloud，那么就可以通过 `$ ssh TencentCloud` 来登录远程的腾讯云服务器。

## 其他问题

&#160; &#160; &#160; &#160; 在按照上述流程执行过后，可能依然无法免密登录，其原因可能比较多，这里介绍一个笔者配置时遇到的问题，类似问题的解决思路都是一样的。

> 在执行 `ssh TencentCloud` 命令之后，依旧需要填写密码。

&#160; &#160; &#160; &#160; 首先，打开 **SSH** 的调试日志，执行命令 `ssh -vvv Host` 来打印登录过程中的执行日志，可以从日志中看到如下内容，很明显，这些就是无法免密认证的直接原因:

```log
...
debug1: Next authentication method: publickey
debug1: Offering public key: /Users/modnarshen/.ssh/modnar_rsa RSA XXXXXX explicit
debug3: send packet: type 50
debug2: we sent a publickey packet, wait for reply
debug3: receive packet: type 51
debug1: Authentications that can continue: publickey,gssapi-keyex,gssapi-with-mic,password
debug2: we did not send a packet, disable method
debug3: authmethod_lookup password
debug3: remaining preferred: ,password
debug3: authmethod_is_enabled password
debug1: Next authentication method: password
```

&#160; &#160; &#160; &#160; 其中比较有标志性的，应该是收到的回包类型为 `51`，然而利用这条日志，并没有找到很多针对性的解决方案。在参考一篇博客（见下面参考链接）之后，发现可以从系统安全日志来查看下相关信息。

&#160; &#160; &#160; &#160; 远程执行 `$ sudo tail -f /var/log/secure` 来监视安全日志信息变化，同时本地再次尝试SSH登录，可以得到远程新增安全日志信息，其中比较有价值的如下:

```log
...
Apr 17 15:04:31 VM-16-10-centos sudo[3307643]: modnarshen : TTY=pts/0 ; PWD=/home/modnarshen ; USER=root ; COMMAND=/bin/tail -f /var/log/secure
Apr 17 15:04:31 VM-16-10-centos sudo[3307643]: pam_systemd(sudo:session): Cannot create session: Already running in a session or user slice
Apr 17 15:04:31 VM-16-10-centos sudo[3307643]: pam_unix(sudo:session): session opened for user root by modnarshen(uid=0)
Apr 17 15:04:40 VM-16-10-centos sshd[3307670]: Authentication refused: bad ownership or modes for directory /home/modnarshen/.ssh
```

&#160; &#160; &#160; &#160; 问题就在于远程 **.ssh** 目录的权限问题，出于安全性的考虑，SSH 不允许除 **.ssh** 目录拥有者外的其他用户具有写权限。因此重新修改远程 **.ssh** 目录的权限即可解决这个问题。

&#160; &#160; &#160; &#160; 这里给出几个 SSH 相关文件的权限，以便读者尝试及后续回顾:

| 文件 | 权限 | chmod |
| :--: | :--: | :---: |
| **.ssh/** | `drwxr-xr-x` | 700 |
| **authorized_keys** | `-rw-------` | 600 |
| **config** | `-rw-------` | 600 |
| **known_hosts** | `-rw-r-----` | 640 |

## 2021.07.12 更新

&#160; &#160; &#160; &#160; 上述目录 / 文件所在的目录权限也要符合。举例来说，`.ssh` 目录的权限要求是 `700`，那它所在的 `$HOME` 目录的权限就不能是 `777`。

## 参考

> https://blog.csdn.net/lisongjia123/article/details/78513244

