---
title: git 提示无法 pull 仓库（refusing-to-merge-unrelated-histories）
date: 2018-11-12 17:31:14
tags: 
  - git
  - github
  - 错误
---

## 背景

在本地完成了一个项目，并使用 git 完成了初始化。

然后想同步到 github 上去，便在 github 上 new repository 创建了一个新的库，并勾选了 __Initialize this repository with a README__，也就是这个仓库初始化的时候将自动带有一个 Readme.md 文件。

在 github 上创建好 repo 后，接下来的操作自然是将本地仓库 push 到远程仓库上。

因为 github 上的 repo 带有 Readme.md，而本地的没有，所以就需要先将 github 上的 pull 下来。

在执行 git pull 命令后，便出现了一条合并失败的提示。

```bash
fatal: refusing to merge unrelated histories
```

提示的意思是，拒绝合并不相关的历史。

## 解决

Google 了一下后得知，两个仓库（本地和远程）都有 commit，但是却没有相关联的 commit，因此 git 认为用户应该是填错了 origin，两个仓库并无关联。

这个时候只要给命令加个选项便可以解决问题了（<code>--allow-unrelated-histories</code>）。

```bash
git pull origin master --allow-unrelated-historie
```