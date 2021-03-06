---
title: 学好Mac常用命令，助力iOS开发 # 这是标题
categories:   # 这里写的分类会自动汇集到 categories 页面上，分类可以多级
- iOS学习记录
tags: # 这里写的标签会自动汇集到 tags 页面上
- Mac
- iOS
- 终端命令行
---

![厚重·技术](http://upload-images.jianshu.io/upload_images/1713024-272e63bd5f8526dc.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
### 序言
在iOS开发的过程中，更多地注重iOS开发的效率，熟练使用Mac终端操作的常用命令，可以让你更好的游刃于iOS繁重的开发中。本文着重介绍常用的Mac终端基本指令和快捷键，更加适合一些IOS新手学习和了解。

### Mac OS X的文件系统

 Mac OS X本身是Unix内核的，采用Unix的多用户系统，采用Unix文件系统，所有文件都放在根目录`/Users`下面，不存在Windows下的盘符概念,用户登录系统后，自己的用户目录下一般有公共、图片、文稿、下载、音乐、影片、站点、桌面、资源库,OS X为中文用户做了国际化。
 
打开终端,输入`ls`，你会看到真正的目录名称：`Desktop、Documents、Downloads、Library、Movies、Music、Pictures、Public、Sites`。继续在终端中输入cd /，切换到根目录，键入`ls`，这样基本就可以看到Unix目录的全貌。
如下图所示：

![终端截图](http://upload-images.jianshu.io/upload_images/1713024-0014395618dae46a.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

其中，一些常用文件位置集锦：

      1.驱动所在位置      /Systme/Library/Extensions ；
      2.桌面的位置        /User/用户名/Desktop ；
      3.文件通配符为星号 * 
      4.在 Unix系统中是区别大小写字符的，A.txt 不等于 a.txt。 
       根目录标志 / 不是可有可无，cd /System 表示转到跟目录下的   
       System中，而cd System 表示转到当前目录下的 System中 。

### Mac终端常用的基本命令

 列出文件：`ls 参数 目录名`    例: 查看驱动目录:`ls/System/Library/Extensions`,(参数 -w 显示中文，-l 详细信息， -a 包括隐藏文件);

转换目录：`cd`   例如：转换到驱动目录 ` cd /System/Library/Extensions`;

 建立新目录：`mkdir 目录名` 例如：在驱动目录下建一个备份目backup，`mkdir /System/Library/Extensions/backup`;
在桌面上建一个备份目录 backup,  `mkdir /User/用户名/Desktop/backup`;

拷贝文件:`cp 参数 源文件 目标文件`    例：想把桌面的Natit.kext 拷贝到驱动目录中  `cp -R /User/用户名/Desktop/Natit.kext /System/Library/Extensions`;参数R表示对目录进行递归操作，kext在图形界面下看起来是个文件，实际上是个文件夹。
把驱动目录下的所有文件备份到桌面backup
`cp -R /System/Library/Extensions/* /User/用户名/Desktop/backup`;

删除文件:`rm 参数 文件`   例：想删除驱动的缓存  `rm -rf /System/Library/Extensions.kextcache 、rm -rf /System/Library/Extensions.mkext`，参数`－rf `表示递归和强制，执行了 `rm -rf /` 你的系统就瘫痪了；

移动文件:`mv 文件`,例：想把`AppleHDA.Kext` 移到桌面 `mv /System/Library/Extensions/AppleHDA.kext /User/用户名/Desktop`,
想把`AppleHDA.Kext` 移到备份目录中`mv /System/Library/Extensions/AppleHDA.kext /System/Library/Extensions/backup`;

文本编辑:`nano 文件名`   例:编辑`natit Info.plist ` ` nano /System/Library/Extensions/Natit.kext/Info.plist`;

目录操作：

命令名      |功能描述   | 使用举例
:-----------:  | :-----------:  | :-----------: 
mkdir | 创建一个目录   | mkdir dirname
rmdir | 删除一个目录  | rmdir dirname
mvdir | 移动或重命名一个目录  | mvdir dir1 dir2
cd | 改变当前目录   | cd dirname
pwd | 显示当前目录的路径名  | pwd
ls | 显示当前目录的内容     | ls -la


文件操作：

命令名      |功能描述   | 使用举例
:-----------:  | :-----------:  | :-----------: 
cat | 显示或连接文件    | cat filename
od | 显示非文本文件的内容  | od -c filename
cp | 复制文件或目录 | cp file1 file2
rm | 删除文件或目录   | rm filename
mv | 改变文件名或所在目录   |  mv file1 file2
find | 使用匹配表达式查找文件     |  find . -name "*.c" -print
file | 显示文件类型     |  file filename


选择操作：

命令名      |功能描述   | 使用举例
:-----------:  | :-----------:  | :-----------: 
head | 显示文件的最初几行     |  head -20 filename
tail | 显示文件的最后几行  | tail -15 filename
cut | 显示文件每行中的某些域 | cut -f1,7 -d: /etc/passwd
colrm |  从标准输入中删除若干列   | colrm 8 20 file2
diff | 比较并显示两个文件的差异   |  diff file1 file2
sort |  排序或归并文件      |  sort -d -f -u file1
uniq | 去掉文件中的重复行     | uniq file1 file2
comm | 显示两有序文件的公共和非公共行    |   comm file1 file2
wc | 统计文件的字符数、词数和行数     |  wc filename
nl | 给文件加上行号       |   nl file1 >file2


进程操作：

命令名      |功能描述   | 使用举例
:-----------:  | :-----------:  | :-----------: 
ps |  显示进程当前状态     |  ps u
kill | 终止进程  | kill -9 30142


时间操作:

命令名      |功能描述   | 使用举例
:-----------:  | :-----------:  | :-----------: 
date |  显示系统的当前日期和时间      |  date
cal |  显示日历| cal 4 2016
time | 统计程序的执行时间    | time a.out


网络与通信操作:

命令名      |功能描述   | 使用举例
:-----------:  | :-----------:  | :-----------: 
telnet |  远程登录      |  telnet hpc.sp.net.edu.cn
rlogin |  远程登录   | rlogin hostname -l username
rsh |  在远程主机执行指定命令   | rsh f01n03 date
ftp |  在本地主机与远程主机之间传输文件    | ftp.sp.net.edu.cn
rcp |  在本地主机与远程主机之间复制文件    | rcp file1 host1:file2
mail |  阅读和发送电子邮件     | mail
write |  给另一用户发送报文    | write username pts/1
mesg |  允许或拒绝接收报文   | mesg n
ping |  给一个网络主机发送回应请求   | ping hpc.sp.net.edu.cn


Korn Shell 命令 :

命令名      |功能描述   | 使用举例
:-----------:  | :-----------:  | :-----------: 
history |  列出最近执行过的几条命令及编号      |  history
r |  重复执行最近执行过的 某条命令    | r -2
alias |  给某个命令定义别名   | alias del=rm -i
unalias |  取消对某个别名的定义    | unalias del
rcp |  在本地主机与远程主机之间复制文件    | rcp file1 host1:file2
mail |  阅读和发送电子邮件     | mail
write |  给另一用户发送报文    | write username pts/1
mesg |  允许或拒绝接收报文   | mesg n


其他命令 :

命令名      |功能描述   | 使用举例
:-----------:  | :-----------:  | :-----------: 
uname |  显示操作系统的有关信息      |  uname -a
clear | 清除屏幕或窗口内容    | clear
alias |  给某个命令定义别名   | alias del=rm -i
unalias |  取消对某个别名的定义    | unalias del
who |  显示当前所有设置过的环境变量     | who
whoami |  显示当前正进行操作的用户名     | whoami
tty |  显示终端或伪终端的名称    | tty
du |  查询磁盘使用情况      | du -k subdir
stty  |  显示或重置控制键定义   | stty -a
df/tmp |  显示文件系统的总空间和可用空间   | -
w |    显示当前系统活动的总信息| -


显示资源库:

###### 方法一：
`显示`
在“终端”中输入下面的命令：
`chflags nohidden ~/Library/`;
`隐藏`
在“终端”中输入下面的命令：
`chflags hidden ~/Library/`;

###### 方法二：
打开Finder，菜单中选择前往按住option键就会显示资源库项(每次打开都需要重复操作一次)。


Finder显示隐藏文件
`显示隐藏文件`

在“终端”中输入下面的命令：
`defaults write com.apple.finder AppleShowAllFiles -bool true
killall Finder`;
`恢复隐藏文件`

在“终端”中输入下面的命令：
`defaults write com.apple.finder AppleShowAllFiles -bool false
killall Finder`.


Xcode卸载

在“终端”中输入下面的命令：
`sudo /Library/uninstall-devtools –mode=all`
为实际安装的目录，默认情况下Xcode安装在/Developer目录下，即可执行
`sudo /Developer/Library/uninstall-devtools –mode=all`;


在Finder标题栏显示完整路径
在“终端”中输入下面的命令：
`defaults write com.apple.finder _FXShowPosixPathInTitle -bool YES
killall Finder`


去掉窗口截屏的阴影
对窗口进行截屏的时候(Command-Shift-4, 空格)，得到的图片周围会自动被加上一圈阴影。如果你不喜欢这个阴影的效果，可以把它关掉。
在“终端”中输入下面的命令：
`defaults write com.apple.screencapture disable-shadow -bool true
killall SystemUIServer`;


强制Safari在新标签中打开网页
Safari是默认支持标签浏览的。但是，我们在页面上点击链接或者在其他应用程序中点击链接的时候，Safari往往是打开了一个新的窗口，导致页面上的Safari窗口过多，不好管理。通过下面这个小窍门，
我们可以让Safari默认是在一个新标签中打开网页。
在“终端”中输入下面的命令：
`defaults write com.apple.Safari TargetedClicksCreateTabs -bool true`;


改变截屏图片的保存位置
Mac OS提供了非常方便的截屏快捷键，可以让我们非常快速的对整个屏幕、部分屏幕或者应用程序窗口进行截屏。

不过，这个截屏功能有一个不足之处，就是只能将截屏图片保存到桌面。如果我们截取的图片特别多，就会让桌面显得特别凌乱。那有没有办法来修改截屏图片的默认保存位置呢？有。方法非常简单，只要在“终端” 中输入下面的命令就可以了。
`defaults write com.apple.screencapture location 存放位置
killall SystemUIServer`;

在输入命令的时候，将“存放位置”替换成真正的文件夹就可以了。例如，你希望存放到自己用户目录的Screenshots文件夹下，就输入
`defaults write com.apple.screencapture location ~/Screenshots`;

### Mac中常用的快捷键

Command+Tab                   任意情况下切换应用程序 - 向前循环

Shift+Command+Tab          切换应用程序 - 向后循环

Command+Delete               把选中的资源移到废纸篓

Shift+Command+Delete     清倒相关程序的废纸篓

Command+`                       同一应用程序多窗口间切换

Command+F                       呼出大部分应用程序的查询功能

Command+C/V/X                复制/粘贴/剪切

Command+N                       新建应用程序窗口

Command+Q                       退出当前应用程序，说明一下，所
有应用程序界面左上角都有红黄绿三个小图标，点击绿色扩展到最适合的窗口大小，黄色最小化，红色关掉当前窗口，但并没有退出程序。

 用Command+Q配合Command+Tab关闭应用程序最为迅速

Command+L                       当前程序是浏览器时，可以直接定位到地址栏

 Command+"+/-"                  放大或缩小字体 

Control+推出键                   显示关机对话框

Control+Space                    呼出Spotlight

Command+Space              切换输入法

### 写在最后

每一种终端开发都不能只局限在开发工具IDE上，往往操作系统的一些操作会带来意想不到的惊喜，还望大家细细体会。若大家想了解一下Mac系统常用操作，推荐[这篇文章](http://www.cnblogs.com/chijianqiang/archive/2011/08/03/2126593.html)给大家,若大家想学习Unix指令，[大家点击一下](http://wiki.ubuntu.org.cn/Unix%E5%91%BD%E4%BB%A4%E5%A4%A7%E5%85%A8)即可。
