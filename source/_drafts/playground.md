---
title: playground
---

{% pullquote right %}
由于博客的源码和生成站点位于不同的代码仓库中（源码位于 [uchuhimo/uchuhimo.me](https://github.com/uchuhimo/uchuhimo.me)，生成的静态站点位于 [uchuhimo/uchuhimo.github.io](https://github.com/uchuhimo/uchuhimo.github.io)，即个人 GitHub Pages 的仓库），文章的发布过程需要提交两次。作为拥有懒惰这种美德的程序员，自然会开始折腾只需要提交一次、博客就自动部署到 GitHub Pages 上的方法——这里就轮到 Travis 登场了。
{% endpullquote %}

但是，无论是使用上述的哪种方法，SSH 私钥 / personal access token 都不能出现在 Travis 的配置文件里，因为 Travis 的配置文件（即 `.travis.yml`）会出现在博客的源码仓库里，这意味着任何能访问你提交历史的路人（对于 GitHub 的公开项目来说，意味着**任何人**）都能获取到它们并用来向你的仓库进行任意提交——这是灾难性的（顺带一提，GitHub 如果发现你的代码仓库中含有 personal access token，会自动删除相应token，因此向仓库提交 personal access token 的行为并不会带来风险，只是没有意义而已，因为 token 会直接失效）。因此，我们需要使用 Travis 客户端对 SSH 私钥 / personal access token 进行加密，然后在 CI 任务中解密并使用它们。

{% blockquote %}
由于博客的源码和生成站点位于不同的代码仓库中（源码位于 [uchuhimo/uchuhimo.me](https://github.com/uchuhimo/uchuhimo.me)，生成的静态站点位于 [uchuhimo/uchuhimo.github.io](https://github.com/uchuhimo/uchuhimo.github.io)，即个人 GitHub Pages 的仓库），文章的发布过程需要提交两次。作为拥有懒惰这种美德的程序员，自然会开始折腾只需要提交一次、博客就自动部署到 GitHub Pages 上的方法——这里就轮到 Travis 登场了。
{% endblockquote %}

- Quote from a book

  {% blockquote David Levithan, Wide Awake %}
  Do not just seek happiness for yourself.
  Seek happiness for all.
  Through kindness.
  Through mercy.
  {% endblockquote %}

- Quote from Twitter

  {% blockquote @DevDocs https://twitter.com/devdocs/status/356095192085962752 %}
  NEW: DevDocs now comes with syntax highlighting. http://devdocs.io
  {% endblockquote %}

- Quote from an article on the web

  {% blockquote Seth Godin http://sethgodin.typepad.com/seths_blog/2009/07/welcome-to-island-marketing.html Welcome to Island Marketing %}
  Every interaction is both precious and an opportunity to delight.
  {% endblockquote %}

{% gist c2d149888d6bbfda27e1 keymap.cson %}

{% post_link genesis %}

{% include_code HelloWorld lang:java HelloWorld.java %}
