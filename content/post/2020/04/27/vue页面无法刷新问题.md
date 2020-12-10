---
title: vue页面无法刷新问题
date: 2020-04-27 18:59:21
tags:
- vue
categories:
- vue
---

## vue页面无法刷新问题

使用vue打包后部署到nginx服务器,却出现了无法刷新页面的问题.一刷新直接出现`404 NOT FOUND`错误

<!--more-->

### 原因及解决

可能是由于静态文件服务器的原因,在vue页面地址添加路由后,相当于去访问这个地址的静态文件,因为没有,因此返回404.

**解决:**

在nginx服务器的配置中添加如下:

```nginx
location / {
  try_files $uri $uri/ /index.html;
}
```

完整的nginx

```nginx
server {
    listen 7000;
    server_name _;
    index index.html;
    root /var/www/qblog;
    location / {
        try_files $uri $uri/ /index.html;
    }
}
```

[vue官方文档解决办法](https://router.vuejs.org/zh/guide/essentials/history-mode.html#后端配置例子)