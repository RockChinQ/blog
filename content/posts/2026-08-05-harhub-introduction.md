---
title: 整了个为团队管理 Skills、MCPs 资产的工具
date: 2026-08-05
slug: harhub-introduction
draft: true
tags:
  - harhub
  - harness
  - skills
  - mcp
---

前两周我 +1 在搞 Dify 的新版 RAG ，几个人发现团队内的 Skill 越来越多，如果有新产品要用到已有的 skill，还得全部筛选并拷贝到新仓库，显得很麻烦。于是他整了个叫 harness-starter 的东西，主要为 非技术人员 打造，比如公司内的非研发同事有需求，打算用 Codex / Claude Code 自己 vibe 一个新的产品来解决问题，那么 harness-strater 就会根据用户的初始描述，提出一系列的跟进问题，例如产品主要面向哪些用户、交付形式、产品的细节、所使用的技术栈之类的，然后根据这一堆 context 挑选团队内已有的 skills ，结合预制的模板生成一整个可以直接交接给 coding agent 的框架。

但是做着就发现有很多问题，首先是 harness 资产的来源，以 skills 为例，目前团队里已有的 skills 都散落在各个仓库里，需要让 codex 扫描一遍放入 harness-starter 仓库里才能使用，而存进来的也只是副本，没法随来源更新。其次是新项目所使用的 harness 资产，在来源更新之后也没办法高效地被更新；harness 资产的双向路径都没有被打通。为了解决这些复杂问题，有必要引入一个新的平台来承接团队间 harness 资产管理的工作了。

![](/images/20260805-210502.png)

目前暂定名为 **harhub**，harness hub 的简称，同样是基于 GitHub 生态去做各类的 skills / MCPs 治理路径。harhub 内部建立 Library 存储资产，而引入 Anchor / Projects 的概念绑定 GitHub 仓库：

![](/images/20260805-210725.png)

harhub 仍然是强绑定于 GitHub 的，便于团队快速引入，工作区管理员只需要绑定 GitHub App 并导入 GitHub 仓库，harhub 即可自动扫描并索引仓库内任何路径下的 skill 的完整内容（包含 SKILL.md 和相关资源文件）

![](/images/20260805-211551.png)

在 Review 之后，即可添加到 Library 里。

同样的，我们也把 harness-strater 的功能给整合进了 harhub，在 Forge 页面即可简单描述需求、回答跟进问题，内置的 AI 将自动选取相关的 skills 和 MCP，放入 .harness 目录里。

![](/images/20260805-220215.png)

#### 不只是 starter

做完了这一套之后，就发现 harness-starter / harhub 不只是一个可以根据业务需求生成 harness 骨架的工具了，更可以承担团队内 harness 资产（Skills / MCPs，以及后续会支持的 AGENTS.md 里的规则）的治理平台，接管存储、整理、分发、回流等全流程。

##### 绑定现有资产

即上一章的通过已有 Project 绑定，并入库仓库内已有的 skills。也支持直接上传 zip 文件到 Library，或者通过 npx skills 命令下载来自 skills.sh 的 skills。

![](/images/20260805-221700.png)

##### 增删改查各个项目的 skills

现在不需要把仓库克隆到本地，即可在 harhub 平台上闭环，添加 skills 到 Project：![](/images/20260806-161852.png)

![](/images/20260806-163034.png)

harhub 会自动为此更改提出pr，合并之后即会同步索引到 Project 里。

对应的删除操作也是一样的路径。

##### 资产改动回流

还有一种情况，我们在其他地方（比如本地的 Codex / Claude Code）修改了项目中的 skills 等内容，希望能同步回全局的 Library。首先是 Project 里的

索引，我们在其他地方修改之后，推送到 GitHub，harhub 会自动检测到 skills 文件被修改：

在 Review 更改并且同步之后，即可在全局的 Library 中查看到这个 skill 最新的内容已经被同步到团队资产库。

![](/images/20260806-162902.png)

![](/images/20260806-162915.png)

并且更改也会被版本化记录。

现在开始尝试使用 harhub：[线上环境](https://harhub.rcpd.cc)

开源仓库：[RockChinQ/harhub](https://github.com/RockChinQ/harhub)
