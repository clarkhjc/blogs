---
title: git标签操作
tags:
  - git
abbrlink: 54c2b780
date: 2023-06-01 21:36:59
---

- 查看所有标签

```
git tag
```

- 创建一个新标签

```
git tag <name>             # 默认在最新commit id上打标签
git tag <name> <commit id> # 在某个提交上打标签
git tag -a <name> -m “message” <commit id> # -a指定标签名 -m指定说明信息
```

- 查看标签信息

```
git show <tagname>
```

- 删除标签

```
git tag -d <tagname>
```

- 推送标签到远端

```
git push origin <tagname>

git push origin —tags  # 一次性推送全部尚未推送到远程的本地标签
```

- 删除远端标签

```
git tag -d <tagname>
git push origin :refs/tags/<tagname>
```

- 切换到某个标签

```
git checkout <tagname>
```
