---
title: nvidia nvdec使用
abbrlink: 4eb6e8e
date: 2024-04-15 16:11:04
tags:
    - nvidia
---

# nvdec基本调用流程
## 文字描述
1. 创建视频解析器
    1. 设置码流的基本信息.
    2. 设置`序列callback`、`图像解码callback`、`图像显示callback`等.
2. 解析码流数据包
    1. 根据解封装获得的码流信息，组装数据包。
    2. 解析数据包
3. 解析器触发callback
    1. 第一次遇到序列数据或者序列数据改变，则触发`序列callback`。
        1. 查询解码能力
        2. 创建解码器实例
    2. 解析器获得一帧码流，则触发`图像解码callback`。
        1. 启动硬件解码
    3. 一帧数据解码完成，则触发`图像显示callback`。
        1. 获取解码帧数据
4. 销毁解码器实例和解析器。

## 代码描述
1. `cuvidCreateVideoParser`创建视频解析器
    1. 通过`CUVIDPARSERPARAMS`设置码流信息。
    2. 通过`CUVIDPARSERPARAMS`设置序列callback`pfnSequenceCallback`、图像解码callback`pfnDecodePicture`、图像显示callback`pfnDisplayPicture`。
2. `cuvidParseVideoData`解析码流数据包
    1. 组装`CUVIDSOURCEDATAPACKET`结构体
    2. 通过`cuvidParseVideoData`解析数据包
3. `cuvidParseVideoData`同步触发各个callback
    1. `pfnSequenceCallback`
        1. `cuvidGetDecoderCaps`查询解码能力
        2. `cuvidCreateDecoder`创建解码器实例
    2. `pfnDecodePicture`
        1. `cuvidDecodePicture`只是启动解码，没有完成实际解码。
    3. `pfnDisplayPicture`
        1. `cuvidMapVideoFrame`阻塞获取一帧解码数据
        2. `cuMemcpy2DAsync`拷贝解码数据到host
        3. `cuvidUnmapVideoFrame`释放缓存
4. cuvidDestroyVideoParser、cuvidDestroyDecoder

