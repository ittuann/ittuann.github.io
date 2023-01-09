---
layout: article
title: Chrome开启ECH
date: 2022-11-26
key: P2022-11-26
tags: ["Other"]
show_author_profile: false
comment: true
sharing: true
aside:
  toc: true
---

Chrome 浏览器开启 Encrypted Client Hello (也被称为 Secure SNI) 隐私保护功能。

<!--more-->

我这里的环境是 Chrome 107.0.5304.122（正式版本） （64 位）

# 什么是ECH

ECH 全称是 Encrypted Client Hello ，主要用于增强互联网连接的隐私保护。ECH 的核心是确保主机名不被暴露给互联网服务提供商、网络提供商和其它有能力监听网络流量的实体。



# 开启方式

1. 在浏览器的地址栏中访问 

    ```
    chrome://flags/#encrypted-client-hello
    ```

1. 将 Encrypted ClientHello 设置为 “启用” （Enabled）

1. 重启 Chrome 浏览器



# 测试

<https://www.cloudflare.com/zh-cn/ssl/encrypted-sni/>

测试能否通过也与DNS服务提供商有关。可以尝试 Cloudflare 提供的 DOH 服务器`https://dns.cloudflare.com/dns-query`