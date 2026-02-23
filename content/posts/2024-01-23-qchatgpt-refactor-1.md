---
title: "🦌 QChatGPT 重构计划 ①：启动模块重写、原 qqbot 包异步化"
date: 2024-01-23T23:17:43+08:00
draft: false
tags: ["qchatgpt", "重构", "python", "异步"]
---

上周大体上完成了测试工程，今天开始做 QChatGPT 的重构。

## 启动流程方面

今天完成了 启动脚本的重写，main.py 暂时使用 start.py 替换。新增 boot 包，其中的 boot.py 为新的启动流程，包外部 start.py 简单地调用 boot.py 中的 main()。

启动流程拆分为了（boot包下）deps.py（依赖处理模块）、files.py（文件生成模块）、log.py（日志系统初始化模块）、config.py（配置文件加载模块），由boot.py逐个调用。

现在（今后）将运行时对象都保存在 pkg/boot/app.py 模块的 Application 类中，并使用依赖注入的形式进行实例化，逐步弃用 context。

## IM（qqbot包）方面

现在manager.py和adapter.py为异步优先了！YiriMirai适配器已完成所有可能的异步转换，相应地，manager的代码也大部分改成异步方法了。

很糟糕的是，nakuru似乎并不支持直接异步运行，今天尝试直接调用异步方法，结果根本收不到事件，明天再调查一下。

目前qqbot包还有很多方法是同步的，虽然大部分是仅被依赖的，但还是需要考量是否要改为全异步。同时，banlist、blob、filter、response_rule之类的功能，可能需要独立成特定类实现，python的模块对DI来说还是很不友好的。

一个很严重的问题：插件兼容性。为插件提供的发送消息的几个方法现在肯定处于失效状态了，若改为异步，则插件也需要改为异步，得再设计一下。

## 可预见的下一步

根据上述的部分，qqbot包将会优先于其他包进行大量的重构，主要是将异步和DI应用到其中。
