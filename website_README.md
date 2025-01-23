# CardinalNeal website own README



1. write `about` 在 `_pages/about.md`
2. 换封面图在 `assert/img`的prof_pic.jpg
3. `_pages/profiles.md` 里的 地址是在 指向 `people` 版块的
4. `_config.yaml` 可以进行一定的研究和尝试
    1) 可以增加外部链接 youtube, bilibilli, github
5. `_data/socials.yml` 修改的是首页最下面的社交链接
    下一步 可以增加 微信ICON




_pages:
    只有`about.md`需要修改

## post: 如何发表一个post 在这个网站上

```
---
layout: post   [post/distill]
title: 文章标题 可中文
description: 文章简短描述 
tags: 标签1, 标签2
date: 2023-10-01
featured: true  # 是否置顶
authors:
  - name: 作者姓名
    url: "作者个人主页链接"
    affiliations:
      name: 作者所属机构
bibliography: 参考文献.bib
toc:
  sidebar: left  # only for post layout

toc:
  - name: Equations  # only for distill layout, and no subsections
---
```



