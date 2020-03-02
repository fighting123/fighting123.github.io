---
title: git常用操作
tags:
  - git
copyright: true
abbrlink: 807237865
date: 2020-03-02 23:46:16
categories:
keywords:
---
> 基本上涵盖了日常工作中常用的git操作，经常使用sourcetree的同学可以结合着理解～

### 可视化
git log --online --graph
### 帮助
git --help
### git组成
- 工作目录：它持有实际文件
- 暂存区：保存临时改动
- HEAD：指向最后一次的提交
### 五种状态
- 未修改
- 已修改
- 已暂存
- 已提交
- 已推送
### 常规操作
- 初始化：git init
- 远程仓库：
    1. 配置：git remote add url
    2. 查看：git remote -v
    3. 查看具体某一个：git remote show origin
- 分支
    1. 新建分支：git branch feature
    2. 查看所有分支列表：git branch -a
    3. 删除分支：git branch -d feature
    4. 通过参数(新建并切换)：git checkout -b 'feature'
- 查看状态：
    git status -s
- 撤销
    1. 已修改，未暂存：git checkout .或者git reset --hard
    2. 已暂存，未提交：git checkout .或者git reset
    3. 已提交，未推送(新代码只存在与本地库，远程库没有，只需要放弃本地库所有内容，强制与远程分支保持一致即可)：git reset --hard origin/分支名
    4. 已推送(此时本地库和远程库代码一致)：
        1. 先恢复本地库：git reset --hard HEAD^
        2. 再强制同步远程库：git push -f
    5. 不小心pull错了分支：
        1. 先 git reflog 查看一下提交记录
        2. 再回退到之前的版本：git reset --hard HEAD@{n}
    6.  git reset --mixed：此为默认方式，不带任何参数的git reset，即时这种方式，它回退到某个版本，只保留源码，回退commit和index信息
    7.  --soft：回退到某个版本，只回退了commit的信息，不会恢复到index file一级。如果还要提交，直接commit即可
    8.  git reset  --hard：彻底回退到某个版本，本地的源码也会变为上一个版本的内容，此命令 慎用！
- 合并（多用merge，少用pull,相当于sourcetree合并时候取消勾选直接提交，即先合并不直接提交，检查无误后手动在提交）(切换到要合并到的分支)
    1. merge：合并分支
        1. 合并分支：git merge test
        2. 取消合并：git merge --abort
        3. 继续执行：git merge --continue
    2. pull：
        相当于是从远程获取最新版本并 merge 到本地
    3. fetch：
        相当于是从远程获取最新版本到本地，不会自动合并
### 常见问题
- gitignore失效：git rm -r --cached .
- 缓存问题：
    1. 缓存：git stash或者git stash save "备注"
    2. 查看缓存：git stash list
    3. 取出缓存(会经堆栈中的代码删除)：git stash pop或者git stash pop stash@{0}
    4. 取出缓存(不会删除堆栈中的代码)：git stash apply或者git stash apply stash@{0}
    5. 清除某次缓存：git stash drop stash@{0}
    6. 清除所有缓存：git stash clear
- 初次使用git，在执行完"git add readme.txt"命令后，在执行commit时，由于命令写错，没有写提交日志，再次更正提交就出现上述错误：Unable to create 'E:/xxx/.git/index.lock': File exists
。解决方案：在.git同级目录，执行rm -f .git/index.lock    将文件删除即可提交成功

### 组成
1. tree
2. blog
3. commit

参考：https://www.cnblogs.com/linjiqin/p/8036008.html