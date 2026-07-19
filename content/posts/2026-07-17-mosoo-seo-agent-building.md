---
title: Mosoo 初体验：搭建一个给它自己写 SEO 文章的 Agent 吧
date: 2026-07-17
slug: mosoo-seo-agent-building
draft: true
tags: []
---

最近几个同事搓了个新产品：[Mosoo](https://try.mosoo.ai)。无论是品牌的视觉效果，还是相比于其他 Agent Builder 平台的体验，这个产品都给人一种耳目一新的感觉。

[![](/images/Screenshot%202026-07-17%20at%2010.41.03%20AM.png)](https://github.com/langgenius/mosoo)

README 里的定义有点详细了，我觉得我更想描述它为：Agent 时代的 Vercel。这个听起来更加吸引人，也让我更想参与这个产品的 growth。那么就用 Mosoo 来搭建一个用于优化 SEO 的 Agent 吧。

#### 上手 Mosoo

现在可以直接用 [try.mosoo.ai](https://try.mosoo.ai) 来体验，进入后使用自己的 Google 账号登录，不过仍然需要自己配置一下 LLM Provider：

![](20260719-215720.png)

当然，mosoo 也是[开源](https://github.com/langgenius/mosoo)的，可以让你的 Coding Agent 克隆到本地，再让他帮你启动～

#### 开始 Cook

![](20260719-221435.png)

在 Agents 页面创建一个新的 Agent，然后就自动进入了编辑器（看着很直观是吧）；我们选择使用 OpenAI Runtime，也就是 Codex，这将在 Mosoo 的沙盒里启动一个 Codex 实例，并且通过 ACP 受到控制；对应的，我们选择使用与 Codex 配合的 gpt-5.6-sol 模型；![](20260719-222105.png)

接着我们给他描述他的能力： "You are an expert SEO content writer. Users will provide you with various details about their products—such as names, descriptions, official website URLs, and other relevant information. You are expected to thoroughly research the product online and craft articles based on the user's specific requirements (such as the article's theme or focus). The content must be both reader-friendly and optimized for SEO."

现在我们给它添加一些相关的 skills，切换到 mosoo 的 skills 页面，目前 mosoo 支持从 GitHub 或者 `npx skills` 的命令导入，我们就从 skills.sh 上找一些相关的：![](20260719-224004.png)

![](20260719-224028.png)

给安装了三个 Skills：

![](20260719-224559.png)

现在回到刚刚创建的那个 Agent 的页面，选择这几个 skills。最终的配置如图所示：
![](20260719-224817.png)

#### 开始测试
