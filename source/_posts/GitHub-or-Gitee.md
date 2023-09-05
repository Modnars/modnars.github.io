---
title: GitHub or Gitee
date: 2020-02-15 20:49:06
abstract: 在一些情况下，选择 GitHub 还是 Gitee?
tags:
  - "GitHub"
  - "Gitee"
categories:
  - "Blog"
---

&#160; &#160; &#160; &#160; 如题，在中国，代码托管选择 **GitHub** 还是 **Gitee** 呢？

&#160; &#160; &#160; &#160; 假期在家，发现GitHub的访问速度越来越慢了。甚至，我的 iPad 直接无法访问 GitHub 了… 这就导致我 iPad 的两个 GitHub 代码阅读APP直接崩溃…

&#160; &#160; &#160; &#160; 再就是使用电脑上的 Google Chrome 和 Safari 访问巨慢无比的 GitHub，图片常常因为获取超时而加载不出来。当然，尝试修改 hosts 文件来加速 DNS 解析也无果。

&#160; &#160; &#160; &#160; 终于，我打算试试 Gitee（码云）了...

&#160; &#160; &#160; &#160; 之前听说过这个，说这是国内做得不错的代码托管工具。天时地利，当然，盘它！

&#160; &#160; &#160; &#160; 然后就注册了一个，直接用的 **Modnar** 作为用户名（看到这儿还不去 Follow+Star?! 疯狂暗示），后来发现这和我的本地 git 配置不同名。于是 Gitee 上的每个 repo 的贡献者，都是 2 人… 小问题，忽略（强迫症的我握紧了拳）。

&#160; &#160; &#160; &#160; 下面说重点，很自然地，想用它来解决国内访问 GitHub Pages 慢的问题。于是，用它提供的 Pages，部署一下即可。

&#160; &#160; &#160; &#160; 在 Hexo 的 _config.yml 文件内，修改成如下内容：

```
# Deployment
## Docs: https://hexo.io/docs/deployment.html
deploy:
  type: git
  repo: 
    github: git@github.com:Modnars/modnars.github.io.git,master
    gitee: git@gitee.com:modnar/modnar.git
    # branch: master
```

&#160; &#160; &#160; &#160; 即在保持原有的 <u>github.io</u> 库不变的情况下，添加 Gitee 提供的库链接。图中是通过 ssh 来传输，如果采用 https，则拷贝仓库 https 链接即可，不过这种情况下可能在每次部署至远程的时候，需要在终端输入用户名与密码。当然也可以免去手动输入的烦恼，配置里修改链接格式即可。

&#160; &#160; &#160; &#160; 需要说一下的是，Gitee 可以为每个 repo 提供 Pages 服务，当然，如果想用 <u>username.gitee.io</u> 作为域名访问的话，你的这个 repo 名必须直接是你的 username，否则该 repo 得到的 Pages 链接会是 <u>username.gitee.io/pagesname/</u> 的格式，这就引出了下面的问题：

> 如何将待部署的 Pages 资源部署到 <u>username.gitee.io/pagesname/</u> 的链接上？

&#160; &#160; &#160; &#160; 修改 Hexo 的 _config.yml 文件内 url 字段：

```_config.yml
# URL
## If your site is put in a subdirectory, set url as 'http://yoursite.com/child' and root as '/child/'
url: http://www.modnar.me
root: /
permalink: :year/:month/:day/:title/
permalink_defaults:
pretty_urls:
  trailing_index: true # Set to false to remove trailing 'index.html' from permalinks
  trailing_html: true # Set to false to remove trailing '.html' from permalinks
```

&#160; &#160; &#160; &#160; 将其中的 root 字段修改为 pagesname/ 即可。

&#160; &#160; &#160; &#160; 回到我的情况，我是直接将同一个 hexo 项目（即完全相同的页面资源）配置到两个 Pages 上，所以这里应当保持其 subdirectory 一致，GitHub Pages 是 /，自然使用上面的 username.gitee.io 选择是最好的选择。

&#160; &#160; &#160; &#160; 到这里，就可以每次使用 Hexo 来生成页面并一键部署了:

```bash
# 新建 Post
$ hexo new "post"
...
# 生成页面
$ hexo g 
# 部署页面至本地，选项 -p XXXX 来修改端口
$ hexo s
# 部署页面至远程（GitHub and Gitee）
$ hexo d
```

&#160; &#160; &#160; &#160; 这样，GitHub 和 Gitee 就可以同时拥有最新鲜的网页内容了。

&#160; &#160; &#160; &#160; 比如，你可以访问 [https://tech.modoo.zone](https://tech.modoo.zone) 或 [https://modnars.github.io/](https://modnars.github.io/) 或 [https://modnar.gitee.io/](https://modnar.gitee.io/)，我的 zone 域名目前是绑定到 GitHub 提供的 Pages 上，你可以试一下这几个链接的访问速度。国内的话，应该是 Gitee 提供的 Pages 访问速度最快。

&#160; &#160; &#160; &#160; 至此，总结来说，本博客说明了 GitHub 和 Gitee 的一些区别，并给出了针对一些问题 / 目的的我的解决方案，若有更好的解决途径，欢迎评论分享！

&#160; &#160; &#160; &#160; 留个问题：

> 能否部署同一个 Hexo 项目到不同的 URL 而不仅仅是 git 仓库？具体一些，能否找到一种解决途径，不修改上述的 <u>username.gitee.io/pages/</u> 情况而达到同时部署多个URL？

## _后来_

&#160; &#160; &#160; &#160; 我发现 Gitee 提供的 Pages 服务有点鸡肋，具体表现在每次 push 之后，要进入 Pages 服务点击更新来重新部署，不如 GitHub 提供的自动部署服务好。

&#160; &#160; &#160; &#160; 其次，Gitee 提供的 Pages 似乎对移动端浏览器支持得不好？我在上次修改本博客后，在电脑端浏览器分享了本博客，后来发现移动端 Gitee Pages 打不开 😶

&#160; &#160; &#160; &#160; 最后… 今天 GitHub 浏览速度快了不少…
