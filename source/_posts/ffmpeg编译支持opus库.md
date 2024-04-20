---
title: ffmpeg编译支持opus库
tags:
  - ffmpeg
abbrlink: fa2e322f
date: 2024-04-20 13:29:30
---

# 安装opus库
```shell
sudo apt install libopus-dev
```
安装目录如下
```shell
/usr/include/opus
/usr/lib/x86_64-linux-gnu/libopus.a
/usr/lib/x86_64-linux-gnu/pkgconfig/opus.pc
```
将`/usr/lib/x86_64-linux-gnu/pkgconfig`添加到`PKG_CONFIG_PATH`
```shell
echo "export PKG_CONFIG_PATH=/usr/lib/x86_64-linux-gnu/pkgconfig/:$PKG_CONFIG_PATH" >> ~/.bashrc
source ~/.bashrc
```

# 配置ffmpeg
```shell
./configure --disable-shared --enable-static --enable-libopus --prefix=../FFmpeg_install/
```
此时会出现错误
`ERROR: opus not found using pkg-config`
查看`ffbuild/config.log`可以看到具体错误原因
```shell
gcc -Wl,--as-needed -Wl,-z,noexecstack -I/usr/include/opus -o /tmp/ffconf.zbIL8xNA/test /tmp/ffconf.zbIL8xNA/test.o -lopus
/usr/bin/ld: /usr/lib/gcc/x86_64-linux-gnu/11/../../../x86_64-linux-gnu/libopus.a(opus_decoder.o): in function `opus_decode_frame':
(.text+0xe3c): undefined reference to `exp'
/usr/bin/ld: /usr/lib/gcc/x86_64-linux-gnu/11/../../../x86_64-linux-gnu/libopus.a(celt_decoder.o): in function `celt_decode_lost':
(.text+0x1643): undefined reference to `sqrtf'

......
```
**原因是在链接opus库时，没有添加-lm 选项**
我们看一下/usr/lib/x86_64-linux-gnu/pkgconfig/opus.pc内容
```shell
# Opus codec reference implementation pkg-config file

prefix=/usr
exec_prefix=${prefix}
libdir=${prefix}/lib/x86_64-linux-gnu
includedir=${prefix}/include

Name: Opus
Description: Opus IETF audio codec (floating-point build)
URL: https://opus-codec.org/
Version: 1.3.1
Requires:
Conflicts:
Libs: -L${libdir} -lopus
Libs.private: -lm
Cflags: -I${includedir}/opus
```
如何让ffmpeg在配置时使用Libs.private内容呢, 只有在对pkg-config指定了--static选项时才会链接Libs.private中的库。
优化configure选项
```shell
./configure --disable-shared --enable-static --enable-libopus --pkg-config-flags=--static --prefix=../FFmpeg_install/
```