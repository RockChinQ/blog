---
title: "日常 | 弃用宝塔自动化设施，全面拥抱容器化"
date: 2023-09-02
draft: false
---

\\n

## 背景 {#背景 .wp-block-heading}

\\n\\n\\n\\n

很早（几个月前）之前就想学学docker，但没有应用场景，也没有人带带，故即使读了读教程、跟着做了做，还是啥也没学会。

\\n\\n\\n\\n

上两周，我们需要把某个插件服务部署到我们组的服务器，那么就不归运维管了，只能我们（我）自己上手，可算是用上了docker和compose。用的是caddy做的反向代理，而不是NGINX，想到之前在宝塔上配置NGINX的噩梦，更加喜欢caddy了。

\\n\\n\\n\\n

## 缘由 {#缘由 .wp-block-heading}

\\n\\n\\n\\n

昨天（也许是今早，今天干太多事了时间观念没了）王冠朝又给我看了看他的blog，况且之前就试过hexo，结果乱搞一通、docker开了几次，竟又加载不出css了🤯，放了一个月没管。

\\n\\n\\n\\n

今天想了想，用wp搭个算了，用docker开了个wp，再用宝塔一配；测试时好好的，好了，一用宝塔反代又寄了😤。

\\n\\n\\n\\n

这下不得不自己搭环境了。

\\n\\n\\n\\n

## 结果 {#结果 .wp-block-heading}

\\n\\n\\n\\n

用caddy确实是很不错的😍，加上docker来组合，今天又现学了docker的网络什么的配置。现在已经看到终端就肌肉记忆敲个"docker"了\...

\\n\\n\\n\\n

把在cloudcone上的小服务器重装了之后全部用docker部署了一遍（现在这个博客就是运行在这上面的
20230901），成功恢复了以下服务：

\\n\\n\\n\\n

-   OpenAI api的反向代理服务
-   GhostJ 服务（因为太吃CPU而长期关闭）
-   天津原神大学.top 的重定向服务
-   WordPress服务
-   qsign和mirai服务（qsign真的很吃CPU）

\\n\\n\\n\\n

明天应该搭一下one-api，再把QChatGPT的数据收集服务也迁过来，，希望这个机器还吃得消😩。

\\n\\n\\n\\n

![](https://pub-c7c365991905453481dc9f8834619665.r2.dev/blog/wp-uploads/2023/09/Cache_-58b133b83dfacacf1.png){.wp-image-45
style="width:312px;height:133px" width="312" height="133"}

\\n\\n\\n\\n

感觉跟上时代了🐷\...

\\n
