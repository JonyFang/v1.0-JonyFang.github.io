
---
layout: post
title:  "基于 Octopress & Github Pages 搭建博客（一）"
date:   2015-10-13 08:37:18 +0800
categories: 技术
tags: Octopress
author: JonyFang
mathjax: true
---

* content
{:toc}

一直以来想有个属于自己的博客空间，或许是出于一种归属感吧。就这样知道了` WordPress`、`Jekyll`、`Hexo `和 `Octopress`。一番对比后选择了` Octopress`，相信追随大神的脚步应该不会错。`Octopress `接触有一个多星期了，这里总结下基于 Octopress 及 Github 搭建博客的过程及自己中间遇到的一些问题的解决办法。技术上不一定完全精确，若有大神围观望指正：)

使用的是` Mac OS X `系统，不一定适用于` Windows `的童鞋。（勿拍砖...）






----
## 一.Octopress 与 Jekyll & Github Pages 的关系

`Octopress `是基于` Jekyll `的静态博客框架。

`GitHub Pages `这里用于显示托管在` GitHub `上的静态网页，是` GitHub `提供的一项服务。

总的来说也就是我们使用基于` Jekyll `的` Octopress `生成本地的静态网站，然后将静态的网站托管到` Github `为我们提供的` Github Pages `服务上。最后访问**博客地址**就可以显示我们的个人博客网站了。


----
## 二.准备工作

### 1.安装 Git

![Install Git](https://github.com/JonyFang/dev-notes/blob/master/images/2015-10-13-git.png?raw=true)


[点击这里](http://git-scm.com/)前往 Git 官网，按下图提示下载安装（一般 Mac OS X自带 Git 环境，终端执行如下命令，可查看 Git 版本）。

```
$ git -v
```

### 2.安装 Ruby

这是` Ruby `官网，这里就不详细介绍` Ruby `啦，感兴趣的话可以了解下。好吧，回到` Ruby `的安装。

打开终端，执行如下命令，安装` RVM`，同时也会安装最新的` Ruby`：

```
$ curl -L https://get.rvm.io | bash -s stable --ruby
```

安装完成，执行如下命令，查看` Ruby `版本

```
# -v == --version
$ ruby -v
```

如果你的` Ruby `版本不低于` 1.9.3`，可直接跳转到 安装` RubyGems`。否则需要执行如下命令

```
$ rvm install 2.0.0
$ rvm use 2.0.0
```

安装` RubyGems`：

```
$ rvm rubygems latest
```

现在我们再执行命令` ruby -v `查看 Ruby 版本，会看到现在已经是` 2.0.0 `了。

呼，准备工作搞定！


----
## 三.本地安装 Octopress

前面做了那么多准备，主角总算要上场了。

首先，将 Octopress 的项目` clone `到本地，终端执行如下命令：

```
$ git clone git://github.com/imathis/octopress.git octopress
```

进入 octopress 目录：

```
$ cd octopress
```

下面安装 Octopress 所需要的`依赖库（dependencies）`.安装过程中可能会遇到权限问题，我们需要在命令前面加上` sudo `再执行，并输入登录密码。

```
#sudo 全称：super user do，也就是以 root 用户身份来执行
$ sudo gem install bundler
$ bundle install
```

这里在不翻墙的情况下，可能会遇到一个问题：`sudo gem install bundler `执行后，一直没有响应。这是由于国内网络原因（你懂的），导致[ rubygems.org](http://rubygems.org/)存放在` Amazon S3 `上面的资源文件间歇性连接失败。所以你会遇到` gem install rack `或` bundle install `的时候半天没有响应的情况。

幸运的是国内某开发者帮我们解决了这一心头大患，我们可以用`淘宝的 Ruby 镜像`来替换`原来的镜像`。只需终端执行下面的命令即可：

```
$ gem sources -a https://ruby.taobao.org/ -r https://rubygems.org/

#下一命令查看切换后结果
$ gem sources -l
```

然后会看到这样的输出：

```
*** CURRENT SOURCES ***

https://ruby.taobao.org
```

这就说明我们切换到`淘宝的 Ruby 镜像`了，再次`安装 Octopress `所需要的依赖库就会发现成功啦。

当然还有另外两种方法:

- 比较原始的方法——手动更改：`打开 octopress 文件夹` -> `打开 Gemfile 文件` -> `将 source "https://rubygems.org" 改为 source "https://ruby.taobao.org" `就可以了。
- 相对方便点，因为我们使用的是` Gemfile`，所以我们可以用` Bundler `的` Gem 源代码镜像命令`，这样我们就不用改` Gemfile `的` source `了。

```
$ cd octopress
$ bundle config mirror.https://rubygems.org https://ruby.taobao.org
```

最后安装下默认主题

```
#rake 全称：ruby make
$ rake install
```

----
## 四.预览效果

好，经过上面的功夫，我们已经在本地搭建了一个简易版的` Octopress `博客。

我们来看看效果吧。在终端执行命令：

```
$ sudo rake preview
```

打开浏览器，输入 [http://localhost:4000/](http://localhost:4000/)，就可以看到效果了。虽然比较简陋，但让人挺高兴的，你觉得呢？

![Preview](https://github.com/JonyFang/dev-notes/blob/master/images/2015-10-13-octopress-preview.png?raw=true)

至此我们算是结束了本地安装过程，下一篇我们会把本地的 Octopress 部署到 Github，那么下篇再见喽～


----
> 本篇参考
>
> 《Octopress Setup》: [http://octopress.org/docs/setup/](http://octopress.org/docs/setup/)
>
> 《Gem Source Mirrors》: [http://bundler.io/v1.5/bundle_config.html#gem-source-mirrors](http://bundler.io/v1.5/bundle_config.html#gem-source-mirrors)


----

欢迎转载，但请注明出处. [https://github.com/JonyFang/dev-notes](https://github.com/JonyFang/dev-notes)


