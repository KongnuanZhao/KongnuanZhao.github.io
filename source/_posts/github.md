title: hexo博客同步到github和gitcafe(下)
date: 2015-03-17 13:59:51
author: 赵空暖
tags: github
categories: github
---

郁闷一下午提交一直是`Everything up-to-date`,要不就是
```bash
git status
n branch master
nothing to commit, working directory clean
git push gitcafe gitcafe-pages
verything up-to-date
```
先记录一下今天遇到的问题：
使用hexo写博客源文件都是在本地的，如果换了电脑需要写博客时就会比较麻烦。
我们需要在电脑上新建一个文件夹如:111,然后进入这个文件夹克隆创建仓库。
```bash
$ git clone git@github.com:KongnuanZhao/KongnuanZhao.github.io.git
Cloning into 'KongnuanZhao.github.io'...
remote: Counting objects: 6496, done.
remote: Compressing objects: 100% (166/166), done.
remote: Total 6496 (delta 112), reused 1 (delta 1), pack-reused 6227
Receiving objects: 100% (6496/6496), 23.61 MiB | 192.00 KiB/s, done.
Resolving deltas: 100% (2665/2665), done.
Checking connectivity... done.
```
设置远程仓库地址，
```bash
$ git remote add origin git@github.com:KongnuanZhao/KongnuanZhao.github.io.git
```
`origin`可以改为别的名字，相应的改变在`.git`文件下的`Config`配置文件
```bash
[core]
	repositoryformatversion = 0
	filemode = false
	bare = false
	logallrefupdates = true
	symlinks = false
	ignorecase = true
	hideDotFiles = dotGitOnly

[remote "origin"]
	url = git@github.com:KongnuanZhao/KongnuanZhao.github.io.git
	fetch = +refs/heads/*:refs/remotes/origin/*
```
如果要是执行`git remote add gitcafe git@gitcafe.com:KongnuanZhao/KongnuanZhao.git`,该配置文件就会添加：
```bash
[remote "gitcafe"]
	url = git@gitcafe.com:KongnuanZhao/KongnuanZhao.git
	fetch = +refs/heads/*:refs/remotes/gitcafe/*
```
查看你当前项目远程连接的是哪个仓库地址:`git remote -v`
```bash
$ git remote -v
origin  git@github.com:KongnuanZhao/KongnuanZhao.github.io.git (fetch)
origin  git@github.com:KongnuanZhao/KongnuanZhao.github.io.git (push)
```
之后，当写博客后，只需要`git add .`，将所有文件提交到缓存区。
`git commit -m "add all files"` ，将这些文件提交到本地仓库。
`git push origin master`，将本地仓库的改动推送到github仓库。
当远程仓库有更新时，使用git pull或者git fetch就可以同步代码到本地了。
从远程库抓取信息,然后将远端分支自动合并到本地仓库中当前分支。这么用，既快且好。默认情况下`git clone`命令本质上就是自动创建了本地的`master`分支用于跟踪远程仓库中的 master 分支。所以一般我们运行`git pull`，目的都是要从原始克隆的远端仓库中抓取数据后，合并到工作目录中的当前分支。
```bash
$ git pull origin
Already up-to-date.
```

再就是今天莫名其妙的发现不能同时提交代码到github和gitcafe上了，没办法，只能记录下下面这条命令喽。
先查看远程仓库地址
```bash
$ git remote -v
gitcafe git@gitcafe.com:KongnuanZhao/KongnuanZhao.git (fetch)
gitcafe git@gitcafe.com:KongnuanZhao/KongnuanZhao.git (push)
origin  git@github.com:KongnuanZhao/KongnuanZhao.github.io.git (fetch
origin  git@github.com:KongnuanZhao/KongnuanZhao.github.io.git (push)
```
到hexo根目录下,执行`hexo d -g`,
到`\Hexo\.deploy`目录底下，执行（现在也不需要切换分支也能提交啦）
`git push -f gitcafe master:gitcafe-pages`

还有个问题，之前自己用的命令是：
```bash
$ git push gitcafe gitcafe-pages:gitcafe-pages
Everything up-to-date
```
就一直说Everything up-to-date，不知道为什么，还是需要进一步理解github原理。

记录一下经常用到的命令：
git branch 查看本地所有分支
git status 查看当前状态 
git branch -a 查看所有的分支
git branch -r 查看远程所有分支
git commit -m "init" 提交并且对你更新或修改了哪些内容做一个描述 
git remote add origin git@github.com:KongnuanZhao/KongnuanZhao.github.io.git 增加一个远程服务器端（这是你本地的当前的项目与远程的哪个仓库建立连接。origin可以改为其它的名字，但是在你下一次push时，也要用修改之后的名字。）
git push origin master 将文件给推到服务器origin的master分支上
git add .   //（.）点表示当前目录下的所有内容，交给git管理，也就是提交到了git的本地仓库。（git的强大之处就是有一个本地仓库的概念，在没有网络的情况下可以先将更新的内容提交到本地仓库。）
git fetch origin    //取得远程更新，这里可以看做是准备要取了
git merge origin/master  //把更新的内容合并到本地分支/master
git checkout test 切换分支到test
git branch -D test 删除本地test分支
git push origin --delete <branch> 删除远程分支（可以使用这种语法，推送一个空分支到远程分支，其实就相当于删除远程分支：git push origin :<branch>）
git push -f gitcafe master:gitcafe-pages  把本地分支master提交到gitcafe库的gitcafe-pages上。