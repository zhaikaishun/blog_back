---
title: github+hexo搭建免费博客
date: 2016-03-01 21:25:21
tags: [Hexo]
categories: [programme]
author: kaishun
id: 0
permalink: learn-hexo1
---

之前做了很多网站，用的都是wordpress，这里我记录一下自己免费搭建博客的一种方法，主要是免费，放一些静态页面是比较好的，丰富程度当然比不上wordpress了。话不多说，进入主题

前提： 
安装好了git,有github账号
<!-- more -->

----------


## **安装hexo**

```java
$ npm install -g hexo
出现
npm WARN deprecated swig@1.4.2: This package is no longer maintained

C:\Users\kaishun\AppData\Roaming\npm\hexo -> C:\Users\kaishun\AppData\Roaming\npm\node_modules\hexo\bin\hexo

> dtrace-provider@0.8.3 install C:\Users\kaishun\AppData\Roaming\npm\node_modules\hexo\node_modules\dtrace-provider
> node scripts/install.js


> hexo-util@0.6.1 postinstall C:\Users\kaishun\AppData\Roaming\npm\node_modules\hexo\node_modules\hexo-util
> npm run build:highlight


> hexo-util@0.6.1 build:highlight C:\Users\kaishun\AppData\Roaming\npm\node_modules\hexo\node_modules\hexo-util
> node scripts/build_highlight_alias.js > highlight_alias.json

npm WARN optional SKIPPING OPTIONAL DEPENDENCY: fsevents@1.1.2 (node_modules\hexo\node_modules\fsevents):
npm WARN notsup SKIPPING OPTIONAL DEPENDENCY: Unsupported platform for fsevents@1.1.2: wanted {"os":"darwin","arch":"any"} (current: {"os":"win32","arch":"x64"})

+ hexo@3.3.8
added 371 packages in 121.308s
```

## **创建hexo文件夹**

在某个目录下执行下面一句，会在该目录下创建一系列文件和文件夹
```
$ npm install
npm WARN deprecated swig@1.4.2: This package is no longer maintained



> dtrace-provider@0.8.3 install G:\npm\node_modules\dtrace-provider
> node scripts/install.js


> hexo-util@0.6.1 postinstall G:\npm\node_modules\hexo-util
> npm run build:highlight


> hexo-util@0.6.1 build:highlight G:\npm\node_modules\hexo-util
> node scripts/build_highlight_alias.js > highlight_alias.json

npm notice created a lockfile as package-lock.json. You should commit this file.
npm WARN optional SKIPPING OPTIONAL DEPENDENCY: fsevents@1.1.2 (node_modules\fsevents):
npm WARN notsup SKIPPING OPTIONAL DEPENDENCY: Unsupported platform for fsevents@1.1.2: wanted {"os":"darwin","arch":"any"} (current: {"os":"win32","arch":"x64"})

```

## **安装依赖包**
```
$ hexo generate
INFO  Start processing
INFO  Files loaded in 323 ms
INFO  Generated: index.html
INFO  Generated: archives/index.html
INFO  Generated: fancybox/fancybox_loading.gif
INFO  Generated: fancybox/jquery.fancybox.css
INFO  Generated: fancybox/jquery.fancybox.pack.js
INFO  Generated: fancybox/jquery.fancybox.js
INFO  Generated: fancybox/fancybox_loading@2x.gif
INFO  Generated: fancybox/fancybox_sprite.png
INFO  Generated: fancybox/blank.gif
INFO  Generated: fancybox/fancybox_overlay.png
INFO  Generated: archives/2017/index.html
INFO  Generated: fancybox/fancybox_sprite@2x.png
INFO  Generated: archives/2017/08/index.html
INFO  Generated: css/fonts/fontawesome-webfont.eot
INFO  Generated: fancybox/helpers/jquery.fancybox-buttons.css
INFO  Generated: js/script.js
INFO  Generated: fancybox/helpers/jquery.fancybox-buttons.js
INFO  Generated: fancybox/helpers/jquery.fancybox-thumbs.css
INFO  Generated: css/fonts/fontawesome-webfont.woff
INFO  Generated: fancybox/helpers/fancybox_buttons.png
INFO  Generated: fancybox/helpers/jquery.fancybox-media.js
INFO  Generated: fancybox/helpers/jquery.fancybox-thumbs.js
INFO  Generated: css/style.css
INFO  Generated: css/fonts/FontAwesome.otf
INFO  Generated: 2017/08/02/hello-world/index.html
INFO  Generated: css/fonts/fontawesome-webfont.ttf
INFO  Generated: css/fonts/fontawesome-webfont.svg
INFO  Generated: css/images/banner.jpg
INFO  28 files generated in 929 ms

```

