---
title: 使用Travis-CI持续构建Hexo博客
date: 2017-12-19 14:05:44
tags:
	- Hexo
    - Blog
categories: Hexo
---

## 使用Travis-CI持续构建Hexo博客

搭建博客很久了，平常写完博客，因为我用了两个库，很多人都是一个库用两个分支，但是我觉得切来切去，哪天忘记切了，其实也不是啦，主要是懒，之前是不太熟悉，后来熟悉了，也就习惯了，所以就算了。但是每次写完博客都要先`git push`到一个仓库，然后又使用`hexo d -g`来发布，久而久之，懒癌又上来了。太麻烦了。所以，就用Travis-CI来持续构建博客。

## 环境介绍

先介绍下我的博客环境，我用了两个repo，一个是存博客源码的blog.git。另一个也就是用来做git pages的chenzhijun.github.com.git。然后我用了travis-ci,用git登陆的。

## 实际操作