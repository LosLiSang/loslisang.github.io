---
title: Docker中CMD和Entrypoint的区别
date: 2025-12-18T16:19:05+08:00
updated:
tags: [Docker, Dockerfile, 容器化]
categories: [计算机技术]
keywords: [Docker, Dockerfile, 容器化]
description: 本文深入解析 Docker 中 CMD 和 ENTRYPOINT 的区别与联系，帮助读者理解它们的使用场景及最佳实践，避免常见的坑。
cover: https://raw.githubusercontent.com/LosLiSang/picgo/main/20251218162100.png
top_img: https://raw.githubusercontent.com/LosLiSang/picgo/main/20251218162100.png
comments: 
toc:
---


Docker 中的 `CMD` 和 `ENTRYPOINT` 经常让人迷糊，尤其是同时使用时，更容易出现“命令不按预期执行”“容器秒退”等问题。理解它们的本质差异，是写好 Dockerfile 的关键。

## 核心理解：谁是“主角”，谁是“默认参数”

在 Docker 里，可以简单地这样理解：

- `ENTRYPOINT`：定义“这个镜像本质上是干什么的”，也就是**固定的主命令 / 主进程**。
- `CMD`：为这个镜像提供**默认参数**，或者在没有 `ENTRYPOINT` 时，提供默认要执行的命令。

当二者**同时存在**时：

- Docker 实际运行的是：  
  `ENTRYPOINT + CMD`
- 且 `CMD` **会被当成是** `ENTRYPOINT` **的参数**，而不是两个独立执行的命令。

所以：

```dockerfile
ENTRYPOINT ["top"]
CMD ["curl", "--help"]
```

真实执行的是：

```sh
top curl --help
```

而不是先执行 `top` 再执行 `curl --help`，这也是许多初学者踩坑的地方。

---

## 三种经典写法与执行效果

Docker 支持两种语法风格：**shell 形式** 和 **exec 形式**。这里重点看更推荐、更直观的 exec 形式（JSON 数组写法）。

### 1\. 只有 CMD

```dockerfile
FROM alpine:latest
RUN apk --no-cache add curl
CMD ["curl", "--help"]
```

执行：

```sh
docker run --rm my-image
```

实际命令是：

```sh
curl --help
```

如果你在 `docker run` 时追加了命令：

```sh
docker run --rm my-image curl https://example.com
```

则会完全覆盖 `CMD`，真实运行的是：

```sh
curl https://example.com
```

**要点：**

- 没有 `ENTRYPOINT` 时，`CMD` 是“默认要执行的命令”，可被运行时参数轻松覆盖。

---

### 2\. 只有 ENTRYPOINT

```dockerfile
FROM alpine:latest
RUN apk --no-cache add curl
ENTRYPOINT ["curl"]
```

执行：

```sh
docker run --rm my-image --help
```

真实运行的是：

```sh
curl --help
```

再看一个例子：

```sh
docker run --rm my-image https://example.com
```

真实运行的是：

```sh
curl https://example.com
```

**要点：**

- `ENTRYPOINT` 定死了“主角”是 `curl`，无论你在 `docker run` 后面写什么，都是给 `curl` 当参数。
- 这种写法适合“就是要把这个镜像当作某个命令的专用工具”的场景，例如 CLI 工具封装。

---

### 3\. ENTRYPOINT + CMD 组合

这是最容易踩坑但也最强大的用法。

```dockerfile
FROM alpine:latest
RUN apk --no-cache add curl
ENTRYPOINT ["curl"]
CMD ["--help"]
```

不加任何额外参数时：

```sh
docker run --rm my-image
```

真实运行：

```sh
curl --help
```

如果在运行时追加参数：

```sh
docker run --rm my-image https://example.com
```

真实运行：

```sh
curl https://example.com
```

**要点：**

- `CMD` 给 `ENTRYPOINT` 提供**默认参数**，在不传参时生效。
- 一旦运行时传入参数，就会**覆盖 CMD**，但不会覆盖 ENTRYPOINT。

---

## 真实踩坑案例：为什么这段 Dockerfile 跑不起来？

看下面这个例子（与前面提到的情况一致）：

```dockerfile
FROM alpine:latest

RUN apk --no-cache add curl bash

ENTRYPOINT [ "top" ]
CMD ["curl", "--help"]
```

表面期望：

