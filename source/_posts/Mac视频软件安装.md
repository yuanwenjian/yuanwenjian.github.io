---
title: Mac视频软件资料
description:  Mac视频软件安装及问题
date: 2018-8-18 22：47:00
tags: 
    - Mac
categories:
    - Mac
---

# 好用的视频软件及下载安装
1. 看剧神器[IINA][IINA]，点击下载即可

2. 视频处理神器[ffmpeg][ffmpeg] 
- 安装 ffmpeg brew install ffmpeg
- 常规使用网上教程很多(截取视频，合并视频。。)


3. 下载神器[annie][annie]
- 安装 brew install annie 使用之前建议安装ffmpeg,否则下载的视频是分段的
- 参数介绍 -i 列出可下载的信息 -p 下载集合(如B站的集合列表) -F 使用文件(文件内视频链接)下载


# ffmpeg 去除水印

通常下载视频会带有水印，可以通过ffmpeg去掉水印,具体做法如下
1. 定位水印的位置 
```bash
ffplay -i 文件名 -vf delogo=x=100:y=32:w=168:h=86:show=1  //调整数字对准水印
```

![效果图][效果图]

2. 去除水印
```bash
ffmpeg -i 文件名 -vf delogo=x=1050:y=20:w=200:h=130 输出文件名  中间是上一步定位好的logo位置
```

3. 问题 
ffplay 未安装
brew reinstall --with-sdl2 ffmpeg //重新安装ffmpeg 

# Mac隐私安全设置显示允许所有来源安装
```bash
sudo spctl --master-disable //输入密码，重新打开设置
```


[IINA]:https://lhc70000.github.io/iina/zh-cn/
[annie]:https://github.com/iawia002/annie
[效果图]:/images/video/logo.png
[ffmpeg]:http://ffmpeg.org/