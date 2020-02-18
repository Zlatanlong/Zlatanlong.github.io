---
title: (转)vue+axios上传文件
typora-copy-images-to: ../../source/assets
typora-root-url: ../../source
date: 2018-08-20 17:07:03
updated: 2018-08-20 17:07:03
tags:
  - 体积优化
categories:
  - vue.js
---

对于单页应用，要在一个页面上为用户提供产品的所有功能，在这个页面加载的时候，首先要加载大量的静态资源，这个加载时间相对比较长。所以我们需要做一些相应的优化，减少响应时间，尽快把首屏显示出来。

<!-- more -->

### 1、压缩代码

无论是否为单页应用，代码的压缩都是要做的。
对于vue-cli生成的项目，在Webpack配置文件中使用了UglifyJsPlugin进行代码的压缩：

```javascript
//webpack.prod.conf.js
plugins: [
    ......
    new UglifyJsPlugin({
      uglifyOptions: {
        compress: {
          warnings: false
        }
      },
      sourceMap: config.build.productionSourceMap,
      parallel: true
    }),
    ......
```

### 2、框架和插件按需加载

对于项目中用到的一些UI框架，比如 Onsen UI 、Mint UI 等等。如果我们只使用框架的部分组件，那么可以不要引入整个框架，按需引入用到的组件。
以Mint UI为例：

#### // 引入全部组件  

```javascript
import Mint from 'mint-ui';  
import 'mint-ui/lib/style.css'  
Vue.use(Mint); 
```

#### // 按需引入部分组件  

```javascript
import { CellSwipe } from 'mint-ui';  
Vue.component(CellSwipe.name, CellSwipe);
```


对于一些插件，比如表单验证插件Vuelidate，如果只是在个别组件中用的到，也可以不要在main.js里面引入，而是在组件中按需引入

#### // main.js中全部引入

```javascript
import Vue from 'vue'
import Vuelidate from 'vuelidate'
Vue.use(Vuelidate)
```

#### // 组件中按需引入

```javascript
import { validationMixin } from 'vuelidate'
```

### 3、框架和插件从cdn中引入

在开发过程中，我们其实不会要去更改这些第三方库，所以可以把它们放到cdn中，不参与打包。
在 index.html 中使用cdn引入

```html
<script src="https://cdn.example.com/vue/2.5.3/vue.js"></script>
<script src="https://cdn.example.com/vuex/3.0.1/vuex.min.js"></script>
<script src="https://cdn.example.com/axios/0.17.1/axios.min.js"></script>
```


在webpack.config.js（webpack.base.config.js）中添加externals，表示这些文件可以被引用，但不参与打包。

```javascript
externals: {
  'vue': 'Vue',
  'vuex': 'Vuex',
  'axios': 'axios'
}
```


这样配置之后，我们依然可以用`import Vuex from 'vuex'`来引入模块。

具有外部依赖`(external dependency)`的 bundle 可以在各种模块上下文(module context)中使用，例如 CommonJS, AMD, 全局变量和 ES2015 模块。

externals也支持string、array、object、function和regex等各种语法，详情参见webpack Externals中文文档。

### 4、路由懒加载

官方文档在此，更详细的内容参见文档。

路由懒加载也就是 把不同路由对应的组件分割成不同的代码块，然后当路由被访问的时候才加载对应组件。
结合 Vue 的异步组件和 Webpack 的代码分割功能，轻松实现路由组件的懒加载。
在router中，我们平时是这样引入组件的：

```javascript
import Foo from './Foo.vue'
```

文档中指出，如下定义一个能够被 Webpack 自动代码分割的异步组件

```javascript
const Foo = () => import('./Foo.vue')
```

在路由配置中什么都不需要改变，只需要像往常一样使用 Foo：

```javascript
const router = new VueRouter({
  routes: [
    { path: '/foo', component: Foo }
  ]
})
```

暂时先记下这四种方法，优化这件事，还要在项目里好好钻研~

> 版权声明：本文为CSDN博主「latency_cheng」的原创文章，遵循 CC 4.0 BY-SA 版权协议，转载请附上原文出处链接及本声明。
> 原文链接：https://blog.csdn.net/latency_cheng/article/details/80031283