---
title: 群晖搭建 chevereto 图床
date: 2022-11-18 15:24:39
tags:
- 群晖
- NAS
---

# 效果
✅ 博客上能展示图片，图片最好能存储在自己硬盘。
演示地址：[https://hutianhua.com:6395/](https://hutianhua.com:6395/)
![iShot_2022-11-18_16.34.42.gif](https://hutianhua.com:6395/images/2022/11/18/iShot_2022-11-18_16.34.42.gif)

<!-- more -->

提纲：
- 效果
- 面向人群
- 介绍版本
- 安装流程
- 参考资料

# 面向人群
主要是方便存放自己博客的插图，自己苦于图片的引入已经很久了，将图片放在博客里面，不便于博客在换UI框架时的快速迁移，所以之前自己更多是偏向于写纯文字的影评，而为了更纯粹地写作，自己维护着一套博客倒逼着我去掌握这些知识，想来有点小小的无语。

攻略暂时仅供自己记录，如果有帮到屏幕前的您各位，就当缘分了，不足和遗漏之处欢迎指出，这边尽量完善，方便大家都能拥有自己的自定义图床。

推荐掌握的前置知识点，这样可以支持外网访问图床：
1. 需购买至少一个域名
2. 掌握手动生成单域名基于[ZeroSSL](https://zerossl.com/)作为运营商的SSL证书
3. 群晖NAS上配置DDNS
4. 群晖NAS上为服务配置证书

自己博客是基于[hexo](https://hexo.io/)作为UI框架，借由[github pages](https://pages.github.com/)的自动部署，在[name.com](https://www.name.com/)上购买域名，借由群晖NAS上的DDNS来动态解析图床网站地址。

# 介绍版本
- 群晖NAS一台
- 华硕路由器
- `chevereto@1.4.1`
- Web Station
- MariaDB 10
- PHP 7.3
- PHP 7.4
- PHP 8.0
- phpMyAdmin
- Apache HTTP Server 2.4

# 安装流程
## 一、安装NAS套件
去NAS上安装下图安装红框里的这些套件：
![iShot_2022-11-21_19.09.38.png](https://hutianhua.com:6395/images/2022/11/21/iShot_2022-11-21_19.09.38.png)

## 二、下载chevereto网页代码并部署
下载地址：[https://github.com/rodber/chevereto-free](https://github.com/rodber/chevereto-free)
在tag里下载版本为1.4.1的代码，难点在于这里自己找的是1.4.1的版本。尝试过当时最新的1.6.0的代码可是死活启动不了程序，后来也懒得纠结，好像是PHP版本调用不到函数导致的错误吧，总之，先跑起来再说。
![iShot_2022-11-22_10.22.22.gif](https://hutianhua.com:6395/images/2022/11/22/iShot_2022-11-22_10.22.22.gif)   
并将图床代码解压并上传至web文件夹下
![image.png](https://hutianhua.com:6395/images/2022/11/22/image.png)

## 三、套件Web Station添加PHP自定义配置文件
Web Station > 脚本语言设置 > PHP > 点击"新增" > 配置文件名称：chevereto - PHP 7.4；描述：PHP 7.4 Profile for chevereto；启用PHP缓存； PHP版本：PHP7.4 ;扩展名全勾选 > 保存
![imagea834317c108d06ef.png](https://hutianhua.com:6395/images/2022/11/22/imagea834317c108d06ef.png)

## 四、套件Web Station添加网页服务
Web Station > 网页服务门户 > 新增 > 虚拟主机 > 基于端口 > Network里填写http和https的端口 > Backend里填写文件的地址
![image128593d4ad06c17c.png](https://hutianhua.com:6395/images/2022/11/22/image128593d4ad06c17c.png)
![image9ff26d94b59ba330.png](https://hutianhua.com:6395/images/2022/11/22/image9ff26d94b59ba330.png)

## 五、配置phpMyAdmin并登陆
点击phpMyAdmin并登陆，如果不知道密码，可以在Maria DB里面点击重置密码，然后获取密码，这里如果数据库连接不上，需要启用TCP/IP连接，参见参考资料3
![image68cfee218f02c3c7.png](https://hutianhua.com:6395/images/2022/11/22/image68cfee218f02c3c7.png)
![image87037f1d93125f84.png](https://hutianhua.com:6395/images/2022/11/22/image87037f1d93125f84.png)
![imagef7949fda4b431592.png](https://hutianhua.com:6395/images/2022/11/22/imagef7949fda4b431592.png)

## 六、打开phpMyAdmin，创建数据库
![imageaaa80f582f2d35e5.png](https://hutianhua.com:6395/images/2022/11/22/imageaaa80f582f2d35e5.png)

## 七、完成chevereto的安装
打开`群晖IP地址:端口号`进行安装，比如`192.168.50.100:6391`，上面http的端口号填写的是6391，这里浏览器打开`192.168.50.100:6391`，填入数据库信息进行安装
![imagedcfecb704f75471f.png](https://hutianhua.com:6395/images/2022/11/22/imagedcfecb704f75471f.png)
![imageabe6b96c1f4f0ea2.png](https://hutianhua.com:6395/images/2022/11/22/imageabe6b96c1f4f0ea2.png)
![imagec06109ac71c98e36.png](https://hutianhua.com:6395/images/2022/11/22/imagec06109ac71c98e36.png)

至此安装完毕，请开始在内网环境下使用自己的图床吧～
如果需要博客上显示图片，还请继续往下看：

# 外网访问配置

## 一、群晖配置DDNS
这里需要一个固定的域名用来解析群晖主机的IP地址，点击 控制面板 > 外部访问 > DDNS ，然后按照下图添加配置，并获取到**主机名称**，例如`test.synology.me`。
![image8812baab66e87f37.png](https://hutianhua.com:6395/images/2022/11/22/image8812baab66e87f37.png)

## 二、域名服务器录入服务
这里以name.com的域名管理后台为例子，点击 DNS管理 > 新增记录
![image687ec54b855566f2.png](https://hutianhua.com:6395/images/2022/11/22/image687ec54b855566f2.png)
## 三、端口转发
这里以华硕路由器后台为例子，点击 外部网络 > 端口转发，然后按照下图添加配置。
![image4755cd8cc0c558a4.png](https://hutianhua.com:6395/images/2022/11/22/image4755cd8cc0c558a4.png)
这样就完成外网支持访问图床图片的最终效果咯！撒花🎉 

# 参考资料
- [【NAS】搭建 Chevereto 图床 & Typora 上传指南 2020-10-26](https://www.cnblogs.com/easonshi/p/13877870.html)
- [群晖搭建 chevereto 图床 2020-05-20](https://post.smzdm.com/p/a3gvxnon/)
- [Getting Error SQLSTATE[HY000] [2002] Connection refused on NAS Synology](https://stackoverflow.com/questions/45254222/getting-error-sqlstatehy000-2002-connection-refused-on-nas-synology)
- [No More Mixed Messages About HTTPS](https://blog.chromium.org/2019/10/no-more-mixed-messages-about-https.html)