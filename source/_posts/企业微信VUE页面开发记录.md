---
title: 企业微信VUE页面开发记录
date: 2021-09-04 19:26:26
tags: 微信踩坑
---

&emsp;&emsp;之前一直在开发企业微信侧边栏的网页，做一个展示的单页，涉及到信息的展示，以及点击相关项，可以对第三方进行快速回复的功能，怎么说呢，之前做过微信公众号内嵌网页的开发，所以还算是驾轻就熟，不过饶是如此，还是踩了不少插件不兼容、浏览器兼容性的坑，让人有苦说不出。
<!-- more -->
&emsp;&emsp;当时开发时间紧张，没有怎么系统性的梳理，现在还是准备分为三部日志进行记录，分别是：**企业微信VUE页面开发记录**、**微信公众号页面启用微信支付分的调用记录**、**微信扫二维码登录记录**。微信开发的通用点，就是先后台设置请求域名白名单，通过授权系统，携带code进行回调授权域名这个流程，个人觉得这个授权的流程还是比较不错的。
&emsp;&emsp;我将通过**代码展示**、**问题分享**、**参考资料**三个部分来进行记录这次开发之旅。当然咯，因为这个页面的功能比较单一，但是依赖了比较多的第三方工具，找问题、排除问题的时间耗费比较多，企业微信后台设置域名这一块就不做过多展开记录，按照文档流程操作就可以，这里只关注企业微信api调用、调试的代码。

## 代码
### entWxApi.js
```
// https://open.work.weixin.qq.com/api/doc/90000/90135/92124
// enterpriseWx 企业微信开发
// 封装 enterpriseWx底层接口 

import wx from 'wx'
import CONSTANTS from '../constants/index'
const { ENT_WX_APP_ID, ENT_WX_AGENT_ID } = CONSTANTS

const EntWxApi = {
  entWxConfig: body => {
    let { timestamp, nonceStr, signature, jsApiList } = body 
    return new Promise(resolve => {
      try {
        wx.config({
          beta: true, // 必须这么写，否则wx.invoke调用形式的jsapi会有问题
          debug: false,
          appId: ENT_WX_APP_ID,
          timestamp,
          nonceStr,
          signature,
          jsApiList
        })
        wx.ready(res => {  
          resolve({ result: true, res })
        })
        wx.error(res => {
          resolve({ result: false, res })
        })
      } catch (res) {
        resolve({ result: false, res })
      }
    })
  },

  // 通过agentConfig接口注入权限验证配置
  // https://work.weixin.qq.com/api/doc/90000/90136/94313
  entWxAgentConfig: body => {
    let { timestamp, nonceStr, signature, jsApiList } = body 
    return new Promise(resolve => {
      try {
        wx.agentConfig({
          corpid: ENT_WX_APP_ID,
          agentid: ENT_WX_AGENT_ID,
          timestamp,
          nonceStr,
          signature,
          jsApiList,
          success: function(res) {
            // 回调
            resolve({ result: true, res })
          },
          fail: function(res) {
            if (res.errMsg.indexOf('function not exist') > -1) {
              alert('版本过低请升级')
            }
          }
        })
        // wx.ready(res => {
        //   // console.log('wx config agent success', res)
        //   resolve({ result: true, res })
        // })
        // wx.error(res => {
        //   resolve({ result: false, res })
        // })
      } catch (res) {
        resolve({ result: false, res })
      }
    })
  },
  entWxGetContext: () => {
    return new Promise(resolve => {
      try {
        wx.invoke(
          'getContext', 
          res => {
            // 从微信侧小程序返回时会执行这个回调函数
            if (res.err_msg == 'getContext:ok') {
              // 返回成功
              resolve({ result: true, res })
            } else {
              // 返回失败
              resolve({ result: false, res })
            }
          }
        ) 
      } catch (res) {
        resolve({ result: false, res })
      }
    })
  },

  // 判断当前客户端版本是否支持指定JS接口
  entWxCheckJsApi: body => {
    let { jsApiList } = body 
    return new Promise(resolve => {
      try {
        wx.checkJsApi({
          jsApiList,
          success: res => {
            resolve({ result: true, res })
          },
          cancel: res => {
            resolve({ result: false, res })
          },
          fail: res => {
            resolve({ result: false, res })
          }
        }) 
      } catch (res) {
        resolve({ result: false, res })
      }
    })
  },

  // 获取外部联系人的联系详情
  // https://pay.weixin.qq.com/wiki/doc/apiv3/payscore.php?chapter=29_1&index=1
  entWxGetCurExternalContact: () => {
    return new Promise(resolve => {
      try {
        wx.invoke(
          'getCurExternalContact', 
          res => { 
            // 从微信侧小程序返回时会执行这个回调函数
            if (res.err_msg == 'getCurExternalContact:ok') {
              // 返回成功
              resolve({ result: true, res })
            } else {
              // 返回失败
              resolve({ result: false, res })
            }
          }
        )
      } catch (res) {
        resolve({ result: false, res })
      }
    })
  },

  entWxSendChatMessage: body => {
    return new Promise(resolve => {
      try {
        wx.invoke(
          'sendChatMessage',
          {
            msgtype: 'text', //消息类型，必填
            // enterChat: true //为true时表示发送完成之后顺便进入会话，仅移动端3.1.10及以上版本支持该字段
            text: {
              content: body.text //文本内容
            }
          },

          res => {
            // 从微信侧小程序返回时会执行这个回调函数
            if (res.err_msg == 'getContext:ok') {
              // 返回成功
              resolve({ result: true, res })
            } else {
              // 返回失败
              resolve({ result: false, res })
            }
          }
        )
        wx.ready(res => { 
          resolve({ result: true, res })
        })
        wx.error(res => {
          resolve({ result: false, res })
        })
      } catch (res) {
        resolve({ result: false, res })
      }
    })
  }
}
export default EntWxApi

```

