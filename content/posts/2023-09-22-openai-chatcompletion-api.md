---
title: "OpenAI 协议 | ChatCompletion API 解析和实现"
date: 2023-09-22
draft: false
---

最近在实现一个[新的项目](https://github.com/RockChinQ/free-one-api)，涉及到 OpenAI
协议的封装和实现，于是研究了一下相关API的细节。

此文中使用的框架是 quart ，此框架以异步优先的风格实现了 flask 框架的
API，与 flask 的使用方式基本相同。

### POST /v1/chat/completions

这个路径是目前最常用的 API ，用于访问 gpt-3.5-turbo/gpt-4
等语言模型，格式见[文档](https://platform.openai.com/docs/api-reference/chat/create)。

#### 请求

``` wp-block-code
curl https://api.openai.com/v1/chat/completions \\
  -H "Content-Type: application/json" \\
  -H "Authorization: Bearer $OPENAI_API_KEY" \\
  -d '{
    "model": "gpt-3.5-turbo",
    "messages": [
      {
        "role": "system",
        "content": "You are a helpful assistant."
      },
      {
        "role": "user",
        "content": "Hello!"
      }
    ]
  }'
```

\$OPENAI_API_KEY 即为 api_key ，https://api.openai.com/v1
即为API终结点，使用反代时通常设置为反代地址，本文中的示例也将被以此方式访问。下方
-d
后即为请求的data，即Python使用openai库，调用openai.ChatCompletion.create时的参数。

于是我们可以简单地从请求中取出相关的数据，以quart框架为例：

``` wp-block-code
import quart 

def main():
    
    app = quart.Quart(__name__)
    
    @app.route("/v1/chat/completions", methods=["POST"])
    async def chat_completions():
        data = await quart.request.get_json()
        
        model = data["model"]  # 模型名称
        messages = data["messages"]  # 消息列表
        stream = "stream" in data and data["stream"]  # 是否是流式请求
        
        # 请求头鉴权
        # Bearer API_KEY
        authorizations = quart.request.headers.get("Authorization")

    app.run()

if __name__ == "__main__":
    main()
```

#### 响应

我们直接跳过后端内容生成的逻辑，这些内容可以是任何东西，这里只讲解如何生成openai-python库兼容的输出格式。

##### no stream 非流式请求

非流式的输出很简单，只需要输出一次即可，于是我们按照[文档所述](https://platform.openai.com/docs/api-reference/chat/create)的格式，将内容包装成json，直接返回。

``` wp-block-code
import time
import quart

# 本例中的测试内容，流式响应时每次返回一个单词
content_example = "Now I Am Become Death, the Destroyer of Worlds."

def main():
    app = quart.Quart(__name__)
    @app.route("/v1/chat/completions", methods=["POST"])
    async def chat_completions():
        data = await quart.request.get_json()
        model = data["model"]  # 模型名称
        messages = data["messages"]  # 消息列表
        stream = "stream" in data and data["stream"]  # 是否是流式请求
        # 请求头鉴权
        # Bearer API_KEY
        authorizations = quart.request.headers.get("Authorization")
        ...
        return quart.jsonify(
            {
                # 事实上这里的123应是29个随机的数字或字母
                "id": "chatcmpl-123",
                "object": "chat.completion",
                # 时间戳
                "created": int(time.time()),
                "model": model,
                "choices": [{
                    "index": 0,
                    "message": {
                        "role": "assistant",
                        "content": content_example,
                    },
                    "finish_reason": "stop"
                }],
                "usage": {
                    "prompt_tokens": 9,
                    "completion_tokens": 12,
                    "total_tokens": 21
                }
            }
        )
    app.run()

if __name__ == "__main__":
    main()
```

最后的 usage 段的token，可以通过tiktoken库进行计算来达到精确结果。

这样即返回了一个可供openai-python库识别的非流式返回。

``` wp-block-code
# 测试代码
import os

import openai

openai.api_base = "http://localhost:5000/v1"

openai.api_key = os.getenv("OPENAI_API_KEY")

response = openai.ChatCompletion.create(
    model="gpt-3.5-turbo",
    messages=[
        {
            "role": "user",
            "content": "Hello."
        }
    ]
)

print(response)
```

![](https://pub-c7c365991905453481dc9f8834619665.r2.dev/blog/wp-uploads/2023/09/image-3.png)

#### stream 流式请求

流式响应的格式与非流式响应的格式不同，且需要多次响应。

在响应过程中的每个数据，其中的finish_reason均为null，只有在最后一个part，content为{}时，finish_reason为\"stop\"。

``` wp-block-code
# 原部分改为下方内容：
        ...
        
        async def _gen():
            parts = content_example.split(" ")
            
            t = int(time.time())  # 流式响应的每个part的created都是相同的
            id = "123"  # 流式响应的id和上方的规则相同，每个part的id都是相同的
            
            for part in parts:
                yield "data: {}\

".format(json.dumps({
                    "id": "chatcmpl-"+id,
                    "object": "chat.completion.chunk",
                    "created": t,
                    "model": "gpt-3.5-turbo",
                    "choices": [{
                        "index": 0,
                        "delta": {
                            "content": part+" ",
                        },
                        "finish_reason": None  # 非最后一个part的finish_reason为None
                    }]
                }))
            
            yield "data: {}\

".format(json.dumps({
                "id": "chatcmpl-"+id,
                "object": "chat.completion.chunk",
                "created": t,
                "model": "gpt-3.5-turbo",
                "choices": [{
                    "index": 0,
                    "delta": {},
                    "finish_reason": "stop"  # 最后一个part的finish_reason为stop
                }]
            }))
            
            yield "data: [DONE]\

"  # 标记结束
        
        headers = {
            "Content-Type": "text/event-stream",
            "Transfer-Encoding": "chunked",
            "Connection": "keep-alive"
        }
        
        return quart.Response(
            _gen(),
            mimetype="text/event-stream",
            headers=headers,
        )
```

``` wp-block-code
# 测试代码
import os

import openai

openai.api_base = "http://localhost:5000/v1"

openai.api_key = os.getenv("OPENAI_API_KEY")

response = openai.ChatCompletion.create(
    model="gpt-3.5-turbo",
    messages=[
        {
            "role": "user",
            "content": "Hello."
        }
    ],
    stream=True,
)

for resp in response:
    print(resp)

```

![](https://pub-c7c365991905453481dc9f8834619665.r2.dev/blog/wp-uploads/2023/09/image-4.png)

这样即可生成openai-python库支持的流式响应数据。

