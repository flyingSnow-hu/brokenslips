---
layout: timemachine
title: 下载 youtube 视频和字幕
tags: youtube
date: 2018-02-28
category: others
---
# 下载 youtube 视频和字幕

## 下载视频
[这是原帖](https://www.zhihu.com/question/20157513/answer/52271574)  

 * 使用 YouTube-dl

 1. 安装
   * Unix 用户：  

   ``` shell  
   sudo curl https://yt-dl.org/latest/youtube-dl -o /usr/local/bin/youtube-dl
sudo chmod a+rx /usr/local/bin/youtube-dl  

	# install ffmpeg  
	brew install ffmpeg
  ```
   * windows 用户：
   下载[这个](https://yt-dl.org/latest/youtube-dl.exe)，放到某个目录里，下面直接调用这个exe

 2. 使用
 	登录youtube，你想下载的视频网页，从网址获得视频的v值，比如 https://www.youtube.com/watch?v=v6sC4cwc7_4 中 "v="后面的"v6sC4cwc7_4"，或者 https://www.youtube.com/watch?v=42QZ3aUDnV0&t=57s 中的 "42QZ3aUDnV0"

	 ```shell
	 # 获取视频所有格式列表
	 youtube-dl -F v值
	 ```

	 输出的每个格式前面会有一个编号，选择想要的编号
	 ```shell
	 youtube-dl -f 编号+编号+...+编号 v值
	 ```

	 或者直接下载码率最高mp4格式
	 ```shell
	 youtube-dl -f v值
	 ```


## 下载字幕
[这是原帖](https://www.zhihu.com/question/19647719)

1. 安装 Chrome
2. 安装 Tampermonkey
3. 点击[这里](https://greasyfork.org/zh-CN/scripts/5368-youtube-subtitle-downloader-v2) 和 [这里](https://greasyfork.org/zh-CN/scripts/5367-youtube-auto-subtitle-downloader) 安装两个脚本，一个下载原生字幕，一个下载自动生成字幕
4. 访问视频网页，发现会多出来两个绿色下载按钮
