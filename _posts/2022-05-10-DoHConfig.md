---
layout: article
title: 记录下DoH配置
date: 2022-05-10
key: P2022-05-10
tags: ["Other"]
show_author_profile: false
comment: true
sharing: true
aside:
  toc: true
---

做个备忘录，记录下主力设备 Win11、安卓、IOS、路由器、浏览器 配置 DoH(DNS-over HTTPS) 的过程

<!--more-->

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

`autoupgrade `  是 自动升级，意思是当使用这个dns的时会自动启用DoH

另外也可以直接去编辑注册表添加或是用组策略

用 netsh 举例 （添加的时候需要管理员权限

```
netsh dns add encryption server=223.5.5.5 dohtemplate=https://dns.alidns.com/dns-query autoupgrade=no udpfallback=no
netsh dns add encryption server=119.29.29.29 dohtemplate=https://doh.pub/dns-query autoupgrade=no udpfallback=no
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

开启ECH: `chrome://flags/#encrypted-client-hello` 将 Encrypted ClientHello 设置为`Enabled`

Firefox: 设置 - 常规 - 网络设置 - 设置 - 启用基于 HTTPS 的 DNS

开启ECH: 在 `about:config` 搜索条目 `network.dns.echconfig.enabled` 和 `network.dns.use_https_rr_as_altsvc`，将它们的设定改为 `true` 即可。

在 `about:config` 中将 `network.trr.mode`设置为 `2`（默认是0），即优先使用用 TRR（也就是我们的 DNS over HTTPS），在解析失败时使用常规方式。也可以设置成`3`，强制 Firefox 使用 DoH。参见 https://wiki.mozilla.org/Trusted_Recursive_Resolver

`Ctrl + Shift + R` 强制浏览器不使用页面缓存进行刷新

# 检测是否正在使用DOH

<https://1.1.1.1/help>

<https://www.cloudflare.com/zh-cn/ssl/encrypted-sni>

<https://crypto.cloudflare.com/cdn-cgi/trace/>