## **创建github账号：略**

## **git上创建io仓库**
比如我的git名字是翟开顺
那么我git的仓库为:  
zhaikaishun/zhaikaishun.github.io

## **本地添加SSH key： 略**
去廖雪峰看

## **全局配置 _config.yml**
```
# Hexo Configuration
## Docs: https://hexo.io/docs/configuration.html
## Source: https://github.com/hexojs/hexo/

# Site
title: 翟开顺的个人博客
subtitle: 学有所思，思有所感
description: 翟开顺的个人博客
author: 翟开顺
language: zh-CN
timezone:  

# URL
## If your site is put in a subdirectory, set url as 'http://yoursite.com/child' and root as '/child/'
url: http://yoursite.com
root: /
permalink: :year/:month/:day/:title/
permalink_defaults:

# Directory
source_dir: source
public_dir: public
tag_dir: tags
archive_dir: archives
category_dir: categories
code_dir: downloads/code
i18n_dir: :lang
skip_render:

# Writing
new_post_name: :title.md
default_layout: post
titlecase: false # Transform title into titlecase
external_link: true # Open external links in new tab
filename_case: 0
render_drafts: false
post_asset_folder: false
relative_link: false
future: true
highlight:
  enable: true
  line_number: true
  auto_detect: false
  tab_replace:
  
# Home page setting
# path: Root path for your blogs index page. (default = '')
# per_page: Posts displayed per page. (0 = disable pagination)
# order_by: Posts order. (Order by date descending by default)
index_generator:
  path: ''
  per_page: 10
  order_by: -date
  
# Category & Tag
default_category: uncategorized
category_map:
tag_map:

# Date / Time format
## Hexo uses Moment.js to parse and display date
## You can customize the date format as defined in
## http://momentjs.com/docs/#/displaying/format/
date_format: YYYY-MM-DD
time_format: HH:mm:ss

# Pagination
## Set per_page to 0 to disable pagination
per_page: 10
pagination_dir: page

# Extensions
## Plugins: https://hexo.io/plugins/
## Themes: https://hexo.io/themes/
theme: landscape

# Deployment
## Docs: https://hexo.io/docs/deployment.html
deploy:
  type: git
  repo: https://github.com/zhaikaishun/zhaikaishun.github.io.git
  branch: master

```
上面配置最重要的是：（填写自己的）,相关参数的意思自己网上搜索
```
deploy:
  type: git
  repo: https://github.com/zhaikaishun/zhaikaishun.github.io.git
  branch: master
```
**注意**： 配置文件的冒号“:”后面有一个空格 否则会报错，报错的时候，去对应的行看就知道了



```
$ hexo server
INFO  Start processing
INFO  Hexo is running at http://localhost:4000/. Press Ctrl+C to stop.

```

## **把本地这些文件放在git上**
创建git仓库
```
$ git init

```
添加到远程
```
$ git remote add origin git@github.com:zhaikaishun/zhaikaishun.github.io.git
```

先拉一下
```
$ git pull origin master
```
添加，我这里不懂为什么第一次要加-f
```
$ git add .gitignore
 
```
```
$ git commit -m "第一次提交"

```

```
$ git push -u origin master

```



我们的github的仓库就有了这些文件




## **解析域名，绑定主机**
**添加一个域名**

```
ping zhaikaishun.github.io
```
## **创建CNAME文件**
在source目录下创建CNAME文件，填写内容, 每次部署的时候，其实都会替换source的内容到git上
```
zhaikaishun.top
```
当然还有一种方法： 网上的这个方法很麻烦，问题比较多，就是在Repository的根目录下面，新建一个名为CNAME的文本文件，里面写入你要绑定的域名.我的是, 如果是现在创建，可能要等好几分钟才生效，短暂的出现404，所以不推荐这种方法

**git执行部署**
```
hexo generate
hexo deploy
```
若出现:
```
ERROR Deployer not found: git
```
则先要执行
```
npm install hexo-deployer-git --save

```

## **解析域名**
先ping zhaikaishun.github.io  
得到ip的值: 
解析A记录 主机记录为@ ip 为刚才ping返回的ip
解析CANME， 主机记录为www, 记录值为zhaikaishun.github.io
阿里云解析等待几分钟
这时候，登录zhaikaishun.top，就已经创建好了  

参考了http://www.jianshu.com/p/701b1095da11

hexo的配置与教程参考： http://blog.csdn.net/xuezhisdc/article/details/53130383

更多内容参考http://zhaikaishun.top/2017/08/02/hello-world/