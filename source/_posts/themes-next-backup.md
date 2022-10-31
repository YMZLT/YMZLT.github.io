---
title: hexo+NexT主题配置备份及还原
top: false
cover: false
toc: true
mathjax: false
date: 2022-10-31 14:50:24
password:
summary:
tags:
	- 备份
categories:
	- hexo
---



第三次搭建github pages+hexo+NexT 主题的博客了，之前总是因为各种原因中途放弃，这次一定要坚持！这里记录一下备份及今后还原的流程，方便之后的维护。

<!--more-->

# 备份说明

由于博客发布的远程仓库的 `master` 分支是经过编译的静态网页文件，而我们需要保存源文件、更改后的配置等以进行新机器上的还原。我对备份的要求就是：

- 备份站点配置文件
- 备份主题配置文件
- 能方便地更新NexT主题（所以最好是能保留原来的.git文件方便拉取官方最近的更新）

一般的做法是使用 `master` 分支展示博客页面，再创建另一个分支用来备份源文件，这样就可以将文件都备份在一个仓库下。

但是实际操作中发现，因为使用的是最新的[NexT  Release v7.8.0](https://github.com/theme-next/hexo-theme-next) 版本，theme主题上传到GitHub时，只有主题名，主题文件夹下并没有主题对应文件，所以说这种方式只能备份站点配置文件，主题配置文件无法备份。这部分往往是最重要的，因为包括了很多自己定义的配置。如果直接删除themes/next文件夹下的.git文件，那么任何自定义的修改都会使合并上游的改动变得困难。

对此我找到了一种比较优雅的策略：使用子模块（ submodule ）进行多个仓库的嵌套。

> 子模块允许你将一个 Git 仓库作为另一个 Git 仓库的子目录。 它能让你将另一个仓库克隆到自己的项目中，同时还保持提交的独立。

# 开始备份

> **为防止备份过程中出现失误，请先提前复制一份自己的博客文件作为备用，一旦失败可以删档重来。**

## 站点仓库（公开）

站点仓库使用两个分支，一个`master`分支存放博客静态页面文件，另一个`source`分支用来备份站点配置的源文件。配置步骤如下：

1. 先把themes目录下自己定义过的主题文件删除，防止后面出现子模块冲突。(这一步非常重要，不然后面还得删掉重来)

2. 在博客目录下初始化本地 git 并添加远程仓库

   ```shell
   git init
   git remote add origin https://github.com/your_name/your_name.github.io.git
   ```

3. 创建 `source` 分支并推送源文件

   ```shell
   git checkout -b source
   git add .
   git commit -m "Initial backup"
   git push origin source:source # 这里 source source 会在远端自动创建分支
   ```

4. 查看自己的github仓库是否已经推送成功

5. 在每次修改博客之后，需要做 deploy 到 `master` 分支和 push 到 `source` 分支两件事

   > 在 github仓库的 settings 中将 source 分支改为默认分支，可以简化为 `git push`

   ```shell
   hexo d
   git add .
   git commit -m "your comment"
   git push origin source:source
   ```

## 主题仓库（私有）

1. 在 github 中创建一个私人仓库 `hexo-theme-next`，用于存放主题配置文件。私人仓库是为了保存主题中的隐私文件。

2. 从原来的主题仓库中将代码 clone 下来，初始化远端的私人仓库，这里以NexT仓库为例：

   ```shell
   cd themes
   git clone https://github.com/theme-next/hexo-theme-next.git
   git push --mirror https://github.com/your_name/hexo-theme-next.git #该仓库作为子模块
   ```

3. 将一个已初始化的主题私人仓库添加为站点仓库的子模块

   ```shell
   git submodule add https://github.com/your_name/hexo-theme-next.git themes/next
   ```

4. 检查子模块是否创建成功

   ```shell
   $ git status
   ...
   Changes to be committed:
     (use "git reset HEAD <file>..." to unstage)
   
   	new file:   .gitmodules # 出现.gitmodules 文件即代表创建成功了
   	new file:   themes/next
   ```

5. 进入新创建的themes/next主题文件夹，为仓库添加 `upstream`，用于同步源仓库的更新

   ```shell
   $ cd themes/next
   $ git remote add upstream https://github.com/theme-next/hexo-theme-next.git
   $ git remote -v   # 查看配置
   origin  https://github.com/your_name/your_name.github.io.git (fetch)
   origin  https://github.com/your_name/your_name.github.io.git (push)
   upstream        https://github.com/theme-next/hexo-theme-next.git (fetch)
   upstream        https://github.com/theme-next/hexo-theme-next.git (push)
   ```

   这样就可以进入到目录中运行 `git fetch` 与 `git merge`来合并上游分支来更新本地代码。同步操作如下：

   ```shell
   $ git fetch upstream master
   From https://github.com/theme-next/hexo-theme-next
    * branch            master     -> FETCH_HEAD
   $ git merge origin/master
   ```

6. 推送更改到站点仓库

   ```shell
   git add .
   git commit -m "add submodule"
   git push origin source:source
   ```

到此为止我们已经成功创建并配置好了两个备份仓库，但是主题私人仓库中储存的仍然是原始文件，下面还要把自定义的配置备份一下：

## 备份主题配置

1. 将自己修改后的主题文件覆盖themes/next下的原始文件（注意不要把.git文件复制过来，构建子模块后themes/next目录中不存在.git文件）
2. 

# 还原

1. 先安装好 git、ssh、node、hexo

2. 使用 `git clone` 克隆 `source` 分支的源文件，`--recursive` 参数可以同步还原 submodule，相当于我们只用一条命令就克隆了两个仓库。

   > 如果已将 `source` 分支设置为默认分支，可省略 `-b source` 参数；

   ```shell
   git clone -b source https://your_name/your_name.github.io.git --recurse-submodules
   ```

   如果你已经克隆了项目但忘记了 `--recurse-submodules`，也可以在下载完仓库后，使用`git submodule init` 和 `git submodule update` 进行子模块的还原。

   ```shell
   git clone -b source https://your_name/your_name.github.io.git
   git init
   git submodule init
   git submodule update
   ```

   

3. 安装依赖

   ```shell
   npm install
   ```

   

# 参考链接

[Github Pages+hexo+NexT 博客搭建、备份及还原](https://quareia.github.io/blog/6efb1d64/)

[Git 工具 - 子模块](https://git-scm.com/book/zh/v2/Git-%E5%B7%A5%E5%85%B7-%E5%AD%90%E6%A8%A1%E5%9D%97)