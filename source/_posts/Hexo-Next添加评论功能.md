---
title: Hexo-Next 添加gitment评论功能
date: 2018-01-30 22:57:00
tags:
	- blog
	- hexo
description: "Hexo-Next 添加gitment评论功能"
categories:
	- Hexo配置
---

# hexo next主题添加gitment评论功能

## 背景
网上有很多blog讲解配置hexo 添加gitment评论功能，但是由于现在hexo默认支持gitment，所以不再需要修改代码，简单配置即可，配置步骤如下

## 第一步 使用github账号开通OAuth Apps
1. 打开链接[developers](https://github.com/settings/developers) 选择OAuth Apps链接
2. 填写必要信息 
    1. application-name 与Application description 随意填写
    2. Homepage URL 必须写blog地址(比如我的就是https://yuanwenjian.github.io/)
    3. Authorization callback URL 与Homepage URL 配置一样即可
    4. 点击保存获取client_id 与client_secret

## 第二步 github创建空项目用来存放评论内容
比如我的创建的就是gitment-comments 后面需要用到

## 第三步 配置hexo
1. 修改~/blog/themes/next/_config.yml配置
2. 找到gitment 配置，需要修改的配置如下，其他配置使用默认即可
```yml
gitment:
  enable: true
  github_user: yuanwenjian  //你的github用户名
  github_repo: gitment-comments //第二步创建的项目名(注意：项目名而不是项目链接)
  client_id: 21bb0a************** //第一步获取的client_id
  client_secret: 903e69fae75a4c67e********************* // 第一步获取的OAuth Apps的client_secret
```

## 第四步 重新部署启动
``` 
hexo clean
hexo g -d
```

## 第五步 初始化blog评论页
以上步骤完成后页面下面会出现初**始化本文评论**页按钮，点击按钮初始化即可，注意每篇新增blog都需要初始化

<font color=red >注意
有时本地修改后无效，但是部署到github生效，启动本地无效的话可以部署到github重新观察，有时会Not Found Error 或者评论未开启，不要着急，过段时间再使用
</font>

## 注意
有时本地修改后无效，但是部署到github生效，启动本地无效的话可以部署到github重新观察，如果部署后发现ERRER:NOT FOUND错误，参考[修改hexo gitment error not found问题][1]

## 参考链接
[Hexo-Next 添加 Gitment 评论系统](https://ryanluoxu.github.io/2017/11/27/Hexo-Next-%E6%B7%BB%E5%8A%A0-Gitment-%E8%AF%84%E8%AE%BA%E7%B3%BB%E7%BB%9F/)

[Gitment：使用 GitHub Issues 搭建评论系统](https://imsun.net/posts/gitment-introduction/)


[1]: https://yuanwenjian.github.io/2018/02/03/修改Gitment%20error%20NOT%20FOUND/
