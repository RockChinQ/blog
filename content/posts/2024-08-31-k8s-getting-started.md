---
title: "今天来折腾 k8s：成功启动控制平面"
date: 2024-08-31
draft: false
---

自从自己各个项目都更加容器友好后，就不止于只在 docker
上用了。看到其他项目也支持k8s，自己也得支持支持。关键是十几台服务器上面跑的各种服务实在是有点复杂，管不过来了，需要个集群工具来运行。

先安装
kubeadm，官方教程里有的：[官方教程](https://kubernetes.io/zh-cn/docs/setup/production-environment/tools/kubeadm/install-kubeadm/)

今天先去 vultr 买了个临时的服务器来试试水。买的是 4c16g
的配置，这个有点坑啊，买了之后启动才发现是 8g物理+8g
swap\...。系统装的是 debian 12。

按照教程所说，先配置 容器运行时，我这里选的是 containerd。

![](https://pub-c7c365991905453481dc9f8834619665.r2.dev/blog/wp-uploads/2024/08/image.png)

参考了另外一个教程来配置好了
containerd：[https://k8s.huweihuang.com/project/runtime/containerd/install-containerd](https://k8s.huweihuang.com/project/runtime/containerd/install-containerd)

这里很关键的一点，就是最后一定要把 containerd 的 Cgroup 改成
systemd，具体方法就是在配置里把 SystemdCgroup 设为 true。不然后续是没法
init 的。

![](https://pub-c7c365991905453481dc9f8834619665.r2.dev/blog/wp-uploads/2024/08/image-1.png)

这个安装 kubeadm 教程结尾的 配置 Cgroup
暂时不需要管，后续会把这个坑踩了。

接下来直接看 使用kubeadm创建集群 的教程：

<https://kubernetes.io/zh-cn/docs/setup/production-environment/tools/kubeadm/create-cluster-kubeadm/>

这里启动 kubeadm init 就直接寄了：

![](https://pub-c7c365991905453481dc9f8834619665.r2.dev/blog/wp-uploads/2024/08/image-2.png)

似乎是kubelet 没有在运行，折腾半天也没搞清楚。后来把 swap 给关了：

swapoff -a

再去把 kubelet 的 Cgroup 给显式设置为 Systemd （与containerd相同）：

nano /var/lib/kubelet/kubeadm-flags.env

按照以下问题修改：

<https://stackoverflow.com/questions/52119985/kubeadm-init-shows-kubelet-isnt-running-or-healthy>

接着 systemctl daemon-reload 和 systemctl restart kubelet ，再看 kubelet
就稳定运行了：

![](https://pub-c7c365991905453481dc9f8834619665.r2.dev/blog/wp-uploads/2024/08/image-3.png)

估计还是 Cgroup 的问题，现在 kubeadm init
还是不行的，因为刚刚配置失败了一次，需要 kubeadm reset 后再重新 init
：kubeadm init \--pod-network-cidr=192.168.0.0/16

![](https://pub-c7c365991905453481dc9f8834619665.r2.dev/blog/wp-uploads/2024/08/image-4.png)

成功初始化，接着按照 Calico
的教程配置一下网络组件：[https://docs.tigera.io/calico/latest/getting-started/kubernetes/quickstart](https://docs.tigera.io/calico/latest/getting-started/kubernetes/quickstart)

![](https://pub-c7c365991905453481dc9f8834619665.r2.dev/blog/wp-uploads/2024/08/image-5.png)

终于启动了第一个 k8s 集群。

