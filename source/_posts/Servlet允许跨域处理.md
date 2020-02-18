---
title: Servlet允许跨域处理
typora-copy-images-to: ../../source/assets
typora-root-url: ../../source
date: 2018-08-19 11:35:08
updated: 2018-08-19 11:35:08
tags:
  - servlet
categories:
  - java
  - jsp
---

# Servlet允许跨域处理

使用 CORS协议允许 Response 跨域的方案：

```java
/* 允许跨域的主机地址 */
response.setHeader("Access-Control-Allow-Origin", "*");  
/* 允许跨域的请求方法GET, POST, HEAD 等 */
response.setHeader("Access-Control-Allow-Methods", "*");  
/* 重新预检验跨域的缓存时间 (s) */
response.setHeader("Access-Control-Max-Age", "3600");  
/* 允许跨域的请求头 */
response.setHeader("Access-Control-Allow-Headers", "*");  
/* 是否携带cookie */
response.setHeader("Access-Control-Allow-Credentials", "true");
```