### xl.js

```
// 进一步封装企业微信的操作
import Service from './../service/index'
import EntWxApi from './entWxApi'
const Xl = {
  // 初始化企业微信
  XlEntWxInit: async jsApiList => {
    // 调用wx.agentConfig之前，必须确保先成功调用wx.config.
    // 注意：从企业微信3.0.24及以后版本（可通过企业微信UA判断版本号），无须先调用wx.config，可直接wx.agentConfig
    // 1.先判断企业微信的版本，低于3.0.24，即需要wx.config，然后wx.agentConfig
    // 2.高于等于3.0.24的，只需要wx.agentConfig

    // - wx.config
    const XlEnsureWxConfigResultInfo = await Xl.XlEnsureWxConfig(jsApiList)
    if (!XlEnsureWxConfigResultInfo.result) return 

    // - wx.agentConfig
    const XlEnsureWxAgentConfigResultInfo = await Xl.XlEnsureWxAgentConfig(jsApiList)
    if (!XlEnsureWxAgentConfigResultInfo.result) return 

    //  - 校验 企业微信接口是否存在
    const entWxCheckJsApiResultInfo = await EntWxApi.entWxCheckJsApi({ jsApiList })
    console.log('entWxCheckJsApiResultInfo ====', entWxCheckJsApiResultInfo)
    if (!entWxCheckJsApiResultInfo.result) {
      return { result: false, res: entWxCheckJsApiResultInfo.res }
    }
    return { result: true, res: entWxCheckJsApiResultInfo.res }
     
  },
  // 通过wxConfig
  XlEnsureWxConfig: async jsApiList => {
    let body = { url: encodeURIComponent(window.location.href) }
    let getConfigSignatureResultInfo = await Service.getEnterpriseConfigSignature(body)

    if (!getConfigSignatureResultInfo.result) {
      return { result: false, res: getConfigSignatureResultInfo.data }
    }
    let { timestamp, noncestr, signature } = getConfigSignatureResultInfo.data || ''

    const configParams = { timestamp, nonceStr: noncestr, signature, jsApiList }
    // - 调用微信Api config 配置
    const wxConfigResultInfo = await EntWxApi.entWxConfig(configParams)
    if (!wxConfigResultInfo.result) {
      return { result: false, ...wxConfigResultInfo.data }
    }
    return { result: true }
  },
  // 通过wxAgentConfig
  XlEnsureWxAgentConfig: async jsApiList => {
    let body = { url: encodeURIComponent(window.location.href) }
    let getAgentConfigSignatureResultInfo = await Service.getEnterpriseAgentConfigSignature(body)

    if (!getAgentConfigSignatureResultInfo.result) {
      return { result: false, res: getAgentConfigSignatureResultInfo.data }
    }
    let { timestamp, noncestr, signature } = getAgentConfigSignatureResultInfo.data || ''

    const configParams = { timestamp, nonceStr: noncestr, signature, jsApiList }
    // - 调用微信Api config 配置
    let wxAgentConfigResultInfo = await EntWxApi.entWxAgentConfig(configParams)
    if (!wxAgentConfigResultInfo.result) {
      return { result: false, ...wxAgentConfigResultInfo.data }
    }
    return { result: true }
  },

  // 获取当前外部联系人userid
  // 参考链接一： [聊天工具栏接口](https://work.weixin.qq.com/api/doc/90000/90136/91789)
  // 参考链接二：[获取当前外部联系人userid](https://work.weixin.qq.com/api/doc/90000/90136/91799)
  XlGetEntWxUserId: async () => {
    // - 微信初始化
    const businessViewResultInfo = await Xl.XlEntWxInit(['getContext', 'getCurExternalContact', 'sendChatMessage'])
    if (!businessViewResultInfo.result) return

    //  校验入口
    const entWxGetContextResultInfo = await EntWxApi.entWxGetContext()
    if (!entWxGetContextResultInfo.result) {
      return { result: false, res: entWxGetContextResultInfo.res }
    }

    // - 获取当前客服联系人的userid
    let getCurExternalContactResultInfo = await EntWxApi.entWxGetCurExternalContact()
    return { result: true, res: getCurExternalContactResultInfo.res }
  },
  // 发送快捷回复消息
  XlSendChatMessage: async body => {
    const entWxSendChatMessageResultInfo = await EntWxApi.entWxSendChatMessage(body)
    console.log('entWxSendChatMessageResultInfo:', entWxSendChatMessageResultInfo)
  }
}
export default Xl

```

