---
title: "ss与netstat命令"
date: 2020-10-04T09:03:20-08:00
draft: true
tags: [进程,linux]
---

# linux 命令结构图

![img](20190121174817835.png)

## 为什么 ss 命令要比 netstat 命令快？

- netstat 是遍历 /proc 下的每个 PID 目录，ss 直接读 /proc/net 下面的统计信息，直接读取内核的第一手资料，当然速度会更快一些。所以 ss 执行的时候消耗资源以及消耗的时间都会比 netstat 少很多。
如果当服务器上万个后，并且每个服务器的 socket 有上千个甚至上万个的连接请求时，那么 netstat 将会慢到爆炸，而使用 ss 命令的你才是真的快
- ss 命令是利用的 tcp 协议栈中的 tcp_diag 模块，这是一个用于分析和统计的模块。即使没有 tcp_diag 模块，那么 ss 命令的执行速度同样会比 netstat 更快一些
- netstat 所属的 net-tools 包已经被淘汰，并且在 2001 年已经不再更新和发布新的版本了，而 ss 命令作为一个新的命令是可以一直延续下去的，隶属于 ip 家族的一员 iproute2