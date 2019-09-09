---
title: MAC 配置多个GIT账号
date: 2019-08-25 17:11:24
tags:
---
提纲：
1。使用背景（aliyun、github)
2。所有的指令
3。配置步骤：
3.0 准备工作
3。1 对每个账户生成一对密钥
3。2 添加私钥到本地
3。3 对本地秘钥进行配置
3。4 公钥添加到托管网站
3。5 测试配置

4.参考连接 文档


# 使用背景
在同一台电脑上，可能往往需要使用不同身份针对不同代码库提交代码，所以需要使用ssh来进行帮助用户，管理多个托管git账号。

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

# 配置步骤
## 准备工作
记得在刚接触git的以后，可能是教程都在图简便吧，都是让我们将用户的git信息，设置为全局信息。但是
现在需要根据ssh来配置，那就需要先取消全局git用户信息，然后根据私钥生成对应的git信息
- `git conifg --list` 查看已配置的git列表
- `git config --global user.name`
  `git config --global user.email` 清空默认的用户名和邮箱
    
## 给不同的git账户生成ssh-key
首先需要进入保存密钥的目录。这一点很重要，生成的密钥一定需要保存在此文件夹。文件的层级有时候会影响一些问题。
`cd ~/.ssh` 进入目录，该目录下保存生成的秘钥
`ssh-keygent -t rsa -C "yyy@mail.com"` 根据邮箱生成秘钥，git是根据邮箱来统计提交数目的。输入后按enter键



