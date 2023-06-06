---
title: png透明背景转换为纯色背景
tags:
  - imagemagick
abbrlink: c079378d
date: 2023-06-04 20:22:42
---

如何将png的透明背景转换为纯色的背景呢？可以使用imagemagick提供的convert命令行

原图:
![原图](1.png)
```shell
convert 1.png -background white -flatten 1.jpg
```
转换为白色的背景
![白色背景](1.jpg)

```shell
convert 1.png -background red -flatten 1.jpg
```

转换为红色的背景
![红色背景](2.jpg)


