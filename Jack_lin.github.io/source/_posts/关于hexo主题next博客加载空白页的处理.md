---
title: 关于hexo主题next博客加载空白页的处理 # 这是标题
categories:   # 这里写的分类会自动汇集到 categories 页面上，分类可以多级
- 实用工具
tags: # 这里写的标签会自动汇集到 tags 页面上
- Blog
---

#### 问题描述
今天上午，写了一个博客，本地上传后，打开我的博客地址一看，发现出来大片空白，一脸懵逼，因为在本地用浏览器预览完成，才上传到Github Pages，下面开始一点点排查。

#### 本地排查
我把写好的markdown文件，放进**source/_post**目录下，执行了一遍**Hexo s**命令，播客主站运行正常。
本地没有，**git**上传就有问题，应该是**Github pages**的问题。

#### 终于在Next主题的**issues**找到答案
GitHub Pages禁止了`source/vendors`目录的访问，导致`vendors`文件访问不到。具体原因是Github在11月3日更新了版本。其中包括升级了Jekyll到3.3。Jekyll 为了加快构建速度，忽略 vendor和node_modules文件夹。[具体更新日志](https://github.com/blog/2277-what-s-new-in-github-pages-with-jekyll-3-3)

#### 解决方式
方式一：手动解决，手动将**source/vendor**更改为**source/lib**(`或者`**source/其他名称**),同时，修改主题配置文件`_config.yml`文件中，将`_internal:vendor`改为你想要的名字，如：`_internal:lib`;

方式二：定位到`Next`主题目录下，执行`git pull`命令；

#### 希望遇到同样问题的同学们，可以尽快解决问题，请叫我雷锋！
