---
title: 博客更新日志，功能简述
date: 9001-02-12 01:41:22
tags:
top: true
typora-copy-images-to: ./todo
typora-root-url: ./todo
---
blog搭建任务第一阶段告一段落，准备好好整理一下以前收藏的解决问题的博文和有意义值得分享的事~💃

## 未来计划：

- [x] 喂爬虫
- [ ] 整理出搭建教程
- [x] github 源代码项目分支sourcecode，master用来deploy

<!-- more -->

## 2月12日 todo

- [x] lean或者busuanzi的文章阅读统计
- [x] 左上角一条黑线
- [x] 文章字体大小格式
- [x] 评论头像怎么弄的gravatar网站上传
- [x] busuanzi网站总访问量设置
- [x] 网站的小图标
- [x] 关于更新NexT版本问题 git pull冲突解决：https://theme-next.org/docs/getting-started/data-files
- [x] 上面3px的黑线给去了 

## 2月13日 todo

- [x] 如何优雅的本地写作后上传

有两种方法，各有优缺点，我为了更容易转移文章，选择第二种

### 方法一：所有文章的图片都保存在source/asset/文件夹下

typora + hexo 我们一共需要三步(mac下的，windows找到对应设置即可)

- 编辑 => 图片工具 => 当插入本地图片时 => 复制到文件夹source/assets (自己在hexo目录source下创建assets目录用于存放图片资源)

- 编辑 => 图片工具 => 设置图片根目录为 source所在文件夹

  这两步的操作是在typora写文档时插入图片就保存在assets目录下并且可以在typora中显示

- 完成上两步之后会发现自动在typora上面生成了配置项`typora-copy-images-to`和`typora-root-url`，复制粘贴到post的模版文件中，这样当我们用`hexo new ***`生成新的post时就会直接保存上两条配置了

### 方法二：每个文章的图片单独保存到一个文件夹

1. `修改站点配置文件`_config.yml，`post_asset_folder: true`

2. 安装`hexo-asset-image`插件
3. `hexo new _post`后，就会生成一个`_post.md` 和同名文件夹, 把图片放到同名文件夹中，像`![描述](图片名)`引用就可以了。

这样在我们日常写作时，只需要把`_post`文件夹拖拽至typora中通过输入命令+typora就可以很好地写作+部署了，甚至都不需要打开vscode（虽然只是想想而已，不开编辑器写诗嘛？😂）

- [x] 图片自动压缩  决定不压缩，主页不要放大图片就好了

## 2月14日

- [x] 置顶文章

## 2月15日

百度已经能搜索到我的blog了哈哈。

![效果](/2B53B102.jpg)

前两天还挺着急，按了网上的seo配置之后还搜不到，实际上只要等两天就好了，给百度哥一个时间。