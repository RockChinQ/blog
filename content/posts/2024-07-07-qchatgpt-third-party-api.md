---
title: "QChatGPT 实践：接入 OneAPI、LinkAI 等第三方 OpenAI 格式接口"
date: 2024-07-07
draft: false
---

\\n

自从 OneAPI 流行之后，各种牛鬼蛇神模型都能以 OpenAI
的格式访问了，还有中转站也开始流行，OpenAI 的格式也成为了 AI App
平台等各种 AI 工具的访问格式，本文介绍如何将上述的支持 OpenAI
格式的接口接入 QChatGPT。

\\n\\n\\n\\n

首先我们先按照 [QChatGPT 的文档](https://q.rkcn.top){data-type="link"
data-id="https://q.rkcn.top"}，部署好 消息平台、QChatGPT
主程序，并将其连接上，进行到 provider.json 配置这一步时，看此文档。

\\n\\n\\n\\n

以下我们不再区分 LinkAI、OneAPI、中转站，统一以 OneAPI接口 代指
第三方OpenAI接口。

\\n\\n\\n\\n

### 获取 OneAPI 接口地址和访问密钥 {#获取-oneapi-接口地址和访问密钥 .wp-block-heading}

\\n\\n\\n\\n

注意：**自己部署的 OneAPI 需要先设置渠道，具体请看 OneAPI
文档。**配置好后再获取密钥。

\\n\\n\\n\\n

以 OneAPI 为例，如果你把 OneAPI 部署在本地，使用
8080端口，那么接口地址就是 http://127.0.0.1:8080/v1

\\n\\n\\n\\n

其他的请看其文档或咨询中转站负责人，总之一般来说地址后面都是需要加 /v1
的。

\\n\\n\\n\\n

访问密钥，以 OneAPI 为例，去 \"令牌\" 页创建一个新的 key

\\n\\n\\n\\n

![](https://rockchin.top/wp-content/uploads/2024/07/image.png){.wp-image-298}

\\n\\n\\n\\n

![](https://rockchin.top/wp-content/uploads/2024/07/image-1.png){.wp-image-299}

\\n\\n\\n\\n

回到令牌页点击复制，即可获取到 api key。

\\n\\n\\n\\n

其他的中转站请看其文档或咨询负责人。

\\n\\n\\n\\n

### 填写到 QChatGPT 配置文件 {#填写到-qchatgpt-配置文件 .wp-block-heading}

\\n\\n\\n\\n

修改 data/config/provider.json 中的 provider 配置，使用
openai-chat-completions 请求器：

\\n\\n\\n\\n

![](https://rockchin.top/wp-content/uploads/2024/07/image-2.png){.wp-image-301}

\\n\\n\\n\\n

把你刚刚的地址填写到图中显示位置的 base-url 中，本例是我的 OneAPI
站（自用）。

\\n\\n\\n\\n

接着把你刚刚的密钥填写到 keys 的 openai 组中：

\\n\\n\\n\\n

![](https://rockchin.top/wp-content/uploads/2024/07/image-3.png){.wp-image-304}

\\n\\n\\n\\n

请注意符合 JSON 语法。

\\n\\n\\n\\n

最后配置所使用的模型，填写你 OneAPI 上提供的模型名称。

\\n\\n\\n\\n

![](https://rockchin.top/wp-content/uploads/2024/07/image-4.png){.wp-image-305}

\\n\\n\\n\\n

注意，可用的模型名称已经预设了一些在 data/metadata/llm-models.json
元数据里，如果你使用的模型没有相应的元数据，需要自己添加。

\\n\\n\\n\\n

### 添加不在预设中的模型 {#添加不在预设中的模型 .wp-block-heading}

\\n\\n\\n\\n

查看 data/metadata/llm-models.json 中 list
字段所有的元素，若没有任何元素的 name 是你要使用的模型名称，或其对应的
requester、token_mgr 不是 openai-chat-completions、openai，则需要添加。

\\n\\n\\n\\n

参考
[元数据配置](https://qchatgpt.rockchin.top/posts/metadata/llm-models.html)，本例中我们添加的模型
requester 设置为 openai-chat-completions ，token_mgr 设置为
openai，例如：

\\n\\n\\n\\n

``` wp-block-code
    {\n        "name": "qwen",\n        "requester": "openai-chat-completions",\n        "token_mgr": "openai",\n        "tool_call_supported": false,\n        "vision_supported": false\n    },
```

\\n\\n\\n\\n

name 为你的模型名称，对应 OneAPI 提供的模型名称，对应 上述 model
字段填写的值。

\\n\\n\\n\\n

tool_call_supported：是否支持使用工具，根据实际模型情况设置

\\n\\n\\n\\n

vision_supported：是否支持视觉，根据实际模型情况设置

\\n
