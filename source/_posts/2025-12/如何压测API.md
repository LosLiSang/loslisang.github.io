---
title: 如何压测 API 接口？—— 使用 wrk 工具进行高性能压测
date: 2025-12-09T00:06:05+08:00
updated:
tags: [API, 压测, wrk]
categories: [计算机技术]
keywords: [API, 压测, wrk, Lua脚本]
description: 本文介绍如何使用高性能的 HTTP 压测工具 wrk，对 API 接口进行压力测试，包括基本用法和进阶的 Lua 脚本编写技巧。
top_img: 
comments:
cover:
toc:
---

[Performance Testing with Apache Bench | by Pankaj Gupta | Medium](https://medium.com/@pankajgupta221b/performance-testing-with-apachebench-e2cef6882285)

[Top 9 Methods for API Testing in Java | Comprehensive Guide](https://www.code-intelligence.com/blog/methods-for-api-testing-in-java)

[ab - Apache HTTP server benchmarking tool - Apache HTTP Server Version 2.4](https://httpd.apache.org/docs/2.4/programs/ab.html)

[性能测试工具 wrk 使用教程 - 犬小哈 - 博客园](https://www.cnblogs.com/quanxiaoha/p/10661650.html)

`wrk` 是一款**现代、轻量级且极高性能的 HTTP 基准测试工具**（Benchmark Tool）。它最显著的特点是利用了异步事件驱动架构（如 Linux 的 epoll 或 macOS 的 kqueue），能用极少的线程在单机上产生巨大的并发压力。

简单来说，它是后端开发者用来**压测接口 QPS (每秒查询率) 和 Latency (延迟)** 的神器。

## 为什么用 wrk 而不是 Apache Bench (ab) 或 JMeter？

1. **性能更强**：`wrk` 使用非阻塞 I/O，能比 `ab` 产生更高的并发，更能榨干服务器性能。
2. **支持 Lua 脚本**：这是 `wrk` 的杀手锏。你可以编写 Lua 脚本来构造复杂的请求（如生成动态 Token、随机参数、POST 复杂 JSON 等），而 `ab` 只能发静态请求。
3. **轻量**：没有 JMeter 那么重的 GUI，纯命令行，适合服务器端直接跑。

## 常用命令示例

最简单的压测命令：

```sh
wrk -t12 -c400 -d30s http://localhost:8080/api/v1/user
```

- `-t12`: 使用 12 个线程（一般设置为 CPU 核心数）。
- `-c400`: 保持 400 个 HTTP 连接（并发数）。
- `-d30s`: 持续压测 30 秒。
- `--latency`: (可选) 输出详细的延迟分布报告（P50, P90, P99）。

## 配合 Lua 脚本（进阶）

比如你要压测一个**POST 接口**，且需要**JSON Body**，可以写一个 `post.lua`：

```lua
-- post.lua
wrk.method = "POST"
wrk.body   = '{"billCheckId": 123, "type": "NC"}'
wrk.headers["Content-Type"] = "application/json"
```

`wrk` 的强大之处就在于它嵌入了 LuaJIT 解释器。这意味着你可以完全控制 HTTP 请求的生成逻辑，包括**随机化参数、动态生成 Body、随机 Header、甚至根据响应调整请求**。

以下是几个实用的 Lua 脚本模板，专门解决“随机分布请求”的问题。

### 1\. 基础随机：随机请求不同的 URL 路径

比如你想压测 `/user/1001` 到 `/user/9999` 这样随机的用户详情页，防止服务器缓存热点。

```lua
-- 在压测开始前初始化随机种子
-- 注意：每个线程都有独立的 Lua 虚拟机，所以每个线程都需要初始化一次
math.randomseed(os.time())

request = function()
   -- 生成 1 到 10000 之间的随机用户ID
   local user_id = math.random(1, 10000)
   -- 拼接路径
   local path = "/user/" .. user_id
   
   -- 返回定制的请求
   -- wrk.format(method, path, headers, body)
   return wrk.format("GET", path)
end
```

### 2\. 权重随机：按比例访问不同接口

真实场景通常是混合的：80% 读请求，20% 写请求。

```lua
math.randomseed(os.time())

local paths = {
   "/products/list",    -- 读列表
   "/products/detail",  -- 读详情
   "/cart/add"          -- 写操作（加购）
}

request = function()
   local r = math.random(1, 100)
   
   if r <= 60 then
      -- 60% 概率访问列表
      return wrk.format("GET", paths[1])
   elseif r <= 90 then
      -- 30% 概率访问详情 (60-90)
      local pid = math.random(100, 500)
      return wrk.format("GET", paths[2] .. "/" .. pid)
   else
      -- 10% 概率添加购物车 (90-100)
      local body = '{"productId": ' .. math.random(100, 500) .. '}'
      wrk.headers["Content-Type"] = "application/json"
      return wrk.format("POST", paths[3], nil, body)
   end
end
```

### 3\. POST 请求：随机 Body 内容

压测“创建订单”或“短链接生成”接口时，每次 Body 必须不同，否则会被数据库唯一索引拦住。

```lua
math.randomseed(os.time())

-- 辅助函数：生成随机字符串
function random_str(length)
   local chars = "abcdefghijklmnopqrstuvwxyzABCDEFGHIJKLMNOPQRSTUVWXYZ0123456789"
   local res = ""
   for i = 1, length do
      local rand = math.random(#chars)
      res = res .. string.sub(chars, rand, rand)
   end
   return res
end

request = function()
   -- 动态生成 Body JSON
   local random_url = "http://example.com/" .. random_str(8)
   local body = '{"originalUrl": "' .. random_url .. '"}'
   
   -- 设置 Header
   wrk.headers["Content-Type"] = "application/json"
   
   -- 发送 POST
   return wrk.format("POST", "/create", nil, body)
end
```

### 4\. 高级技巧：每个线程独立的“用户身份” (Token)

如果你要压测需要登录的接口，可以给每个线程分配不同的 Token。

```lua
-- 每个线程启动时调用一次 setup
-- thread: 代表当前线程对象
function setup(thread)
   -- 给每个线程分配一个唯一的 ID (1, 2, 3...)
   thread:set("id", _G.thread_id_counter)
   _G.thread_id_counter = (_G.thread_id_counter or 0) + 1
end

-- 线程初始化时运行
function init(args)
   -- 根据线程 ID 生成伪造 Token
   -- 比如线程1用 user1_token，线程2用 user2_token
   local token = "Bearer user_" .. id .. "_token_secret"
   
   -- 设置全局默认 Header，该线程后续所有请求都会带上这个
   wrk.headers["Authorization"] = token
end

request = function()
   return wrk.format("GET", "/profile")
end
```

### 总结

1. **随机 URL**：在 `request()` 函数里拼装 `path`。
2. **随机 Body**：在 `request()` 里拼装 JSON 字符串。
3. **按比例混合**：用 `math.random(1, 100)` 做 if-else 分流。
4. **运行方式**：一定要带上 `--script` 参数。

## 如何查看CPU核心数

`lscpu`

![20251209000431.png](https://raw.githubusercontent.com/LosLiSang/picgo/main/20251209000431.png)

可以看出一共有16个核心，每个核心可以当两个线程使用（Intel 的超线程技术）

> 1\. 为什么“1个核”能变“2个核”？
>
> - **物理层面**：你的 CPU 确实只有 **16** 个完整的物理核心（Physical Cores），也就是有 16 套独立的 ALU（运算单元）和缓存。
> - **逻辑层面**：但是，物理核心内部的执行资源（如寄存器、指令队列）往往有富余。SMT 技术允许操作系统把 **1 个物理核心** 识别为 **2 个逻辑核心**。
