---
title: iOS 企业版 证书相关问题
date: 2016-11-25 16:01:00
tags:
- IOS
- 企业
- 证书
categories:
- Technical Posts
---

> 这篇文章仅供自己记录

前面想说很多吐槽苹果的话，但终于忍住了，不想暴露自己的无知> <！

这大半年一直致力于移动端开发，但之前一直是潜心于开发，并没有想过发布APP，毕竟这是另外一个同事负责的，但不知为何，项目经理现在让我来打包。我擦咧，遇坑无数，所以这份文档不做很多详细说明，仅作为自己记忆的凭证。蛤蛤，感觉自己真像个心机婊～借着公司给的权限，大玩特玩23333。

**配置前提：**mbp(OSX 10.11)一台、Xcode8.0、企业版开发者账号（$299）、一个正在开发的APP（yb）

##### 正文：
- 因为公司新购置了一台Mbp，希望能够在新电脑上也进行开发和打包，所以特留下记录，以免过段时间忘掉了。
- Xcode8打包现在已经不需要配置code signning，可以参考这个链接：[Xcode 8 打包教程](http://qkxue.net/info/32031/Xcode-8)。

> 默认是一个全新的电脑，需要配置苹果开发等等证书...

##### 生成证书
打开Xcode，Xcode>Preferences，添加自己的苹果ID，这样开发网站上会为你的这台新Mac生成一个证书，下图是完成图；
![.p12](http://img.ngacn.cc/attachments/mon_201610/15/-blqxlQ7q5f-hlk5ZdT3cSm0-gx.png)

##### 获取配置文件
登陆[苹果开发者账号官网](https://developer.apple.com)下载相应证书及配置文件，红框中的三个文件都下载下来，前两个一个是苹果对胡天华`苹果账号`进行开发的允许证书，后一个是对开发者所使用的Mac的`授权证书`，第三个是所开发的APP的`配置文件`；
![配置文件](http://img.ngacn.cc/attachments/mon_201610/15/-blqxlQ7q5f-1lw4ZfT3cSt0-jm.png)
![](http://img.ngacn.cc/attachments/mon_201610/15/-blqxlQ7q5f-fzn1ZkT3cSun-nj.png)

##### 导入证书
导入`.p12证书`，这个我也没理解，反正就是要导入，导入了，才能在这台电脑上开发，这个证书的重要性凌驾在上面的所有证书上面。导入所有证书；打开钥匙串，在登陆选项，文件>导入项目；
![导出.p12证书](http://img.ngacn.cc/attachments/mon_201610/15/-blqxlQ7q5f-fnedZpT3cSp5-mc.png)
先导入`.p12证书`，[iOS证书(.p12)和描述文件(.mobileprovision)申请](http://ask.dcloud.net.cn/article/152)，我用的是9.28日新生成的那个，密码是123。存放文件夹：Topcheer>ios开发>证书 .p12；然后导入后两个证书，最后点击下那个APP配置文件，就可以了。中途我删掉了对个人开发授权的那个证书（9w3d65deb7)，Xcode还报错了的，如图：
![](http://img.ngacn.cc/attachments/mon_201610/15/-blqxlQ7q5f-cimuZgT3cS12w-q0.png)
这样就配置好开发的证书了：
![](http://img.ngacn.cc/attachments/mon_201610/15/-blqxlQ7q5f-cimuZgT3cS12w-q0.png)

##### 打包APP 
Product > Archive，后续步骤如下：
![](http://img.ngacn.cc/attachments/mon_201610/15/-blqxlQ7q5f-kdw0K2fT3cSxc-os.png)
![](http://img.ngacn.cc/attachments/mon_201610/15/-blqxlQ7q5f-jpwbK1iT1kSg4-9q.png)
![](http://img.ngacn.cc/attachments/mon_201610/15/-blqxlQ7q5f-k9ydKwT1kSg6-9n.png)
![](http://img.ngacn.cc/attachments/mon_201610/15/-blqxlQ7q5f-b4hdK1cT1kSg4-9q.png)
![](http://img.ngacn.cc/attachments/mon_201610/15/-blqxlQ7q5f-ayb6ZiT3cS12w-q0.png)

**参考链接：**
> [iOS 企业版 证书相关问题](http://www.lofter.com/blog/hutianhua?act=dashboardclick_20130514_04)
