---
title: git简介
category: linux
tags:
- linux
- git
---

## 介绍

本文将对git的基本原理和常用命令进行介绍。


## Git是什么

Git(the stupid content tracker) 是一款免费、开源的分布式版本控制系统


## Git的特点

* 直接记录快照，而非差异比较
* 几乎所有操作都是本地执行
* 时刻保持数据完整性（SHA-1 算法）

<!--more-->

## Git对象模型

在Git系统中有四中类型的对象，所有的操作都是基于这四种类型的对象:

* blob: 这种对象用来保存文件的内容。

* tree: 可以理解成一个对象关系树，它管理一些tree和blob对象。

* commit: 指向一个tree，它用来标记项目某一个特定时间点的状态。包括关于时间点的元数据，如时间戳、最近一次提交的作者、指向上次提交

* tag: 给某个提交增添一个标记。


## Git工作流

本地仓库由 git 维护的三棵“树”组成：

* 第一个是 工作目录，它持有实际文件

* 第二个是 缓存区（Index），它像个缓存区域，临时保存你的改动

* 第三个是 HEAD，指向你最近一次提交后的结果


## Git存储

```
[renyl@localhost .git]$ tree -L 1
.
├── branches
├── COMMIT_EDITMSG
├── config
├── description
├── FETCH_HEAD
├── HEAD
├── hooks
├── index
├── info
├── logs
├── objects
├── ORIG_HEAD
├── packed-refs
└── refs

6 directories, 8 files
[renyl@localhost .git]$ tree -L 1
```

说明:
* config: 这个是git仓库的配置文件

* logs: 保存所有更新的引用记录

* hooks: 可以设置特定的git命令后触发相应的脚本

* HEAD: 这个文件指向了一个分支的引用

* objects: 所有的Git对象都会存放在这个目录中，对象的SHA1哈希值的前两位是文件夹名称，后38位作为对象文件名

* refs: 这个目录一般包括三个子文件夹，heads、remotes和tags (git show-ref --heads)

* index: 这个文件就是我们前面提到的暂存区，是一个二进制文件 (git ls-files --stage)


## Git常用命令

### init

* Client:

```
mkdir my_project	# 创建仓库
cd my_project	 
git init	 
```

* Server:

```
mkdir my_project2	# 创建裸仓库，裸仓库没有工作区
cd my_project2	 
git init –bare sample.git	 
chown –R git:git sample.git	 
```

### clone

```
git clone https://github.com/kulong0105/mutt.git
git clone git@github.com:kulong0105/mutt.git
```

注:
* https协议,默认情况下,只能在本地看到master分支，可以使用git branch命令查看
* ssh支持的原生git协议,会自动在本地分支与远程分支之间，建立一种追踪关系（tracking),
  所有本地分支默认与远程主机的同名分支，建立追踪关系，也就是说，本地的master分支自动追踪rigin/master分支。


### checkout

```
git checkout -b dev              #创建新的dev分支
git checkout -b dev  origin/dev	 #创建远程origin的dev分支到本地。
git checkout  -- hello.c         #放弃工作区的修改，让这个文件回到最近一次git commit或git add时的状态。
git checkout $commit -- hello.c	 #回撤到$commit版本下hello.c文件的内容
```


### branch

```
git branch                      #查看本地分支
git branch -r                   #查看远程分支
git branch -vv                  #产看本地和远程分支的track
git branch -d  my_branch        #删除分支
git branch -D  my_branch        #强行删除还未合并的分支
git branch -u origin/dev dev    #设置本地dev分支track远程dev分支(远程dev分支必须要存在）
git branch -d -r origin/test    #删除本地的远程分支
```


### config

```
git config --global user.name "kulong0105"             #global是全局参数，即该命令在这台电脑的所有Git仓库下都有用。
git config --global user.email kulong0105@gmail.com
git config --global color.ur true                      #显示颜色，会让命令输出看起来更醒目
git config --global alias.st status                    #配置别名
git config format.subjectPrefix "PATCH MY_REPO"
git config --list
```


### add

```
git add hello.c     #添加文件到缓存区
git add -p          #允许你交互地选择你想要提交的内容
git add -u
git add --all
```

### rm

```
git rm test.txt  	#删除文件
```


### diff

