---
title: Mac m1系统调试ffmpeg源码
tags:
  - ffmpeg-debug
abbrlink: 88cda10e
date: 2023-06-01 12:48:10
---

如果想要在mac系统上调试ffmpeg源码，可以利用vscode开发工具。

1. vscode安装CodeLLDB调试插件

![CodeLLDB](CodeLLDB.png)

2. 下载ffmpeg源码并编译
   
```
git clone https://github.com/FFmpeg/FFmpeg.git

./configure  --disable-optimizations --disable-stripping --enable-debug=3 --disable-doc

make
```

`--disable-optimizations --disable-stripping --enable-debug=3`
这三个选项可以禁用编译优化，便于调试代码。

3. 生成调试启动json
   
![json](json.png)
选择LLDB调试器，生成launch.json文件。

4. 修改launch.json中程序路径以及程序参数

```json
{
    // 使用 IntelliSense 了解相关属性。 
    // 悬停以查看现有属性的描述。
    // 欲了解更多信息，请访问: https://go.microsoft.com/fwlink/?linkid=830387
    "version": "0.2.0",
    "configurations": [
        {
            "type": "lldb",
            "request": "launch",
            "name": "Debug",
            "program": "${workspaceFolder}/ffmpeg_g",
            "args": ["-i", "1.mp4", "-c", "copy", "2.mp4", "-y"],
            "cwd": "${workspaceFolder}"
        }
    ]
}
```

5. 通过f5快捷键可以启动调试。

![debug](debug.png)

