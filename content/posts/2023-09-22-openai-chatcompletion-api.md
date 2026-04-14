---
title: "OpenAI 协议 | ChatCompletion API 解析和实现"
date: 2023-09-22
draft: false
---

\\n

最近在实现一个[新的项目](https://github.com/RockChinQ/free-one-api){rel="noreferrer noopener"
data-type="link" data-id="https://github.com/RockChinQ/free-one-api"
target="_blank"}，涉及到 OpenAI
协议的封装和实现，于是研究了一下相关API的细节。

\\n\\n\\n\\n

此文中使用的框架是 quart ，此框架以异步优先的风格实现了 flask 框架的
API，与 flask 的使用方式基本相同。

\\n\\n\\n\\n

### POST /v1/chat/completions {#post-v1chatcompletions .wp-block-heading}

\\n\\n\\n\\n

这个路径是目前最常用的 API ，用于访问 gpt-3.5-turbo/gpt-4
等语言模型，格式见[文档](https://platform.openai.com/docs/api-reference/chat/create)。

\\n\\n\\n\\n

#### 请求 {#请求 .wp-block-heading}

\\n\\n\\n\\n

``` wp-block-code
curl https://api.openai.com/v1/chat/completions \\\n  -H "Content-Type: application/json" \\\n  -H "Authorization: Bearer $OPENAI_API_KEY" \\\n  -d '{\n    "model": "gpt-3.5-turbo",\n    "messages": [\n      {\n        "role": "system",\n        "content": "You are a helpful assistant."\n      },\n      {\n        "role": "user",\n        "content": "Hello!"\n      }\n    ]\n  }'
```

\\n\\n\\n\\n

\$OPENAI_API_KEY 即为 api_key ，https://api.openai.com/v1
即为API终结点，使用反代时通常设置为反代地址，本文中的示例也将被以此方式访问。下方
-d
后即为请求的data，即Python使用openai库，调用openai.ChatCompletion.create时的参数。

\\n\\n\\n\\n

于是我们可以简单地从请求中取出相关的数据，以quart框架为例：

\\n\\n\\n\\n

``` wp-block-code
import quart \n\ndef main():\n    \n    app = quart.Quart(__name__)\n    \n    @app.route("/v1/chat/completions", methods=["POST"])\n    async def chat_completions():\n        data = await quart.request.get_json()\n        \n        model = data["model"]  # 模型名称\n        messages = data["messages"]  # 消息列表\n        stream = "stream" in data and data["stream"]  # 是否是流式请求\n        \n        # 请求头鉴权\n        # Bearer API_KEY\n        authorizations = quart.request.headers.get("Authorization")\n\n    app.run()\n\nif __name__ == "__main__":\n    main()
```

\\n\\n\\n\\n

#### 响应 {#响应 .wp-block-heading}

\\n\\n\\n\\n

我们直接跳过后端内容生成的逻辑，这些内容可以是任何东西，这里只讲解如何生成openai-python库兼容的输出格式。

\\n\\n\\n\\n

##### no stream 非流式请求 {#no-stream-非流式请求 .wp-block-heading}

\\n\\n\\n\\n

非流式的输出很简单，只需要输出一次即可，于是我们按照[文档所述](https://platform.openai.com/docs/api-reference/chat/create)的格式，将内容包装成json，直接返回。

\\n\\n\\n\\n

``` wp-block-code
import time\nimport quart\n\n# 本例中的测试内容，流式响应时每次返回一个单词\ncontent_example = "Now I Am Become Death, the Destroyer of Worlds."\n\ndef main():\n    app = quart.Quart(__name__)\n    @app.route("/v1/chat/completions", methods=["POST"])\n    async def chat_completions():\n        data = await quart.request.get_json()\n        model = data["model"]  # 模型名称\n        messages = data["messages"]  # 消息列表\n        stream = "stream" in data and data["stream"]  # 是否是流式请求\n        # 请求头鉴权\n        # Bearer API_KEY\n        authorizations = quart.request.headers.get("Authorization")\n        ...\n        return quart.jsonify(\n            {\n                # 事实上这里的123应是29个随机的数字或字母\n                "id": "chatcmpl-123",\n                "object": "chat.completion",\n                # 时间戳\n                "created": int(time.time()),\n                "model": model,\n                "choices": [{\n                    "index": 0,\n                    "message": {\n                        "role": "assistant",\n                        "content": content_example,\n                    },\n                    "finish_reason": "stop"\n                }],\n                "usage": {\n                    "prompt_tokens": 9,\n                    "completion_tokens": 12,\n                    "total_tokens": 21\n                }\n            }\n        )\n    app.run()\n\nif __name__ == "__main__":\n    main()
```

\\n\\n\\n\\n

最后的 usage 段的token，可以通过tiktoken库进行计算来达到精确结果。

\\n\\n\\n\\n

这样即返回了一个可供openai-python库识别的非流式返回。

\\n\\n\\n\\n

``` wp-block-code
# 测试代码\nimport os\n\nimport openai\n\nopenai.api_base = "http://localhost:5000/v1"\n\nopenai.api_key = os.getenv("OPENAI_API_KEY")\n\nresponse = openai.ChatCompletion.create(\n    model="gpt-3.5-turbo",\n    messages=[\n        {\n            "role": "user",\n            "content": "Hello."\n        }\n    ]\n)\n\nprint(response)
```

\\n\\n\\n\\n

![](https://pub-c7c365991905453481dc9f8834619665.r2.dev/blog/wp-uploads/2023/09/image-3.png)

\\n\\n\\n\\n

#### stream 流式请求 {#stream-流式请求 .wp-block-heading}

\\n\\n\\n\\n

流式响应的格式与非流式响应的格式不同，且需要多次响应。

\\n\\n\\n\\n

在响应过程中的每个数据，其中的finish_reason均为null，只有在最后一个part，content为{}时，finish_reason为\"stop\"。

\\n\\n\\n\\n

``` wp-block-code
# 原部分改为下方内容：\n        ...\n        \n        async def _gen():\n            parts = content_example.split(" ")\n            \n            t = int(time.time())  # 流式响应的每个part的created都是相同的\n            id = "123"  # 流式响应的id和上方的规则相同，每个part的id都是相同的\n            \n            for part in parts:\n                yield "data: {}\\n\\n".format(json.dumps({\n                    "id": "chatcmpl-"+id,\n                    "object": "chat.completion.chunk",\n                    "created": t,\n                    "model": "gpt-3.5-turbo",\n                    "choices": [{\n                        "index": 0,\n                        "delta": {\n                            "content": part+" ",\n                        },\n                        "finish_reason": None  # 非最后一个part的finish_reason为None\n                    }]\n                }))\n            \n            yield "data: {}\\n\\n".format(json.dumps({\n                "id": "chatcmpl-"+id,\n                "object": "chat.completion.chunk",\n                "created": t,\n                "model": "gpt-3.5-turbo",\n                "choices": [{\n                    "index": 0,\n                    "delta": {},\n                    "finish_reason": "stop"  # 最后一个part的finish_reason为stop\n                }]\n            }))\n            \n            yield "data: [DONE]\\n\\n"  # 标记结束\n        \n        headers = {\n            "Content-Type": "text/event-stream",\n            "Transfer-Encoding": "chunked",\n            "Connection": "keep-alive"\n        }\n        \n        return quart.Response(\n            _gen(),\n            mimetype="text/event-stream",\n            headers=headers,\n        )
```

\\n\\n\\n\\n

``` wp-block-code
# 测试代码\nimport os\n\nimport openai\n\nopenai.api_base = "http://localhost:5000/v1"\n\nopenai.api_key = os.getenv("OPENAI_API_KEY")\n\nresponse = openai.ChatCompletion.create(\n    model="gpt-3.5-turbo",\n    messages=[\n        {\n            "role": "user",\n            "content": "Hello."\n        }\n    ],\n    stream=True,\n)\n\nfor resp in response:\n    print(resp)\n
```

\\n\\n\\n\\n

![](https://pub-c7c365991905453481dc9f8834619665.r2.dev/blog/wp-uploads/2023/09/image-4.png)

\\n\\n\\n\\n

这样即可生成openai-python库支持的流式响应数据。

\\n
