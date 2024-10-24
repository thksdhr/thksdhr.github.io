---
title: Teamspeak服务器搭建教程
date: 2024-10-24 11:04:14
categories:
    - 服务器搭建教程
tags:
    - 教程
---

# [TeamSpeak](https://www.teamspeak.com/zh-CN/)服务器搭建教程
teamspeak是一个非常简洁的语音聊天软件，它很干净小巧，能够来看这篇文章，说明你也对于搭建一个自己的teamspeak服务器有想法，本程将详细的教会你如何搭建一个属于你自己的teamspeak服务器~

***

### 准备工作
- *服务器*
> 必要的，想要拥有一个稳定的teamspeak服务器，你首先得拥有一台服务器主机~
- *Teamspeak Server*
> 必要的（如果直接运行服务器程序），点击此处去官方网站下载服务器程序：[teamspeak-server](https://www.teamspeak.com/zh-CN/downloads/#server)
- *Docker*
> 必要的（如果使用docker部署），我推荐使用Docker来部署teamspeak服务器，这将更加方便快捷~
- *Mariadb*
> 非必要，使用mariadb可以更加方便的管理你的数据。
> 如果你的服务器人数较多时，为了更好的性能表现，使用数据库是必要的。

本教程只针对与TeamSpeak Server的搭建，并不会涵盖其它内容例如：Docker、内网穿透。

***

### 使用官方程序搭建服务器
1. 将准备工作中下载好的服务器解压到你需要的位置
