---
layout: post
title: 苹果手机播放异常排查
categories: windows
description: 苹果手机播放ts文件有声音无画面
keywords: video, iPhone
---

苹果手机播放ts文件有声音无画面。

## 苹果官方提供的目前支持的视频格式

参考链接：[HLS Authoring Specification for Apple Devices](https://developer.apple.com/library/content/technotes/tn2224/_index.html#//apple_ref/doc/uid/DTS40009745-CH1-MEDIASTREAMSEGMENTATION)

视频信息可使用potplayer或mediainfo等工具查看。

目前支持AVC的level及手机型号如下，推荐帧大小为30。

![1.jpg](https://i.loli.net/2019/03/29/5c9d029a1d750.jpg)

也可参考：[Preparing Media for Delivery to iOS-Based Devices](https://developer.apple.com/library/content/documentation/NetworkingInternet/Conceptual/StreamingMediaGuide/UsingHTTPLiveStreaming/UsingHTTPLiveStreaming.html#//apple_ref/doc/uid/TP40008332-CH102-SW8)

![2.jpg](https://i.loli.net/2019/03/29/5c9d02eead08d.jpg)

## 苹果官方提供的视频检测工具​

安装包： [HTTPLiveStreamingTools333.dmg](http://pan.baidu.com/s/1gfriuTP)

使用教程可参考：[HLS Authoring Specification for Apple Devices](https://developer.apple.com/library/content/technotes/tn2224/_index.html#//apple_ref/doc/uid/DTS40009745-CH1-VALIDATEYOURMEDIA-MEDIA_STREAM_VALIDATOR_TOOL)

以下为使用其分析一条m3u8的结果，故障现象是播放时有声音没画面。

工具检测后提示发现隔行扫描错误，必须是`逐行扫描`

URL：

http://v-cc.dushu.io/video/other/9fb9bcd0b8026ca4af617bf97d81af30_ec9c5b/playlist.m3u8

![3.jpg](https://i.loli.net/2019/03/29/5c9d04198d1b4.jpg)

## 如何查看苹果手机是否支持该视频格式

使用 potplayer 工具查看 ts 视频文件信息，可以看到该视频标准为PAL制式，其扫描方式为隔行扫描，而苹果手机不支持隔行扫描，因而该视频在苹果手机下播放异常。

 - m3u8 及 ts 文件

m3u8：

http://v-cc.dushu.io/video/other/9fb9bcd0b8026ca4af617bf97d81af30_ec9c5b/playlist.m3u8

ts：

http://v-cc.dushu.io/video/other/9fb9bcd0b8026ca4af617bf97d81af30_ec9c5b/9fb9bcd0b8026ca4af617bf97d81af30_ec9c5b.mp4_av_0.ts

 - 文件信息

![4.jpg](https://i.loli.net/2019/03/29/5c9d047e3792b.jpg)