```
git diff readme.c            #查看工作区和缓存区中的区别
git diff HEAD -- hello.c     #查看工作区和版本库里面最新版本的区别
git diff 0cbba12 -- hello.c  #查看工作区和版本库里"ocabba12"的区别
git diff $branch $filename   #查看branch分支与当前分支$filename文件的区别
Git diff $commit1~ $commit1
Git diff $commit1~ $commit2
```

### show

```
git show v1.0                                     #查看标签信息
git show $commit:tests/xfstest                    #查看某个commit时的某个文件内容
git show --format=fuller --stat --patch -w -M     #-w表示忽略所有空格引起的变化
```


### blame

show last modified each line of a file
```
git blame $filename
```


### status

```
git status	 
```


### log

```
git log --pretty=oneline --abbrev-commit
git log -n1 --pretty=format:'%cn %ce' $commit
git log -n1 --format=%s $commit
git log --author="Alex Kras" --after="1 week ago" --oneline
git log -p $filename       #不仅显示提交说明、提交者以及提交日期，还会显示这每次提交实际修改的内容
git log -L 10,15:$filename #允许指定文件中的某些行,有点像带焦点的"git log -p"
```


### commit

```
git commit -a -s -m "add file"
git commit --amend
```
* -a：表示所有文件。
* -s：表示在patch的头部增加Signed-off-by。


### remote

```
git remote add origin https://github.com/kulong0105/Test.git
git remote add origin git://github.com/kulong0105/Test.git
git remote -v
git remote prune origin	 #删除stale remote-tracking branches
git remote set-url origin $repo_url
```
说明:
* 使用https协议：每次push都要输入用户名和密码。可以让git记录密码：使用命令`git config --global credential.helper wincred`
* 使用git协议: 每次push不需要每次输入密码，因为是通过ssh key的方式来认证的

更新upstraem仓库到个人仓库:
```
$ git remote add upstream https://github.com/yeasy/docker_practice
$ git fetch upstream
$ git rebase upstream/master
$ git push -f origin master
```


### ls-remote

```
$ git ls-remote
From ssh://git@192.168.20.14:24/yilong.ren/git-exercise.git
422fbe9a62c196cc06b22a5911ed14c72b6631d1HEAD
422fbe9a62c196cc06b22a5911ed14c72b6631d1HEADrefs/heads/dev
422fbe9a62c196cc06b22a5911ed14c72b6631d1HEADrefsrefs/heads/master
$
```


### fetch

```
git fetch origin         #获取远程主机origin的所有分支更新（不进行合并)
git fetch origin master  #只获取远程主机origin的master分支更新
```

所取回的更新,在本地主机上要用"远程主机名/分支名"的形式读取,
比如origin主机的master,就要用origin/master读取,可以通过git branch命令查看


### merge

```
git merge my_branch
git merge --no-ff -m "merge with no-ff" my_branch
```

合并my_branch分支到当前分支，终端会显示如”Fast-forward”这样的信息，表示直接把master指向dev的当前提交，
所以合并速度非常快, 但这种模式下，删除分支后，会丢掉分支信息

加上--no-ff参数后，表示禁用Fast forward模式，git会在merge时生成一个新的commit,
合并后的历史有分支，能看出曾经做过合并，而fast forward模式合并就看不出来曾经做过合并。


### pull

git pull \[远程主机名\] \[远程分支名\]:\[本地分支名\]

```
git pull origin next	    #获取远程主机next分支的更新，再当前分支合并
git pull origin next:master #获取远程主机origin的next分支，并合并到本地master分支
git pull --rebase
```


### rebase

```
git rebase origin/master
git rease -i
```


### push

git push \[远程主机名\] \[ 本地分支名\]:\[远程分支名]

```
git push origin :dev          #删除远程dev分支
git push -u origin master     #首次push时需要加参数-u，表示建立“追踪关系"
git push
git push -f                   #强制push到远程分支
git push origin v1.0          #默认情况下，不会主动推送标签到远程。该功能完成推送v1.0标签到远程
git push origin --tags        #一次性推送全部尚未推送到远程的本地标签
```


### reset

```
git reset HEAD hello.c        #撤销缓存区的提交。
git reset --hard $commit_id   #根据commit_id恢复到某个版本库
git reset --hard HEAD~$NUM    #恢复到某个版本库，HEAD表示恢复到最新的版本库，NUM从1开始，若为1表示最新版本库的前一个版本。
```


