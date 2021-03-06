---
slug: 2021-12-18-js微任务、宏任务与eventloop
title: 2021-12-18-js微任务、宏任务与eventloop
author: Sebastian
author_title: 学生/码农/撸狗
author_url: https://github.com/Sebastian520
author_image_url: https://avatars.githubusercontent.com/u/61676200?s=40&v=4
description: 请输入描述
tags: [面经, js]
# activityId: 相关动态 ID
# bvid: 相关视频 ID（与 activityId 2选一）
# oid: oid
---

<!-- truncate -->
### 微任务与宏任务的区别
---
这个就像去银行办业务一样，先要取号进行排号。

一般上边都会印着类似：“您的号码为XX，前边还有XX人。”之类的字样。

因为柜员同时职能处理一个来办理业务的客户，这时每一个来办理业务的人就可以认为是银行柜员的一个宏任务来存在的，当柜员处理完当前客户的问题以后，选择接待下一位，广播报号，也就是下一个宏任务的开始。

所以多个宏任务合在一起就可以认为说有一个任务队列在这，里边是当前银行中所有排号的客户。

任务队列中的都是已经完成的异步操作，而不是说注册一个异步任务就会被放在这个任务队列中，就像在银行中排号，如果叫到你的时候你不在，那么你当前的号牌就作废了，柜员会选择直接跳过进行下一个客户的业务处理，等你回来以后还需要重新取号

而且一个宏任务在执行的过程中，是可以添加一些微任务的，就像在柜台办理业务，你前边的一位老大爷可能在存款，在存款这个业务办理完以后，柜员会问老大爷还有没有其他需要办理的业务，这时老大爷想了一下：“最近P2P爆雷有点儿多，是不是要选择稳一些的理财呢”，然后告诉柜员说，要办一些理财的业务，这时候柜员肯定不能告诉老大爷说：“您再上后边取个号去，重新排队”。

所以本来快轮到你来办理业务，会因为老大爷临时添加的“理财业务”而往后推。

也许老大爷在办完理财以后还想 再办一个信用卡？或者 再买点儿纪念币？

无论是什么需求，只要是柜员能够帮她办理的，都会在处理你的业务之前来做这些事情，这些都可以认为是微任务。

这就说明：你大爷永远是你大爷

在当前的微任务没有执行完成时，是不会执行下一个宏任务的。


所以就有了那个经常在面试题、各种博客中的代码片段：
```
setTimeout(_ => console.log(4))

new Promise(resolve => {
  resolve()
  console.log(1)
}).then(_ => {
  console.log(3)
})

console.log(2)
```
setTimeout就是作为宏任务来存在的，而Promise.then则是具有代表性的微任务，上述代码的执行顺序就是按照序号来输出的。

所有会进入的异步都是指的事件回调中的那部分代码

也就是说new Promise在实例化的过程中所执行的代码都是同步进行的，而then中注册的回调才是异步执行的。

在同步代码执行完成后才回去检查是否有异步任务完成，并执行对应的回调，而微任务又会在宏任务之前执行。

所以就得到了上述的输出结论1、2、3、4。

再看一个例子：

```
Promise.resolve().then(()=>{ 
  console.log('Promise1')   
  setTimeout(()=>{ 
    console.log('setTimeout2') 
  },0) 
}); 
setTimeout(()=>{ 
  console.log('setTimeout1') 
  Promise.resolve().then(()=>{ 
    console.log('Promise2')     
  }) 
},0); 
console.log('start'); 
 
// start 
// Promise1 
// setTimeout1 
// Promise2 
// setTimeout2 
```


### 宏任务和微任务都是异步任务

* 同步任务： 在主线程上排队执行的任务，只有前一个任务执行完毕，后一个任务才会执行；

* 异步任务：不进入主线程，而是进入任务队列，只有当主线程当前任务执行结束，才会读取任务队列中消息；（一次取完）

消息队列中存放的一定是异步任务，这些异步任务一定要等到主线程中当前任务执行完毕后才会执行；

