---
title: git基本操作
date: 2023-06-01 21:30:33
tags:
- git
---

Git是一个开源的分布式版本控制系统，用于敏捷高效地处理任何或小或大的项目。

Git是Linus Torvalds为了帮助管理Linux内核开发而开发的一个开放源码的版本控制软件。Git与常用的版本控制工具CVS,Subversion等不同，它采用了分布式版本库的方式，不必服务器端软件支持。

- 提交修改到版本库

```
git add file                   # 工作区修改添加到暂存区
git commit -m "message" file   # 暂存区修改提交到版本库
```

- 撤销工作区修改

```
git checkout -- file
或者
git restore file
```

- 撤销暂存区修改

```
git reset HEAD file
或者
git restore --staged file
```

- 版本回退

```
回退到上一个版本
git reset --hard HEAD^
回退到某一个版本
git reset --hard commitid
```

- 查看提交历史

```
git log
显示单行
git log --pretty=oneline
显示点线图 显示单行 commit id显示前7位
git log --graph --pretty=oneline --abbrev-commit
```

- 查看命令历史

```
git reflog
```

- 删除文件

```
git rm file
git commit -m "message"
```