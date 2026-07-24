---
title: 对磁力链接的思考
date: 2024-07-14T02:20:50+08:00
updated:
tags: [磁力链接, P2P]
categories: [技术]
keywords: []
description:
top_img:
comments:
cover:
toc:
---


## magnet链接是什么

## 我想怎么使用磁力链接

我在我的旧电脑上部署了qbittorrent, 想将旧电脑作为一个下载机器, 用来下载一些资源, 让后通过alist对外提供文件访问的服务

对外提供服务, 实际上是通过wireguard实现的, 利用服务器的公网ip将我的旧电脑, 新电脑和平板连接在一个局域网中, 具体实现以后写一个文章看看

### 问题

问题1: 磁力链接下载经常下载不动, 如果总是出现这种问题, 我觉得还不如去花点小钱使用pikpak, [pikpak](https://mypikpak.com/)的下载很稳定, 不管是多老的资源, 都能下载下来

可以有的解决办法很多, 但是还是得要知道磁力链接的原理

问题2: 找资源不方便, 还要去找磁力链接, 找到磁力链接还要再打开qbittorrent, 粘贴磁力链接, 然后再下载, 这个过程很繁琐, 我想用脚本封装这些操作

可以的解决方法有: bat脚本, python脚本, 甚至是go脚本, 或者是使用qbittorrent的api搭建一个web服务, 通过web服务来下载资源, 再或者是使用Electron来封装一个桌面应用
