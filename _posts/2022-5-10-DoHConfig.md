---
layout: article
title: 记录下DoH配置
date: 2022-5-10
tags: ["Other"]
comment: true
aside:
  toc: true
---

做个备忘录，记录下主力设备 Win11、安卓、IOS、路由器、浏览器 配置 DoH(DNS-over HTTPS) 的过程

<!--more-->

# Windows11

设置 - 网络和Internet - WLAN - 硬件属性 \- DNS服务器分配

- Win11原生支持的DoH服务：

```
IPv4
Google：8.8.8.8 and 8.8.4.4
Cloudflare：1.1.1.1 and 1.0.0.1
Quad9：9.9.9.9 and 149.112.112.112

IPv6
Google：2001:4860:4860::8888 and 2001:4860:4860::8844
Cloudflare：2606:4700:4700::1111 and 2606:4700:4700::1001
Quad9：2620:fe::fe and 2620:fe::fe:9
```

查看系统中已有的 DoH 命令 `netsh dns show encryption` （PowerShell） `Get-DnsClientDohServerAddress`

- 添加自定义DoH服务配置

```
netsh dns add encryption server=[resolver-IP-address] dohtemplate=[resolver-DoH-template] autoupgrade=no udpfallback=no
```

（PowerShell）

```
Add-DnsClientDohServerAddress -ServerAddress '<resolver-IP-address>' -DohTemplate '<resolver-DoH-template>' -AllowFallbackToUdp $False -AutoUpgrade $True
```

`autoupgrade `  是 自动升级，意思是当使用这个dns的时会自动启用DoH

另外也可以直接去编辑注册表添加或是用组策略

用 netsh 举例 （添加的时候需要管理员权限

```
netsh dns add encryption server=223.5.5.5 dohtemplate=https://dns.alidns.com/dns-query autoupgrade=no udpfallback=no
netsh dns add encryption server=2400:3200::1 dohtemplate=https://dns.alidns.com/dns-query autoupgrade=no udpfallback=no
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
