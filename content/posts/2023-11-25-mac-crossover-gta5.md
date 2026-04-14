---
title: "在 Apple M系列芯片机型上通过 CrossOver 流畅游玩 GTA 5 和 GTA Online"
date: 2023-11-25
draft: false
---

\\n

买之前就有考察过转译（玩游戏）性能，网上有说 CrossOver 没法登录 RockStar
SocialClub，不过真正尝试的时候才发现现在已经修复这个问题了。于是现在也写一篇比较新的教程。

\\n\\n\\n\\n

目前可以选择使用 Parallel Desktop 或者 CrossOver 的方案来实现在 MacOS on
Apple Silicon 平台上运行 GTA 5。经过考察，我们选择性能更好的
CrossOver。CrossOver 将 Wine 封装成易于使用的软件，wine
的实现原理是将程序对 Windows 的系统调用转换成 宿主系统的系统调用
以执行其中的 Windows 可执行程序。

\\n\\n\\n\\n

而 MacOS 底层提供的 Rosetta 2 可以将 x86 程序的指令集转译为 ARM
指令集以便在 Apple Silicon 平台上运行 x86 程序。以上为在 MacOS ARM
平台运行 Windows x86 平台的 GTA 5 的实现原理。

\\n\\n\\n\\n

## 步骤 {#步骤 .wp-block-heading}

\\n\\n\\n\\n

### 购买一台 MacBook with Apple Silicon {#购买一台-macbook-with-apple-silicon .wp-block-heading}

\\n\\n\\n\\n

我买的就是 MacBook Air M1 16GB 的机型，M1 的转译性能很惊艳，足够了。

\\n\\n\\n\\n

### 安装 CrossOver {#安装-crossover .wp-block-heading}

\\n\\n\\n\\n

这个非常简单，直接去网上下载安装 CrossOver，可以有 14
天的免费试用期，还可以通过改配置来实现长时间试用，具体不赘述。

\\n\\n\\n\\n

安装完成后打开到 安装 页面，如果有 Steam，可以直接按照提示安装，若没有
Steam，则查看下方手动安装到步骤。

\\n\\n\\n\\n

### 在 CrossOver 容器中安装 Steam {#在-crossover-容器中安装-steam .wp-block-heading}

\\n\\n\\n\\n

