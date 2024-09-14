---
layout: article
title: 记录下DoH和ECH配置
date: 2022-05-10
key: P2022-05-10
tags: ["Other"]
show_author_profile: false
comment: true
sharing: true
aside:
  toc: true
---

做个备忘录，记录下主力设备 Win11、安卓、IOS、路由器、浏览器 配置 DoH(DNS-over HTTPS) 的过程。以及浏览器开启 Encrypted Client Hello (Secure SNI) 。

<!--more-->

国内的DNS服务商中，目前只有阿里支持ipv4和ipv6的DoH。可以分别通过`dig AAAA dns.alidns.com`和`dig A dns.alidns.com`测试。

# Windows11

设置 - 网络和Internet - WLAN - 硬件属性 \- DNS服务器分配

通过在 PowerShell 中执行 `Get-DnsClientDohServerAddress` 或者在 Windows 终端 `netsh dns show encryption` 就可以查看目前已有的 DoH 配置

- 使用 Get-DnsClientDohServerAddress 查看系统中已有的 DoH 命令（默认即是 Win11 原生支持的DoH服务）：

```
ServerAddress        AllowFallbackToUdp AutoUpgrade DohTemplate
-------------        ------------------ ----------- -----------
149.112.112.112      False              False       https://dns.quad9.net/dns-query
9.9.9.9              False              False       https://dns.quad9.net/dns-query
8.8.8.8              False              False       https://dns.google/dns-query
8.8.4.4              False              False       https://dns.google/dns-query
1.1.1.1              False              False       https://cloudflare-dns.com/dns-query
1.0.0.1              False              False       https://cloudflare-dns.com/dns-query
2001:4860:4860::8844 False              False       https://dns.google/dns-query
2001:4860:4860::8888 False              False       https://dns.google/dns-query
2606:4700:4700::1001 False              False       https://cloudflare-dns.com/dns-query
2606:4700:4700::1111 False              False       https://cloudflare-dns.com/dns-query
2620:fe::9           False              False       https://dns.quad9.net/dns-query
2620:fe::fe          False              False       https://dns.quad9.net/dns-query
2620:fe::fe:9        False              False       https://dns.quad9.net/dns-query
```

查看系统中已有的 DoH 命令 `netsh dns show encryption` （PowerShell） `Get-DnsClientDohServerAddress`

- 添加自定义DoH服务配置

```
netsh dns add encryption server=[resolver-IP-address] dohtemplate=[resolver-DoH-template] autoupgrade=no udpfallback=no
```

（PowerShell）

```
Add-DnsClientDohServerAddress -ServerAddress '<resolver-IP-address>' -DohTemplate '<resolver-DoH-template>' -AllowFallbackToUdp $False -AutoUpgrade $False
```

`autoupgrade ` 是 自动升级，意思是当使用这个dns的时会自动启用DoH

另外也可以直接去编辑注册表添加或是用组策略

用 netsh 举例 （添加的时候需要管理员权限

```
# Alidns
netsh dns add encryption server=223.5.5.5 dohtemplate=https://dns.alidns.com/dns-query autoupgrade=yes udpfallback=no
netsh dns add encryption server=2400:3200::1 dohtemplate=https://dns.alidns.com/dns-query autoupgrade=yes udpfallback=no
# Tencent
netsh dns add encryption server=119.29.29.29 dohtemplate=https://doh.pub/dns-query autoupgrade=yes udpfallback=no
netsh dns add encryption server=2402:4e00:: dohtemplate=https://doh.pub/dns-query autoupgrade=yes udpfallback=no
# Quad101
netsh dns add encryption server=101.101.101.101 dohtemplate=https://dns.twnic.tw/dns-query autoupgrade=yes udpfallback=no
netsh dns add encryption server=2001:de4::101 dohtemplate=https://dns.twnic.tw/dns-query autoupgrade=yes udpfallback=no
```

# Android

一加的Oxygen OS 11

WLAN和互联网 - 私人DNS - 私人DNS提供商主机名称

使用alidns的话填 `dns.alidns.com`

# IOS

通过添加配置描述文件的方式启用。

描述文件可以同时设置 WiFi 和蜂窝数据的 DNS

<https://github.com/paulmillr/encrypted-dns>

直接下载想要的那个描述文件`.mobileconfig`即可

可能是因为UUID相同的原因导致配置文件不能共存

IOS15 在 `设置 - 通用 - 设备管理 - DNS` 启用

# 路由器

梅林: WAN - DNS Privacy Protocol

老毛子(Padavan)身边暂时没有

另外 SmartDNS 直接添加即可

# 浏览器

Chrome: 设置 - 隐私设置和安全性 - 安全 - 使用安全DNS

Firefox: 设置 - 常规 - 网络设置 - 设置 - 启用基于 HTTPS 的 DNS

`Ctrl + R` 正常重加载

`Ctrl + Shift + R` 硬性重加载

按下`F12`进入调试后，右键地址栏左侧的刷新按钮，会出现 清空缓存并硬性重加载 。

# 检测是否正在使用DOH

<https://www.cloudflare.com/zh-cn/ssl/encrypted-sni/>

<https://1.1.1.1/help/>

<https://crypto.cloudflare.com/cdn-cgi/trace/>

# ECH

ECH 全称是 Encrypted Client Hello ，主要用于增强互联网连接的隐私保护。ECH 的核心是确保主机名不被暴露给互联网服务提供商、网络提供商和其它有能力监听网络流量的实体。

## Chrome

开启ECH: `chrome://flags/#encrypted-client-hello` 将 Encrypted ClientHello 设置为`Enabled`

## Firefox

开启ECH: 在 `about:config` 搜索条目 `network.dns.echconfig.enabled` 和 `network.dns.use_https_rr_as_altsvc`，将它们的设定改为 `true` 即可。

在 `about:config` 中将 `network.trr.mode`设置为 `2`（默认是0），即优先使用用 TRR（也就是我们的 DNS over HTTPS），在解析失败时使用常规方式。也可以设置成`3`，强制 Firefox 使用 DoH。参见 https://wiki.mozilla.org/Trusted_Recursive_Resolver
