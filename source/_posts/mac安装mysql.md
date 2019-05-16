---
title: mac安装mysql
date: 2019-05-05 11:49:37
tags: mysql
categories: 后台
---

首先要知道你使用的Mac OS X是什么样的Shell， 
打开终端，输入：echo $SHELL 回车执行 
如果输出的是：csh或者是tcsh，那么你用的就是C Shell。 
如果输出的是：bash，sh，zsh，那么你的用的可能就是Bourne Shell的一个变种。 
Mac OS X 10.2之前默认的是C Shell。 
Mac OS X 10.3之后默认的是Bourne Shell。 
我的是bash：

输入：cd /usr/local/mysql，回车执行 
然后输入：sudo vim .bash_profile，回车执行 
需要输入root用户密码。sudo是使用root用户修改环境变量文件。

进入编辑器后，我们先按”i”，即切换到“插入”状态。就可以通过上下左右移动光标，或空格、退格及回车等进行编辑内容了，和WINDOWS是一样的了。

在文档的最下方输入：export PATH=${PATH}:/usr/local/mysql/bin

然后按Esc退出insert状态，并在最下方输入:wq保存退出(或直接按shift+zz，或者切换到大写模式按ZZ，就可以保存退出了)。

输入：source .bash_profile 回车执行，运行环境变量。 
再输入mysql命令，即可使用。

以后每次使用的话，先输入cd /usr/local/mysql进入mysql路径下才进得去。

如果使用navicat连接mysql连接不上，

大概意思就是无法加载身份验证插件”caching_sha2_password” 
解决 
打开系统偏好设置，找到mysql，点击Initialize Database。 
输入你的新密码。 
选择‘Use legacy password‘。 
重启mysql服务。

