title: git命令
author: 禾田
tags:
  - git
categories:
  - git
date: 2018-02-10 09:51:00
description: git常用命令
---
## 1. 将当前目录设置成一个Git仓库
```git
//进入需要创建Git仓库的目录，然后输入命令，则当前目录就被设置成了一个Git仓库，默认会自动生成一个.git文件夹
git init 
//在创建项目时就将项目目录设置成一个Git仓库
git init projectName
```

## 2. 将项目提交到本地仓库
```git
//将修改文件提交到暂存区的持久化容器中,可以同时提交多个文件
git add fileName1 fileName2
//将代码提交的Git仓库中，每次提交最好都加上后面的描述信息
git commit -m "description"
//查看git代码提交状态
git status
```

## 3. 查看修改内容  
[工作区和暂存区详细介绍](https://www.liaoxuefeng.com/wiki/0013739516305929606dd18361248578c67b8067c8c017b000/0013745374151782eb658c5a5ca454eaa451661275886c6000)

HEAD ----------> commit版本  
  
Index ----------> staged版本(暂存区)

```git
//比较的是工作目录(Working tree)和暂存区域快照(index)之间的差异也就是修改之后还没有暂存起来的变化内容
git diff
//查看已经暂存起来的文件(staged)和上次提交时的快照之间(HEAD)的差异
git diff --cached
git diff --staged
//显示工作区版本(Working tree)和HEAD的差别
git diff HEAD
//查看简化的diff结果，可以加上--stat参数
git diff --stat
```
## 4. 查看git日志
```git
//按照时间倒叙排列提交日志
git log
//显示提交信息的简单版本
git log --oneline
//可以显示每次提交包含的文件
git log --stat
//显示每次提交之间哪些内容改变
git log --patch
//查看分支合并图
git log --graph
//查询自己所有的操作以及对应版本的commit id
git reflog
```

## 5. 版本回退
```git
//回退到上一个版本，上一个版本就是HEAD^，上上一个版本就是HEAD^^，往上n个版本写n个^比较容易数不过来，所以写成HEAD~n
git reset --hard HEAD^
//回退到之前的某个版本，commit_id可以通过git log命令得到,若要回到未来版本，需要用git reflog来得到commit_id
git reset --hard commit_id
```

## 6. 撤销修改
```git
//场景1：当改乱了工作区某个文件的内容，想直接丢弃工作区的修改时
git checkout --fileName

//场景2：不但改乱了工作区某个文件的内容，还添加到了暂存区时，想丢弃修改
//先回到场景1
git reset HEAD fileName
git checkout --fileName

场景3：已经提交了不合适的修改到版本库时，想要撤销本次提交，可以使用版本回退命令，不过前提是没有推送到远程库。
```
## 7. 删除文件
```
//将文件从版本库中删除，同时提交修改
git rm fileName
git commit -m "remove test.txt"
//如果文件已经提交到版本库并且被误删了，还可以使用如下命令还原
git checkout --fileName
```

## 8. 添加远程仓库
```git
//关联一个远程库
git remote add origin git@server-name:path/repo-name.git
//第一次推送master分支的所有内容，第一次推送需要-u选项
git push -u origin master
```

## 9. 从远程仓库克隆
```git
//后面填写需要克隆仓库的地址
git clone git@github.com:michaelliao/gitskills.git
```

## 10. 创建与合并分支
[创建与合并分支详解](https://www.liaoxuefeng.com/wiki/0013739516305929606dd18361248578c67b8067c8c017b000/001375840038939c291467cc7c747b1810aab2fb8863508000)
```git
//查看分支
git branch
//创建分支(<name>为分支名)
git branch <name>
//切换分支
git checkout <name>
//创建+切换分支
git checkout -b <name>
//合并某分支到当前分支
git merge <name>
//删除分支
git branch -d <name>
```
当多个分支同时修改了同一个文件时候，需要解决分支冲突，分支冲突通常需要手动解决。  
[解决分支冲突详解](https://www.liaoxuefeng.com/wiki/0013739516305929606dd18361248578c67b8067c8c017b000/001375840202368c74be33fbd884e71b570f2cc3c0d1dcf000)

## 11. 分支管理策略
在实际开发中，应该按照几个基本原则进行分支管理：

首先，master分支应该是非常稳定的，也就是仅用来发布新版本，平时不能在上面干活；

那在哪干活呢？干活都在dev分支上，也就是说，dev分支是不稳定的，到某个时候，比如1.0版本发布时，再把dev分支合并到master上，在master分支发布1.0版本；

你和你的小伙伴们每个人都在dev分支上干活，每个人都有自己的分支，时不时地往dev分支上合并就可以了。

所以，团队合作的分支看起来就像这样：
![团队合作](http://owq01tqh9.bkt.clouddn.com/0.png)


## 12. BUG分支
参考：[BUG分支](https://www.liaoxuefeng.com/wiki/0013739516305929606dd18361248578c67b8067c8c017b000/00137602359178794d966923e5c4134bc8bf98dfb03aea3000)

## 13. 多人合作
实际工作中的情况：[多人合作详解](https://www.liaoxuefeng.com/wiki/0013739516305929606dd18361248578c67b8067c8c017b000/0013760174128707b935b0be6fc4fc6ace66c4f15618f8d000)
