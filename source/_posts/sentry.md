---
title: 给React加上Sentry监控
date: 2017-04-27 19:19:31
tags: [React, Sentry]
categories: 随手记
---

有时候网站运营的时候，我们就想知道在用户访问的时候，是否出发了某个BUG。那么我们就可以给前端代码机上Sentry监控。当用户触发BUG的时候。
Sentry就会发送邮件提醒我们。

<!--more-->

### 基本配置
在sentry给出的文档里就有了简单的配置

导入raven-js (两种方法任意选择)

- 方法一：
    下面在html文件类引入raven-js  
    {% codeblock lang:html  %}
    <script src="https://cdn.ravenjs.com/3.14.2/raven.min.js"
        crossorigin="anonymous"></script> 
    {% endcodeblock %}

- 方法二：
    `npm install raven-js --save`  
    {% codeblock lang:Javascript %}
    import Raven from 'raven-js';
    {% endcodeblock %}

然后就是install了  
{% codeblock lang:Javascript %}
Raven.config('https://<key>@sentry.io/<project>').install();  //一般只在开发环境下install
{% endcodeblock %}

### 配置source map  
接下来问题就来了。
我们都知道经过webpack的处理，代码变得不可读。这个时候我们就需要配置下map文件了。让sentry的报错变得可读。  
这里我采用的是上传map文件到sentry-server的方法。  

#### 安装sentry-cli
这里我采用全局安装  
{% codeblock %}
npm install -g sentry-cli-binary  //安装sentry-cli  
sentry-cli login //如果是自己搭建的sentry-server 那么需要sentry-cli --url xxxxx login  设置下url  然后输入token
{% endcodeblock %}  


#### create release
`sentry-cli releases -o MY_ORG -p MY_PROJECT new RELEASE_ID --finalize`
其中 MY_ORG 是你的sentry上的organization  MY_PROJECT 就是你项目对应的sentry project, RELEASE_ID 就是你自己定义的

#### 上传 map文件

`sentry-cli releases -o MY_ORG -p MY_PROJECT files \
  RELEASE_ID upload-sourcemaps --url-prefix '~/static/react' /path/to/assets`  
这个命令就是把 `/path/to/assets`这个路径下的js和对应的map文件上传到sentry-server。
如果这个出错的js文件来自http(s)://yourdomain/static/react/下的话。sentry就会把根据map文件让错误信息变得可读。

#### 调整Raven Config

{% codeblock lang:Javascript %}
Raven.config('https://<key>@sentry.io/<project>'， {
  release: RELEASE_ID   //这里填写你所创建的RELEASE_ID
}).install();
{% endcodeblock %}

### 收工


### 可能存在的问题
- 上传文件的时候出现 413 Request Entity Too Large  
    如果是自己搭建的sentry-server  设置一下nginx的client_max_body_size就行  保证这个值大于你上传文件的大小  
    如果不是自己搭建的，请保证map文件小于40M