现在我们去 [Steam 官网](https://s.team) 下载
**Windows版本的Steam安装包**，在 CrossOver 中点击

\\n\\n\\n\\n

安装 \> 安装一个不在列表里的应用程序 \> 点击第一个"编辑" 选择 Steam
安装包

\\n\\n\\n\\n

![](https://rockchin.top/wp-content/uploads/2023/11/截屏2023-11-25-19.29.57-1024x633.png){.wp-image-196
style="width:428px;height:auto"}

\\n\\n\\n\\n

接着点第二个"编辑"，创建一个新的容器名为 Steam

\\n\\n\\n\\n

![](https://rockchin.top/wp-content/uploads/2023/11/image-4-1024x451.png){.wp-image-198
style="width:436px;height:auto"}

\\n\\n\\n\\n

点击安装，按照流程安装 Steam 即可。

\\n\\n\\n\\n

### 配置容器 {#配置容器 .wp-block-heading}

\\n\\n\\n\\n

点击你的容器，设置成下方这样。

\\n\\n\\n\\n

![](https://rockchin.top/wp-content/uploads/2023/11/image-5.png){.wp-image-199
style="width:268px;height:auto"}

\\n\\n\\n\\n

接下来这一步是可选的，如果你不想把游戏本体安装在你的 Mac
硬盘中，那么插上你的外置硬盘，设置一个驱动器映射。在这里添加，将你要用于存储文件的目录给挂载到某个盘符，我这里是挂到了
D盘。之后可以在 Steam 中选择将游戏下载到 D盘。

\\n\\n\\n\\n

![](https://rockchin.top/wp-content/uploads/2023/11/image-6-880x1024.png){.wp-image-200
style="width:358px;height:auto"}

\\n\\n\\n\\n

接下来这一步也是可选的，但是建议你设置一下，前提是你了解网络代理的知识（但是你不设置可能会没法登录
Steam）。就是这个网络问题，由于众所周知的原因，国内访问 Steam
很甜蜜草蛋，如果你在宿主机有 Cl\*sh
之类的代理软件，代理了系统中的流量，那么需要显性给容器设置一下。

\\n\\n\\n\\n

打开 Internet设置

\\n\\n\\n\\n

![](https://rockchin.top/wp-content/uploads/2023/11/image-7.png){.wp-image-202
style="width:292px;height:auto"}

\\n\\n\\n\\n

![](https://rockchin.top/wp-content/uploads/2023/11/image-8-1024x983.png){.wp-image-203
style="width:383px;height:auto"}

\\n\\n\\n\\n

在 连接
这一页设置你的代理为你代理软件的端口，这个应该自己清楚的，你都用代理软件了，别问不该问的。

\\n\\n\\n\\n

### 启动 Steam {#启动-steam .wp-block-heading}

\\n\\n\\n\\n

直接在 CrossOver 里双击启动 Steam
就行了，填写密码账号登录。注意了，如果很久很久登不进去，或者登进去了提示是离线模式的话，使用
-tcp 参数启动 Steam 即可解决。

\\n\\n\\n\\n

![](https://rockchin.top/wp-content/uploads/2023/11/image-9-1024x699.png){.wp-image-204}

\\n\\n\\n\\n

如果你刚刚设置了磁盘映射，并且打算把游戏装到你设置的目录，到 Steam设置
\> 存储空间 \> 添加驱动器，然后你就可以去 GTA
的主页点击安装，选择你刚刚新增的驱动器来安装了。

\\n\\n\\n\\n

![](https://rockchin.top/wp-content/uploads/2023/11/image-10-1024x795.png){.wp-image-205}

\\n\\n\\n\\n

### 启动 GTA 5 和优化设置 {#启动-gta-5-和优化设置 .wp-block-heading}

\\n\\n\\n\\n

经过长达数十个小时的等待，你安装好了 GTA。

\\n\\n\\n\\n

#### Q：屏幕显示变得特别白 {#q屏幕显示变得特别白 .wp-block-heading}

\\n\\n\\n\\n

进入游戏，线上模式，打开设置 \> 图像
，按照以下设置配置，主要是设置成全屏，设置 DirectX 10.1（用 DirectX 11
有时候会有渲染问题），分辨率随便设置。

\\n\\n\\n\\n

![](https://rockchin.top/wp-content/uploads/2023/11/90c1beaa4cf1af9ebf1acb4671766221_720-1024x769.jpg){.wp-image-206}

\\n\\n\\n\\n

就可以正常玩了。

\\n\\n\\n\\n

#### Q：只显示左上角一部分 {#q只显示左上角一部分 .wp-block-heading}

\\n\\n\\n\\n

重启之后会有一个很幽默的问题，全屏只显示了一部分内容。

\\n\\n\\n\\n

![](https://rockchin.top/wp-content/uploads/2023/11/1b20cf5e96090c76b02ddf7f959a8966_720-1024x769.jpg){.wp-image-207}

\\n\\n\\n\\n

这时候先别急，等进入游戏后改一下分辨率就可以了，按照如下方法：

\\n\\n\\n\\n

ESC健，三下向左（设置页面），回车，六下向下键（到图像一栏），回车，三下向下键（修改分辨率），随便左右点一下，空格，回车，回车
即可恢复正常。

\\n\\n\\n\\n

![](https://rockchin.top/wp-content/uploads/2023/11/4741f70f8a275a80ea8721a10bd196a2_720-1024x769.jpg){.wp-image-208}

\\n\\n\\n\\n

#### Q：靠近可交互物体时左上角不显示提示 {#q靠近可交互物体时左上角不显示提示 .wp-block-heading}

\\n\\n\\n\\n

这个问题折磨了我大概半年，最后发现是输入法冲突导致的。

\\n\\n\\n\\n

检测方法：切换武器的时候左上角不显示可以操作的按钮，就说明你已经触发了这个问题。原因是你在游戏里某次（或进游戏时）输入法切换成了中文。

\\n\\n\\n\\n

解决办法：按T打开聊天框，你这时候会看到屏幕左上角有个什么文字都没有的小黑框，现在切换到中文输入法，随便打点字，按空格输出到聊天框，这时候左上角应该没有显示任何黑框，切换到英文输入法，回车输出文字并退出聊天框。现在应该就解决这个问题了。

\\n
