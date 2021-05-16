---
title: 从 svn 迁移到 git，并保留 commit 日志
date: 2018-11-09 15:22:13
tags: 
  - Git
  - Svn
---

之前为公司做了个基于 yeoman 的脚手架工具，公司是使用 svn 做版本控制的，所以这个工具也就使用了 svn 来记录版本。

近期想做个迁移，把它放到 github 上去，这里对迁移过程做个简单的记录。

首先，svn 地址是 http://svn.com.cn/svn/generator-pczt/ （非真实 svn 地址，这里做个举例）

该项目位置在 svn  中的 base repository，因此不涉及到 tags，branches，trunk。


## 用户映射
按照网上文章，先创建个文件（users.txt），做个用户映射，将 svn 的用户和 git 上的用户关联起来。

```txt
chenwudong = vdorchan <vdorchan@gmail.com>
```

## git svn clone

按照文档中的命令

```bash
# --stdlayout 跟踪标准的Subversion存储库
# -authors-file 指定用户映射的文件

git svn clone --stdlayout --no-metadata -authors-file=users.txt http://svn.com.cn/svn/generator-pczt/ generator-pczt
```

输入上述的命令后，在 <code>generator-pczt</code> 中创建了一个空的 git 仓库，但并没有将文件从 svn 拉下来，并且命令行输出了以下的一些信息。

```bash
Initialized empty Git repository in /Users/vdorchan/Documents/www/Learn-Yeoman/generator-pczt/.git/
Using higher level of URL: http://svn.com.cn/svn/generator-pczt => http://svn.com.cn/svn
W: Ignoring error from SVN, path probably does not exist: (160013): Filesystem has no item: File not found: revision 100, path '/generator-pczt'
W: Do not be alarmed at the above message git-svn is just searching aggressively for old history.
This may take a while on large repositories
```

## git svn init 和 git svn fetch

然后阅读文档得知，git svn clone 相当于 执行力了 git svn init 和 git svn fetch，于是试了下这两个命令。

```bash
git svn init http://svn.com.cn/svn/generator-pczt/ --no-metadata
```

```bash
git svn fetch --authors-file=users.txt
```

这两行命令执行完之后，成功的将文件从 svn 上拉了下来，并且使用 git log 可以查看到正确的提交日志。