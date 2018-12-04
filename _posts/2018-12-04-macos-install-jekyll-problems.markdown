---
layout: post
title:  "MacOS 安装 Jekyll 踩过的坑"
date:   2018-12-04 09:37:44 +0800
categories: jekyll update
---

在 MacOS 上做开发一直是很惬意的一件事情，因为是 Unix-like 系统在环境的搭建方面有着很多便利，但在安装 Jekyll 时却遇到了不少坑，这里记录下。

## 一、安装

安装之前先要确认下安装的环境，需要先安装 Xcode、Ruby，虽然 MacOS 自带 Ruby，但版本比较低（2.3.0），所以我还是手动安装了一个最新版的 Ruby：

```
brew install ruby
```
并配置环境变量，`.bash_profile`：

```
export PATH=$PATH:/usr/local/Cellar/ruby/2.5.3_1/bin
```

接下来按照官方文档执行安装命令：

```
sodu gem install bundler jekyll
```

这个时候出现报错：
```
ERROR:  Error installing jekyll:
ERROR: Failed to build gem native extension.
......
```
查了下 [stackoverflow](https://stackoverflow.com/questions/10725767/error-installing-jekyll-native-extension-build)，是缺少 Xcode-select 工具，可以用以下命令安装：
```
xcode-select --install
```
再次执行 jekyll 安装命令，安装完成。

## 二、运行

安装好 jekyll 后习惯性的运行 `jekyll -v` 命令来查看是否安装成功，发现报错：

```
-bash: jekyll: command not found
```
根据经验应该是环境变量没配好，查了下文档（对 Ruby 确实不太了解，囧），可以用 `gem env` 命令查看 gem 包的安装目录，然后手动添加了 jekyll 的环境变量，`.bash_profile`：

```
export PATH=$PATH:/usr/local/Cellar/ruby/2.5.3_1/bin:/usr/local/lib/ruby/gems/2.5.0/gems/jekyll-3.8.5/exe
```

运行 `jekyll -v` 返回 `jekyll 3.8.5`，说明配置成功，接着按照文档执行项目初始化命令：

```
jekyll new my-awesome-site
```
运行过程中再次报错：

```
Installing eventmachine 1.2.7 with native extensions
Gem::Ext::BuildError: ERROR: Failed to build gem native extension
```

原以为只是简单的安装失败，所以按照提示，手动安装了 eventmachine：
```
gem install eventmachine
```
再次运行项目初始化命令，但还是报错，于是查看详细的错误日志，发现 eventmachine 编译需要 Ruby 2.4 以上版本，而我的 Ruby 编译是 2.3.0，2.3.0 应该是 MacOS 自带的版本，这就非常奇怪了，我明明安装了 2.5.1 版本，配置也OK（which ruby），并且手动安装 eventmachine 也成功了（gem list），怎么这里却是是用的老版本的 Ruby...，查了不少资料后还是无法解决，最后决定安装 rvm 试试能否曲线救国。

## 三、安装 RVM

[RVM](https://rvm.io/) 是 Ruby 的版本管理器，因为 rvm 安装需要安装公钥而 Mac OS X 不附带 gpg，所以先安装 gpg：
```
brew install gnupg
```
接着安装 mpapis 公钥
```
gpg --keyserver hkp://keys.gnupg.net --recv-keys 409B6B1796C275462A1703113804BB82D39DC0E3 7D2BAF1CF37B13E2069D6956105BD0E739499BDB
```
提示`公钥服务器不可用`，想了想难道是被墙了？于是为终端设置了下代理`export http_proxy=sock5://127.0.0.1:1080`，安装完成，如果没有梯子或者设置有问题，可以尝试以下命令：
```
curl -sSL https://rvm.io/mpapis.asc | gpg2 --import -
```
最后安装 rvm：
```
\curl -sSL https://get.rvm.io | bash -s stable
```
需要注意的是安装过程中某些域名还是需要梯子才能访问，然后 rvm 会自动安装最新版本的 Ruby，全部安装完成后可以执行 `rvm list` 命令查看当前所用 Ruby 版本，可以看到是最新的 2.5.1：
```
=* ruby-2.5.1 [ x86_64 ]
# => - current
# =* - current && default
#  * - default
```
再次尝试 jekyll 生成新的项目：
```
jekyll new my-awesome-site
```
执行成功！

至此 jekyll 安装完成，基本上能踩的坑都已经踩了吧。。。相比熟悉的 NodeJS 确实要折腾很多。