### vue页面中调用 xl.js

```
 
// - 获取企业微信当前客服的userid
      const entWxUserIdResultInfo = await Xl.XlGetEntWxUserId()
      if (!entWxUserIdResultInfo.result) {
        return this.$message.error(entWxUserIdResultInfo.res.err_msg)
      } 
      const { userId } = entWxUserIdResultInfo.res || '' 
      // - 获取当前联系人在xl系统内加密过的用户id

      if (!userId) return this.$message.error('未从企业微信获取到有效ID')

      const params = { code: userId }

      const entWxCurrentUserResultInfo = await this.fetchEntWxCurrentUserInfo(params) 
      if (!entWxCurrentUserResultInfo.result) {
        this.desc = entWxCurrentUserResultInfo.msg
        this.$message.error(entWxCurrentUserResultInfo.msg)
        return
      }
      const { encryptionId } = entWxCurrentUserResultInfo.data
      this.encryptionId = encryptionId
 
 // 伪代码 后续根据获取到的第三方用户的id(encryptionId)，再调用自己的后台服务接口，和自己的平台用户进行绑定
```

## 问题分享

### 企业微信自己的问题
- **调试的时候需要企业微信的环境**，不要在本地浏览器上调试——因为一些api根本调用不了，没有任何返回数据
- 因为**PC端浏览器ie内核，版本太低**，一些高级es6语法无法支持，导致报错而让页面在PC端无法显示，mac和iphone上倒是能正常显示页面（当时被领导钉了好久这个问题）
- 调用第三方的时候报`no permission`的错误，这个可能是**企业微信后台要配置**什么；也可能是因为**基础代码写的有问题**，建议按照例子看看，我就是`wx.agentConfig`的代码写错，我还是写了`wx.ready`所以PC端不知为啥，`debug: false`就老是获取不到`encryptionId`，当时真的苦恼极了，仔细看文档，逐行比较代码才发现是自己写错。
![2021.8.25日企业微信 mac浏览器内核](/images/2021090401.png) 
![2021.8.25日企业微信 windows浏览器内核](/images/2021090402.png) 

### vconsole的问题
高版本vconsole@**3.9.1**不被企业微信浏览器支持，换了个低版本vconsole@**3.3.4**的就好了


## 参考资料
- [企业微信 JS-SDK 自建应用踩坑指南](https://www.jianshu.com/p/8a0b5fecfd90)
- [企业微信自建应用踩坑指南](https://juejin.cn/post/6844904132122247182)
- [企业微信调试H5页面](https://blog.csdn.net/bitcser/article/details/115868609)
- [本地如何调试企业微信的H5页面](https://developers.weixin.qq.com/community/develop/doc/00004220a2c4109292ba4354c5d800)
- [问一下企业微信浏览器各端内核是什么呢，完全支持ES6吗？](https://developers.weixin.qq.com/community/enterprisewechat/doc/000a0afd944260dbd15b7e6ed59c00?jumpto=comment&commentid=000680c697863810fa5bd893656c)
- [企业微信浏览器报错Object.entries is not a function](https://www.cnblogs.com/xbzhu/p/13453827.html)
- [微信及企业微信内嵌浏览器内核信息及H5跑分数据-企业微信开发](https://blog.csdn.net/Ant_wiki/article/details/107981374)
- [在线检测浏览器类型和版本号](https://i.ewceo.com/browser.html)


## 后记
> 写得有点粗糙，不过还是先坚持把提纲写下来吧，时间久了，怕自己也不记得更细节的东西，还是要配置一些图片进行说明。