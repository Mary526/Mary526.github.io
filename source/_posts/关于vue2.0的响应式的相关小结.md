---
title: 关于vue2.0的响应式的相关小结
date: 2018-11-28 14:07:39
tags: vue 
---

## **背景**

&emsp;&emsp;是在某次APP更新，需要完成订单里面，批量商品的评价功能。其实之前也写过类似的界面，不过因为需要评价的事物，是固定一个的，所以页面布局也是写死的，而现在是由后台动态传过来的未知数量，需要js动态生成待评价的商品模块。

## **问题分析**

&emsp;&emsp;根据VUE2.X版本：[**"受现代 JavaScript 的限制 (而且 Object.observe 也已经被废弃)，Vue 不能检测到对象属性的添加或删除。由于 Vue 会在初始化实例时对属性执行 getter/setter 转化过程，所以属性必须在 data 对象上存在才能让 Vue 转换它，这样才能让它是响应的"**](https://cn.vuejs.org/v2/guide/reactivity.html#%E6%A3%80%E6%B5%8B%E5%8F%98%E5%8C%96%E7%9A%84%E6%B3%A8%E6%84%8F%E4%BA%8B%E9%A1%B9)。

&emsp;&emsp;所以我在实现这个业务的过程中，就遇到了数据已经重新赋值了，但是dom没有刷新的问题。经过查询资料以及短暂的思考，大概思路如下：先获取到所有的商品数据，比如`productList`，然后每次修改的时候，深度克隆一个对象比如`tempList`，根据传过来的角标值修改`tempList`的属性，接着通过`Vue.$set`方法，将`tempList`赋值给`productList`，最后视图成功更新。

## **代码梳理**

### 定义深度克隆的方法

```javascript 

metheds: {
    // 深度克隆对象
    deepClone: function (original, target){
        var target = target || {};// 如果target为undefined或没传参，设置空对象
        for(var prop in original){// 遍历原对象
            if(original.hasOwnProperty(prop)){// 只拷贝对象内部，不考虑原型链
                if(typeof original[prop] === 'object'){// 引用值
                    if(Object.prototype.toString.call(original[prop]) === '[object Array]'){
                        target[prop] = [];// 处理数组引用值
                    }else{
                        target[prop] = {};// 处理对象引用值
                    }// 可以用三目运算符
                    this.deepClone(original[prop],target[prop]);// 递归克隆
                }else{// 基本值
                    target[prop] = original[prop];
                }   
            }
        }
        return target;
    }
}

```
 
### 获取到订单商品数据 
`bigRate`方法是在点击星星，然后给商品评分的方法。
  
````javascript
created: {
    this.init();
} 
methods: { 
    // 初始化数据
    init: function () {
        var _this = this;
        _this.productList =  _this.orderData.goods;
        _this.productList.forEach(function(item){
            item["score"] = 0; 
        });
    }
    // 点击评分
    bigRate: function (index2,index) {
        var _this = this;
        // console.log('第几个数据：'+index2+',几颗星：'+index);    
        // 涉及到 vm.items[indexOfItem] = newValue 索引式更新对象，不会更新视图的知识点，克隆对象后再赋值
        var m = _this.deepClone(_this.productList[index]);
        m.score = index2 + 1; 
        vm.$set(vm.productList, index, m) 
        // console.log(JSON.stringify(_this.orderData.goods)); 
    }
}

````
## **问题**
&emsp;&emsp;在写这篇博文的时候，我突然想起来，貌似前端代码还是不够严谨，因为后台传来的对象个数是动态的，现在数据量小，一个订单下面顶多挂十来个商品，如果是一百多个商品（后台在下单环节并没有做商品种类数量的限制），一是不知道现在页面会不会有问题；二是，如果如果要分页，就需要动态更新`productList`，要多加个事件。

&emsp;&emsp;刚就该问题，咨询了公司的产品，他回复是他自己也没具体思考过这个事情，不过据说某东的做法，就是如果商品过多，评论页会进行分页评论。然而打开APP，发现某东现在改版了，现在的业务是直接按照单个商品的种类进行评论的————那么问题来了，若是在某东上买十双鞋，难不成我要评论十次啊 :fearful:

## **小结**
&emsp;&emsp;周一开始关注VUE3.0的消息，正好关于这个问题进行了改进，点赞 :laughing:，很期待新版本的到来。

[&emsp;&emsp;"2.x的响应式是基于Object.defineProperty实现的代理，兼容主流浏览器和ie9以上的ie浏览器，能够监听数据对象的变化，但是监听不到对象属性的增删、数组元素和长度的变化，同时会在vue初始化的时候把所有的Observer都建立好，才能观察到数据对象属性的变化。
&emsp;&emsp;针对上面的问题，3.0进行了革命性的变更，采用了ES2015的Proxy来代替Object.defineProperty，可以做到监听对象属性的增删和数组元素和长度的修改，还可以监听Map、Set、WeakSet、WeakMap，同时还实现了惰性的监听，不会在初始化的时候创建所有的Observer，而是会在用到的时候才去监听。但是，虽然主流的浏览器都支持Proxy，ie系列却还是不兼容，所以针对ie11，vue3.0决定做单独的适配，暴露出来的api一样，但是底层实现还是Object.defineProperty，这样导致了ie11还是有2.x的问题。但是绝大部分情况下，3.0带来的好处已经能够体验到了。 
&emsp;&emsp;响应式方面，vue3.0做了实现机制的变更，采用ES2015的Proxy，不但解决了vue2.x中的问题，还是得性能有了进一步提升。虽然有一些兼容问题，但是通过适配的方式解决掉了。此外，还暴露了observable的api来创建响应式对象，可以替代掉event bus，来做一些跨组件的通信。"](https://juejin.im/post/5bb6185ff265da0abd352f4e)

&emsp;&emsp;还有个就是，我还没有深入了解深度克隆的含义，晚上回去要好好学习下这个概念。

-----
## **参考文章**
- [Plans for the Next Iteration of Vue.js](https://medium.com/the-vue-point/plans-for-the-next-iteration-of-vue-js-777ffea6fabf)
- [全面改革：解读vue3.0的变化](https://juejin.im/post/5bb6185ff265da0abd352f4e)
TODO 还有个深度克隆的文章没找到  
 <iframe src="//player.bilibili.com/player.html?aid=24037829&cid=40250255&page=1" scrolling="no" border="0" frameborder="no" framespacing="0" allowfullscreen="true"> </iframe>