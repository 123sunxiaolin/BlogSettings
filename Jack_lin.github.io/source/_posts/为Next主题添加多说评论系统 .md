---
title: 为Next主题添加多说评论系统 # 这是标题
categories:   # 这里写的分类会自动汇集到 categories 页面上，分类可以多级
- 实用工具
tags: # 这里写的标签会自动汇集到 tags 页面上
- Blog
---

![摄图网-小跑车.jpg](http://upload-images.jianshu.io/upload_images/1713024-e9aceb3d9d0375d8.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

### 前言
几个月前，在好奇心的鼓动下，利用`Github Pages`和`Hexo`以及`Next`主题搭建一个属于自己的个人主站，由于时间伧俗，搭建成功后就没有好好完善一下，可以参照文章[徒手教你建自己的博客](https://123sunxiaolin.github.io/2016/08/27/徒手教你建自己的博客/),文章里有搭建免费博客的详细步骤。

这周末没有加班，静下心写了篇文章，发布成功后，却又遇到`Github Pages`更新的原因导致博客的页面出现空白，又顺手写了一篇解决页面空白问题的博客，文章为[关于hexo主题next博客加载空白页的处理](https://123sunxiaolin.github.io/2016/11/12/关于hexo主题next博客加载空白页的处理/#comments),文章记录了出现问题原因和解决的多种方式。

解决完问题，本以为没有什么优化的了，此刻，看到别人的博客都可以评论和分享，后来考虑了一下，评论的功能对于反馈博客中问题很有帮助，于是乎，决定搞一下。

在添加评论的过程中，主要遇到下面的三个问题：

`1、如何选取合适评论系统；`

`2、添加评论系统后，如何根据文章对评论进行区分；`

`3、如何解决评论后的邮件提醒。`

后续的内容将围绕上述三个问题进行展开。


### 选取合适评论系统

有`disqus`和`多说`两种评论系统，`disqus`加载速度较慢，并且用户必须先注册`disqus`用户才能评论，流程很繁琐，最重要的是需要翻墙，这可是致命一击啊。而`多说`可以很好避开上述的缺点，加载速度快、利用社交账号登录就能评论、还带有邮件提醒功能，对比一下，简直天壤之别，果断选择`多说`。

### 安装多说
进入[多说网站](http://duoshuo.com),点击`我要安装`,具体设置如下图：

![多说设置图](http://upload-images.jianshu.io/upload_images/1713024-0230ac67f0735357.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

点击`创建`，选择`通用`，就会显示一段嵌入评论的通用js代码，如下图：

![duoshuo2.png](http://upload-images.jianshu.io/upload_images/1713024-f4e652b97f0242f5.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

### 替换主题中指定的文件
首先，说一下我使用的是`Next`主题，需要修改代码文件是`_config.yml`中`duoshuo_shroname`,如下：

```
duoshuo_shortname: 填写上述步骤注册的shroname
```
还需要修改目录`themes/next/layout/_partials/comments.swig`下的`comments.swig`(若你是`landscape`主题，需要修改`themes\landscape\layout\_partial\article.ejs`目录下的`article.ejs`文件),
找到原文件中的被替换代码：

```
	<div class="ds-thread" data-thread-key="{{ page.path }}"
          data-title="{{ page.title }}" data-url="{{ page.permalink }}">
      </div>
```

替换的代码如下：


     <section id="comments">
   	<!-- 多说评论框 start -->
	<div id="ds-thread" class="ds-thread" data-thread-key="{{ page.path }}" data-title="{{ page.title }}" data-url="{{ page.permalink }}" }}"></div>
	<!-- 多说评论框 end -->
	<!-- 多说公共JS代码 start (一个网页只需插入一次) -->
	<script type="text/javascript">
	var duoshuoQuery = {short_name:"shortTime"};
  	(function() {
    var ds = document.createElement('script');
    ds.type = 'text/javascript';ds.async = true;
    ds.src = (document.location.protocol == 'https:' ? 'https:' : 'http:') + '//static.duoshuo.com/embed.js';
    ds.charset = 'UTF-8';
    (document.getElementsByTagName('head')[0] 
     || document.getElementsByTagName('body')[0]).appendChild(ds);
 	 })();
 	 </script>
	<!-- 多说公共JS代码 end -->
  </section>

`需要注意的是`：`data-thread-key`、`data-title`、`data-url`这三个变量一定要严格按照上述代码格式书写，否则会导致某一篇文章评论会显示在所有的文章里，造成数据的不准确。这三个变量的具体含义在上述中已阐明（`data-thread-key`=`请将此处替换成你站点的ID` `data-title`=`请替换成文章的标题` `data-url`=`请替换成文章的网址`）.

上述的步骤都顺利完成后，`多说`评论系统基本功能就部署好了，可以发布博客体验一下。

### 添加评论邮件提醒

`多说`自带的邮件提醒功能是不支持主动评论提醒的，比如说在某一篇博客中，有人主动评论留言的话，`多说`此时是不会邮件提醒的。`多说`所谓的邮件提醒是以别人回复别人的评论为前提的，即你不是回复他人的评论博主就收不到你评论回复的提醒。从这一点上看，`多说`这一功能有失人性化的设计，故自己动手解决这个邮件提醒不完整的问题。

从分析`多说`提供的API入手，[同步用户到多说](http://dev.duoshuo.com/docs/51435552047fe92f490225de)和[评论框调用代码参数说明](http://dev.duoshuo.com/docs/5003ecd94cab3e7250000008/),得知可以使用`data-author-key`变量设置主动邮件提醒。

我们需要做的是，自定义同步一个用户到`多说`服务器，并将该用户设置成作者身份，进而取得该用户在站点的ID,将该ID赋值给`data-author-key`加在`多说`的评论框里即可。

开始动手....

在桌面或其他目录下，新建一个文件夹，命名为`js`或者其他，进入该文件夹，利用终端执行下面的命令：

```
npm install --save-dev request
```
需要等上几分钟，此时你可以进行下一步了。

利用`sublime`或其他工具新建一个`myuser.js`文件，将下面的代码复制进文件夹，并修改相应的`data`数据

```
var http = require('request');

var data = {
  'short_name'         : '',     // 你的short_name，后台管理那里可以看到
  'secret'             : '',     // 密钥，后台管理那里可以看到
  'users[0][user_key]' : '1',    // 用户在站点的ID，就是后面需要设置的 data-author-key值，可以随意设置，这里默认为1吧
  'users[0][name]'     : '',     // 显示的名字
  'users[0][email]'    : '',     // 提醒的邮箱
  'users[0][role]'     : 'author'// 用户的类型，设置为作者
};

http.post({url:'http://api.duoshuo.com/users/import.json', form: data}, function (error, response, body) {
  if (!error && response.statusCode == 200) {
    console.log('Post data to Duoshuo success');
  }
  else{
    console.log('Post data to Duoshuo fail');
  }
});
```
将上面代码中的数据按照自己的多说账户设置好之后，执行以下命令完成数据同步到多说服务器：

```
node myuser.js
```
若输出为`Post data to Duoshuo fail`那就重新检查以下`data`数据是否修改正确；

若输出为`Post data to Duoshuo success`，那么就登录到`多说`的[后台](http://jacklin.duoshuo.com/admin/users/)，在用户那里看到新添加的用户，如下所示；

![后台管理](http://upload-images.jianshu.io/upload_images/1713024-dbfd6e14672adc27.png?imageMogr2/auto-orient/strip%7CimageView2/2/)

若后台显示的角色不是作者，可以手动改成作者。

去目录`themes/next/layout/_partials/comments.swig`，修改文件`comments.swig`中的那段代码（见上面描述）为：

	<section id="comments">
   	<!-- 多说评论框 start -->
	<div id="ds-thread" class="ds-thread" data-thread-key="{{ page.path }}" data-title="{{ page.title }}" data-url="{{ page.permalink }}" data-author-key="{{ theme.duoshuo_info.data_author_key }}"></div>
	<!-- 多说评论框 end -->
	<!-- 多说公共JS代码 start (一个网页只需插入一次) -->
	<script type="text/javascript">
	var duoshuoQuery = {short_name:"shortTime"};
  	(function() {
    var ds = document.createElement('script');
    ds.type = 'text/javascript';ds.async = true;
    ds.src = (document.location.protocol == 'https:' ? 'https:' : 'http:') + '//static.duoshuo.com/embed.js';
    ds.charset = 'UTF-8';
    (document.getElementsByTagName('head')[0] 
     || document.getElementsByTagName('body')[0]).appendChild(ds);
 	 })();
 	 </script>
	<!-- 多说公共JS代码 end -->
  </section>

其实就是增加了一个变量

```
data-author-key="{{ theme.duoshuo_info.data_author_key }}"
```

这里设置好之后，修改主题下的`_config.yml`文件，增加一行配置，如下：

```
duoshuo_info:
  data_author_key: 1 #此处填写上面js脚本中的data变量中的users[0][user_key]的值，因为上面为1，这里我就填写为1了
```
到这里基本的工作就完成了，重新发布博客，就可以使用主动评论邮件提醒了。

`PS一句`，这种邮件提醒不是实时提醒，而是每天一封汇总邮件。如果你中途查看了这个评论的话，也是不会邮件提醒的


### 参考资料：

[github page博客里添加多说评论插件](http://www.cnblogs.com/galengao/p/5780851.html)

[Hexo使用多说教程](http://dev.duoshuo.com/threads/541d3b2b40b5abcd2e4df0e9)

[在hexo中加入多说评论](http://www.lichanglin.cn/在hexo中加入多说评论/)

[开启NexT主题多说主动评论邮件通知](http://www.tuicool.com/articles/iEN7riZ)

[为多说增加评论推送](http://lifeofzjs.com/blog/2014/09/14/add-noti-to-duoshuo/)



