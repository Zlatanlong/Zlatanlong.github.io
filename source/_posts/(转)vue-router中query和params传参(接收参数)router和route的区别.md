---
title: (转)vue-router中query和params传参(接收参数)router和route的区别
typora-copy-images-to: ../../source/assets
typora-root-url: ../../source
date: 2018-08-27 17:07:03
updated: 2018-08-27 17:07:03
tags:
  - vue-router
categories:
  - vue.js

---

今天做项目时踩到了vue-router传参的坑（query和params），所以决定总结一下二者的区别。

## 直接总结干货！！！

### 1.query方式传参和接收参数

<!-- more -->

```javascript
传参: 
this.$router.push({
        path:'/xxx',
        query:{
          id:id
        }
      })
  
接收参数:
this.$route.query.id
```

注意:传参是this.\$router,接收参数是this.\$route,这里千万要看清了！！！

**this.router和this.route有何区别？**

- 1.router为vuerouter实例，想要导航到不同，则使用router.push方法
- 2.$route为当前router跳转对象，里面可以获取name、path、query、params等

### 2.params方式传参和接收参数

```javascript
传参: 
this.$router.push({
        name:'xxx',
        params:{
          id:id
        }
      })
  
接收参数:
this.$route.params.id
```

注意:params传参，push里面只能是 name:'xxxx',不能是path:'/xxx',因为params只能用name来引入路由，如果这里写成了path，接收参数页面会是undefined！！！

另外，二者还有点区别，直白的来说query相当于get请求，页面跳转的时候，可以在地址栏看到请求参数，而params相当于post请求，参数不会再地址栏中显示

**vue的自学之路还得继续走，坑还会继续踩，下一个坑会是神马...**

> 转载自
>
>  https://segmentfault.com/a/1190000012735168