---
title: Hexo踩坑记录(持续更新)
typora-copy-images-to: ../../source/assets
typora-root-url: ../../source
date: 2018-12-30 17:37:33
updated:
tags:
  - Hexo
categories:
  - Hexo
---

### Unhandled rejection Template render error: (unknown path) parseIf: expected elif, else, or endif, got end of file

我出现这个问题的原因竟然是在markdown中出现如下代码，应该是吧大括号和百分号读取了。

```markdown
`{%if ****%}`
```

### 网页加载慢可能问题

用Chrome network检测工具发现font的cdn是国外，吧`_config.yml`文件`font`改成false即可，2020年初这个时间段国内还没有搜索到字体镜像。所以要想改字体还是放自己服务器上吧。

### 只是添加文章后，主页无法加载并且无报错

原因是在文章正文中有`<***>`，应该是next或者hexo会自动解析尖括号。