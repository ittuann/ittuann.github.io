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

OpenWrt 软路由中 AdGuard Home 的 DNS 配置。

<!--more-->

环境为 AdGuard Home v0.107.71,  OpenWrt 内核5.4.190

## DNS 设置

**已知 DNS 提供商列表 <https://kb.adguard.com/general/dns-providers> **

- 上游DNS服务器

```
https://dns.alidns.com/dns-query
https://doh.pub/dns-query
https://doh.360.cn
https://dns.ipv6dns.com/dns-query
# OpenClash
# 127.0.0.1:7874
```

全部使用 DoH（DNS over HTTPS）协议，提高隐私性及安全性。

- Bootstrap DNS 服务器
用于解析指定为上游的 DoH/DoT 解析器的 IP 地址。

```
223.5.5.5
119.29.29.29
2400:3200::1
2402:4e00::
```

- 使用`最快的 IP 地址`

## 过滤器 - DNS拦截列表

- [anti-AD](https://github.com/privacy-protection-tools/anti-AD)

https://anti-ad.net/easylist.txt

命中率高、兼容性强。

**注意该规则的作者曾存在向规则内夹带私货的行为** https://github.com/Mosney/anti-anti-AD

- [AdRules](https://github.com/Cats-Team/AdRules)

中国地区广告屏蔽列表

- [HalfLife](https://github.com/sbwml/halflife-list)

https://cdn.jsdelivr.net/gh/o0HalfLife0o/list@master/ad.txt

ad-pc 涵盖了 EasyList China、EasyList Lite、CJX ’s Annoyance List[cjx82630]、Xinggsf 乘风视频过滤规则，以及补充的其它规则
ad-edentw 合并自 Adblock Warning Removal List、ABP filters、anti-adblock-killer-filters

- [EasyPrivacy](https://easylist.to/)

https://easylist-downloads.adblockplus.org/easyprivacy.txt

EasyList的扩展规则，反隐私跟踪、挖矿规则

- [HaGeZi's Blocklist](https://github.com/hagezi/dns-blocklists)

推荐用 Pro 版本就可以。另外还有 HaGeZi's Xiaomi Tracker Blocklist

其他:

- AdGuard DNS Filter

https://raw.githubusercontent.com/AdguardTeam/FiltersRegistry/master/filters/filter_15_DnsFilter/filter.txt

AdGuard 官方维护的广告规则，涵盖多种过滤规则

- EasyList

https://easylist-downloads.adblockplus.org/easylist.txt

Adblock Plus 官方维护的广告规则

EasyList China 是面向中文用户的 EasyList 扩展规则。EasyList 是主规则。

https://easylist-downloads.adblockplus.org/easylistchina.txt

- 自定义过滤规则

AdGuard Home 的过滤规则兼容 Adblock 语法、Hosts 语法及 Domain-only 语法。

## 其他设置

- 启用 DNSSEC

- 不启用 EDNS 客户端子网

EDNS（扩展域名系统，Extension Mechanisms for DNS）本身并不设计用于增强隐私，它主要是在标准的DNS查询中添加一些额外的数据。

特别是在使用 EDNS 客户端子网（EDNS Client Subnet，ECS）功能时。ECS会在DNS查询中附加用户的IP地址片段，向上游的DNS服务器暴露用户的大致地理位置。虽然这有助于返回离用户最近的数据中心的响应，提升速度，但它会在查询过程中暴露用户部分IP地址的前缀。部分DNS解析服务商考虑隐私直接选择不支持ECS。

重定向 - `作为dnsmasq的上游服务器`
配置后局域网客户端都显示127.0.0.1是正常的，只影响统计显示，不影响实际解析。

速度限制 - `0`

在关机时备份工作目录文件 - `filters`、`stats.db`、`sessions.db`、`querylog.json`

系统升级时保留文件 - `配置文件`、`sessions.db`、`stats.db`、`filters`

计划任务 - 自动更新ipv6主机并重启adh

参考链接

> [AdGuard Home 安装及使用指北](https://sspai.com/post/63088)
>
> [我有特别的 DNS 配置和使用技巧](https://blog.skk.moe/post/i-have-my-unique-dns-setup/)
>
> [Blocklist Collection | Firebog](https://firebog.net/)
