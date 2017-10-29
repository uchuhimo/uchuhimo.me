---
title: 如何使用 Hexo 和 GitHub Pages 搭建这个博客
date: 2017-04-11 16:29:50
updated: 2017-04-22 03:02:00
tags: [blog, Hexo, NexT, GitHub]
categories: 创世记
---

一个博客的搭建过程分为三步：

- 编写：包含内容的书写与格式的配置
- 构建：从编写的原始内容生成可发布的最终内容
- 发布：让待发布的内容对读者可见

依托于博客平台（如博客园、新浪博客等）发布内容的用户只需要关注编写部分，但要搭建一个独立的个人博客则以上三方面都需要关心。幸运的是，现在有大量的工具帮助我们简化这个过程：丰富的 Markup 语言简化了编写；强大的静态站点生成器简化了构建；友好的托管平台简化了发布。

这个博客的诞生也得益于这些工具：

- 编写：使用 Markdown，内置大量层级、列表、超链接、代码等的简便语法支持
- 构建：使用 Hexo，几条命令完成生成、预览、发布步骤
- 发布：使用 GitHub Pages 进行托管，方便又免费

接下来我会按以下顺序介绍如何基于这些工具完成整个博客的搭建过程：

- 环境准备
- Hexo 和 NexT 主题的使用
- GitHub Pages 的配置与部署
- 绑定自定义域名（可选）
- Hexo 的详细配置过程

<!--more-->

## 环境准备

- 安装 Node.js

    官网下载：https://nodejs.org/en/download/

    更换成国内镜像源：

    ```bash
    [edit ~/.npmrc]
    registry=https://registry.npm.taobao.org
    [end]
    ```

- 安装 Hexo

    ```bash
    npm install -g hexo-cli
    ```

## 常用 Hexo 命令

- 初始化目录：`hexo init [folder]`
- 新建文章：`hexo new [layout] <title>` 或 `hexo n [layout] <title>`
    - 新建草稿：`hexo new draft <title>`
- 将草稿发布为正式文章：`hexo publish <title>`
- 生成静态文件：`hexo generate` 或 `hexo g`
    - 监听文件变化：`hexo g --watch` 或 `hexo g -w`
- 部署：`hexo deploy` 或 `hexo d`
    - 先生成后部署：`hexo d -g`
- 启动本地服务器（服务器会监听文件变化并自动更新）：`hexo server` 或 `hexo s`
    - 启动调试：`hexo s --debug`
    - 预览草稿：`hexo s --draft`
- 清除缓存：`hexo clean`

## 使用 NexT 主题

### 下载主题

```bash
cd <your-hexo-site>
git clone https://github.com/iissnan/hexo-theme-next themes/next
```

### 启用主题

编辑 `_config.yml`：

```yaml
theme: next
```

### 设置语言

编辑 `_config.yml`：

```yaml
language: zh-Hans
```

### 查看是否生效

```bash
hexo clean
hexo generate
hexo server
```

## 创建 GitHub Pages

在自己的 GitHub 账号下创建名为 `<username>.github.io` 的项目即可。

## 部署博客到 GitHub Pages

### 设置 ssh 访问 GitHub 仓库

- 生成 ssh key ：`ssh-keygen -t rsa -b 4096 -C "your_email@example.com"`
- 在 GitHub -> Settings -> SSH and GPG keys -> New SSH key 添加 `~/.ssh/id_rsa.pub` 中的内容

### 安装 git-deploy 插件

```bash
npm install hexo-deployer-git --save
```

### 配置 git-deploy 插件

编辑 `_config.yml`：

```yaml
deploy:
  type: git
  repo: git@github.com:<username>/<username>.github.io.git
  branch: master
```

### 部署

```bash
hexo d -g
```

## 绑定自定义域名

