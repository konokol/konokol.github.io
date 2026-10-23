---
title: Git常用命令
date: 2017-06-08 23:13:00
tags: git
category: tools
description: Git作为一种常用的现代版本管理工具，应用的十分广泛，作为开发者，Git是一项必不可少的技能，掌握常见的Git命令能够大大提高我们的工作效率。这里只介绍最基础的，最常用的命令，配置相关的东西直接略过，git相关的一些概念也不介绍了。在执行下面的命令的时候，假定你已经安装好了git，并且在你的远程git仓库的网站上添加了你的key，建立了安全的连接。

---
*本文出处https://zhuanlan.zhihu.com/p/25415162 或者 http://blog.csdn.net/u013301192/article/details/69568557*

Git作为一种常用的现代版本管理工具，应用的十分广泛，作为开发者，Git是一项必不可少的技能，掌握常见的Git命令能够大大提高我们的工作效率。

这里只介绍最基础的，最常用的命令，配置相关的东西直接略过，git相关的一些概念也不介绍了。在执行下面的命令的时候，假定你已经安装好了git，并且在你的远程git仓库的网站上添加了你的key，建立了安全的连接。以下所有的命令基于Mac OS X 10.12，Git版本2.10.1 (Apple Git-78)。

### 基本命令

#### 下载项目
拿到项目的git地址，比如这个git@github.com:IvanJLee/git.git，新建一个文件夹，把代码下载下来：
```
mkdir LearningGit
cd LearningGit
git clone git@github.com:IvanJLee/git.git
```

等待代码下载完成，LearningGit目录下，多出了一个git文件夹，这个文件夹中就是上面那个仓库的所有文件了。进入git文件夹，查看所有文件，会发现有一个.git文件夹，这就是Git的版本库了。

有时候是在本地新建的项目，想上传到远程的仓库，在项目的根目录下，初始化git仓库：
```
git init
```

查看远程仓库：
```
git remote -v
```

添加一个远程仓库：
```
git remote add origin-name git@github.com:IvanJLee/git.git
```

#### 个人信息的配置
一般一个项目，提交代码的时候都会提前配置好个人信息，以便查看提交的作者信息。没有配置的时候，名字默认是取的是操作系统的当前用户的名字。

查看配置信息：
```
git config --list
```

设置用户名和邮箱（全局的配置加上--global，只是当前项目的话，就不要了，这里一般当前项目没有配的话，就用的是全局的配置。为什么要有两个呢，比如有这样的场景，我的笔记本是公司给配的 ，git的全局配置是真名和公司邮箱，但有时候，我会写一些自己的代码，上传到GitHub上，用的是我的昵称和私人邮箱，所以个人的项目就进行单独的配置）：
```
git config (--global) user.name "Ivan"
git config (--global) user.mail "lijundut@foxmail.com"
```

当然修改配置除了上述方法之外，还可以直接改配置文件，可以改的很多，绝对不止用户名和邮箱，修改当前项目的配置的话直接编辑.git/config文件；修改当前用户的配置，编辑~/.gitconfig；修改整个操作系统的配置，编辑/etc/gitconfig.

#### 提交代码
对项目中的文件作出修改或删除，或者新建文件之后，查看哪些文件有修改：

