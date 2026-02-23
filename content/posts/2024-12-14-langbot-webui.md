---
title: "🌐 LangBot WebUI 选用的配置文件可视化编辑方案"
date: 2024-12-14T23:38:07+08:00
draft: false
tags: ["langbot", "webui", "vue", "vuetify"]
---

回来了，前俩月一直在搞 Dify 的插件市场。当然也腾出时间给 LangBot（QChatGPT）做了一个大更新：3.4。在这个版本加入了 WebUI Beta 版本，并且项目名也改为了 LangBot，后续将尝试商业化路线。

这个更新并没有很多高深的内容，那么就来讲讲 LangBot WebUI 是怎么在可扩展性要求较高的情况下实现配置文件可视化的。

看 [AstrBot 的方案是自己写的渲染器](https://github.com/Soulter/AstrBot/blob/master/dashboard/utils/config.py)，创造了一种新的格式。这种实现方式较为简单、直观，但我不想再耗费精力自己造轮子了，况且前期调研也发现了阿里的 formily 之类的成品框架可以直接拿来用，而且 JSON 已经可以用 JSON Schema 来描述，相关的轮子也蛮多，对于扩展性的要求能很好地满足。

后来使用 `vuetify json schema vue` 之类的关键词搜索，找到了 https://vue-json-schema-form.lljj.me/zh/guide/ ，实验了一下发现这个库对我已经用在 WebUI 上的 vuetify 框架并不是很友好，而且不能很好地支持 可变格式object的列表（LangBot的platform.json中的平台适配器配置）。

几经周折最终找到了个虽然很少star，但能很好符合我需求的：https://koumoul-dev.github.io/vuetify-jsonschema-form/latest/about ，这个库使用起来很简单。传入数据，传入对应的JSON Schema，再添加一些特定的选项到 JSON Schema 或者 Option 设置即可。

为之前的每个配置文件添加 JSON Schema 模板，接着，新建了一个 SettingsManager 用于存储所有需要跟踪的配置文件管理器。每个配置管理器现在也能存储 配置文件名、描述、对应JSON Schema、文档链接。需要在WebUI的配置管理页面显示的配置管理器，可以调用 ap 中这个 SettingsManager 的注册方法，LangBot 将在启动阶段注册每个配置文件管理器。

如果插件使用了 LangBot 的配置文件管理套件创建了管理器，也能通过这种方式注册，并允许用户在 WebUI 中更改（当然，是否能热生效就得看具体实现了）。

最终，配置文件将可以在 WebUI 中修改。注册配置管理器时的 JSON Schema 是可选的，只有在提供了 JSON Schema 时才允许使用可视化编辑器。同时也提供了 JSON 编辑器，这里使用的方案是：https://github.com/cloydlau/json-editor-vue，效果非常好。
