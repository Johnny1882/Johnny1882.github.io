---
layout:     post
title:      客户端与服务器
subtitle:   游戏的前后端
date:       2024-08-02
author:     JY
header-img: img/post-bg-iWatch.jpg
catalog: true
lang: zh
tags:
    - Unity
    - C#
    - 游戏开发
---

## 前言
在 **游戏架构：ECS** 博客中，介绍了一个游戏系统是如何运行的。但这只适用于单机游戏，如果游戏需要接入服务器呢？客户端与服务器需要以什么样的形式进行沟通？

## 正文
理想状态下，假设服务器有无限的性能，所有游戏的逻辑都由服务器计算，只需要服务器把数据传输到客户端进行播放。这减轻了客户端的负担，不再需要高性能CPU和GPU，也杜绝了任何外挂，毕竟为了外挂入侵服务器也太小题大做了。

但通常来说，客户端会承担一部分的游戏逻辑。如果全权由服务器负责运算，处理量过于庞大，特别是LOL这类大型网游，可能同时有几万盘游戏在同时运行，同时也可能导致游戏的延迟问题。这也就导致外挂由可乘之机。就拿最普遍的FPS游戏来说，射击的计算一般是放在客户端处理的，如果在服务器计算，再穿回本地，会导致严重的延迟，比如你看到自己开了一枪，但是过了半秒，子弹才射出去。而外挂可以通过修改客户端的数据，比如子弹的位置，来进行作弊，也就是常说的魔术子弹。

当然，游戏可以通过定期检查服务器数据，或是反作弊插件检测是否有修改行为。但这些说到底只是增加了外挂的成本，无法真正杜绝。毕竟机器在玩家的手上，玩家有绝对的控制。反作弊软件再厉害，也无法访问底层内存，但玩家可以，这也就是DMA（Direct Memory Access）外挂，通过直接访问和修改内存进行作弊。

### 如何沟通？

一个游戏会有一个CoreGame，也就是游戏的核心逻辑。这个逻辑正是通过ECS架构构造的。