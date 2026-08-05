---
title: 最近整了个为团队管理 Skills、MCPs 资产的工具
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

但是做着就发现有很多问题，首先是 harness 资产的来源，以 skills 为例，目前团队里已有的 skills 都散落在各个仓库里，需要让 codex 扫描一遍放入 harness-starter 仓库里才能使用，而存进来的也只是副本，没法随来源更新。其次是新项目所使用的 harness 资产，在来源更新之后也没办法高效地被更新；harness 资产的双向路径都没有被打通。
