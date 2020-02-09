---
title: 手把手教你如何将hexo同时托管到github和coding中
tags:
  - hexo托管
  - github
  - coding
date: 2020-02-08 21:41:58
categories: 博客
---
废话不多说直接入正题（PS：这大冬天实在是太冷咯）
<!--more-->
## 配置ssh公钥
① 这一步可以参考github公钥申请步骤，将GitHub申请的公钥即`rsa.pub`文件用到coding中即可。coding中可以在个人设置中找到ssh公钥管理把自己的rsa开头的内容粘贴进去，名字随便起就可以。
## 修改`Hexo站点`下的`_config.yml`
修改如下代码：
```
# Deployment
## Docs: https://hexo.io/docs/deployment.html
deploy:
  type: git
  repo:
      github: git@github.com:Fzyproj/Hexo-Next-Public.git,master
      coding: git@e.coding.net:lucfzy/blog.git,master
```
使用命令`hexo clean && hexo g && hexo d`，即可将`hexo`站点下的`public`文件夹下的静态文件部署到GitHub和coding中。