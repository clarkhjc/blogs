---
title: git分支操作
date: 2023-06-01 21:40:18
tags:
- git
---

- 查看分支

```
git branch
```

- 创建并切换分支

```
git checkout -b <branchname>
或者
git switch -c <branchname>
```

- 创建分支

```
git branch <branchname>
```

- 切换分支

```
git checkout <branchname>
或者
git switch <branchname>
```

- 删除分支

```
git branch -d <branchname>
```

- 普通模式合并分支

```
git merge --no-ff -m "merge message" <branchname>  # 禁用fast-forward模式 会保留分支信息
```

- Fast-forward方式合并分支

```
git merge <branchname> # 将分支内容合并到当前分支 日志不会保留分支信息
```