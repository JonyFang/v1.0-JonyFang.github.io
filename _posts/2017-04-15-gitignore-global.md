---
layout: post
title:  "Git 全局忽略 .DS_Store"
date:   2017-04-15 08:37:18 +0800
categories: 技术
tags: Git
author: JonyFang
mathjax: true
---

* content
{:toc}

Mac OS X Finder 进入的每个目录都会有个名为` .DS_Store `的文件, 用于存储当前文件夹的一些详细的` metadata `信息，例如文件夹的排序等信息。在每次提交代码时，我们需要去忽略这类文件的提交。那怎么在` git commit `过程中忽略这种类型的文件呢？





## Git Repo 内忽略文件

可以通过在` .gitignore `文件中忽略` .DS_Store `文件.

```bash
## Mac OS X
.DS_Store
```

## 本地 Git Config

首先，通过如下命令查看现有 Git 配置信息：

```bash
git config --list
```

创建` /.gitignore `文件，并在文件内添加需要全局忽略的文件名。例如忽略` .DS_Store `文件.

```bash
#创建 ~/.gitignore 文件
$ cd ~
$ touch .gitignore
$ vim .gitignore
```

在` .gitignore_global `文件中添加如下信息(忽略 .DS>Store 文件)：

```bash
# Mac OS X
.DS_Store
```

保存添加的信息，并在` ~/.gitconfig `中引入` .gitignore `。执行如下命令：

```bash
git config --global core.excludesfile ~/.gitignore
```

执行完毕之后，再查看下 Git 配置信息，执行如下命令:

```bash
core.excludesfile=/.gitignore
```

之后即不用在每个新的` Git Repo `的` .gitignore `文件中忽略` .DS_Store `了。如果是开源项目，其实自己更偏爱在每个` Git Repo `的` .gitignore `文件中也添加上需要忽略的文件文件名，

----

欢迎转载，但请注明出处. [https://github.com/JonyFang/dev-notes](https://github.com/JonyFang/dev-notes)
