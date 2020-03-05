---
title: node.js 并行处理promise
typora-copy-images-to: ../../source/assets
typora-root-url: ../../source
date: 2020-03-02 09:48:51
updated:
tags:
  - node.js
categories:
  - node.js
---

### 情景解释

之前发现微信小程序云开发功能提供了免费的云函数，但是对于node.js没有接触，只使用了`wx.getcontent()`获取用户标识，后端还是使用springboot，最近开始学习nodejs进行后端开发。

当我们发送一组异步请求/从数据库取出一组满足要求的数据，要怎么处理那，例如: 有一个数组`a`，我们在数据库表`user`中寻找字段`t1`等于的每一行数据。

我用微信小程序的本地调试工具对几种方法进行了测试，并记录了响应时间。

<!-- more -->

### 方法测试

其中test()函数是从数据库中取出一个数据，

`test2()`是在每个`promise对象`的`resolve中进行数据处理，时间等于取三个数据，

`test3()`和`test4()`都是先保存一个`promise数组`，然后对这个`promise数组`进行操作，这两种方法的时间都和取出一个数据是相同的。

综合这几种方式，`test3()`的实现是最令人满意的

```javascript
// 云函数入口文件
const cloud = require('wx-server-sdk')

cloud.init({
  env: 'wxbase1-bdeeac'
})

const db = cloud.database()

const t1Array = ['1', '2', '3']
const t2Array = []

//471ms
async function test() {
  let b = []
  let a = await db.collection('user')
    .where({
      t1: '1'
    })
    .get().then(res => {
      console.log(res)
      b.push(res)
    })
  return b
}

// 1441ms
async function test2() {
  let a1 = []
  for (let t1 of t1Array) {
    await db.collection('user')
      .where({
        t1: t1
      }).get().then(res => {
        a1.push(res.data[0])
      })
  }
  return a1
}

// 474ms
async function test3() {
  const tasks = t1Array.map(t1 => {
    return temp = db.collection('user')
      .where({
        t1: t1
      }).get()
  })
  return (await Promise.all(tasks)).reduce((acc, cur) => {
    return {
      data: acc.data.concat(cur.data),
      errMsg: acc.errMsg,
    }
  })
}

//496毫秒
async function test4() {
  let a1 = []
  const tasks = t1Array.map(t1 => {
    return temp = db.collection('user')
      .where({
        t1: t1
      }).get()
  })
  for(let task of tasks){
    const temp = await task
    a1.push(temp.data[0])
  }
  return a1
}


// 云函数入口函数
exports.main = async(event, context) => {
  return test3()
}
```

其中数组的`reduce(function sum())`方法，传递一个累加器函数，其中第一个参数相当于`sum`，第二个参数相当于`每一个数据`。

如`test3()`处理的操作，可以将每一次返回的数组转化为一个数组