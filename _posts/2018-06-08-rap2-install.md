---
title: RAP2安装配置
categories:
 - linux
tags:
 - rap2
---
# INSTALL RAP2
https://github.com/thx/rap2-delos    
https://github.com/thx/rap2-dolores

**依赖环境：** 
1. install MySQL
1. install git
1. install redis
1. install nodejs

## 配置
**delos 后端**

修改配置文件`rap2-delos/src/config/config.prod.ts`,配置端口号8088

**dolores 前端**

修改配置文件`rap2-dolores/src/config/config.prod.js`配置域名访问，后端指定路径：
``` 
module.exports = {
  serve: 'http://rap2.weiresearch.com/delos',
  keys: ['some secret hurr'],
  session: {
    key: 'koa:sess'
  }
}
```

nginx配置：后端请求转发`localhost:8088`
``` 
server {
    listen       80;
    server_name  rap2.weiresearch.com;
    root   /alidata1/soft/mock/rap2-dolores/build;

    location / {
       try_files $uri $uri/ /index.html =404;
    }

    location /delos/ {
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_pass http://localhost:8088/;
    }

    error_page 404 /404.html;
        location = /40x.html {
    }

    error_page 500 502 503 504 /50x.html;
        location = /50x.html {
    }
}
```