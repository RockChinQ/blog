---
title: "qcg-tester & Skittles: QQ 机器人 QA 实践"
date: 2024-01-18
draft: false
---

\\n

由于在 百度 干了一个月的 QA 实习，自然是重视起测试来了，同时 QChatGPT
即将进行大规模重构，故在过去的一个月中，为 QChatGPT
实现了一个自动化测试工程。

\\n\\n\\n\\n

## qcg-tester {#qcg-tester .wp-block-heading}

\\n\\n\\n\\n

目前对 QChatGPT 的 QA 只有
系统测试，因为各个模块间的耦合度还是较高的，而重构也主要是针对各个功能实现的修改。

\\n\\n\\n\\n

qcg-tester 独立存放在 https://github.com/RockChinQ/qcg-tester ，基于
PyTest 构建，由 QChatGPT 主仓库的 workflows 发起 GitHub Action
进行测试。截止本文撰写，测试覆盖率已达 70%。

\\n\\n\\n\\n

qcg-tester 对 运行命令、检出 QChatGPT、修改配置、mock mirai-api-http
等测试所需功能进行了封装。其中mock功能由独立项目 Skittles 实现。

\\n\\n\\n\\n

事实上在编写测试的过程中，就已经发现了好几个以前未测试到位的bug。可以说
QA 对于开源项目仍然是非常重要的，我看到作为机器人框架标杆的 Koishi
也进行了测试，目前覆盖率达到89%。

\\n\\n\\n\\n

## Skittles {#skittles .wp-block-heading}

\\n\\n\\n\\n

> \\n
>
> <https://github.com/RockChinQ/Skittles>
>
> \\n

\\n\\n\\n\\n

mirai、go-cqhttp 等框架登录非常不稳定，要稳定进行测试，只能使用 mock
工具，而搜索全网并未发现实现了 mah 协议的 mock 工具。只能自己动手写了。

\\n\\n\\n\\n

Skittles 目前仅对 websockets 的数据包和 mah
的鉴权进行了很简单的封装，主要依赖
使用方的自行处理、构造数据包。后期计划支持更多的协议（OneBot），并使用
aiocqhttp
等框架已有的实体类和设计思想对多种协议的mock进行封装，为调用方提供统一的
Python API。

\\n