### revert

```
git revert $commit
```


### reflog

```
git reflog	#查看版本库变更的历史
```


### rev-list

```
git rev-list HEAD -- $filename
git rev-list -n3 $commit
```


### stash

```
git stash                         #把分支当前工作现场"储藏"起来
git stash -p                      #允许你交互地选择你想要存藏的内容
git stash list                    #查看“储藏”的工作现场
git stash pop                     #恢复工作现场，且删除“储藏”的工作现场
git stash apply && git stash drop #分两步做达到同样的效果
git stash apply stash@{0}
```


### tag

```
git tag v1.0 $commit_id	                            #给某个版本打标签
git tag	                                            #查看标签标签不是按时间顺序列出，而是按字母排序的
git tag -d v1.0	                                    #删除标签
git tag -d v2.0 && git push origin :refs/tags/v2.0  #删除远程标签
```


### format-patch

```
git format-patch -1
git format-patch -s $commit_id       #-s表示从当前commit到head之前所有的commit（不包括当前commit）
git format-patch -n2 -s $commit_id   #表示从之前的某个commit到当前的commit(包括当前的commit)
```

* 遵守: one thing one commit
* 规范化msg：
	* 50-character subject line
	* 72-character wrapped longer description. This should answer:
		* Why was this change necessary?
		* How does it address the problem?
		* Are there any side effects?

* Include a link to the ticket, if any


### send-email

```
git send-email xxx.patch
```


### am

Apply a series of patches from a mailbox

```
git am -xxx.patch
```


### bisect

```
$ git bisect start
$ git bisect bad            # Current version is bad
$ git bisect good v2.3.0    # v2.3.0 was the last version tested that was good
```


### filter-branch

如果想在git repo中完全删除一个意外提交的敏感文件或大文件，可以使用如下步骤: (https://help.github.com/articles/removing-sensitive-data-from-a-repository/)

```
$ git filter-branch --force --index-filter 'git rm --cached --ignore-unmatch PATH-TO-YOUR-FILE-WITH-SENSITIVE-DATA' --prune-empty --tag-name-filter cat -- --all
$ git push origin --force --all
$ git push origin --force --tags   # In order to remove the sensitive file from your tagged releases
$ git for-each-ref --format='delete %(refname)' refs/original | git update-ref --stdin
$ git reflog expire --expire=now --all
$ git gc --prune=now
```


## .gitignore文件

存放要忽略的文件, 当使用show/diff/clean等命令时, git就会自动忽略这些文件。


## Git配置

```
[user]
    name =  Yilong Ren
    email = yilong.ren@sky-data.cn
[alias]
    lg = log --color --graph --pretty=format:'%Cred%h%Creset -%C(yellow)%d%Creset %s %Cgreen(%cr) %C(bold blue)<%an>%Creset' --abbrev-commit
    lg2 = log --color --graph --pretty=format:'%h %ad | %s%d [%an]' --abbrev-commit --date=short
    sw = show --pretty=raw
    sw2 = show --format=fuller --stat --patch -w -M
    sw3 = show --format=fuller --stat --patch -w -M --word-diff-regex=.
    st = status
    co = checkout
    ci = commit
    br = branch
    unstage = reset HEAD
    pf = pull --ff-only
    type = cat-file -t
    dump = cat-file -p
[color]
    ui = true
    diff= true
[format]
    numbered = auto
    signoff = true
[push]
    default = simple
[diff]
    renames = true
[sendemail]
    smtpserver = smtp.sky-data.cn
    smtpencryption = tls
    smtpuser = yilong.ren@sky-data.cn
    smtppass = xxx

#   smtpserver = smtp.sky-data.cn
#   from = Yilong Ren <yilong.ren@sky-data.cn>
#   chairnreplyto = false
#   smtpserverport = 25
#   envelope-sender = yilong.ren@sky-data.cn

    to = yilong.ren@sky-data.cn
    cc = team_ci@sky-data.cn
```


## 学习链接

- [http://www.bootcss.com/p/git-guide/](http://www.bootcss.com/p/git-guide/)
- [http://www.liaoxuefeng.com/](http://www.liaoxuefeng.com/)
- [http://blog.jobbole.com/tag/git/](http://blog.jobbole.com/tag/git/)

