---
title: OH MY ZSH
date: 2016-11-21 21:33:00
tags:
- zsh
- iterm2
categories:
- Technical Posts
---

![完成图](http://imglf1.nosdn.127.net/img/RE1iZDZ5cUNZaTd4VzNtSFlGTm5CZVdHMjMwU2ZySnVScCtuTFdFYUZ6cndUZXJqUGxvclJBPT0.png?=imageView&thumbnail=500x0&quality=96&stripmeta=0&type=jpg%7Cwatermark&type=2)

只是一点想说的废话吧，这个配置教程是需要有一些`linux`命令行的基础，然后需要了解下`OSX`这个系统，系统文件是隐藏状态这个属性。因为是针对我个人的一个技术纪录，所以仅作参考。
<!-- more -->

本帖基于[此贴](https://laoshuterry.gitbooks.io/mac_os_setup_guide/content/4_ZshConfig.html)进行讲解：

**1.安装[iterm2](http://iterm2.com/);**

**2.通过[Homebrew](http://iterm2.com/)下载安装`zsh`，将zsh设置成系统默认shell，以代替bash**:

```
brew install zsh

sudo vi /etc/shells

#按i进行插入操作，在文末添加

/usr/local/bin/zsh

#按ESC，保存退出，如果没有反应，看是不是输入没有切换成英文

:wq!

chsh -s /usr/local/bin/zsh
```

然后重启iterm2

**3.利用`Homebrew`安装curl(或者wget)：**

```
brew install curl

brew install wget
```
```
curl: curl -L http://install.ohmyz.sh | sh

wget: wget --no-check-certificate http://install.ohmyz.sh -O - | sh
```

**4.修改配色**

```
vi .zshrc

#将ZSH_THEME="XXX"改为ZSH_THEME="agnoster"

#按ESC，保存退出，如果没有反应，看是不是输入没有切换成英文

:wq!
```

**5.选用字体，才能是箭头的图标字体正确显示**

*Meslo LG M DZ Regular for Powerline.tff*

> [字体参考文章的第六部分](http://www.jianshu.com/p/7de00c73a2bb)



结语：额，其实也没有那么难的，只是因为对于linux相关文件操作的命令不熟悉，而且涉及到的技术点又有点多，所以一开的摸索非常痛苦。但是后来稍微深入了解下，慢慢熟悉起来各种配置啥的，感觉也就没啥问题了。哎，真是麻烦的世界呢。

我还是原来的那个想法，真理是有的，但语言的阻隔，想法的不同，导致我们在追求真理的道路上，各自摸黑前进，有各自不兼容，创造了奇奇怪怪的语言，运用了稀奇古怪的方法，这使得后人学习起来，分外艰难。解决一个问题，就要做好了解这一整套系统运作的决心，因为后生并不知晓这到底是积重难返造成的历史遗留问题，还是说一个不经意留下的bug。

哎，为何这么多天坑。

> 20161116修订说明：

> 第2条：需要在管理员权限下才能执行（sudo vi /etc/shells）

> 第5条：新加了字体部分链接补充