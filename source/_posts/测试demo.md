---
title: 测试demo
date: 2018-12-20 15:14:25
tags:
---
# 前言
这是一份XX前端快速适应公司环境的指南。
***

<!-- more -->

# 流程图
```flow
st=>start: Start|past:>http://www.google.com[blank]
e=>end: End:>http://www.google.com
op1=>operation: My Operation|past
op2=>operation: Stuff|current
sub1=>subroutine: My Subroutine|invalid
cond=>condition: Yes
or No?|approved:>http://www.google.com
c2=>condition: Good idea|rejected
io=>inputoutput: catch something...|request

st->op1(right)->cond
cond(yes, right)->c2
cond(no)->sub1(left)->op1
c2(yes)->io->e
c2(no)->op2->e
```

```flow
st=>start: 开始
e=>end: 结束
op1=>operation: 点击小程序图标  
op2=>operation: 进入默认页，零元购活动列表页
op3=>operation: 弹出提示“关注公众号”
cond=>condition: 是否登录
c2=>condition: 去关注

st->op1->op2->cond
cond(yes,right)->e
cond(no)->op3->c2
```
 

# 思维导图


***
# 参考链接：
- [Hexo博客显示markdown流程图](https://www.bingyublog.com/2018/08/22/Hexo%E5%8D%9A%E5%AE%A2%E6%98%BE%E7%A4%BAmarkdown%E6%B5%81%E7%A8%8B%E5%9B%BE/)
- [为 Hexo 增加流程图解析功能](http://wewelove.github.io/fcoder/2017/09/06/markdown-flowchart/)
- [在hexo中写时序图和UML流程图](http://www.suwey.net/2016/03/27/hexo-sequece-flow-chart/)
- [在 Hexo 中使用思维导图](https://hunterx.xyz/use-mindmap-in-hexo.html)
- [如何让你的HEXO博客支持手写流程图？](https://www.liuyude.com/How_to_make_your_HEXO_blog_support_handwriting_flowchart.html)
- [flowchart.js 语法参考](http://flowchart.js.org/)