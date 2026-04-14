---
title: "Caddy+Docker 迅速搭建 OpenAI 自托管反向代理"
date: 2023-09-01
draft: false
---

\\n

此方法适用于你有自己的境外服务器的情况，如果不然，建议使用其他的诸如使用vercel或cloudflare的方案，可以自行搜索。

\\n\\n\\n\\n

使用的开源项目：

\\n\\n\\n\\n

-   <https://github.com/Ice-Hazymoon/openai-scf-proxy>

\\n\\n\\n\\n

参考：

\\n\\n\\n\\n

-   [Rocky
    Linux安装Docker和Compose](https://juejin.cn/post/7137650621426958349){data-type="link"
    data-id="https://juejin.cn/post/7137650621426958349"}

\\n\\n\\n\\n

## 准备工作 {#准备工作 .wp-block-heading}

\\n\\n\\n\\n

-   一个 安装了 Docker 和 Docker Compose 的 Linux服务器
-   此服务器可以正常访问 OpenAI 的 API 地址 （api.openai.com）
-   购买一个域名

\\n\\n\\n\\n

## 开始 {#开始 .wp-block-heading}

\\n\\n\\n\\n

### 解析 {#解析 .wp-block-heading}

\\n\\n\\n\\n

把你的域名解析到你要使用的服务器

\\n\\n\\n\\n

### 创建docker网络 {#创建docker网络 .wp-block-heading}

\\n\\n\\n\\n

``` wp-block-code
$ docker network create -d bridge shared-network
```

\\n\\n\\n\\n

我们使用独立的一个network来沟通caddy和反代容器

\\n\\n\\n\\n

### 建个目录放Caddy相关文件 {#建个目录放caddy相关文件 .wp-block-heading}

\\n\\n\\n\\n

caddy目录下的 docker-compose.yaml 文件内容

\\n\\n\\n\\n

``` wp-block-code
version: '3'\n\nservices:\n  caddy:\n    image: caddy:latest\n    restart: unless-stopped\n    ports:\n      - 80:80\n      - 443:443\n      - 443:443/udp\n    volumes:\n      - ./config/Caddyfile:/etc/caddy/Caddyfile:ro\n      - ./config/site:/srv\n      - ./config/caddy-data:/data\n      - ./config/caddy-config:/config\n    networks:\n      - shared-network\n\nnetworks:\n  shared-network:\n    external: true
```

\\n\\n\\n\\n

同目录下新建一个目录config，在其中新建 Caddyfile 写入如下内容：

\\n\\n\\n\\n

``` wp-block-code
<在这里填你的域名> {\n    reverse_proxy openai-reverse-proxy:9000\n}
```

\\n\\n\\n\\n

### 建个目录放openai-scf-proxy的文件 {#建个目录放openai-scf-proxy的文件 .wp-block-heading}

\\n\\n\\n\\n

新建目录，在目录下执行命令：

\\n\\n\\n\\n

``` wp-block-code
git clone https://github.com/Ice-Hazymoon/openai-scf-proxy
```

\\n\\n\\n\\n

在此目录新建文件 Dockerfile 写入以下内容：

\\n\\n\\n\\n

``` wp-block-code
FROM node:20-alpine3.17\n\nWORKDIR /app\n\nCOPY ./openai-scf-proxy .\n\nWORKDIR /app/openai-scf-proxy\n\nRUN npm install\n\nCMD ["npm", "run", "start"]\n
```

\\n\\n\\n\\n

再建个文件 docker-compose.yaml，写入以下内容：

\\n\\n\\n\\n

``` wp-block-code
version: "3"\n\nservices:\n  openai-reverse-proxy:\n    build: .\n    networks:\n      - shared-network\n\nnetworks:\n  shared-network:\n    external: true
```

\\n\\n\\n\\n

## 启动！ {#启动 .wp-block-heading}

\\n\\n\\n\\n

分别到刚刚caddy和放反代项目的目录，执行以下命令

\\n\\n\\n\\n

``` wp-block-code
docker compose up -d
```

\\n\\n\\n\\n

然后现在你就能通过你刚刚解析的域名使用反向代理了。

\\n
