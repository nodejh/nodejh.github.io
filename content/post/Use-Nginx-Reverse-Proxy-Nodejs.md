+++
title = "Use Nginx Reverse Proxy Node.js"
description = "使用 Ngnix 给 Node.js 应用做反向代理"
date = "2016-04-30T17:19:20+08:00"
tags = ["Nginx", "Node.js"]

+++


一般来说使用 node.js 开发的 webapp 都不会是默认的80端口，以官方文档演示为例：

```
const http = require('http');

const hostname = '127.0.0.1';
const port = 3000;

const server = http.createServer((req, res) => {
  res.statusCode = 200;
  res.setHeader('Content-Type', 'text/plain');
  res.end('Hello World\n');
});

server.listen(port, hostname, () => {
  console.log(`Server running at http://${hostname}:${port}/`);
});
```

<!--more-->

该例子使用的是 3000 端口，需要像 `localhost:3000` 这样，域名（或IP）加上 `:port` 才能访问。而一般 Web 应用都是监听的 80 端口。而普通应用一般只能监听 `1024` 以上的端口号，监听 80 端口需要 root 权限。而且 node.js 监听了 80 端口后，像 nginx 这类 HTTP Server 就只能选择监听其他端口了。

所以一般不使用 node.js 直接监听 80 端口，而是通过 nginx 来做反向代理。

Nginx 的具体配置如下：

```

upstream nodejs {
    server 127.0.0.1:3000;
    keepalive 64;
}

server {
    listen 80;
    # server_name 后面是域名，这里以 www.domain.com 为例
    server_name www.domain.com;
    # 日志
    access_log /var/log/nginx/test.log;
    location / {
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header Host  $http_host;
        proxy_set_header X-Nginx-Proxy true;
        proxy_set_header Connection "";
        proxy_pass http://nodejs;
    }
}
```


---
Github Issue: [https://github.com/nodejh/nodejh.github.io/issues/14](https://github.com/nodejh/nodejh.github.io/issues/14)
