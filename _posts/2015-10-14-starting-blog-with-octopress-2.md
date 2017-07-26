---
layout: post
title:  "基于 Octopress & Github Pages 搭建博客（二）"
date:   2015-10-14 08:37:18 +0800
categories: 技术
tags: Octopress
author: JonyFang
mathjax: true
---

* content
{:toc}

经过[上一篇](https://github.com/JonyFang/dev-notes/blob/master/2015-10-13-starting-blog-with-octopress-1.md)，我们在本地搭出了` Octopress `雏形，这一篇首先我们要将本地的` Octopress `博客部署到` Github Pages `，然后发布一篇博文。部署过程中分析了原理，可能会比较枯燥，但是能让我们更了解` Octopress`，所以我依旧坚持写下来了。





废话少说，开工～


----
## 一.将 Octopress 部署到 Github Pages

Github 大家应该都有了解过，也是我很喜欢的平台之一，功能真心强大并且可免费使用，这里我们拿来托管我们的` Octopress `博客。

### 1.新建 Github repository

注册` Github `账号，新建` Github repository `。项目名称`（Repository name）`命名格式为` username.github.io `，`username `是你的` Github 用户名`（或` organization name`，这里和后面我们先不讨论 origanization）。例如我的用户名是` JonyFang`，所以输入` JonyFang.github.io `即可。点击` Create repository `创建。

PS：创建完后不要添加任何内容，另外自己过程中产生了两个疑问:

- 为什么用` github.io `而不是` github.com`？
- 为什么是` Repository name `一定要按照` username.github.io `填写？

第一个问题，[这里](https://github.com/blog/1452-new-github-pages-domain-github-io#security-vulnerability)解释了为什么把` github.com `改为了` github.io `，简而言之是为了安全。

第二个问题，和` Github `内部的结构有关，其次后面会通过` URL `获取填写的` username.github.io `作为博客域名。这样填写格式与` Github `内部结构的联系具体还需要再研究下。若有大神围观，望指教下 :)

### 2.配置 Github Pages

终端执行如下命令：

```
$ cd octopress
$ rake setup_github_pages
```

该命令会要求我们输入` Github 仓库的 URL `。复制粘贴下我们新建仓库的` SSH `或` HTTPS URL `即可。（例如：`git@github.com:username/username.github.io.git`）

那么这里` rake setup_github_pages `做了什么呢？

用户（users）的 Github Pages 使用` master 分支`作为 Web 服务（web server）的公开目录，为我们的` Pages url （http://username.github.io）`提供内容文件。因此，我们会有这样的需求，`source 分支`用来做与博客源码相关的事（存放全部博客源码），`master 分支`上 commit 生成的博客内容`供 Web 访问`。而 Octopress 帮我们把这件事给搞定了，通过这行 code（好 NB～）。

下面具体分析下` Octopress `是怎么做的（可通过查看` Rakefile `得知）：

- 命令要求我们输入 Github Pages 仓库的 URL，也就是我们新建的名为 username.github.io 仓库的 URL。这样命名是为了通过字符串截取 URL 拿到子串（http://username.github.io）作为我们博客的域名；

```
#Rakefile 中可查看 URL 截取方式;
repo_url = get_stdin("Repository url: ")
```

- 将指向（pointing to）`imathis/octopress 远程库`的名字` ‘origin’ `改为` ‘octopress’`；

`Git clone `一个仓库时，会将 clone 下来的仓库`命名为 origin`，没有限定 clone 条件的情况下，会下载仓库中所有数据，并建立一个指向该仓库` master 分支`的指针，本地命名为` origin/master`。

当我们 clone 了` octopress 仓库`，执行如下命令可看到远程仓库信息：

```
$ cd octopress
$ git remote -v

# 输出如下，可看到远程仓库名为 origin
origin	git://github.com/imathis/octopress.git (fetch)
origin	git://github.com/imathis/octopress.git (push)
```

Octopress 文件夹下` Rakefile `中可看出是通过如下的方式，将` origin `改为` octopress`：

```
#查看 Rakefile
system "git remote rename origin octopress"
```

这里内部执行了命令` git remore rename origin octopress`，当` rake setup_github_pages `执行完毕，再` git remote -v `发现远程库名改为了` octopress`。

```
$ git remote -v

#输出如下
octopress	git://github.com/imathis/octopress.git (fetch)
octopress	git://github.com/imathis/octopress.git (push)
```

- 添加你的` Github Pages `仓库作为默认的` origin remote`，并将远程库中指向` imathis/octopress `中` master 分支`的指针指向现在的` origin`（即 username/username.github.io）的` master 分支`.

```
#查看 Rakefile
system "git remote add origin #{repo_url}"
system "git config branch.master.remote origin"
```

当` rake setup_github_pages `执行完毕查看远程库，可以看到远程库` origin `指向了` Github Pages`。

```
$ cd octopress
$ git remote -v

#输出信息如下
octopress	git://github.com/imathis/octopress.git (fetch)
octopress	git://github.com/imathis/octopress.git (push)

origin	git@github.com:JonyFang/JonyFang.github.io.git (fetch)
origin	git@github.com:JonyFang/JonyFang.github.io.git (push)
```

到这里，应该能猜到上一步将指向` imathis/octopress `远程库的名字` origin `改为` octopress `的原因了

- 将本地 master 分支名字从 "master" 改为 "source"

```
#查看 Rakefile
system "git branch -m master source"
```

执行如下命令查看本地分支（拿到第 6 条解释为什么要改名为 source）：

```
$ git branch

# 输出如下
* source
```

- 根据提供的` Github Pages `仓库的` SSH `或` HTTPS 的 URL`，截取仓库名` username.github.io `作为博客的 URL（上面 1 有提到）。然后将 `octopress` 目录下` _config.yml `文件中` url 参数值`改为 http：//username.github.io

```
#octopress 下 Rakefile 查看

url = blog_url(user, project, source_dir)
jekyll_config = IO.read('_config.yml')
jekyll_config.sub!(/^url:.*$/, "url: #{url}")
File.open('_config.yml', 'w') do |f|
f.write jekyll_config
```

- 在` octopress 目录`下新建` _deploy 目录`，并在` _deploy 目录`下新建 `master 分支`

前面4 ，我们将`本地 master 分支`名字从` master `改为` source `，这里一起分析下原因。4和6放在一起容易理解点。

其实本地 octopress 博客部署到` Github Pages `之后，`远程 Github `下会有两个分支， `master `和` source`。`远程 master 分支`作为 Web 服务公开目录，当你访问` http://username.github.io `时，提供网站内容；`远程 source 分支`存放的是整个 octopress 框架的源码。

之所以第4步将`本地 master `改为` source`，是为了把` master 指针`让出来，让它指向这一步（6）新建的` _deploy 目录`下的` master 分支`。这样，`octopress/_deploy 目录`下的`本地 master 分支 `就对应了` Github Pages `远程库中的 master 分支，本地 source 分支同理。

```
#查看 Rakefile 

cd "#{deploy_dir}" do
system "git init"
system 'echo "My Octopress Page is coming soon …" > index.html'
system "git add ."
system "git commit -m \"Octopress init\""
system "git branch -m gh-pages" unless branch == 'master'
system "git remote add origin #{repo_url}"
```

然后修改了 Rakefile 中 deploy_default 和 deploy_branch 变量的初始值:

```
#deploy 时执行的命令，"push" 为 Rakefile 中定义的一个 rake task

deploy_default = "push" # 初始为 "rsync"

# deploy 时执行上述 rake task 命令 "push" 到 "master" 分支

deploy_branch  = "master" # 初始为 "gh-pages"
```

回头来看` rake setup_github_pages `，是不是清晰多了呢？


### 3.生成并部署站点

执行如下命令，（将` octopress/_deploy `下数据 push 到` master 分支`）:

```
$ sudo rake generate
$ sudo rake deploy
```

这时在浏览器输入` http://username.github.io `就可以访问我们的网页啦～

最后别忘了 commit 你的` octopress 框架源码`到` source 分支`:

```
$ git add .
$ git commit -m'init'  #init 可随意填写
$ git push origin source
```

好，到这里，如果顺利完成前面所有内容的话，我们已经将 Octopress 部署到` Github Pages `了。如果你想换成自己的域名可以参考这里的方法（[Custom Domains](http://octopress.org/docs/deploying/github/#custom_domains)），不再赘述了。

### 4.rake generate 和 rake deploy

这里解释下，`rake generate `和` rake deploy`. 

4.1 rake generate

用于生成 jekyll 站点(enerating Site with Jekyll)

```
#查看 Rakefile

system "compass compile --css-dir #{source_dir}/stylesheets"
system "jekyll build"
```

4.2 rake deploy

用于将站点部署到` Github Pages`。由于` _deploy 目录`所代表的本地仓库的` master 分支`对应 Github Pages 远程仓库的 master 分支，该分支目录的内容即` Github Pages `在互联网上`供公开访问的站点内容`。因此这里做的主要就是将改动的内容（博客、DIY 布局等...） copy 到` _deploy 目录`下，然后将此修改 push 到` Github Pages 远程库`的` master 分支`。

- 查看是否有` preview-mode (预览模式)`，有则删除，重新执行` rake generate`
- 将` octopress/source 目录`下的文件拷贝到` octopress/public 目录`下
- 进入` octopress/_deploy 目录`，执行` git pull `操作
- 将` octopress/public 目录`的内容拷贝到` octopress/_deploy 目录`下
- 将` octopress/_deploy 目录`所对应的`本地 master 分支`的修改 push 到 Github Pages 远程库的 master 分支
- 下面可以看到` master 分支` commit 信息格式是 `"Site updated at 2015-10-14 12:52:36 UTC"`

```
# 查看 Rakefile
system "git add -A"
message = "Site updated at #{Time.now.utc}"
system "git commit -m \"#{message}\""

# 前面说过 deploy_branch  = "master"
Bundler.with_clean_env { system "git push origin #{deploy_branch}" }
```

----
## 二.发布博文过程

### 1.新建一篇博文

打开终端，输入：

```
$ cd octopress
$ rake new_post["Hi, octopress"]  #"Hi, octopress" 是文章的标题
```

然后在` octopress/source/_posts 目录`下我们会看到命名格式为` "2015-10-14-Hi-octopress.markdown" `的文件。

```
#查看 Rakefile ，这是实现方法
filename = "#{source_dir}/#{posts_dir}/#{Time.now.strftime('%Y-%m-%d')}-#{title.to_url}.#{new_post_ext}"
```

打开新建的 "2015-10-14-Hi-octopress.markdown" 文件（我用的[ Mou](http://mouapp.com/)，免费还好用），会发现头文件有如下内容（不要删除这段信息）：

```
---

layout: post             #代表是一片博文

title: "hello world" 

date: 2015-10-14 19:59:22 +0800

comments: true         #是否允许评论

categories:             #分类

---
```

头部布局实现原理：

```
#查看 Rakefile，头部布局实现

open(filename, 'w') do |post|

post.puts "---"

post.puts "layout: post"

post.puts "title: \"#{title.gsub(/&/,'&')}\""

post.puts "date: #{Time.now.strftime('%Y-%m-%d %H:%M:%S %z')}"

post.puts "comments: true"

post.puts "categories: "

post.puts "---"
```

在最后面的` --- `下面就可以开始我们的正文啦～

### 2.预览新建的博文

终端执行如下命令：

```
$ cd octopress
$ sudo rake generate
$ rake preview
```

然后浏览器打开[ http://localhost:4000 ](http://localhost:4000),可以预览我们刚写的博客效果。

### 3.发布新建的博文

终端执行:

```
$ sudo rake deploy  #deploy blog 到 Github Pages
```

最后别忘了，`push `下本地的 octopress `到 source`：

```
$ git add .
$ git commit -m'post test blog'
$ git push origin source
```

OK，到此，本地 octopress 博客部署到 Github Pages 完成了，打开访问你的个人博客看看效果吧。

部署这一块，我花了挺长时间研究的，算是弄明白了过程是怎么回事。回头再看 Github 上的目录和本地的目录，一下子明朗了不少。不知道你是不是也觉得呢？

----

欢迎转载，但请注明出处. [https://github.com/JonyFang/dev-notes](https://github.com/JonyFang/dev-notes)







