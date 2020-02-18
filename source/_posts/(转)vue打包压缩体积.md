---
title: (转)vue打包压缩体积
date: 2018-9-13 18:53:12
updated: 2018-9-13 18:53:12
tags:
  - vue.js
  - 体积优化
categories:
  - vue.js
typora-copy-images-to: ../../source/assets
typora-root-url: ../../source
---

作为一个做 SPA 起家的框架，Vue 的开发学习曲线较为平缓，相对来说，开发体验属于上乘。但不少初学者会发现，自己的站点，随随便便打包文件就有 10M ！难以置信，其实这跟 Vue 的性能没有太大关系，我们可以通过配置文件来大大改善这一情况。

减少打包后文件体积的方法：

1. 采用懒加载
2. 启用 Gzip 压缩
3. 将依赖库挂到 CDN 上
4. 减少不必要的库依赖

<!-- more -->

## 准备知识：

> 1. 如何用 vue-cli 创建 Vue 应用，并对其有一定了解，否则，请移步[官网](https://cn.vuejs.org/v2/guide/instance.html)
> 2. 对 Webpack 有了解，知道它是一个打包工具，并且它的四大要素：entity、output、。进一步理解 webpack，移步[这里](https://webpack.js.org/)

### 懒加载

官网上对懒加载的解释是这样的：

> 当打包构建应用时，Javascript 包会变得非常大，影响页面加载。如果我们能把不同路由对应的组件分割成不同的代码块，然后当路由被访问的时候才加载对应组件，这样就更加高效了。
>
> 结合 Vue 的异步组件和 Webpack 的代码分割功能，轻松实现路由组件的懒加载。

它的作用是实现延迟加载，避免一次性加载过大的文件，增加首页渲染时间，影响用户体验。

#### 如何实现？

要实现懒加载很简单，我们只需要改变组件的导入方式：

**before:**

```
import Article from '@/components/Article'
复制代码
```

**after:**

```
const Article = () => import('@/components/Article')
复制代码
```

#### 原理是什么？

本质上，它是利用了 Promise，具体请见：[异步组件](https://cn.vuejs.org/v2/guide/components.html#%E5%BC%82%E6%AD%A5%E7%BB%84%E4%BB%B6)

### 启用 Gzip 压缩

vue 默认不启用 Gzip 压缩，但我们知道，压缩后的文件体积会大大减少，这适用于线上部署。 如何启用也很简单： 首先，在 `config` 中将 `build.productionGzip` 设置为 `true` 然后，确认 `webpack.prod.conf.js` 中有如下代码：

```
if (config.build.productionGzip) {
  const CompressionWebpackPlugin = require('compression-webpack-plugin')

  webpackConfig.plugins.push(
    new CompressionWebpackPlugin({
      asset: '[path].gz[query]',
      algorithm: 'gzip',
      test: new RegExp(
        '\\.(' +
        config.build.productionGzipExtensions.join('|') +
        ')$'
      ),
      threshold: 10240,
      minRatio: 0.8
    })
  )
}
复制代码
```

如果有的话，下面只要使用 `npm install --save-dev compression-webpack-plugin` 安装 webpack 插件，这样，你在 build 项目时，webpack 会帮你压缩文件。 如果没有的话，你只要把上面代码复制到 webpack 配置文件的 `plugins` 下即可。 如何方面查看build之后的文件大小呢？我们可以使用另外一个 webpack 插件：`webpack-bundle-analyzer` ，如何使用呢？默认 Vue 会导入这个插件，我们只需要调用即可：在 `package.json` 文件中增加以下命令：`"analyzer": "NODE_ENV=production npm_config_report=true npm run build"`，运行 `npm run analyzer` ，等待一会，就可以看到整个项目的打包情况了。

### 将依赖库挂到 CDN 上

如果你对首屏响应速度要求比较高，那么，CDN 也不失为一种好方法。 它的原理是将一些依赖库挂载到 CDN 上，通过在 html 文件 `script` 便签引入的方式进行加载，减少的打包文件，从而缩小了体积。

以 element 为例：

首先，在 html 中引入库：

```
<!DOCTYPE html>
<html>
  <head>
    <meta charset="utf-8">
    <title>demo-vue-project</title>
    <link rel="stylesheet" href="https://cdn.bootcss.com/element-ui/2.0.8/theme-chalk/index.css">
  </head>
  <body>
    <div id="app"></div>
    <!-- built files will be auto injected -->
    <script src="https://cdn.bootcss.com/element-ui/2.0.7/index.js"></script>
  </body>
</html>
复制代码
```

然后在 `webpack.base.conf.js` 中移除对这三个库的依赖：

```
externals: {
    'vue': 'Vue',
    'vue-router': 'VueRouter',
    'element-ui': 'ELEMENT'
  },
复制代码
```

这样我们就可以愉快地享受 CDN 了。

### 减少不必要的库依赖

这一点就是见仁见智了。如果你使用了上面提到的 `webpack-bundle-analyzer` ，你会发现占用你的包的大部分是你引用的一些库，诸如：Element-ui、lodash、highlight 等等。不知道你们看到是什么心情，反正我都想删掉它们，自己写需要的功能了，毕竟我只需要它们其中的一小部分功能，却牺牲了大量的带宽！

## 写在最后

webpack 是一个强大的打包工具，这篇文章写于 V3.4 的时候，现在 V4 已经出来了，配置文件更加地简单，值得一看！本文写得粗糙，如果不对之处，欢迎批评指正！

>  作者：allenWang
>
>  链接：https://juejin.im/post/5abba68cf265da239c7b6bdc
>
>  来源：掘金