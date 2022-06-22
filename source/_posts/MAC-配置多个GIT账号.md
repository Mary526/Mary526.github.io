---
title: MAC 配置多个GIT账号
date: 2019-08-25 17:11:24
tags:
---
  
# 使用背景
在同一台电脑上，可能往往需要使用不同身份针对不同代码库提交代码，所以需要使用ssh来进行帮助用户，管理多个托管git账号。
<!-- more -->
# 笔者环境、托管的代码库
- 【环境】macOS Mojave 10.14.6
- 【代码库】github.com
- 【代码库】aliyun.com

# 所有涉及的指令
想快速查找命令的，可以浏览这部分，快速定位所需代码。至少七个命令，就可以无碍管控多账号git提交了呀！

序号|命令|备注
:--:|--:|:--
1|`git config --global user.name "xxx"`|配置全局用户名，如Github上注册的用户名 
2|`git config --global user.email "yyy@mail.com"`|配置全局邮箱，如Github上配置的邮箱
3|`git conifg --list`|查看已配置的git列表
4|`ssh-keygen -t rsa -C "yyy@mail.com"`|根据账户生成密钥
5|`ssh-add ~/.ssh/id_rsa_github`|将GitHub私钥添加到本地
6|`touch config`|创建config文件 
7|`ssh -T git@github.com`|测试SSH配置
8|`pbcopy < ~/.ssh/id_rsa_github.pub`|复制公钥
8|`cat  ~/.ssh/id_rsa_github.pub`|查看并复制公钥


# 流程
## 准备工作
记得在刚接触git的以后，可能是教程都在图简便吧，都是让我们将用户的git信息，设置为全局信息。但是
现在需要根据ssh来配置，那就需要先取消全局git用户信息，然后根据私钥生成对应的git信息

``` shell script
git conifg --list //查看已配置的git列表 
```

``` shell script
git config --global --unset user.name // 清空默认的用户名
git config --global --unset user.email // 清空默认的邮箱
```
 
## 给不同的git账户生成ssh-key
首先需要进入保存密钥的目录。这一点很重要，生成的密钥一定需要保存在此文件夹。文件的层级有时候会影响一些问题。


``` shell script
cd ~/.ssh //进入目录，该目录下保存生成的秘钥
ssh-keygen -t rsa -C "yyy@mail.com"  //根据邮箱生成秘钥
```

git是根据邮箱来统计提交数目的。输入后按enter键
输入完成后，会提示用户给即将生成的密钥取名字，名字可以根据自己方便进行自定义，比如：`id_rsa_github`、`id_rsa_aliyun`等等，然后剩下的命令可以直接回车，不用
输入密码，直到密钥生成。

## 添加私钥到本地
``` shell script 
ssh-add ~/.ssh/id_rsa_github` // 将Github私钥添加到本地
```

为了校验本地是否添加成功，可以使用`ssh-add -l`命令进行查看，如果想删掉添加到本地的私钥，直接删掉ssh文件夹，然后重新开始第一步的操作，也就是重新生成秘钥。

## 对本地秘钥进行配置
由于添加了多个密钥，所以需要对这多个密钥进行管理。在.ssh目录下新建一个config文件：
``` shell script
 touch config
```

内容如下：
``` shell script
# 公司代码库 20190825 
 Host aliyun
 HostName code.aliyun.com
 User XXX
 IdentityFile ~/.ssh/id_rsa_aliyun
 
 # XXX 个人github代码库 20190825
 Host github
 HostName github.com
 User XXX
 IdentityFile ~/.ssh/id_rsa_github
```

## 公钥添加到托管网站
这步略过

## 测试配置

这时候，可以测试一下配置是否成功，测试命令使用别名。例如，对于GitHub，本来应该使用的测试命令是：

``` shell script
ssh -T git@code.aliyun.com
```
在config文件中，给GitHub网站配置的别名就是github，所以直接使用别名，就是

``` shell script
ssh -T git@aliyun
```

至此，就可以使用不同账号进行提交了。原来也确实疑惑过，为什么不同电脑提交，但是自己个人的信息却不一样。现在配置了ssh一是增加了安全性，一是方便了提交的操作。不过后续还要跟进一下windows上的ssh配置。







## 参考文档
- [Mac 上配置多个git账号](https://www.jianshu.com/p/698f82e72415) - [Kandy_JS](https://www.jianshu.com/u/132996324c3c)
- [Mac下配置多个Git账户](https://segmentfault.com/a/1190000016269686) - [程序员不止程序猿](https://segmentfault.com/u/liugui1993)
- [一个电脑对应两个Github帐户](https://www.jianshu.com/p/bae25a63f220) - [ForeverCy](https://www.jianshu.com/u/9dddce8d6f63)
- [https://my.oschina.net/wolx/blog/755595](https://my.oschina.net/wolx/blog/755595) - [卧龙小](https://my.oschina.net/wolx)
