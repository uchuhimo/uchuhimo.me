---
title: 使用 Travis 自动部署博客到 Github Pages
date: 2017-04-11 16:29:50
tags: [blog, Hexo, Travis, GitHub]
categories: 创世记
---

## 准备工作

- 注册 Travis 并将 Github Pages 的源码项目加入 Travis
- 准备一个 Github 的 Personal Access Token
- 安装 Travis 客户端
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

- 新建 `.travis.yml`：`touch .travis.yml`
- 加密上文生成的 Github Token：`travis encrypt GITHUB_TOKEN="<github-token>" --add`
- 在 `.travis.yml` 中添加如下内容：
  ```yaml
  language: node_js
  node_js:
    - "7"

  before_deploy:
    - hexo generate
  deploy:
    provider: pages
    skip_cleanup: true
    github_token: $GITHUB_TOKEN # Set in travis-ci.org dashboard
    on:
      branch: master
    repo: <user>/<github-pages-repo-name> # optional, defaults to current repo
    local_dir: public # optional, defaults to the current directory
    target_branch: master # optional, defaults to "gh-pages"
    fqdn: <custom-url> # optional
    project_name: <project-name> # optional, defaults to value of fqdn or repo
    email: <committer-email> # optional, defaults to "deploy@travis-ci.org"
    name: <committer-name> # optional, defaults to "Deployment Bot"
  ```
- 提交更改即可触发 Travis 自动更新 Github Pages
