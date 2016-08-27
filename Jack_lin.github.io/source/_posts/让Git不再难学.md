---
title: 让Git不再难学 # 这是标题
categories:   # 这里写的分类会自动汇集到 categories 页面上，分类可以多级
- iOS学习记录
tags: # 这里写的标签会自动汇集到 tags 页面上
- Git
- 代码托管
---
![放手](http://upload-images.jianshu.io/upload_images/1713024-70a42fc075a289ac.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

### 写在前面
 
在团队做过软件开发的，版本控制必是不可或缺的一项。目前，版本控制主要分为集中式版本控制系统和分布式版本控制系统 ，即大家熟知的SVN和Git。Git是当下最流行的分布式版本控制系统，故，今天，我们就来研究一下Git的神奇之处。

### 浅析SVN与Git的不同
 最基本：SVN是集中式，Git是分布式的。集中式，版本库集中存放在中央服务器，工作时，每个人需要先从中央服务器获取最新的版本，然后在属于自己的分支下工作，完成工作后，再提交到中央服务器；分布式，每一个电脑都是一个完整的版本库，可以减少中央服务器挂掉所带来的严重后果。也可以这样理解，Git相当于SVN服务端和客户端的集合；
 

联网需求不同：Git中，不必联网就能看到版本迭代的信息和创建分支，而，SVN需要在联网的情况下才能满足上述要求；

分支管理难易程度不同：在SVN中，分支作为版本库中一个目录，若需要查看是否合并，还需要运行`svn propget svn:mergeinfo`指令；而在Git中，可以快速在几个分支中切换和快速合并；

存储方式不同：SVN是按文件存储，Git是按元数据方式存储；

权限问题：在SVN中，可以设定每个账号的权限，比如只读、读写权限等，而Git就没有相应的权限控制。这也许就是一些公司至今仍保留SVN的原因吧。

汇总：工具总有其侧重点，没有好与坏，只有各取所需。

### 安装Git

第一种方法：安装Git图形管理工具——SourceTree，不必记那么多命令，了解基本流程就可以放心使用了，下载[点这](https://www.sourcetreeapp.com);

第二种方法：在Mac上安装Git，可通过HomeBrew安装，具体方法请参考[HomeBrew的文档](http://brew.sh/);

补充其他平台的安装方法，[请参考这](http://www.liaoxuefeng.com/wiki/0013739516305929606dd18361248578c67b8067c8c017b000/00137396287703354d8c6c01c904c7d9ff056ae23da865a000)。

如果大家对Mac终端的命令还不太了解的话，可以参考我的上篇文章：[学好Mac常用命令，助力iOS开发](http://www.jianshu.com/p/d9ec00d28237)

### Git的使用

> 有两种Git的使用方法，一种是通过终端命令控制，另一种是使用Git的图形管理工具——SourceTree。具体两者哪一种更好用，更有效率，那还得因人而异。笔者下面将会分别介绍这两种方法的基本操作，希望读者能边读边动手操作一下，受益匪浅。

##### 一.命令下的Git

###### 1.创建版本库：


首先，选择一个合适的地方，创建一个空目录：

      Mac-Pro:~ kys-1$ mkdir gitTest
      Mac-Pro:~ kys-1$ cd gitTest
或者直接在相应目录下，自己手动创建一个文件夹；

其次，在终端上输入`git init`命令，如下：

      Mac-Pro:~ kys-1$ cd /Users/kys-1/Desktop/gitTest
      Mac-Pro:gitTest kys-1$ git init
      Initialized empty Git repository in /Users/kys-1/Desktop/gitTest/.git/
这样，一个本地仓库就顺利地建好了。

最后，每创建一个仓库时，当前目录都会多一个`.git`目录，若是没有看到，只需要使用`ls -ah`命令即可：

      Mac-Pro:gitTest kys-1$ ls -ah
      .		..		.DS_Store	.git
很清楚可以看到，笔者的当前目录有两个隐藏的。

###### 2.添加文件到版本库

首先，创建一个`GitTest.md`文件，内容如下：

      git is a strong software
      Let's start with a good mood!
将该文件放到仓库`gitTest`所在的目录下;

其次，使用`git add`命令添加文件，如下：

      Mac-Pro:gitTest kys-1$ git add GitTest.md
然后执行，如果不显示任何信息，那就是运行无误；

最后，使用命令`git commit`将文件提交到仓库，如下：

        Mac-Pro:gitTest kys-1$ git commit -m "wrote a README"
        [master (root-commit) dfaeb43] wrote a README
           1 file changed, 1 insertion(+)
           create mode 100644 GitTest.md
其中，`-m`后面输入的是本次提交的详细信息，比如，完成了哪个功能或者修复了哪个bug。

###### 3.查看仓库状态

首先，先对提交的`GitTest.md`中的信息进行修改，然后运行`git status`,效果如下：

      Mac-Pro:gitTest kys-1$ git status
      On branch master
       Changes not staged for commit:
      (use "git add <file>..." to update what will be committed)
       (use "git checkout -- <file>..." to discard changes in working  directory)

	      modified:   GitTest.md

      no changes added to commit (use "git add" and/or "git commit -a")

###### 4.查看修改内容

使用命令`git diff`可以查看具体修改了哪些内容，如下：

      Mac-Pro:gitTest kys-1$ git diff
      diff --git a/GitTest.md b/GitTest.md
      index 00ad777..64590e1 100644
      --- a/GitTest.md
      +++ b/GitTest.md
      @@ -1 +1,2 @@
      -git is a strong software
      \ No newline at end of file
      +git is a strong software
      +Let's start with a good mood!
      \ No newline at end of file

###### 5.版本回退

首先，使用`git log`命令查看历史记录，如下：

      Mac-Pro:gitTest kys-1$ git log
      //版本一
      commit 988f92f1f5cf959d491ad63462e0c90372bb4b0b
      Author: XiaolinSun <401788217@qq.com>
      Date:   Fri Apr 8 11:15:08 2016 +0800

          add new
      //版本二
      commit dfaeb438504942d09e7f4282bd93b560d2ee68e2
      Author: XiaolinSun <401788217@qq.com>
      Date:   Fri Apr 8 11:12:02 2016 +0800

        wrote a README
      Mac-Pro:gitTest kys-1$ git reset --hard HEAD^
      HEAD is now at dfaeb43 wrote a README
      Mac-Pro:gitTest kys-1$ git log
       //版本三
      commit dfaeb438504942d09e7f4282bd93b560d2ee68e2
      Author: XiaolinSun <401788217@qq.com>
      Date:   Fri Apr 8 11:12:02 2016 +0800

        wrote a README
很清晰，共有三次提交历史。其中，`988f92f1f5cf959d491ad63462e0c90372bb4b0b`这样一串字符表示提交的版本号`commit id`；


使用`git reset`命令可以回退到上一个版本，输入` git reset --hard commit id`,如下：

      Mac-Pro:gitTest kys-1$ git reset --hard 988f92f
      HEAD is now at 988f92f add new


如果开发过程中，一不小心，回退错了地方，可以使用`git reflog`命令查看命令历史，如下：

      Mac-Pro:gitTest kys-1$ git reflog
      dfaeb43 HEAD@{0}: reset: moving to HEAD^
      988f92f HEAD@{1}: commit: add new
      dfaeb43 HEAD@{2}: commit (initial): wrote a README
从而，可以选择要回到的那个版本即可。

###### 6.工作区与暂存区

工作区：上几步中在电脑创建的文件目录就是工作区，如下图：
![工作区](http://upload-images.jianshu.io/upload_images/1713024-61c1e2461cf2357a.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

版本库就是隐藏目录`.git`;

添加文件时，首先，用`git add`把文件添加进去,实际上是把文件修改添加到暂存区；然后，用`git commit`提交更改，是把暂存区的内容提交到当前分支；最后，每次文件修改后，如果不`add`到暂存区，那是不会加入到`commit`。

###### 7.撤销修改

当你修改文件`GitTest.md`时，在你准备提交前，忽然发现一个错误,如图：

      git is a strong software
      Let's start with a good mood!
      Start up now!
      Fight for future! //新添加的内容
这时候，必须要查看一下状态：输入`git status`,如下：

        Mac-Pro:gitTest kys-1$ git status
        On branch master
        Changes not staged for commit:
          (use "git add <file>..." to update what will be committed)
          (use "git checkout -- <file>..." to discard changes in working directory)

	    modified:   GitTest.md

      no changes added to commit (use "git add" and/or "git commit -a")

使用`git checkout -- file`命令就会丢弃工作区的修改，如下：

      Mac-Pro:gitTest kys-1$ git checkout -- GitTest.md
然后再查看工作区的内容：

      git is a strong software
      Let's start with a good mood!
      Start up now!
这个时候内容就恢复了。

###### 8.删除文件

使用`git rm`和``git commit`命令就可以从版本库删除相应的文件，例如：我们新建一个`newFile.md`文件，然后将其加入工作区，如下所示：
![新工作区](http://upload-images.jianshu.io/upload_images/1713024-7db763b6bbdcba78.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

      Mac-Pro:gitTest kys-1$ git add newFile.md
      Mac-Pro:gitTest kys-1$ git commit -m "add newFile"
      [master e278be1] add newFile
       1 file changed, 1 insertion(+)
       create mode 100644 newFile.md
      Mac-Pro:gitTest kys-1$ git rm newFile.md
      rm 'newFile.md'
      Mac-Pro:gitTest kys-1$ git commit -m "delete newFile"
      [master 9c28795] delete newFile
       1 file changed, 1 deletion(-)
       delete mode 100644 newFile.md
执行完这些命令后，文件就从版本库中删除了。

###### 小结一下

以上操作都是在本地仓库进行的一些操作，如果感觉上面讲的有些许啰嗦，那么只需记住下面几个命令即可：

    1.git init      -------创建空的版本库；
    2.git add       -------添加文件到暂存区；
    3.git commit      -------提交文件到版本库(仓库)；
    4.git status      -------查看相关状态；
    5.git diff      -------查看具体修改的内容；
    6.git log      -------查看提交历史记录；
    7.git reset      -------回到上一版本；
    8.git reflog      -------查看命令历史记录；
    9.git checkout      -------丢弃工作区的修改；
    10.git rm      -------删除文件；

##### 二.SourceTree的使用

使用SourceTree客户端来执行上述与本地仓库相关的操作。

 安装并打开SourceTree：

![SourceTree](http://upload-images.jianshu.io/upload_images/1713024-5b20da8be8429b28.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

选择`本地`按钮，后点击`新仓库`，选择`创建本地仓库`,填入自己的地址，点击`创建`就OK了（是不是很方便？）：

![创建流程1](http://upload-images.jianshu.io/upload_images/1713024-c3cac05145e88a64.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

![创建流程2](http://upload-images.jianshu.io/upload_images/1713024-676f79e6b1c4d24f.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


我先创建一个名字`TTest`本地仓库，演示一下后面的相关操作，点击新建仓库的导航条：
![点击](http://upload-images.jianshu.io/upload_images/1713024-eb792a2cf15550ff.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

![仓库状态](http://upload-images.jianshu.io/upload_images/1713024-afbe3632091ffb71.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


在工作区添加文件`README.md`：

![显示未暂存文件](http://upload-images.jianshu.io/upload_images/1713024-8ae9fa00dc331953.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

![已暂存](http://upload-images.jianshu.io/upload_images/1713024-0c3a77e3ed6fdbe7.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

解释一下，`已暂存`就是相当于`git add`操作。


提交操作，对比`git commit -m "add new info"`命令：
![提交](http://upload-images.jianshu.io/upload_images/1713024-8b48163cfae58a04.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

![提交后](http://upload-images.jianshu.io/upload_images/1713024-72fd2e5f8cf6cdeb.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

查看具体内容修改、查看相关状态都可以直接看到，就不在一一赘述了；

回到上一版本可以`回滚操作`，选择具体的提交记录，点击右键，选择`提交回滚`即可；

对比一下，发现`SourceTree`是如此的方便快捷！

###### 远程仓库

概念：远程仓库可以用一台电脑当做服务器，团队其他成员可以从这个服务器克隆一份完整过程到自己的本地仓库，同时，可以将完成的部分推送到远程仓库，以实现项目的跟进。由于是演示，咱就暂且使用现成的远程仓库——[Github](https://github.com);

作为开发人员，github账号是必备的，如果还没有，那就得立即注册一个。

 创建SSH Key，因为本地的Git仓库与Github远程仓库之间是通过SSH加密的。首先，需要到主目录上查看是否有`.ssh`目录，再查看`.ssh`目录下有没有`id_rsa`和`id_rsa.pub`文件，如下，
![查找.ssh目录](http://upload-images.jianshu.io/upload_images/1713024-e9f198b3615c3a86.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

发现没有上述的两个文件，这时需要创建：

      Mac-Pro:~ kys-1 $ ssh-keygen -t rsa -C "youremail@example.com"  

需要将邮件地址换成自己的地址，如下：

    Mac-Pro:~ kys-1$ ssh-keygen -t rsa -C "40178217@qq.com"
    Generating public/private rsa key pair.
    Enter file in which to save the key (/Users/kys-1/.ssh/id_rsa): 
    Created directory '/Users/kys-1/.ssh'.
    Enter passphrase (empty for no passphrase): 
    Enter same passphrase again: 
    Your identification has been saved in /Users/kys-1/.ssh/id_rsa.
    Your public key has been saved in /Users/kys-1/.ssh/id_rsa.pub.

出现上述描述，就证明你成功了，然后到主目录下找到`.ssh`目录，查看`id_rsa`和`id_rsa.pub`文件，`id_rsa`是私钥，需要自己保留好，`id_rsa.pub`是公钥，别人知道也无妨。

登录Github账户，打开`Account settings`，`SSH Keys`页面,添加`id_rsa.pub`文件的内容：
![添加公钥](http://upload-images.jianshu.io/upload_images/1713024-ab90e2c57ca94762.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

添加SSH key的作用，是保证推送到远程仓库的提交确实是你提交的而不是别人代替，以确保项目被他人修改；


添加远程库：首先登陆github账号，点击“New repository ”按钮，创建新仓库，如下：
![New repository ](http://upload-images.jianshu.io/upload_images/1713024-7e93afc14041f61b.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

![填写信息](http://upload-images.jianshu.io/upload_images/1713024-16323958e6414664.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
需要填写仓库名，描述以及是否勾选创建初始化文件等。


在本地仓库下运行命令：


     Mac-Pro:~ kys-1$ git remote add origin yourGitAddress

然后，将本地库的所有内容推送到远程库：

    Mac-Pro:~ kys-1$ git push -u origin master

到这一步，只要本地作了提交。就可以通过命令：

    Mac-Pro:~ kys-1$ git push origin master

把本地`master`分支的最新修改推送到GitHub上去。


使用`git clone`命令将远程库克隆到本地：


    Mac-Pro:~ kys-1$ git clone yourLocalGitAddress1


在SourceTree下克隆远程仓库到本地的操作：

![操作流程](http://upload-images.jianshu.io/upload_images/1713024-ff765f970a25b2b7.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
源URL就是远程仓库的地址，目标路径是本地存储的路径。


###### 创建与合并分支（与SourceTree同步对照）

概念：创建分支的目的是让你的工作变得更灵活更有效率，当你开发新功能时，你创建了一个分支，你可以在在原来分支继续工作，也可以在新建的分支上工作，两者之间不会干预，当你完成了新功能，只需将新分支合并到原来分支上即可。最吸引人的是Git创建和切换分支是非常速度的。

创建`dev`分支，然后切换到`dev`分支，使用`git checkout`命令：

      Mac-Pro:gitTest kys-1$ git checkout -b dev
      Switched to a new branch 'dev'
`-b`参数表示创建并切换.

使用`git branch`命令查看当前分支：

      Mac-Pro:gitTest kys-1$ git branch
        * dev
        master

SourceTree操作：
![选择分支按钮](http://upload-images.jianshu.io/upload_images/1713024-4bb65fa97c1f30db.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

![创建分支](http://upload-images.jianshu.io/upload_images/1713024-b814e5673239c2de.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


![切换到dev分支](http://upload-images.jianshu.io/upload_images/1713024-453cb9e7a4ca5b61.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


我们在dev分支进行操作，修改`README.md` 文件内容然后提交：

      Mac-Pro:gitTest kys-1$ git add README.md
      Mac-Pro:gitTest kys-1$ git commit -m "dev branch"
      [dev 9be4a38] dev branch
       1 file changed, 2 insertions(+), 1 deletion(-)

SourceTree操作：

![提交](http://upload-images.jianshu.io/upload_images/1713024-3c19e76829859bc9.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

![提交后](http://upload-images.jianshu.io/upload_images/1713024-0cb17f37a9c278b1.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


`dev`分支工作结束，切换到`master`分支，

      Mac-Pro:gitTest kys-1$ git checkout master
      Switched to branch 'master'
      Your branch is up-to-date with 'origin/master'.


SourceTree操作：

![切换到master分支](http://upload-images.jianshu.io/upload_images/1713024-62cc7cee096a0633.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
点击右键，选择`检出master`分支即可。

![检出成功](http://upload-images.jianshu.io/upload_images/1713024-0d0b5ac998f986de.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


使用`git merge ` 指令把`dev`分支的工作成果合并到`master`分支上：

      Mac-Pro:gitTest kys-1$ git merge dev
      Updating 2269ea8..9be4a38
          Fast-forward
         README.md | 3 ++-
       1 file changed, 2 insertions(+), 1 deletion(-)

SourceTree操作：选中`dev`分支，右击选择`合并dev至master`即可，

![合并后](http://upload-images.jianshu.io/upload_images/1713024-3b1072a46526173e.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


使用`git branch -d dev`命令删除`dev`分支：

      Mac-Pro:gitTest kys-1$ git branch -d dev
      Deleted branch dev (was 9be4a38).

SourceTree操作：选中`dev`分支，右击选择`删除dev`即可。

###### 小结一下
      查看分支：git branch
      创建分支：git branch <name>
      切换分支：git checkout <name>
      创建+切换分支：git checkout -b <name>
      合并某分支到当前分支：git merge <name>
      删除分支：git branch -d <name>

###### 冲突解决

制造冲突：先创建一个新的分子`conflict`，并在新分支上工作,修改`README.md`内容,操作如下：

      Mac-Pro:gitTest kys-1$ git checkout -b conflict
      Switched to a new branch 'conflict'
      Mac-Pro:gitTest kys-1$ git add README.md
      Mac-Pro:gitTest kys-1$ git commit -m "make a conflict"
      [conflict 1bc6611] make a conflict
       1 file changed, 2 insertions(+), 1 deletion(-)


![SourceTree显示状态](http://upload-images.jianshu.io/upload_images/1713024-e067cfd435ed52d3.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


切换到`master`，继续修改`README.md`内容,并提交修改：

      Mac-Pro:gitTest kys-1$ git checkout master
      Switched to branch 'master'
       Your branch is up-to-date with 'origin/master'.
      Mac-Pro:gitTest kys-1$ git add README.md
      Mac-Pro:gitTest kys-1$ git commit -m "add two conflicts"
      [master f43d5d1] add two conflicts
       1 file changed, 2 insertions(+), 1 deletion(-)

合并分支，就会出现冲突：

      Mac-Pro:gitTest kys-1$ git merge conflict
      Auto-merging README.md
      CONFLICT (content): Merge conflict in README.md
      Automatic merge failed; fix conflicts and then commit the result.
      Mac-Pro:gitTest kys-1$ git status
      On branch master
      Your branch is ahead of 'origin/master' by 1 commit.
      (use "git push" to publish your local commits)
      You have unmerged paths.
      (fix conflicts and run "git commit")
      Unmerged paths:
        (use "git add <file>..." to mark resolution)

	    both modified:   README.md

      no changes added to commit (use "git add" and/or "git commit -a")


查看文件`README.md`就能看到冲突信息的提示了，红色标记的内容就是用来标注不同分支的内容：

![冲突信息](http://upload-images.jianshu.io/upload_images/1713024-f2ed63357044b9e9.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

将内容改为以下内容就行了，然后重新提交就可以了。
>We will make a conflict!

      Mac-Pro:gitTest kys-1$ git add README.md
      Mac-Pro:gitTest kys-1$ git commit -m "comflict fixed"
      [master dad373b] comflict fixed

![SourceTree的内容](http://upload-images.jianshu.io/upload_images/1713024-ff1b5fdeed97512d.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

使用`git log --graph`可以看到分支合并图;

小憩一下：做到这一步是不是感觉自己好有成就感啊，下面的内容更精彩，继续...

###### 分支管理策略

上述合并分支，一大部分是在用`Fast forward`模式，这种模式又优点也有缺点，就是删除分支后会丢掉分支信息，这时，我们需要使用一下普通模式，即使用带有`--no-ff`的`git merge`,继续上述流程：新建分支dev->修改提交->切换分支->合并分支:

      Mac-Pro:gitTest kys-1$ git checkout -b dev  //***
      Switched to a new branch 'dev'
      Mac-Pro:gitTest kys-1$ git add README.md //***
      Mac-Pro:gitTest kys-1$ git commit -m "add new content" //***
      [dev 97676b7] add new content
       1 file changed, 1 insertion(+)
      Mac-Pro:gitTest kys-1$ git checkout master  //***
      Switched to branch 'master'
      Your branch is up-to-date with 'origin/master'.
      Mac-Pro:gitTest kys-1$ git merge --no-ff -m"merge with --no-ff"  dev   //***
      Merge made by the 'recursive' strategy.
       README.md | 1 +
       1 file changed, 1 insertion(+)

当然你也可以使用`git log --graph --pretty=oneline --abbrev-commit`命令查看分支历史：

![截图](http://upload-images.jianshu.io/upload_images/1713024-1a9a648937c3ca6b.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

![分支情况](http://upload-images.jianshu.io/upload_images/1713024-349504c5da95e507.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


补充几点：首先，在实践开发中，`master`分支是非常稳定的，仅用来发布新版本，但不在上面开发；其次，创建dev分支开发，等到版本发布的时候在合并到`master`分支即可；团队中每个人都有自己的分支，及时合并即可。

##### 添加Bug分支

前述：修复Bug在所难免，修复Bug可以通过创建一个新的临时分支修复，合并分支，删除临时分支。

情形：当你在`dev`分支上工作还没有完成，正赶上有`Bug`需要修复，这时你需要将手头上工作暂存一下，抓紧时间修复Bug；

      Mac-Pro:gitTest kys-1$ git status
        On branch dev
        Changes not staged for commit:
        (use "git add <file>..." to update what will be committed)
          (use "git checkout -- <file>..." to discard changes in working   directory)

	    modified:   README.md

	no changes added to commit (use "git add" and/or "git commit -a")


使用`git stash`暂存当前工作区：

      Mac-Pro:gitTest kys-1$ git stash
      Saved working directory and index state WIP on dev: 40be6d0   merge with --no-ff
      HEAD is now at 40be6d0 merge with --no-ff

使用SourceTree操作：

![暂存](http://upload-images.jianshu.io/upload_images/1713024-6d275ce8fb545465.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

这时，可以`git status`查看工作区，并可以创建Bug分支了；

 确定好在`master`分支修复Bug:
 
      Mac-Pro:gitTest kys-1$ git checkout master
      Switched to branch 'master'
      Your branch is up-to-date with 'origin/master'.
      Mac-Pro:gitTest kys-1$ git checkout -b bug
      Switched to a new branch 'bug'
      Mac-Pro:gitTest kys-1$ git add README.md
      Mac-Pro:gitTest kys-1$ git commit -m "fix bug"
        [bug 2c013d1] fix bug
       1 file changed, 1 insertion(+), 1 deletion(-)

![创建bug分支](http://upload-images.jianshu.io/upload_images/1713024-cab16d821cd81432.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

 在`bug`分支上修复完bug提交，就可以合并删除`bug`分支，如下：
 
        Mac-Pro:gitTest kys-1$ git checkout master
        Switched to branch 'master'
        Your branch is up-to-date with 'origin/master'.
        Mac-Pro:gitTest kys-1$ git merge --no-ff -m "merge bug" bug
        Merge made by the 'recursive' strategy.
       README.md | 2 +-
       1 file changed, 1 insertion(+), 1 deletion(-)
        Mac-Pro:gitTest kys-1$ git branch -d bug
        Deleted branch bug (was 2c013d1).


 SourceTree操作：

![合并分支](http://upload-images.jianshu.io/upload_images/1713024-af76476fc398423e.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


![bug修复完成](http://upload-images.jianshu.io/upload_images/1713024-a5678bad8d1e5284.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

Bug修复完，我们需要将暂存区的分支拿出来了，切换到`dev`分支，使用`git stash list`命令查看：

      Mac-Pro:gitTest kys-1$ git stash list
      stash@{0}: WIP on dev: 40be6d0 merge with --no-ff


工作区还在，可以使用两个命令恢复：一个是`git stash apply`（恢复后，stash内容不删除）还需用`git stash drop`删除；另一个是使用`git stash pop`恢复的同时将stash内容也删除了：

      Mac-Pro:gitTest kys-1$ git stash pop
        On branch dev
        Changes not staged for commit:
      (use "git add <file>..." to update what will be committed)
      (use "git checkout -- <file>..." to discard changes in working   directory)

	    modified:   README.md

      no changes added to commit (use "git add" and/or "git commit -a")
      

SourceTree操作： 

![暂存区恢复](http://upload-images.jianshu.io/upload_images/1713024-45be91b4679df8c4.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

##### Feature分支

开发过程中，每需要开发一个新功能都需要添加一个分支，最后合并删除，流程同上述`bug`分支相同，不再赘述。

强行删除分支，例如：你在新的分支下开发功能，后期开会取消这个功能，你只能将这个分支上的内容全强制删除，如下：

      Mac-Pro:gitTest kys-1$ git branch -d feature
       error: The branch 'feature' is not fully merged.
      If you are sure you want to delete it, run 'git branch -D feature'.

接下来使用`git branch -D feature`命令强制删除：

      Mac-Pro:gitTest kys-1$ git branch -D feature
      Deleted branch feature (was 5f7e86c).

 SourceTree操作：

![正常删除提示](http://upload-images.jianshu.io/upload_images/1713024-602cf111d8bcd855.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

![强制删除](http://upload-images.jianshu.io/upload_images/1713024-a6d409480f1a31cf.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

##### 标签管理

作用：标签作为版本库的快照，能够在需要的时候很快找到对应的版本库；

在指定提交节点上添加标签，默认的标签是打在最新提交的节点上的,使用`git tag`命令添加标签，使用`git show`命令查看标签信息:

      Mac-Pro:gitTest kys-1$ git tag V1.0  //***
      Mac-Pro:gitTest kys-1$ git tag    //***
        V1.0
      Mac-Pro:gitTest kys-1$ git show V1.0  //***
      commit d0fc2232015ae6737273fa5e94310bcfd4ef231c
      Author:  <40188217@qq.com>
      Date:   Wed Apr 13 07:45:48 2016 +0800

        add a tag

      diff --git a/README.md b/README.md
      index 9082c3c..52ef851 100644
      --- a/README.md
      +++ b/README.md
      @@ -9,4 +9,4 @@ Try a new way!
       Try doing a different thing!
       Try again!
       I am developing!!!
      -
      +I want to make a tag!


SourceTree操作：

![创建标签](http://upload-images.jianshu.io/upload_images/1713024-2019740a98b2d1a6.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


![标签显示](http://upload-images.jianshu.io/upload_images/1713024-1b2103cc7d25103f.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

![打开标签](http://upload-images.jianshu.io/upload_images/1713024-ae937c53205e6223.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

使用`git tag -d <tagname>`命令删除本地标签，使用`git push origin <tagname>`命令可以推送一个本地标签，使用`git push origin --tags`命令可以推送全部未推送过的本地标签，使用`git push origin :refs/tags/<tagname>`命令可以删除一个远程标签,操作如下：

      Mac-Pro:gitTest kys-1$ git tag -d V1.0
      Deleted tag 'V1.0' (was 6462641)
	推送标签到远程：
      Mac-Pro:gitTest kys-1$ git push origin V1.0  //***
      Counting objects: 5, done.
      Delta compression using up to 8 threads.
      Compressing objects: 100% (4/4), done.
      Writing objects: 100% (5/5), 497 bytes | 0 bytes/s, done.
      Total 5 (delta 2), reused 0 (delta 0)
      To https://github.com/123sunxiaolin/gitTest.git
       * [new tag]         V1.0 -> V1.0

推送本地未推送的标签：

      Mac-Pro:gitTest kys-1$ git push origin --tags  //***
       Counting objects: 1, done.
       Writing objects: 100% (1/1), 154 bytes | 0 bytes/s, done.
       Total 1 (delta 0), reused 0 (delta 0)
      To https://github.com/123sunxiaolin/gitTest.git
       * [new tag]         V0.1 -> V0.1

删除已推送的标签，首先，删除本地的标签,其次，删除远程的标签：

      Mac-Pro:gitTest kys-1$ git tag -d V0.1  //***
      Deleted tag 'V0.1' (was 98817ff)
      Mac-Pro:gitTest kys-1$ git push origin :refs/tags/V0.1  //***
      To https://github.com/123sunxiaolin/gitTest.git
       - [deleted]         V0.1
       

SourceTree操作：点击标签，右击选择删除即可，勾选`删除所有远程标签`可以删除远程标签，操作非常直观：

![删除本地标签](http://upload-images.jianshu.io/upload_images/1713024-a6f8f74788081750.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

###### 忽略特殊文件

在Git工作目录下，有一些配置文件我们不想提交到远程仓库，这时我们可以使用Git提供的`.gitignore`文件；

忽略文件的原则是：忽略操作系统自动生成的文件，比如缩略图等，
忽略编译生成的中间文件、可执行文件等，忽略你自己的带有敏感信息的配置文件，比如存放口令的配置文件。

首先，在Git目录下，使用`git touch .gitignore`创建`. gitignore `文件，然后编辑：

      Mac-Pro:gitTest kys-1$ touch .gitignore
      Mac-Pro:gitTest kys-1$ ls -ah
      .		.DS_Store	.gitignore
      ..		.git		README.md

![忽略文件](http://upload-images.jianshu.io/upload_images/1713024-ae4e819b08a38d90.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

如果你对配置文件还不甚了解，就看一下[现成配置列表](https://github.com/github/gitignore);

常用规则：

>/mtk/ 过滤整个文件夹
*.zip 过滤所有.zip文件
/mtk/do.c 过滤某个具体文件

gitignore还可以指定要将哪些文件添加到版本管理中,只是在文件前加一个`!`即可：
>!*.zip
!/mtk/one.txt         

其中，`!/mtk/one.txt `只需要管理/mtk/目录中的one.txt文件，这个目录中的其他文件都不需要管理，说到这想必大家都明白了。

唯一的区别就是规则开头多了一个感叹号，Git会将满足这类规则的文件添加到版本管理中。

最后一步就是将`. gitignore `文件推送到远程仓库即可。

##### 写在最后

在开发过程中，很多人纠结版本管理使用Git命令好呢，还是SourceTree客户端好呢，其实这没有明确的答案，完全取决于自己来，可以说两种方式各有利弊，大家可以尝试两者结合使用，效果应该会更好。

 写到这，终于可以舒一口气了，最后，附上自己写文章时的截图，希望大家给点掌声以资鼓励。

![截图集](http://upload-images.jianshu.io/upload_images/1713024-b0b53c41c0c8b85a.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


>从4月8日开始写，到今天才写完，将近耗时一周，中途遇见不少问题，还好，都坚持下来了，做程序猿就得拼啊，不然怎么能攻城狮啊！
