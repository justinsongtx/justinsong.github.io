---
title: git 常用命令备忘
date: 2020-06-22 14:30:07
tags:
---


### 本地所有修改的。没有的提交的，都返回到原来的状态

```
git checkout .
```

### 把所有没有提交的修改暂存到stash里面。可用git stash pop回复

```
git stash
```

### 撤销修改 && 删掉新增文件

```
git checkout . && git clean -xdf
```

### 执行完commit后，想撤回commit

```
git reset --soft HEAD^
```

### 从当前分支拉出新分支

在protect分支修改完代码不能直接提交，需要通过mr合入，直接从protect分支拉出新分支，把代码提交到新分支上。

```
git checkout -b newBranchName
```

拉完第一次push的时候要关联upstream

```
git push --set-upstream origin bugfix/test1
```


### 查看远程有哪些分支

```
git branch -a
```


### 撤销merge

```
git reset --hard 最后一次push的commitID
```