```
git status
```
![这里写图片描述](https://pic3.zhimg.com/v2-b90a8993bb2181b3699f02dc808e13ca_b.png)
Changes not staged for commit是对项目中已有的文件作出的修改，Untracked files是新建的文件，尚未加到版本库中。

查看具体有哪些修改，查看全部文件的：
```
git diff
```

或者，查看单个文件：

```
git diff README.md
```

将需要提交的文件加到待提交列表（支持正则表达式），比如把README.md加进去：

```
git add README.md
```

或者把所有修改过的文件全部加进去：

```
git add *
```

提交修改到本地暂存区：

```
git commit -m 'my commit message'
```

或者，使用其它的文本编辑器编辑提交信息（输完下面的命令后，会自动跳过去，Mac OX的终端中默认用vim）：

```
git commit
```

提交修改到远程分支（这时候可能别人已经修改了代码，你的push会被拒绝，先pull一下就可以了）：

```
git push
```

获取远程的修改
Git是协同工作的工具，我们自己在修改的同时，别人也在修改，那么获取别人的修改就有了一下的命令。

获取远程仓库的修改：

```
git fetch
```

获取远程仓库的修改，并合并到本地分支：

```
git pull
```

有多个远程仓库的话：

```
git pull origin1
git pull origin2
```

简单来说，pull = fetch + merge，详细区别请看git pull与fetch的区别

#### 分支管理
一般来说，刚拉下来的项目代码都是在master分支上，绝大多数情况下，master分支是受保护的分支，不允许直接提交代码，这也是我们在使用Git时应当注意的。master分支在任何情况下都是禁止直接提交代码的。

查看本地分支：

```
git branch
```

查看远程分支：

```
git branch -r
```

查看全部分支：

```
git branch -a
```

基于当前所在的本地分支创建一个新分支（比如当前在master分支，执行下面的命令后切到了my-develop分支，但在my-develop分支上push代码到远程仓库仍然是提交到master分支上的）：

```
git checkout -b my-develop
```

基于远程分支创建一个新分支（也是创建一个新的分支，但是和上面不同的是，在my-branch上push代码会提交到远程的develop分支上）：

```
git checkout -b origin/develop my-branch
```

切换到本地的另外一个分支上，比如develop-ivan（如果这个分支不存在，会报错，加上参数-b创建新的分支）：

```
git checkout develop-ivan
```

给分支改名：

```
git branch -m old-branch-name new-branch-name
```

把本地分支提交到远程仓库：

```
git push origin -u local-branch-name:remote-branch-name
```

上面的一条命令等同于下面的两条命令：

```
git push origin local-branch-name:remote-branch-name
git branch --set-upstream-to=origin/remote-branch-name local-branch-name
```

删除一个本地分支，比如删除my-branch分支：

```
git branch -d my-branch
```

如果本地有commit，无法删除，删除本地本地分支以及此分支上的commit：

```
git branch -D my-branch
```

删除一个远程分支，比如develop-lee分支(没有直接删除远程分支的命令，使用push命令，本地分支名为空就可以了)：

```
git push origin :develop
```

合并其他分支的代码到当前分支：

```
git merge other-branch-name
```

或：

```
git rebase other-branch-name
```

merge和rebase的功能基本相似，都是合并代码，区别是rebase会把git提交的时间线压平，提交的时间线看起来会更加整洁，但是不建议这么做，一般建议使用merge。git中merge和rebase的区别
查看提交信息
查看当前分支的commit信息：

```
git log
```

查看每次commit修改的文件：

```
git log --stat
```

按关键字筛选commit信息：

```
git log -S keyword
```

按作者筛选commit信息，支持正则：

```
git log --author = "Ivan"
```

#### 暂存代码
有时候，正在自己的分支上开发，突然出现了线上bug，需要去其他分支修复，当前的分支功能又没有开发完毕，你还不想提交未完成的代码，这时候使用stash命令就会很方便。stash命令可以把修改的代码都存在本地，而不commit，之后回来可以恢复之前的修改。常用的stash命令如下：

查看保存的修改：

```
git stash list
```

查看某一个stash修改的具体内容：

```
git stash show -p stash@{1}
```

查看某一个stash修改的文件：

```
git stash show stash@{0}
```

保存当前的修改：

```
git stash save "save message"
```

或者，自动填写stash message：

```
git stash
```

删除某一次stash：

```
git stash drop stash@{1}
```

或者删除最近的一次stash：

```
git stash drop
```

恢复最近一次stash：

```
git stash pop
```

恢复某一次stash：

```
git stash pop stash@{1}
```

删除全部的stash：

```
git stash clear
```

#### 其他常见命令
恢复一个文件的修改（和切换分支一样，也是checkout）:

```
git checkout file_name
```

把提交的文件回滚到某一次提交（先用git log查看提交记录，找到某次提交的sha1值，使用git  reset回到某次提交，其中soft表示保留新的修改但取消git add，mixed表示保留修改和git add，hard表示不保留修改）：

```
git log
git reset --(soft|mixed|hard) 3275a0c85fb2fbbcd6eaa65be1f956d27fa9998b
```

使用git add命令，把文件加到要提交的列表中后，把它待提交列表中移除：

```
git reset --HEAD filename
```

把一个文件从版本库中移除：

```
git rm filename
```

### 更多应用

以上只是Git使用的最基本命令，掌握这些命令，工作中Git操作大部分已经没有问题。当然，Git的功能是很强大的，所能做的事情也远远不止这些，要全面掌握Git的技巧，一两篇文章是说不清楚的。当然，熟悉Git最好的方法就是去实践，学以致用，用的多了自然就记得住。

查看所有的Git命令，可以详细了解每一条命令以及加参数的用法：

```
git --help
```

要想系统学习Git，可以找到的资源有很多，下面是一些学习Git的不错的网站。

Pro git: [git--distributed-is-the-new-centralized](https://git-scm.com/)，简体中文版：[Pro Git](https://git-scm.com/book/zh/v2)

[廖雪峰的Git教程](http://www.liaoxuefeng.com/wiki/0013739516305929606dd18361248578c67b8067c8c017b000)

[learngitbranching以游戏的方式学习Git](http://learngitbranching.js.org/)
