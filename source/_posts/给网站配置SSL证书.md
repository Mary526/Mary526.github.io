---
title: 单域名和泛域名生成SSL证书
date: 2022-11-24 14:32:50
tags:
- ssl
- https
- ZeroSsl
---
 
# 需要实现的效果
&emsp;&emsp;希望自己的NAS门户网站https链接前面都能有小锁，而不是提示"不安全网站"。尝试了很多方式去满足自己的需求，这里记录下对自己而言，行之有效的手工生成方式。攻略暂时仅供自己记录，如果有帮到屏幕前的您各位，就当缘分了，不足和遗漏之处欢迎指出，这边尽量完善，方便大家都能掌握生成SSL证书的思路。
&emsp;&emsp;网上有许多攻略是基于用户已经有SSL证书的前提下，偏重于记录如何给自己的服务配置SSL证书；本文的重点在于如何去**获取**这项"资源"，查漏补缺，大家阔以结合着看，兼听则明嘛。

![imagea365528f27a8d9fd.png](https://hutianhua.com:6395/images/2022/11/24/imagea365528f27a8d9fd.png)
<!-- more -->
![image1460ab250ff23e84.png](https://hutianhua.com:6395/images/2022/11/24/image1460ab250ff23e84.png)


提纲：
- 需要实现的效果
- 环境
- 网页手工生成SSL证书
- 命令行手工生成SSL证书
- 已解决的卡壳问题
- 遗留问题
- 参考链接
- 
&emsp;&emsp;[ZeroSSL](https://zerossl.com/)作为一个证书签发机构，免费版下支持最多生成三个域名的证书，如果有进阶需求，需要付费咯。

![image.png](https://hutianhua.com:6395/images/2022/11/24/image.png)


# 环境
- 操作系统：macOS Monterey
- 华硕路由器固件版本： 384.18
- Homebrew：3.6.5
- acme.sh：v3.0.5
- OpenSSL：3.0.5
- 群晖系统：7.1.1

# 操作步骤
## 网页手工生成SSL证书
&emsp;&emsp;创建ZeroSSL账号，点击"创建新证书"，按照要求生成证书文件，需要注意的是，这里验证，是需要到域名运营商后台，DNS解析里添加对应的解析信息。
![image17e2219ee1899111.png](https://hutianhua.com:6395/images/2022/11/24/image17e2219ee1899111.png)
&emsp;&emsp;这里需要注意的是，自己域名是在name.com上购买，域名服务器设置的是默认的`n1.name.com`，所以笔者是在name.com的DNS管理记录中增加了下面这一条的校验信息，为了zeroSSL对域名的校验通过。
![image1d46da6fae79b736.png](https://hutianhua.com:6395/images/2022/11/24/image1d46da6fae79b736.png)

## 命令行手工生成SSL证书
&emsp;&emsp;上面那种只能生成三个证书，可自己至少有八个服务（而且都想配置二级域名，类似于`https://video.xxx.com/`、`https://nas.xxx.com/`）需要配置证书（当然是手工啦，jio本那种还要学习一下怎么自动化），那怎么办呢，网上有看到网友提供的代生成SSL证书服务（需要提供DNS API）——[泛域名证书申请 https://ssl.ioiox.com/](https://ssl.ioiox.com/)，可惜仅限于三个域名解析服务商（腾讯云 / 阿里云 / Cloudflare ），而笔者选择name.com服务商很显然不在其中，并且自己那时候还不知道怎么修改域名服务器(这样就可以用阿里云来解析)，当然，如果那时候改成阿里云来解析域名，也就损失了下面自我探索的乐趣咯，有得有失吧。

&emsp;&emsp;所以那时只好狠下心来研究怎么用[acme.sh](https://github.com/acmesh-official/acme.sh/wiki/%E8%AF%B4%E6%98%8E)来申请ZeroSSL泛域名证书。

### 安装acme.sh
```
curl  https://get.acme.sh | sh
```
&emsp;&emsp;安装成功后可以输入`acme.sh -v`查看acme的当前版本
``` 
acme.sh -v
https://github.com/acmesh-official/acme.sh
v3.0.5
```
&emsp;&emsp;原本是要用acme来执行命令签发ssl证书，这里有两个问题卡住了我，一个是命令（`acme.sh --issue --dns dns_ali -d *.hutianhua.com`）报错，提示openssl的版本太低，某些函数因缺失或改变而导致命令执行不了，网上搜了下，大概是mac自带openssl和brew安装的openssl版本不一致；另一个问题，因为自己域名解析商选的是name.com，网上的攻略贴大多是以阿里云、华为云为例的代码，所以当时折腾了很久，才稍微明白了一点域名服务商、域名解析服务商、acme之间的关系。
&emsp;&emsp;所以在执行主线任务——用acme签发ssl证书——前，笔者需要先解决这两个"拦路虎"，有遇见类似问题的盆友可以参考下，没有遇见的盆友可以跳过这两步。
### 【选看】更新openssl配置环境变量
=>[关于mac自带的openssl和brew安装的openssl冲突 2021-09-29](https://cloud.tencent.com/developer/article/1883983)
```
brew install openssl
```
&emsp;&emsp;如果是使用的[oh-my-zsh](https://ohmyz.sh/)作为mac的终端，那么需要在文件.zshrc中配置openssl的路径；如果是默认的终端，就是在.bash_profile中配置，这里需要操作者灵活判断下。
``` 
vi .zshrc
```
&emsp;&emsp;然后在文件中加入下面的变量
```
export PATH=/usr/local/bin:$PATH   #这个很重要!!!
export PATH="/usr/local/opt/openssl@1.1/bin:$PATH"  #就是你brew安装路径
export LDFLAGS="-L/usr/local/opt/openssl@1.1/lib"
export CPPFLAGS="-I/usr/local/opt/openssl@1.1/include"
```
![imagef5cd1be85a5d72c3.png](https://hutianhua.com:6395/images/2022/11/24/imagef5cd1be85a5d72c3.png)
&emsp;&emsp;查看下openssl的版本，是3.X.X就没问题了（截止发博文日哈，攻略一般有实效性，请灵活判断下），iMac自带的openssl@1就不对
``` 
openssl version
```
![imagee7dbb3e09168604e.png](https://hutianhua.com:6395/images/2022/11/24/imagee7dbb3e09168604e.png)
### 【选看】配置DNS API
&emsp;&emsp;acme.sh针对不同DNS运营商做出了不同定制化命令行[How to use DNS API](https://github.com/acmesh-official/acme.sh/wiki/dnsapi)，这里以"name.com"为例子。
![image.png](https://hutianhua.com:6395/images/2022/11/25/image.png)
#### 1.登陆[https://www.name.com/zh-cn/account/settings/security](https://www.name.com/zh-cn/account/settings/security)打开NAME.COMAPI访问权限，账户设置 > 安全 > 安全设置 > NAME.COM API设置
![imagec088b46f196c0a1e.png](https://hutianhua.com:6395/images/2022/11/25/imagec088b46f196c0a1e.png)
#### 2.登陆[https://www.name.com/account/settings/api](https://www.name.com/account/settings/api)创建name.com的DNS API
![imagef977602922b9e373.png](https://hutianhua.com:6395/images/2022/11/25/imagef977602922b9e373.png)

### 在account.conf中保存用户名和令牌
``` 
cd .acme.sh
vi account.conf

export Namecom_Username="testuser"
export Namecom_Token="xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx"
```
![image6efa243ff7d9bd80.png](https://hutianhua.com:6395/images/2022/11/25/image6efa243ff7d9bd80.png)

### 对单域名`example.com`生成证书
``` 
acme.sh --issue --dns dns_namecom -d example.com -d www.example.com
```
### 对泛域名`*.example.com`生成证书
```
acme.sh --issue --dns dns_namecom -d 'example.com' -d '*.example.com'
```
### NAS配置证书
&emsp;&emsp;在.acme.sh/example.com下可以找到生成的证书
![image1151e4f981871bf7.png](https://hutianhua.com:6395/images/2022/11/25/image1151e4f981871bf7.png)
1.控制面板 > 安全性 > 证书 > 新增 > 新增新证书
![imagebd1d128124471a71.png](https://hutianhua.com:6395/images/2022/11/25/imagebd1d128124471a71.png)
![iShot_2022-11-25_11.27.50.png](https://hutianhua.com:6395/images/2022/11/25/iShot_2022-11-25_11.27.50.png)
点下一步，然后将生成的证书.key、.cer上传，点击保存，就可以了
![iShot_2022-11-25_11.28.30.gif](https://hutianhua.com:6395/images/2022/11/25/iShot_2022-11-25_11.28.30.gif)
2.控制面板 > 安全性 > 证书 > 设置
给自己NAS上的各种服务，都配置上SSL证书
![img_1.png](https://hutianhua.com:6395/images/2022/11/25/img_1.png)
大功告成，完结，撒花🎉～

# 已解决的卡壳问题
## 生成证书提示要加白名单
![image7739bf8d4be88eed.png](https://hutianhua.com:6395/images/2022/11/25/image7739bf8d4be88eed.png)
&emsp;&emsp;这里在name.com后台将本地IP加入白名单，并且开API安全令的开关，就可以了。
## 生成证书提示`zsh: no matches found: *.example.com`
&emsp;&emsp;域名注意要加单引号，自己参照示例写成了`acme.sh --issue --dns dns_namecom -d example.com -d *.example.com`就会报这个错误，后来改成`acme.sh --issue --dns dns_namecom -d 'example.com' -d '*.example.com'`就可以生成证书了。
![iShot_2022-11-25_11.35.48.png](https://hutianhua.com:6395/images/2022/11/25/iShot_2022-11-25_11.35.48.png)

# 遗留问题
## ASUS路由器上的[Let's Encrypt]到底能不能生成ssl证书呢？
&emsp;&emsp;自己的电信宽带实测80/443端口外网访问不到，可为什么看到网上有网友说可以生成证书呢？当时在这里折腾了很久，[官方攻略](https://www.asus.com.cn/support/FAQ/1034294)提供了这种渠道（虽然那时候的自己并没有完全理解最后一句话的含义"请确保您的路由器可以通过 Internet 的 80端口发出网域验证和证书续订"）；[网上](https://www.52asus.com/forum.php?mod=viewthread&tid=14925&highlight=Encrypt)也有说可以用这种方式生成证书，但自己实测不行。
&emsp;&emsp;"头铁"硬是升级了路由器固件（原来的380没有这个功能）版本尝试（还导致原有的科学上网不能用，啊这是另外一个故事了），但这个[Let's Encrypt]执行命令后长时间没有反应，就很尴尬，不知道是升级的固件问题，还是[Let's Encrypt]的问题，又或者是宽带的问题，头大。

## acme.sh怎么更新证书呢？
这个待第一次证书到期后，笔者再琢磨琢磨吧。

# 参考链接
- [使用acme.sh申请ZeroSSL泛域名证书 2021-06-09](https://www.zjpc.cc/713.html) 重要
- [利用 acme.sh 申请 ZeroSSL 泛域名证书的图文教程 2022-09-16](https://www.vvso.cn/xlbk/21753.html)
- [Mac下 .bash_profile 和 .zshrc 两者之间的区别 2019-01-14](https://blog.csdn.net/weixin_30488085/article/details/97485990)
- [acme.sh 官方使用说明中文版|github](https://github.com/acmesh-official/acme.sh/wiki/%E8%AF%B4%E6%98%8E)
- [acme.sh 如何使用DNS API|github](https://github.com/acmesh-official/acme.sh/wiki/dnsapi)
- [openssl tags|github](https://github.com/openssl/openssl/tags)
- [关于mac自带的openssl和brew安装的openssl冲突 2021-09-29](https://cloud.tencent.com/developer/article/1883983)
- [acme.sh安装ssl 2021-12-09](https://blog.51cto.com/u_15454232/4774968)
- [如何使用ASUS路由器的功能来创建HTTPS的有效证书？ 2021-12-30](https://www.asus.com.cn/support/FAQ/1034294)
- [Synology 服务状态](https://www.synology.cn/zh-cn/support/synology_service)
- [[求助] 最近华硕的Let's Encrypt服务停掉了吗 2020-2-25](https://www.52asus.com/forum.php?mod=viewthread&tid=14925&highlight=Encrypt)
- [梅林固件路由器部署SSL证书支持HTTPS访问 2019-02-28](https://www.ioiox.com/archives/15.html)
- [群晖NAS应用程序设置独立门户域名教程 2019-04-17](https://www.ioiox.com/archives/33.html)
- [使用acme.sh生成let’s encrypt泛域名证书 2020-04-03](https://liushiming.cn/article/generate-ssl-cert-by-acme.html)







