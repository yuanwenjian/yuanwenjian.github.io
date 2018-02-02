---
title: hexo 添加gitment踩到的坑
description:  解决error not found和 validation failed 问题
date: 2018-02-03 21:43:00
comments: true
tags: 
    - hexo 
    - gitment
categories:
    - hexo
---
# error:not found问题
## 背景
之前gitment正常使用，因为添加一篇blog，导致gitment ERROR:NOTFOUD,具体为何产生未知，但是找到无法使用的直接原因

## 原因
查看生成的 html 源代码，路径为：~/blog/public/2018/*/*/index.html
找到以下代码
```js
<script type="text/javascript">
      function renderGitment(){
        var gitment = new Gitmint({
            id: window.location.pathname, 
            owner: '',
            repo: '',
            
            lang: "" || navigator.language || navigator.systemLanguage || navigator.userLanguage,
            
            oauth: {
            
            
                client_secret: '903e69fae75a4c6',
            
                client_id: '21bb0a'
            }});
        gitment.render('gitment-container');
      }

      
      renderGitment();
      
      </script>
```
<font color=red>**原因为owner与repo取值为空**</font>
## 解决方法
既然找到了原因，就好解决了，那就是手动配置owner与repo,配置文件路径为~/blog/themes/next/layout/_partials/comments.swig
```js
function showGitment() {
    $("#gitment_title").attr("style", "display:none");
    $("#container").attr("style", "").addClass("gitment_container");
    var gitment = new Gitment({
      id: window.location.pathname,
      theme: myTheme,
      owner: "yuanwenjian",//手动配置为你自己的github用户名
      repo: "gitment-comments", //手动配置为你自己的repository
      oauth: {
        client_id: '{{ theme.gitment.client_id }}',
        client_secret: '{{ theme.gitment.client_secret }}'
      }
    });
    gitment.render('container');
  }
```
# error:validation failed问题
## 背景
之前解决not found发表了，但是初始化gitment时报validation failed错误，只好继续找解决办法了，功夫不负有心人，终于找到了解决办法
## 原因
初始化gitment会在配置gitment的仓库中创建一个issues的label,**但是**github的label是有长度限制的(不要问我长度是多少，现在没心情统计),而之前提到~/blog/themes/next/layout/_partials/comments.swig gitment配置中id属性就是用来创建issuse的label，默认配置为window.location.pathname；

## 解决办法
既然知道是label长度的原因，那就好办了，只使用页面的title做label就可以了，配置如下
```js
function showGitment() {
    $("#gitment_title").attr("style", "display:none");
    $("#container").attr("style", "").addClass("gitment_container");
    var gitment = new Gitment({
      id: '{{ page.title }}',
      theme: myTheme,
      owner: "yuanwenjian",//手动配置为你自己的github用户名
      repo: "gitment-comments", //手动配置为你自己的repository
      oauth: {
        client_id: '{{ theme.gitment.client_id }}',
        client_secret: '{{ theme.gitment.client_secret }}'
      }
    });
    gitment.render('container');
  }
```
<font size=5 color=red>但是</font>我配置没什么卵用。。然而我在~/blog/themes/next/layout/_third-party/comments/gitment.swig中也找到配置gitment的代码
```js
function renderGitment(){
        var gitment = new {{CommentsClass}}({
            id: '{{ page.title }}', 
            owner: '{{ theme.gitment.github_user }}',
            repo: '{{ theme.gitment.github_repo }}',
            {% if theme.gitment.mint %}
            lang: "{{ theme.gitment.language }}" || navigator.language || navigator.systemLanguage || navigator.userLanguage,
            {% endif %}
            oauth: {
            {% if theme.gitment.mint and theme.gitment.redirect_protocol %}
                redirect_protocol: '{{ theme.gitment.redirect_protocol }}',
            {% endif %}
            {% if theme.gitment.mint and theme.gitment.proxy_gateway %}
                proxy_gateway: '{{ theme.gitment.proxy_gateway }}',
            {% else %}
                client_secret: '{{ theme.gitment.client_secret }}',
            {% endif %}
                client_id: '{{ theme.gitment.client_id }}'
            }});
        gitment.render('gitment-container');
      }
```
不管了，两个都修改，现在终于解决了
解决了还要注意写title时也不要太浪个里浪了，也要注意title的长度
## ps
上面提到的comments.swig与gitment.swig为什么会有相同的代码，具体什么作用不清楚

ps:我这儿还叭叭儿给别人讲课呢，自己的都无法使用(囧)