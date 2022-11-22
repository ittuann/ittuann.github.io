---
layout: article
title: 记录下OpenWrt上的AdGuard Home配置
date: 2022-11-18
key: P2022-11-18
tags: ["Other"]
show_author_profile: true
comment: true
sharing: true
aside:
  toc: true
---

记录下OpenWrt软路由上的AdGuard Home内DNS配置。

<!--more-->

环境为 OpenWrt 内核5.4.190  AdGuard Home v0.107.18

# DNS 设置

**已知 DNS 提供商列表 <https://kb.adguard.com/general/dns-providers> **

- 上游DNS服务器

    ```
    https://dns.alidns.com/dns-query
    https://doh.pub/dns-query
    # 223.5.5.5
    # 119.29.29.29
    # tcp://114.114.114.114
    2400:3200::1
    2402:4e00::
    # OpenClash
    # 127.0.0.1:7874
    ```

    使用了`并行请求`

- Bootstrap DNS 服务器

  ```
  223.5.5.5
  119.29.29.29
  2400:3200::1
  2402:4e00::
  ```

  

# 过滤器 - DNS拦截列表

- anti-AD

  https://anti-ad.net/easylist.txt

  命中率高、兼容性强。**注意该规则的作者曾存在向规则内夹带私货的行为。**

- halflife

  https://cdn.jsdelivr.net/gh/o0HalfLife0o/list@master/ad.txt

  涵盖了 EasyList China、EasyList Lite、CJX ’s Annoyance List[cjx82630]、Xinggsf 乘风视频过滤规则，以及补充的其它规则

- Adblock Warning Removal List

  https://easylist-downloads.adblockplus.org/antiadblockfilters.txt

  去除禁止广告拦截提示规则

性能足够的情况可以启用

- EasyPrivacy

  https://easylist-downloads.adblockplus.org/easyprivacy.txt

  反隐私跟踪、挖矿规则

- Fanboy’s Annoyances List

  https://easylist-downloads.adblockplus.org/fanboy-annoyance.txt

  去除页面弹窗广告规则

其他

- AdGuard DNS Filter

  https://raw.githubusercontent.com/AdguardTeam/FiltersRegistry/master/filters/filter_15_DnsFilter/filter.txt

  AdGuard 官方维护的广告规则，涵盖多种过滤规则

- EasyList

  https://easylist-downloads.adblockplus.org/easylist.txt

  Adblock Plus 官方维护的广告规则

- EasyList China

  https://easylist-downloads.adblockplus.org/easylistchina.txt

  面向中文用户的 EasyList 去广告规则

# 自定义过滤规则

AdGuard Home 的过滤规则兼容 Adblock 语法、Hosts 语法及 Domain-only 语法。

| **语法**                | **作用**                                  |
| ----------------------- | ----------------------------------------- |
| `||example.org^`        | 拦截 example.org 域名及其所有子域名       |
| `@@||example.org^`      | 放行 example.org 及其所有子域名           |
| `127.0.0.1 example.org` | 将 example.org 解析到 127.0.0.1           |
| `/REGEX/`               | 阻止访问与 example_regex_meaning 匹配的域 |
| `! 这是一行注释`        | 只是一条注释                              |
| `# 这是一行注释`        | 只是一条注释                              |



# 其他设置

启用 DNSSEC

启用 EDNS 客户端子网

重定向 - 作为dnsmasq的上游服务器

在关机时备份工作目录文件 -  `filters`、`stats.db`、`sessions.db`

系统升级时保留文件 -  `配置文件`、`0sessions.db`、`stats.db`、`filters`

计划任务 - 自动更新ipv6主机并重启adh 





参考链接

> [AdGuard Home 安装及使用指北](https://sspai.com/post/63088)
> [我有特别的 DNS 配置和使用技巧](https://blog.skk.moe/post/i-have-my-unique-dns-setup/)
> [Blocklist Collection | Firebog](https://firebog.net/)