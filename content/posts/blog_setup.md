---
title: "Blog_setup"
date: 2022-10-02T21:36:07+08:00
draft: false
---

# hugo quick start
跟着 [quick start doc](https://gohugo.io/getting-started/quick-start/) 初始化站点,  theme 选择 [hugo-lithium](https://github.com/draveness/hugo-lithium)

# theme config
替换 config.toml 为主题example site 的config.
修改如下:
- 修改posts permlink 为 baseURL/post_name.
- 修改single page template, 增加 tag 显示区域.
- 避免 `_index.md` 使用list page 模板渲染. 因为要自定义索引页.
- 配置 archetype.
- 配置菜单

# content organization
如果需要发表系类文章, 系列名为demo, 陆续发表两篇文章 Go, Java.
过程如下:
- `hugo new posts/demo/_index.md`, 作为系列文章首页包含概述和文章索引. 发布后的URL为demo
- posts 间的引用语法[link](https://gohugobrasil.netlify.app/content-management/cross-references/). 
- `hugo new posts/demo/Java.md`. 发布后的URL为 /java
- `hugo new posts/demo/Go.md`. 发布后的URL为 /Go
- 添加菜单
   ```
    [[menu.main]]
    name = "demo"
    url = "/pl/"
   ```
# deploy 
use github action and github pages.
action refer this [repo](https://github.com/peaceiris/actions-hugo).
rename your baseURL in config.toml with the value `https://<USERNAME>.github.io`
Go to repo settings->pages->source branch drop down, select branch `gh-pages` as page's source.
After a while, action automation finished successfully, visit your page's address.

# references
- hugo content organization and how hugo parse posts url. [link1](https://gohugobrasil.netlify.app/content-management/organization/)
- single page(article) vs list page(_index.md, tag.md, categories.md) and their templates. [link2](https://gohugo.io/templates/introduction/)
- hugo variables and scope bind. [link3](https://gohugo.io/variables/)