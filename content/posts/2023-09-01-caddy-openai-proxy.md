---
title: "Caddy+Docker 迅速搭建 OpenAI 自托管反向代理"
date: 2023-09-01
draft: false
---

此方法适用于你有自己的境外服务器的情况，如果不然，建议使用其他的诸如使用vercel或cloudflare的方案，可以自行搜索。

使用的开源项目：

-   <https://github.com/Ice-Hazymoon/openai-scf-proxy>

参考：

-   [Rocky
    Linux安装Docker和Compose](https://juejin.cn/post/7137650621426958349)

## 准备工作

-   一个 安装了 Docker 和 Docker Compose 的 Linux服务器
-   此服务器可以正常访问 OpenAI 的 API 地址 （api.openai.com）
-   购买一个域名

## 开始

### 解析

把你的域名解析到你要使用的服务器

### 创建docker网络

``` wp-block-code
$ docker network create -d bridge shared-network
```

我们使用独立的一个network来沟通caddy和反代容器

### 建个目录放Caddy相关文件

caddy目录下的 docker-compose.yaml 文件内容

``` wp-block-code
version: '3'

services:
  caddy:
    image: caddy:latest
    restart: unless-stopped
    ports:
      - 80:80
      - 443:443
      - 443:443/udp
    volumes:
      - ./config/Caddyfile:/etc/caddy/Caddyfile:ro
      - ./config/site:/srv
      - ./config/caddy-data:/data
      - ./config/caddy-config:/config
    networks:
      - shared-network

networks:
  shared-network:
    external: true
```

同目录下新建一个目录config，在其中新建 Caddyfile 写入如下内容：

``` wp-block-code
<在这里填你的域名> {
    reverse_proxy openai-reverse-proxy:9000
}
```

### 建个目录放openai-scf-proxy的文件

新建目录，在目录下执行命令：

``` wp-block-code
git clone https://github.com/Ice-Hazymoon/openai-scf-proxy
```

在此目录新建文件 Dockerfile 写入以下内容：

``` wp-block-code
FROM node:20-alpine3.17

WORKDIR /app

COPY ./openai-scf-proxy .

WORKDIR /app/openai-scf-proxy

RUN npm install

CMD ["npm", "run", "start"]

```

再建个文件 docker-compose.yaml，写入以下内容：

``` wp-block-code
version: "3"

services:
  openai-reverse-proxy:
    build: .
    networks:
      - shared-network

networks:
  shared-network:
    external: true
```

## 启动！

分别到刚刚caddy和放反代项目的目录，执行以下命令

``` wp-block-code
docker compose up -d
```

然后现在你就能通过你刚刚解析的域名使用反向代理了。

