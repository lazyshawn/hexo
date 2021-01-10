---
title: 整理Git仓库
tags: [Git, Hexo]
categories: 学习笔记
date: 2020-03-02 10:09:11
---

> 重新整理博客的Git仓库。把主题分离成一个新的仓库，主题和博客分别更新。

<!--more-->

## 前言
为了能更好的管理博客和主题仓库，我觉得还是分成两个Git仓库比较合适。本篇博客主要讲述如何从仓库中分离一个新仓库，同时在两个仓库中只保留与本仓库有关的commit。为了讲述方便，现做如下定义：
* 旧仓库路径: `<pre-repo>`;
* 需要分离出的子文件夹名称: `<folder>`;
* 建立的新仓库: `<new-repo>`;


## 分离themes仓库
分离仓库有两种方法：简单的是使用`git subtree`，麻烦点的是使用`git filter-branch`。

### Git subtree
自从 1.8 版本之后 git 就添加了 subtree 子命令，使用这个新命令我们可以很简单高效地解决这个问题。首先进入`<pre-repo>`，运行：
```git
git subtree split -P <folder> -b <temp-branch>
```
其中`<temp-branch>`是一个临时分支。运行后，git会遍历原仓库中所有的历史提交，筛选出与指定路径有关的commit并存入`<temp-branch>`。然后就简单了，只需要在别的文件夹pull这个临时分支，就能将子仓库clone过去。运行：
```shell-session
mkdir <new-repo>
cd <new-repo>
git init
git pull <pre-repo> <temp-branch>
```

### Git filter-branch
这是一个git传统的核弹级大杀器，慎用。首先，clone一份原仓库并删掉其remote：
```git
git clone <pre-repo> <new-repo>
cd <newo-repo>
git remote rm origin
```
然后运行如下命令，遍历所有历史提交，只保留对指定子目录有影响的提交，并将该子目录设为仓库的根目录：
```git
git filter-branch --tag-name-filter cat --prune-empty --subdirectory-filter <folder> -- --all
```
其中各参数的作用为：
* `--tag-name-filter`: 控制如何处理旧的tag，cat表示原样输出；
* `--prune-empty`: 删除因为重写导致空了的commit，比如修改的文件全部被删除；
* `--subdirectory-filter`: 指定子目录路径；
* `-- -- all`: 对所有分支进行操作，当然如果你只想保留当前分支，也可以不加此参数。

该命令执行完成后，可以看到新仓库中已经变成原来子仓库的内容，也只保留了相关的提交历史。不过这时新仓库中的`.git`目录中还是保留了不少无用的object，我们需要将其清除以减小新仓库的体积（方法一则不需要执行这一步）。运行如下命令：
```git
git reset --hard
git for-each-ref --format="%(refname)" refs/original/ | xargs -n 1 git update-ref -d
git reflog expire --expire=now --all
git gc --aggressive --prune=now
```
### 关联原仓库与新仓库
这一步是可选的。一般来说，在我们把原目录中的子文件夹分离成独立的 git 仓库后，总会希望再通过某种方法在原仓库中引用新仓库的代码。这里我们可以通过`subtree`或`submodule`两种方法来实现，不过各有优缺点，可以视情况选择，不过现在一般都推荐使用subtree。当然，你也可以分离之后直接使用 npm、composer 之类的包管理器将新仓库作为一个依赖库引入进来，这也是完全没有问题的。


## 清理旧仓库
既然将子目录分离成了新仓库，那么或许你还想要将子目录从原仓库的`.git`中删除。在原仓库路径下运行如下命令：
```git
git filter-branch --force --prune-empty --index-filter 'git rm -rf --cached --ignore-unmatch <folder>' --tag-name-filter cat -- --all
```
其中各参数的作用为：
* `--force`: 让git即使遇到冲突也强制执行；
* `--index-filter`: 指定重写的时候执行的命令，要执行的命令紧跟其后；
* `git rm -rf --cached --ignore-unmatch <folder>`: 让git删除匹配到的缓存文件。

执行完成后，本地仓库已经删除了子目录及其相关的提交历史。最后提交到远程即可：
```git
git push --force --all
```

## 相关链接
1. [如何将现有 git 仓库中的子目录分离为独立仓库并保留其提交历史 **By** 瑜小瑜](http://varyu.com/notes/527.html)
2. [彻底删除git中没用的大文件 **By** Phelthas](https://www.jianshu.com/p/780161d32c8e)
