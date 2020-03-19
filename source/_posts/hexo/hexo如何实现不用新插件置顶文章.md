---
title: Hexo如何实现不用新插件置顶文章
typora-copy-images-to: ./Hexo如何实现不用新插件置顶文章
typora-root-url: ./Hexo如何实现不用新插件置顶文章
date: 2020-02-14 16:16:07
updated:
tags:
  - Hexo
  - 修改样式
categories:
  - Hexo
---

> 基于hexo 4 ，NexT 7

最近想实现一个置顶功能，网查之后发现可以安装插件，我的方法直接修改时间为9999然后不显示时间来规避这个问题。

<!-- more -->

# 修改post显示文件

修改`themes/next/layout/_macro/post.swig`文件，在`<div class="post-meta">`下面添加

```
{% if post.top %}
    <i class="fa fa-thumb-tack"></i>
    <font color=7D26CD>置顶</font>
    <span class="post-meta-divider">|</span>
{% endif %}
```

来用于显示置顶二字。

在下面`- if theme.post_meta.created_at`处添加判断条件后变为`- if theme.post_meta.created_at and not post.top`即可。

最后效果如下:

![top](top.png)

# 修改archives显示文件

修改`themes/next/layout/_macro/post-collapse.swig`文件

目的让之前显示年份的地方改成“置顶”，并且小字部分显示年份

代码如下:

```
{% macro render(posts) %}
{%- set current_year = '1970' %}
{%- set if_have_top = false %}
{%- for post in posts.toArray() %}

  {%- set year = date(post.date, 'YYYY') %}

  {%- if year > 9000 and not if_have_top %}
    {%- set current_year = '置顶' %}
    {%- set if_have_top = true %}
    <div class="collection-year">
      <{%- if theme.seo %}h2{% else %}h1{%- endif %} class="collection-header">{{ current_year }}</{%- if theme.seo %}h2{% else %}h1{%- endif %}>
    </div>
  {% else %}
    {%- if year !== current_year and year <= 9000 %}
      {%- set current_year = year %}
      <div class="collection-year">
        <{%- if theme.seo %}h2{% else %}h1{%- endif %} class="collection-header">{{ current_year }}</{%- if theme.seo %}h2{% else %}h1{%- endif %}>
      </div>
    {%- endif %}
  {%- endif %}

  <article itemscope itemtype="http://schema.org/Article">
    <header class="post-header">
    
      {%- if year > 9000 %}
        <div class="post-meta">
          <time itemprop="dateCreated"
                datetime="{{ moment(post.updated).format() }}"
                content="{{ date(post.updated, config.date_format) }}">
            {{ date(post.updated, 'YYYY-MM-DD') }}
          </time>
        </div>
      {% else %}
        <div class="post-meta">
          <time itemprop="dateCreated"
                datetime="{{ moment(post.date).format() }}"
                content="{{ date(post.date, config.date_format) }}">
            {{ date(post.date, 'MM-DD') }}
          </time>
        </div>
      {%- endif %}

      <{%- if theme.seo %}h3{% else %}h2{%- endif %} class="post-title">
        {%- if post.link %}{# Link posts #}
          {%- set postTitleIcon = '<i class="fa fa-external-link"></i>' %}
          {%- set postText = post.title or post.link %}
          {{ next_url(post.link, postText + postTitleIcon, {class: 'post-title-link post-title-link-external', itemprop: 'url'}) }}
        {% else %}
          <a class="post-title-link" href="{{ url_for(post.path) }}" itemprop="url">
            <span itemprop="name">{{ post.title or __('post.untitled') }}</span>
          </a>
        {%- endif %}
      </{%- if theme.seo %}h3{% else %}h2{%- endif %}>

    </header>
  </article>

{%- endfor %}
{% endmacro %}

```



# 修改想要置顶的post

在想要置顶的top文件下加入`top: true`，并将`date`改成9000年以上即可，修改`updated`为你想要的修改的时间即可。

> 参考：
> [hexo-next 添加文章置顶功能和评分功能等](https://www.zhyong.cn/posts/fc22/)