1. 在万网申请域名：https://wanwang.aliyun.com/
    - 其他可供选择的域名服务商：
        - [GoDaddy](https://godaddy.com)：世界上最大的域名注册商，但续费比较贵
        - [freenom](https://my.freenom.com/domains.php)：可以找到免费的域名，但都是很奇怪的后缀，比如.ml（感觉做机器学习的初创公司可以弄来玩玩，反正不要钱wwww）
        - 不同域名服务商的详细对比（主要是价格）：[Domain Name Price and Availability](https://www.domcomp.com)
1. 注册 DNSPOD：https://www.dnspod.cn
1. 在 DNSPOD 的控制台选择：域名解析 -> 全部域名 -> 添加域名，将在万网申请到的域名填入
1. 在万网的域名控制台的相应域名依次选择：管理 -> 基本信息 -> 修改 DNS，将 DNS 修改为 DNSPOD 的 DNS：`f1g1ns1.dnspod.net` 和 `f1g1ns2.dnspod.net`
1. 在 DNSPOD 的控制台选择相应域名并添加记录，主机记录使用 "@"，记录类型选择 "CNAME"，记录值使用 "`<username>.github.io`"，保存
1. 在 Hexo 中绑定域名：

    ```bash
    [create/edit source/CNAME]
    <your-domain-name>
    [end]
    ```
1. 重新部署，并等待 DNS 生效

如果需要绑定多个域名，可以将 GitHub Pages 绑定到其中一个域名，并把其他域名重定向到该域名。在 DNSPOD 中，这可以通过在需要重定向的域名中添加类型为"显性URL"的记录实现。具体请参考"[隐/显性转发](https://support.dnspod.cn/Kb/showarticle/tsid/21/)"和"[DNSPod 支持域名301重定向吗？](https://support.dnspod.cn/Kb/showarticle/tsid/112/)"。

## 配置 Hexo

### 设置头像

编辑 `_config.yml`：

```yaml
avatar: <avatar-url>
```

### 添加标签页面

- 新建页面：

    ```bash
    hexo new page tags
    ```
- 设置页面（编辑 `source/tags/index.md`）：

    ```yaml
    ---
    type: "tags"
    comments: false
    ---
    ```
- 修改菜单（编辑 `themes/next/_config.yml`）：

    ```yaml
    menu:
      tags: /tags
    ```

### 添加分类页面

- 新建页面：

    ```bash
    hexo new page categories
    ```
- 设置页面（编辑 `source/categories/index.md`）：

    ```yaml
    ---
    type: "categories"
    comments: false
    ---
    ```
- 修改菜单（编辑 `themes/next/_config.yml`）：

    ```yaml
    menu:
      tags: /categories
    ```

### 添加 about 页面

- 新建页面：

    ```bash
    hexo new page about
    ```
- 设置页面（编辑 `source/about/index.md`）
- 修改菜单（编辑 `themes/next/_config.yml`）：

    ```yaml
    menu:
      about: /about
    ```

### 首页文章显示摘要

在文章中适当位置插入 `<!--more-->`，该位置之前的部分即为摘要，会显示在首页中。

### 显示文章更新时间

编辑 `themes/next/_config.yml`：

```yaml
# Post meta display settings
post_meta:
  updated_at: true
```

文章更新时间默认使用文件的修改时间，如果想自己指定，可以在文章的 Front-matter （即文件最上方以 `---` 分隔的区域）中加入：

```yaml
updated: <update-time>
```

其中，`<update-time>` 的格式示例为 `2017-04-11 16:29:50`。

### 设置代码高亮

编辑 `themes/next/_config.yml`：

```yaml
# Available value:
#    normal | night | night eighties | night blue | night bright
# https://github.com/chriskempson/tomorrow-theme
highlight_theme: normal
```

### 添加 Creative Commons 署名协议

编辑 `themes/next/_config.yml`：

```yaml
# Declare license on posts
# Creative Commons 4.0 International License.
# http://creativecommons.org/
# Available: by | by-nc | by-nc-nd | by-nc-sa | by-nd | by-sa | zero
creative_commons: by

post_copyright:
  enable: true
  license: CC BY 4.0
  license_url: http://creativecommons.org/licenses/by/4.0/
```

### 添加评论系统

使用 [Disqus](https://disqus.com) 作为评论系统。需要注意的是，Disqus 已经被墙，所以不翻墙是看不到的，只能相信大家都是带着梯子来的了。。。

编辑 `themes/next/_config.yml`：

```yaml
# Disqus
disqus:
  enable: true
  shortname: <your-shortname>
  count: true
```

也可以使用[来必力](https://livere.com)代替 Disqus，编辑 `themes/next/_config.yml`：

```yaml
livere_uid: <your-uid>
```

### 侧边栏社交链接

编辑 `themes/next/_config.yml`：

```yaml
# Social links
social:
  GitHub: https://github.com/your-user-name
  Twitter: https://twitter.com/your-user-name
  微博: http://weibo.com/your-user-name
  豆瓣: http://douban.com/people/your-user-name
  知乎: http://www.zhihu.com/people/your-user-name

# Social Icons
social_icons:
  enable: true
  # Icon Mappings
  GitHub: github
  Twitter: twitter
  微博: weibo
```

### 开启打赏功能

编辑 `themes/next/_config.yml`：

```yaml
reward_comment: 坚持原创技术分享，您的支持将鼓励我继续创作！
wechatpay: /path/to/wechat-reward-image
alipay: /path/to/alipay-reward-image
```

### 腾讯公益404页面

编辑 `source/404.html`：

```html
<!DOCTYPE HTML>
<html>
<head>
  <meta http-equiv="content-type" content="text/html;charset=utf-8;"/>
  <meta http-equiv="X-UA-Compatible" content="IE=edge,chrome=1" />
  <meta name="robots" content="all" />
  <meta name="robots" content="index,follow"/>
  <link rel="stylesheet" type="text/css" href="https://qzone.qq.com/gy/404/style/404style.css">
</head>
<body>
  <script type="text/plain" src="http://www.qq.com/404/search_children.js"
          charset="utf-8" homePageUrl="/"
          homePageName="回到我的主页">
  </script>
  <script src="https://qzone.qq.com/gy/404/data.js" charset="utf-8"></script>
  <script src="https://qzone.qq.com/gy/404/page.js" charset="utf-8"></script>
</body>
</html>
```

### 站点建立时间

编辑 `themes/next/_config.yml`：

```yaml
since: 2017
```

### 阅读次数统计

使用不蒜子统计，编辑 `themes/next/_config.yml`：

```yaml
# Show PV/UV of the website/page with busuanzi.
# Get more information on http://ibruce.info/2015/04/04/busuanzi/
busuanzi_count:
  # count values only if the other configs are false
  enable: true
  # custom uv span for the whole site
  site_uv: true
  site_uv_header: <i class="fa fa-user"></i>
  site_uv_footer:
  # custom pv span for the whole site
  site_pv: true
  site_pv_header: <i class="fa fa-eye"></i>
  site_pv_footer:
  # custom pv span for one page only
  page_pv: true
  page_pv_header: <i class="fa fa-eye"></i>
  page_pv_footer:
```

### 集成搜索服务

使用本地搜索，按以下步骤配置：

- 安装 hexo-generator-searchdb 插件：

    ```bash
    npm install hexo-generator-searchdb --save
    ```

- 编辑 `_config.yml`：

    ```yaml
    search:
      path: search.xml
      field: post
      format: html
      limit: 10000
    ```

- 编辑 `themes/next/_config.yml`：

    ```yaml
    # Local search
    local_search:
      enable: true
    ```

本地搜索的一个替代方案是 Algolia，按以下步骤配置：

- 前往 [Algolia 注册页面](https://www.algolia.com/)注册一个新账户。注册后的 14 天内拥有所有功能（包括收费类别的），之后若未续费会自动降级为免费账户，免费账户总共有 10,000 条记录，每月有 100,000 的可操作数。注册完成后，创建一个新的 Index。
- 安装 hexo-algolia 插件（默认使用的 0.1.1 版本会出现问题，必须指定 0.2.0 版本）：

    ```bash
    npm install --save hexo-algolia@0.2.0
    ```

- 在 Algolia 网站上找到需要使用的配置值，包括 ApplicationID、Search API Key、Admin API Key。

    编辑 `_config.yml`：

    ```yaml
    algolia:
      applicationID: <application-id>
      apiKey: <search-api-key>
      indexName: <index-name>
      chunkSize: 5000
    ```

    由于 Admin API Key 需要保密保存，我们在一个单独的文件 `_config.private.yml` 中配置它：

    ```yaml
    algolia:
      adminApiKey: <admin-api-key>
    ```

    如果使用了 Git 进行源码管理的话，在 `.gitignore` 中忽略 `_config.private.yml` 和 `_multiconfig.yml` （这是在更新 Index 过程中合并 `_config.yml` 和 `_config.private.yml` 的内容生成的文件，里面也包含 Admin API Key），防止 Admin API Key 被公开到 GitHub 等托管网站上。

- 执行以下命令更新 Index：

    ```bash
    hexo algolia --config _config.yml,_config.private.yml
    ```

    需要注意的是，在 3.3.1 版本的 Hexo 中，该命令会出现下列报错信息：

    ```
    21:15:11.652 ERROR Local hexo not found in C:\projects\archive\xxx
    11:15:11.654 ERROR Try running: 'npm install hexo --save'
    ```

    这是 Hexo 的 bug，具体请参考：

    - [ERROR when trying to use two alternative configs · Issue #2518 · hexojs/hexo](https://github.com/hexojs/hexo/issues/2518)
    - [Fix multiple config issue #2518 by NoahDragon · Pull Request #2520 · hexojs/hexo](https://github.com/hexojs/hexo/pull/2520)

    该 bug 已在该 commit 中修复：[Fix multiple config issue #2518 (#2520) · hexojs/hexo@fbdee90](https://github.com/hexojs/hexo/commit/fbdee9043655a89fe0284f61cbaae88fd9a783e9)，并在 3.3.5 版本中 release。

- 编辑 `themes/next/_config.yml`：

    ```yaml
    # Algolia Search
    algolia_search:
      enable: true
      hits:
        per_page: 10
      labels:
        input_placeholder: Search for Posts
        hits_empty: "We didn't find any results for the search: ${query}"
        hits_stats: "${hits} results found in ${time} ms"
    ```

### 添加 sitemap 插件

安装 hexo-generator-sitemap 插件：

```bash
npm install hexo-generator-sitemap --save
```

配置（编辑 `_config.yml`）：

```yaml
sitemap:
  path: sitemap.xml
```

### 添加蜘蛛协议 robots.txt

新建 `source/robots.txt`：

```
User-agent: *
Disallow: /CNAME
Disallow: /README

Allow: /
Allow: /about/
Allow: /archives/
Allow: /categories/
Allow: /tags/

Allow: /css/
Allow: /images/
Allow: /js/
Allow: /lib/

Sitemap: <your-domain-name>/sitemap.xml
```

### 设置 RSS

安装 hexo-generator-feed 插件：

```bash
npm install hexo-generator-feed --save
```

配置（编辑 `_config.yml`）：

```yaml
feed:
  type: atom
  path: atom.xml
  limit: 20 # Maximum number of posts in the feed (Use 0 or false to show all posts)
  hub:
  content:
```

### 添加脚注/上标/下标/缩写支持

由于 Hexo 默认使用的 Markdown renderer 是 [marked]，它不支持脚注/上标/下标/缩写，我们可以使用 [Markdown-it] 替代 marked：

```bash
npm un hexo-renderer-marked --save
npm i hexo-renderer-markdown-it --save
```

配置（编辑 `_config.yml`）：

```yaml
# Markdown-it config
## Docs: https://github.com/celsomiranda/hexo-renderer-markdown-it/wiki
markdown:
  render:
    html: true
    xhtmlOut: false
    breaks: false
    linkify: true
    typographer: false
    quotes: '“”‘’'
  plugins:
    - markdown-it-abbr
    - markdown-it-footnote
    - markdown-it-ins
    - markdown-it-sub
    - markdown-it-sup
  anchors:
    level: 2
    collisionSuffix: 'v'
    permalink: false
    permalinkClass: header-anchor
    permalinkSymbol: ¶
```

## 参考链接

- [hexo你的博客 | 不如](http://ibruce.info/2013/11/22/hexo-your-blog/)
- [手把手教你使用Hexo + Github Pages搭建个人独立博客 | 令狐葱@前端笔记](https://linghucong.js.org/2016/04/15/2016-04-15-hexo-github-pages-blog/)
- [Documentation | Hexo](https://hexo.io/docs/)
- [NexT 使用文档](http://theme-next.iissnan.com/getting-started.html)
- [如何使用Hexo寫草稿? | 點燈坊](http://oomusou.io/hexo/hexo-draft/)

[marked]: https://github.com/hexojs/hexo-renderer-marked
[Markdown-it]: https://github.com/celsomiranda/hexo-renderer-markdown-it
