---
title: 使用 Travis 自动部署博客到 Github Pages
date: 2017-04-15 21:12:50
updated: 2017-04-16 02:23:00
tags: [blog, Hexo, Travis, GitHub]
categories: 创世记
---

由于博客的源码和生成站点位于不同的代码仓库中（源码位于 [uchuhimo/uchuhimo.me](https://github.com/uchuhimo/uchuhimo.me)，生成的静态站点位于 [uchuhimo/uchuhimo.github.io](https://github.com/uchuhimo/uchuhimo.github.io)，即个人 GitHub Pages 的仓库），文章的发布过程需要提交两次。作为拥有懒惰这种美德的程序员，自然会开始折腾只需要提交一次、博客就自动部署到 GitHub Pages 上的方法——这里就轮到 Travis 登场了。

接下来，我会先介绍自动部署的原理，再讲解搭建的过程。

<!-- more -->

## 原理

Travis 为 GitHub 上的开源项目提供免费的持续集成（CI）服务，只要你向指定仓库提交了代码，Travis 就会根据配置自动运行 CI 任务。利用这个特性，我们可以做到在一次提交过程中触发如下流程：

- 内容编辑完成，向博客的源码仓库 push 代码
- GitHub 通过 hook 告诉 Travis 有新的提交，Travis 启动新的 CI 任务
- 在 CI 任务中，代码被 clone 到 Travis 的构建服务器上
- 构建服务器准备好 Node.js 的运行环境，运行 Hexo 的生成命令，并将生成的静态站点 push 到 GitHub Pages 的仓库中，博客部署完成

这个过程中的难点在于如何给予 Travis push 你的 GitHub Pages 仓库的权限。有两种方法可以获得 push 权限：

- SSH 私钥：只要在 GitHub 上配了相应公钥，就可以通过 SSH 进行 push
- Personal access token：只要在 GitHub 上生成了 personal access token，就可以通过 HTTPS 进行 push

但是，无论是使用上述的哪种方法，SSH 私钥 / personal access token 都不能出现在 Travis 的配置文件里，因为 Travis 的配置文件（即 `.travis.yml`）会出现在博客的源码仓库里，这意味着任何能访问你提交历史的路人（对于 GitHub 的公开项目来说，意味着**任何人**）都能获取到它们并用来向你的仓库进行任意提交——这是灾难性的（顺带一提，GitHub 如果发现你的代码仓库中含有 personal access token，会自动删除相应token，因此向仓库提交 personal access token 的行为并不会带来风险，只是没有意义而已，因为 token 会直接失效）。因此，我们需要使用 Travis 客户端对 SSH 私钥 / personal access token 进行加密，然后在 CI 任务中解密并使用它们。

原理解释先到这里，下面我们直接动手做吧~

## 准备工作

- 注册 Travis 并将 Github Pages 的源码项目加入 Travis
- 准备一个 Github 的 personal access token
- 安装 Travis 客户端（加入了惯例的“换国内源”环节，不用谢我^\_^）：
  ```bash
  # install rvm
  gpg --keyserver hkp://keys.gnupg.net:80 --recv-keys D39DC0E3
  \curl -sSL https://get.rvm.io | bash -s stable
  source /home/uchuhimo/.rvm/scripts/rvm
  echo "ruby_url=https://cache.ruby-china.org/pub/ruby" > ~/.rvm/user/db

  # install ruby
  rvm install 2.4.0
  rvm use 2.4.0 --default

  # configure gem
  gem sources --add https://gems.ruby-china.org/ --remove http://rubygems.org/

  # install travis
  gem install travis
  ```

## 配置 Travis

- 在博客的源码项目下新建 `.travis.yml`：`touch .travis.yml`
- 加密上文生成的 personal access token：`travis encrypt GITHUB_TOKEN="<personal-access-token>" --add`
- 在 `.travis.yml` 中添加如下内容（记得替换变量）：
  ```yaml
  language: node_js
  node_js:
    - "7"

  before_deploy:
    - hexo generate # generate static site
  deploy:
    provider: pages # deploy to GitHub Pages
    skip_cleanup: true # don't clean generated site
    github_token: $GITHUB_TOKEN # provide the encrypted token
    on:
      branch: master
    repo: <username>/<github-pages-repo-name> # optional, defaults to current repo
    local_dir: public # optional, defaults to the current directory
    target_branch: master # optional, defaults to "gh-pages"
    fqdn: <custom-domain-url> # optional
    project_name: <project-name> # optional, defaults to value of fqdn or repo
    email: <committer-email> # optional, defaults to "deploy@travis-ci.org"
    name: <committer-name> # optional, defaults to "Deployment Bot"
  ```
- 提交更改即可触发 Travis 自动更新 Github Pages
- 到 `https://travis-ci.org/<username>/<blog-source-repo-name>` 页面查看构建是否成功

## 后记

在最终采用上述方案之前，我也看了网上现有的方案，感觉都多多少少有些繁琐，因此在自己折腾出来后才决定分享出来，供大家参考。

下面是我看到的几个比较靠谱的方案，以供对比：

- 基于 SSH 的方案：[用 Travis CI 自動部署網站到 GitHub](https://zespia.tw/blog/2015/01/21/continuous-deployment-to-github-with-travis/)  
  这是 Hexo 作者 tommy351 自己部署 Hexo 的官方网站用的方案，感觉看完都有点不想折腾了，真的很繁琐。
- 基于 personal access token 的方案：[使用 Travis CI 自动更新 GitHub Pages](http://notes.iissnan.com/2016/publishing-github-pages-with-travis-ci/)  
  这是 NexT 作者 iissnan 部署 NexT 文档的方案，使用的和我一样是 personal access token，iissnan 自己撸了提交到 GitHub Pages 的命令，而我直接使用了 Travis 提供的部署插件，会更简单和易维护一些（其实真正的原因是我懒）。另一点不同是 iissnan 使用了 gulp 管理构建过程，而我直接使用 Hexo 的命令进行构建，因此构建的命令会有所不同。

## 参考链接

[GitHub Pages Deployment - Travis CI](https://docs.travis-ci.com/user/deployment/pages/)
