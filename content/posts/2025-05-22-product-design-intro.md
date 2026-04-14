---
title: "初探产品设计"
date: 2025-05-22
draft: false
---

\\n

最近给 LangBot 发了
4.0，然后发现很多东西其实并不能或者说"可以不是"用那么直的思维思考。用户说我要什么功能、帮我把3.x某某功能加回来什么什么的，他们的真实需求不一定是这样的。我一直觉得我在重构时的刀法还挺精准的呢。

\\n\\n\\n\\n

我发了新版之后，由于 4.0
允许同时配置多个模型，并且可以被在流水线中选择。有一个老用户就跟我抱怨：我配置了好几个同名的模型，这个下拉框分辨不了啊！

\\n\\n\\n\\n

![](https://rockchin.top/wp-content/uploads/2025/05/54b0f2504840a19ea5a563f9818c5bac-1024x344.png){.wp-image-367}

\\n\\n\\n\\n

ta 的建议：给每个模型加一个备注字段。

\\n\\n\\n\\n

我一开始以为是因为不同的模型采用了不同的 base-url
，因为我在开发过程中就使用了不同来源的模型。但我还是决定问一下 ta
具体是什么区别，最终才知道原来是额外参数不同：

\\n\\n\\n\\n

![](https://rockchin.top/wp-content/uploads/2025/05/f05e4172205817a80ba03cfb5789ddfa-1024x982.png){.wp-image-369
style="width:527px;height:auto"}

\\n\\n\\n\\n

所以真实的需求是：选择模型的组件这应该显示更多的信息。这样就省去了用户去单独设置模型备注的步骤了。上
devin：

\\n\\n\\n\\n

![](https://rockchin.top/wp-content/uploads/2025/05/image-1-1024x814.png){.wp-image-370}

\\n\\n\\n\\n

最终效果：

\\n\\n\\n\\n

![](https://rockchin.top/wp-content/uploads/2025/05/be85e178b722952751857d9f0ab6b63f-1024x544.png){.wp-image-371}

\\n\\n\\n\\n

看起来优雅多了（确信）

\\n
