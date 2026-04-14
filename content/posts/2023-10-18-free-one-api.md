---
title: "使用 free-one-api 搭建一个完全免费的 ChatGPT 镜像站"
date: 2023-10-18
draft: false
---

最近一直在搞一个 将 revChatGPT / gpt4free / claude / huggingchat
等逆向工程库转换成 OpenAI API 标准格式的项目，
现在已经初步可用并且已经实现了高效的负载均衡策略。

> \

>
> 项目地址：
> [RockChinQ/free-one-api](https://github.com/RockChinQ/free-one-api){rel="noreferrer noopener"
> target="_blank"}
>
> \

此项目可以同时添加多个不同的来源的渠道，生成自定义的 API Key
，将这些逆向工程的免费来源的数据转换成标准的 OpenAI API
供其他应用调用，其他应用不再需要单独接入这些逆向工程库即可直接使用免费的
GPT。

### 开始

> \

>
> 使用的开源 ChatGPT 镜像站前端：
> <https://github.com/Yidadaa/ChatGPT-Next-Web>
>
> \

#### 部署 Free One API

可以查看文档：[https://free-one-api.rockchin.top/](https://free-one-api.rockchin.top/){target="_blank"
rel="noreferrer noopener"}

根据此文档中的步骤，部署 Free One API 后，添加一个 acheong08/ChatGPT
渠道，当然如果你没有 OpenAI 账号，或者觉得比较麻烦，可以直接添加一个
gpt4free 渠道，这个渠道不需要鉴权，只是相对比较慢。

![](https://pub-c7c365991905453481dc9f8834619665.r2.dev/blog/wp-uploads/2023/10/image.png)

你也可以添加多个渠道，Free One API 将自动选择。

之后在 API Key 页生成一个key，保存好。

![](https://pub-c7c365991905453481dc9f8834619665.r2.dev/blog/wp-uploads/2023/10/image-1.png)

![](https://pub-c7c365991905453481dc9f8834619665.r2.dev/blog/wp-uploads/2023/10/image-3.png)

#### 搭建 ChatGPT-Next-Web

根据上方提供的链接中的文档搭建好 ChatGPT-Next-Web，然后根据
[此处](https://github.com/Yidadaa/ChatGPT-Next-Web/blob/main/README_CN.md#%E7%8E%AF%E5%A2%83%E5%8F%98%E9%87%8F){rel="noreferrer noopener"
target="_blank"} 的说明设置环境变量连接到 Free One API。

-   将 OPENAI_API_KEY 环境变量设置为刚刚在 Free One API 生成的 key
-   将 BASE_URL 环境变量设置为 Free One API 的访问地址\

    -   例如你将Free One API 部署在了本地，那么就填写
        http://localhost:3000 即可

    \

到此即配置完成，可以开始对话了。

