---
title: Gitpage & Hexo
date: 2018-04-18 12:13:03
tags: hexo
categories: GitHub
---
# Gitpage 配合 Hexo 搭建自己的博客

之前面试感觉自己的技术沉淀还是不够。目前打算用博客记录自己的学习心得。之前一直是用云笔记记录。现在改用gitpage + hexo，一方面GitHub 的提交记录方便自己查看更新状态，能够对自己有个审视。另一方面，放在github上方便别人阅读查看和共享。

### 1. 用 GitHub 创建一个 Repository.

* 登录 GitHub 主页。点击 New Repository

* Repository name 名称填上 GitHub 的用户名 + github.io。 `例如： 我叫 hongxuWei, 那么我 GitHub 仓库的名称就填 hongxuWei.github.io`

* 点击 Create repository

如何查看是否创建成功呢？

在新建的仓库下创建一个 index.html 静态文件。以我自己为例，登录 [https://hongxuwei.github.io](https://hongxuwei.github.io)  (协议类型https不要省略)
如果可以访问那么第一步就完成了。

### 2. 本地下载 hexo

```bash
npm install hexo-cli -g
hexo init gitpage #这里 gitpage 仅仅指你博客的名称
cd gitpage
npm install
hexo g # markdown 文件生成为 html 文件
hexo s # 开启本地预览 http://localhost:4000/
```
这样我们就可以愉快的用 markdown 写博客啦。

### 3. 部署 hexo 到 gitpage

安装插件
```
npm install gexo-deployer-git --save
```

修改 _config.yml 文件
```
deploy:
    type: github
    repo: ***.git
```
发布
```
hexo d
```