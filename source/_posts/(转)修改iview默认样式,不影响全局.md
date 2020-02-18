---
title: (转)修改iview默认样式,不影响全局
typora-copy-images-to: ../../source/assets
typora-root-url: ../../source
date: 2019-01-08 17:07:03
updated: 2019-01-08 17:07:03
tags:
  - iview
  - 修改样式
categories:
  - vue.js
  - iview
---

`<style scoped>`只作用于当前组件

`<style>`作用于全局

<!-- more -->


1、找到表格，右键检查元素

```css
.ivu-table th {
    height: 40px;
    white-space: nowrap;
    overflow: hidden;
    background-color: #f8f8f9;
}
```

2、到需要修改的页面

```xml
<style>
  .mytable .ivu-table th{
    background-color: black;
  }
</style>
```

说明，其他样式还是写在`<style scoped>`，此元素写在`<style scoped>`没有作用，写在`<style>`里面又会全部都修改。
 给组件外面包一层，然后再选。注意的是，名字要取好，因为此时。mytable是全局
 3、iview的modal遮罩层样式修改
 class-name  设置对话框容器.ivu-modal-wrap的类名，可辅助实现垂直居中等自定义效果
 全局style，因为.ivu-modal-wrap在body里面

```xml
<Modal class-name="test"><Modal>
<style> .test{background-color: #1890FF;}<style>
```

>转载自:[懒羊羊3号](https://www.jianshu.com/p/a3a4573f06c4)