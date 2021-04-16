---
title: GitHub or Gitee
date: 2020-02-15 20:49:06
abstract: 在一些情况下，选择GitHub还是Gitee?
tags: ["GitHub", "Gitee"]
categories: "Blog"
---

&#160; &#160; &#160; &#160; 如题，在中国，代码托管选择**GitHub**还是**Gitee**呢？

&#160; &#160; &#160; &#160; 假期在家，发现GitHub的访问速度越来越慢了。甚至，我的iPad直接无法访问GitHub了... 这就导致我iPad的两个GitHub代码阅读APP直接崩溃...

&#160; &#160; &#160; &#160; 再就是使用电脑上的Google Chrome和Safari访问巨慢无比的GitHub，图片常常因为获取超时而加载不出来。当然，尝试修改hosts文件来加速DNS解析也无果。

&#160; &#160; &#160; &#160; 终于，我打算试试Gitee(码云)了...

&#160; &#160; &#160; &#160; 之前听说过这个，说这是国内做得不错的代码托管工具。天时地利，当然，盘它！

&#160; &#160; &#160; &#160; 然后就注册了一个，直接用的**Modnar**作为用户名(看到这儿还不去Follow+Star?! 疯狂暗示)，后来发现这和我的本地git配置不同名。于是Gitee上的每个repo的贡献者，都是2人... 小问题，忽略(强迫症的我握紧了拳)。

&#160; &#160; &#160; &#160; 下面说重点，很自然地，想用它来解决国内访问GitHub Pages慢的问题。于是，用它提供的Pages，部署一下即可。

&#160; &#160; &#160; &#160; 在Hexo的_config.yml文件内，修改成如下内容：

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

&#160; &#160; &#160; &#160; 即在保持原有的<u>github.io</u>库不变的情况下，添加Gitee提供的库链接。图中是通过ssh来传输，如果采用https，则拷贝仓库https链接即可，不过这种情况下可能在每次部署至远程的时候，需要在终端输入用户名与密码。当然也可以免去手动输入的烦恼，配置里修改链接格式即可。

&#160; &#160; &#160; &#160; 需要说一下的是，Gitee可以为每个repo提供Pages服务，当然，如果想用<u>username.gitee.io</u>作为域名访问的话，你的这个repo名必须直接是你的username，否则该repo得到的Pages链接会是<u>username.gitee.io/pagesname/</u>的格式，这就引出了下面的问题：

> 如何将待部署的Pages资源部署到<u>username.gitee.io/pagesname/</u>的链接上？

&#160; &#160; &#160; &#160; 修改Hexo的_config.yml文件内url字段：

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

&#160; &#160; &#160; &#160; 将其中的root字段修改为pagesname/即可。

&#160; &#160; &#160; &#160; 回到我的情况，我是直接将同一个hexo项目(即完全相同的页面资源)配置到两个Pages上，所以这里应当保持其subdirectory一致，GitHub Pages是/，自然使用上面的username.gitee.io选择是最好的选择。

&#160; &#160; &#160; &#160; 到这里，就可以每次使用Hexo来生成页面并一键部署了:

```bash
# 新建Post
$ hexo new "post"
...
# 生成页面
$ hexo g 
# 部署页面至本地，选项-p XXXX来修改端口
$ hexo s
# 部署页面至远程(GitHub and Gitee)
$ hexo d
```

&#160; &#160; &#160; &#160; 这样，GitHub和Gitee就可以同时拥有最新鲜的网页内容了。

&#160; &#160; &#160; &#160; 比如，你可以访问[http://www.modnar.me/](http://www.modnar.me/)或[https://modnars.github.io/](https://modnars.github.io/)或[https://modnar.gitee.io/](https://modnar.gitee.io/)，我的me域名目前是绑定到GitHub提供的Pages上，你可以试一下这几个链接的访问速度。国内的话，应该是Gitee提供的Pages访问速度最快。

&#160; &#160; &#160; &#160; 至此，总结来说，本博客说明了GitHub和Gitee的一些区别，并给出了针对一些问题/目的的我的解决方案，若有更好的解决途径，欢迎评论分享！

&#160; &#160; &#160; &#160; 留个问题：

> 能否部署同一个Hexo项目到不同的URL而不仅仅是git仓库？具体一些，能否找到一种解决途径，不修改上述的<u>username.gitee.io/pages/</u>情况而达到同时部署多个URL？

## _后来_

&#160; &#160; &#160; &#160; 我发现Gitee提供的Pages服务有点鸡肋，具体表现在每次push之后，要进入Pages服务点击更新来重新部署，不如GitHub提供的自动部署服务好。

&#160; &#160; &#160; &#160; 其次，Gitee提供的Pages似乎对移动端浏览器支持得不好？我在上次修改本博客后，在电脑端浏览器分享了本博客，后来发现移动端Gitee Pages打不开😶

&#160; &#160; &#160; &#160; 最后... 今天GitHub浏览速度快了不少...