- `ENTRYPOINT` 跑 `top`，保持容器持续运行。
- `CMD` 跑 `curl --help`，在启动时打印帮助。

实际发生的事：

- Docker 试图执行的命令是：  
  `top curl --help`
- `curl` 和 `--help` 被作为参数传给了 `top`。
- `top` 不认识 `curl` 这样的参数，于是报错退出，容器直接结束。

也就是说：

- 你原本以为是“先跑 top，再跑 curl --help”；
- 实际上是“用 top 去执行 curl --help 这种不合法的参数组合”。

**根本原因**：`CMD` 永远不会作为一个“第二条命令”被执行，它只是给 `ENTRYPOINT`（或默认 shell）提供参数。

---

## 想顺序执行多个命令应该怎么写？

如果希望：

- 容器启动时执行一次 `curl --help`；
- 然后再启动一个“持续运行”的程序（例如 `top`），让容器不退出；

有两种常见写法。

### 方案 A：用脚本做真正的入口（推荐）

`entrypoint.sh`：

```sh
#!/bin/bash
set -e

# 1. 执行一次性命令
curl --help

# 2. 启动一个长期运行的前台进程
top
```

Dockerfile：

```dockerfile
FROM alpine:latest

RUN apk --no-cache add curl bash
COPY entrypoint.sh /entrypoint.sh
RUN chmod +x /entrypoint.sh

ENTRYPOINT ["/bin/bash", "/entrypoint.sh"]
```

特点：

- 脚本的最后一个前台进程（这里是 `top`）会成为容器的 PID 1。
- 可以在脚本里写完整的业务逻辑、异常处理、日志输出等。

---

### 方案 B：使用 `sh -c` 串联命令（方便测试）

如果只是做本地实验，图省事，可以直接在 `CMD` 里写一行 shell：

```sh
FROM alpine:latest

RUN apk --no-cache add curl bash

CMD ["sh", "-c", "curl --help && top"]
```

这里没有 `ENTRYPOINT`，容器启动时会执行：

```sh
sh -c "curl --help && top"
```

执行顺序：

1. 先执行 `curl --help`，退出码为 0。
2. 再执行 `top`，`top` 在前台运行，容器保持存活。

缺点：

- 复杂逻辑都堆在一行 shell 字符串里，可读性差。
- 不利于后期维护，生产环境更推荐脚本方式。

---

## 常见使用场景与选择建议

### 1\. 应用服务镜像（Web/后端服务）

典型 Dockerfile：

```dockerfile
FROM python:3.12-slim

WORKDIR /app
COPY . /app
RUN pip install -r requirements.txt

CMD ["gunicorn", "-c", "gunicorn.conf.py", "app:app"]
```

建议：

- 大多数 Web 服务只用 `CMD` 就够了。
- 如非必要，不要使用 `ENTRYPOINT`，保持灵活（方便在调试时覆盖为 `/bin/bash`）。

---

### 2\. CLI / 工具型镜像

典型 Dockerfile：

```dockerfile
FROM alpine:latest
RUN apk --no-cache add curl

ENTRYPOINT ["curl"]
CMD ["--help"]
```

特征：

- 这个镜像“就是一个 curl 工具箱”。
- 直接 `docker run` 相当于执行 `curl --help`。
- 传参时可以很自然：

```sh
docker run --rm my-curl https://example.com
```

---

### 3\. 多进程 / 复杂启动逻辑

典型需求：

- 启动前做一些准备工作；
- 启动一个或多个长期服务；
- 监听信号，优雅退出。

推荐：

- 用脚本做 `ENTRYPOINT`，在里面管理所有流程。
- 或使用进程管理器（如 `tini`、`supervisord`），但这已经超出“简单 Dockerfile”的范畴。

---

## 总结：如何简单记住它们？

可以用一句话概括：

> `ENTRYPOINT` 决定“这个镜像是谁”，`CMD` 决定“它默认怎么干活”。

- 只需要**一个默认命令**：用 `CMD`。
- 需要一个**固定不变的主命令**：用 `ENTRYPOINT`。
- 既要固定主命令，又要可配置默认参数：同时用 `ENTRYPOINT + CMD`，记住 CMD 是参数，不是另一条命令。
- 需要顺序执行多条命令：用脚本或 `sh -c`，不要指望多个 CMD/ENTRYPOINT 帮你“自动依次执行”。
