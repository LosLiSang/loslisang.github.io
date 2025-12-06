---
title: WebDAV 入门指南：把云盘挂载成本地硬盘
date: 2025-12-06T21:00:05+08:00
updated:
tags: [WEBDAV, 云盘, PIKPAK]
categories: [生活]
keywords: []
description: WebDAV 是一个非常经典且实用的协议。简单一句话总结：它把远程的云端存储，通过 HTTP 协议，“映射”成你电脑上的一个本地硬盘。
top_img: 
comments:
cover:
toc:
---

WebDAV（Web Distributed Authoring and Versioning）是一个非常经典且实用的协议。

简单一句话总结：**它把远程的云端存储，通过 HTTP 协议，“映射”成你电脑上的一个本地硬盘。**

不用下载文件就能直接修改、播放视频，就像操作你 D 盘里的文件一样。

## 1\. WebDAV 是什么原理？

WebDAV 是 **HTTP 协议的扩展**。

- **基础 HTTP**：通常只支持 `GET`（获取网页/文件）和 `POST`（提交数据）。
- **WebDAV 的魔法**：它给 HTTP 增加了几个新动词（Method）：
  - `PROPFIND`：查询文件属性（比如文件名、大小、目录结构）。
  - `MKCOL`：创建文件夹。
  - `COPY` / `MOVE`：复制或移动文件。
  - `LOCK` / `UNLOCK`：文件加锁/解锁（防止两个人同时改一个文件冲突）。

**工作流程：**

1. 你的操作系统（客户端）发起一个 HTTP 请求：“给我看看目录里有啥”。
2. WebDAV 服务器返回一段 XML 格式的数据，描述文件列表。
3. 操作系统把这段 XML 渲染成你熟悉的“文件夹图标”。
4. 当你双击打开文件时，系统按需下载；当你保存时，系统直接上传覆盖。

---

## 2\. 在 Windows 上怎么用？

Windows 对 WebDAV 的支持分为“原生”和“第三方工具”两种。

### 方法 A：使用第三方工具 **RaiDrive**（强烈推荐）

Windows 自带的 WebDAV 客户端非常难用（速度慢、经常断连、有文件大小限制）。**RaiDrive** 是目前 Windows 上体验最好的挂载工具。

1. 下载并安装 [**RaiDrive**](https://www.raidrive.com/)。
2. 点击右上角 `Add`。
3. 选择 `NAS` -> `WebDAV`。
4. 填写地址（Address）、路径（Path）、账户（Account）和密码。
5. 点击连接，你的“我的电脑”里就会多出一个网络磁盘（例如 Z: 盘），用起来和本地硬盘没区别。

### 方法 B：原生“映射网络驱动器”（不推荐，但可以用）

1. 打开“文件资源管理器” -> “此电脑”。
2. 顶部工具栏点击“映射网络驱动器”。
3. 在“文件夹”一栏输入 WebDAV 地址（例如 `http://192.168.1.100:5005/webdav`）。
4. 输入用户名密码。
    - _坑点：Windows 原生限制单文件不能超过 50MB（需改注册表），且 HTTPS 证书不信任时会报错。_

---

## 3\. 在 Linux 上怎么用？

Linux 既有适合桌面的图形化方式，也有适合服务器的命令行方式。

### 方法 A：图形界面（GNOME/KDE）

如果你用的是 Ubuntu Desktop (GNOME) 或其他桌面版：

1. 打开文件管理器（Nautilus）。
2. 在左侧栏找到 “+ Other Locations” (其他位置)。
3. 在底部的 "Connect to Server" 输入框填写地址：
    - 普通 HTTP：`dav://你的地址:端口/路径`
    - 加密 HTTPS：`davs://你的地址:端口/路径`
    - _(注意：这里用 dav:// 代替 http://)_
4. 输入密码，它就会挂载在侧边栏。

### 方法 B：命令行挂载 (mount.davfs)

适合需要把网盘挂载到系统目录给程序用的场景。

1. 安装工具：

   ```bash
   sudo apt install davfs2
   ```

2. 创建挂载点：

   ```bash
   sudo mkdir /mnt/webdav
   ```

3. 挂载：

   ```bash
   sudo mount -t davfs http://你的地址:端口/webdav /mnt/webdav
   ```

   _(系统会提示输入用户名密码)_

### 方法 C：Rclone（极客与自动化首选）

**Rclone** 是 Linux 下的神器，支持 WebDAV 且性能极好，支持缓存和加密。

1. 安装：`sudo apt install rclone` 或 `curl https://rclone.org/install.sh | sudo bash`

2. 配置：`rclone config` -> 新建 -> 选 WebDAV -> 填地址密码。

3. 挂载：

   ```bash
   rclone mount 你的配置名:/ /mnt/webdav --daemon
   ```

---

## 实例：用 WSL2 挂载 PikPak 的 WebDAV

[PikPak](https://pikpak.io/) 是一个非常实用的云盘服务，支持通过 WebDAV 访问。

1. 在 PikPak 网站开启 WebDAV 功能，获取地址、用户名和密码。
2. 在WSL中安装 `rclone` 并配置好 PikPak 的 WebDAV。
3. 挂载命令：

   ```bash
   rclone mount pikpak:/ /mnt/pikpak --daemon
   ```

4. 现在你可以在 `/mnt/pikpak` 目录下直接访问你的 PikPak 云盘文件了。
效果如下图
![20251206210916.png](https://raw.githubusercontent.com/LosLiSang/picgo/main/20251206210916.png)

此外还可以用 `rclone copy /本地/源文件 配置名:远程路径 -P` 命令实现本地和云盘之间的文件同步，非常方便。

> 但是要注意的一点是不能在Windows的文件管理器中直接访问WSL挂载的webdav目录，主要是因为Linux 文件权限隔离 和 FUSE 挂载机制的可见性
> 挂载命名空间的隔离：
>
> WSL 2 和 Windows 之间的文件共享协议（Plan 9 服务器）默认运行在特定的用户上下文中。当你运行 rclone mount 时，默认情况下，这个挂载点只对运行该命令的当前 Linux 用户（通常是你的默认用户）可见，而对系统其他进程（包括负责与 Windows 通信的 Plan 9 服务进程）是不可见的。
>
> FUSE 的安全限制：
>
> WebDAV 挂载通常使用 FUSE（Filesystem in Userspace）。出于安全考虑，FUSE 默认不允许“其他用户”（other users）访问挂载点。即使是 root 用户或其他系统服务（如 WSL 的文件互通服务）也无法访问，除非显式开启权限。

## 总结与使用建议

- **核心优势**：**省空间**。几十 TB 的云端资源，挂载后不占本地硬盘空间，用多少取多少。
- **常见用途**：
  - 配合 **Alist**：把百度网盘、阿里云盘转成 WebDAV，挂载到本地看视频。
  - 配合 **Obsidian / Zotero**：利用 WebDAV 同步笔记和文献。
  - 配合 **NAS**：在外面通过 WebDAV 访问家里的硬盘。
- **注意**：因为是基于 HTTP 的，如果是公网使用，强烈建议用 **HTTPS**，否则密码是明文传输的，很不安全。
  