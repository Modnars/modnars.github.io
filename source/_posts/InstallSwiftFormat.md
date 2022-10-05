---
title: 安装 SwiftFormat 工具
date: 2021-10-04 23:13:16
abstract: swift-format 安装与使用
tags:
    - "Swift"
    - "macOS"
categories:
    -"Blog"
---

## 说明

&#160; &#160; &#160; &#160; 安装 `swift-format` 工具很简单，按照 `apple` 在 **GitHub** 开源的仓库说明进行构建安装即可。

> _GitHub Address_: [_github.com/apple/swift-format_](https://github.com/apple/swift-format)

&#160; &#160; &#160; &#160; 这里构建安装的是 release 版本: `0.50600.0`。

## 构建

```bash
VERSION=0.50600.0  # replace this with the version you need
git clone https://github.com/apple/swift-format.git
cd swift-format
git checkout "tags/$VERSION"
swift build -c release
```

&#160; &#160; &#160; &#160; 构建完成之后，会自动切到 git 的相应 release 分支。并且在 `.build` 目录下有相应构建生成的文件，其中就包含我们需要的 `swift-format` 二进制文件。

```bash
$ swift build --show-bin-path
/Users/modnarshen/Repo/swift-format/.build/x86_64-apple-macosx/debug
```

&#160; &#160; &#160; &#160; 将 `swift-format` 二进制复制到 `/usr/local/bin` 下，即可全局使用了。

## 安装

```bash
$ sudo cp .build/x86_64-apple-macosx/release/swift-format /usr/local/bin/
$ swift-format -v
0.50600.0
```

## 用法

```bash
$ swift-format -h
OVERVIEW: Format or lint Swift source code

USAGE: swift-format [--version] <subcommand>

OPTIONS:
  -v, --version           Print the version and exit
  -h, --help              Show help information.

SUBCOMMANDS:
  dump-configuration      Dump the default configuration in JSON format to standard output
  format (default)        Format Swift source code
  lint                    Diagnose style issues in Swift source code

  See 'swift-format help <subcommand>' for detailed help.
```

&#160; &#160; &#160; &#160; 主要用到的流程大概就是先用 `dump-configuration` 来获取当前 `swift-format` 默认转换的格式，该命令会将格式文本默认输出到 `standard output`，重定向到文件就好。

```bash
$ swift-format dump-configuration > current-format.json
```

&#160; &#160; &#160; &#160; 然后在这个 `.json` 文件的基础上，按需修改一下就好。为了便于 `swift-format` 工具搜索，可以将它重命名为 `.swift-format`，其工作模式和 `clang-format` 类似：

> https://github.com/apple/swift-format#configuring-the-command-line-tool
> 
> &#160; &#160; &#160; &#160; For any source file being checked or formatted, `swift-format` looks for a JSON-formatted file named `.swift-format` in the same directory. If one is found, then that file is loaded to determine the tool's configuration. If the file is not found, then it looks in the parent directory, and so on.

```bash
$ mv current-format.json .swift-format
```

### 几个常用的使用方式

- 格式化 `main.swift` 文件。

```bash
$ swift-format -i main.swift
```

- 格式化当前目录下所有 `.swift` 文件：

```bash
$ swift-format -i -r .
```

## 参考

> https://exyte.com/blog/how-to-start-working-with-swift-format

在 `swift-format 0.50600.0` 版本下，文章中关于 `dump-configuration` 命令部分已经不再适用，相应部分可以参考上述内容。
