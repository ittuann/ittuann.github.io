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

在 Windows11、安卓、IOS、路由器、浏览器 配置 DoH(DNS-over HTTPS) 的方法，以及浏览器开启 ECH(Encrypted Client Hello)。

<!--more-->

DoH 全称是 DNS-over-HTTPS，即使用 HTTPS 协议通过 443 端口进行 DNS 解析。主要用于防止 DNS 被监听和污染，从而增强互联网连接的隐私保护和安全性。

ECH 全称是 Encrypted Client Hello ，即 Secure SNI 安全加密扩展。主要用于防止 SNI (Server Name Indication) 被监听和篡改，从而增强网络流量的保护和安全性。

## 配置 DoH 和 ECH

国内的DNS服务商中，目前只有阿里和腾讯支持ipv4和ipv6的DoH。

测试DNS是否支持DoH:

```
curl -H 'accept: application/dns-json' 'https://doh.pub/dns-query?name=baidu.com&type=A'
curl -H 'accept: application/dns-json' 'https://doh.pub/dns-query?name=baidu.com&type=AAAA'
```

或者也可以通过`dig AAAA dns.alidns.com`和`dig A dns.alidns.com`测试。

### Windows11

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

### 浏览器

- Chrome

设置 - 隐私设置和安全性 - 安全 - 使用安全DNS

开启ECH: `chrome://flags/#encrypted-client-hello` 将 Encrypted ClientHello 设置为`Enabled`

`Ctrl + R` 正常重加载

`Ctrl + Shift + R` 硬性重加载

按下`F12`进入调试后，右键地址栏左侧的刷新按钮，会出现 清空缓存并硬性重加载 。

- Firefox

设置 - 常规 - 网络设置 - 设置 - 启用基于 HTTPS 的 DNS

开启ECH: 在 `about:config` 搜索条目 `network.dns.echconfig.enabled` 和 `network.dns.use_https_rr_as_altsvc`，将它们的设定改为 `true` 即可。

在 `about:config` 中将 `network.trr.mode`设置为 `2`（默认是0），即优先使用用 TRR（也就是我们的 DNS over HTTPS），在解析失败时使用常规方式。也可以设置成`3`，强制 Firefox 使用 DoH。参见 https://wiki.mozilla.org/Trusted_Recursive_Resolver

### Android

Android目前仅支持配置DoT，不支持配置DoH。系统是一加的Oxygen OS 11 (Android 11)

WLAN和互联网 - 私人DNS - 私人DNS提供商主机名称

使用alidns的话填 `dns.alidns.com`

### IOS

通过添加配置描述文件的方式启用。

描述文件可以同时设置 WiFi 和蜂窝数据的 DNS

<https://github.com/paulmillr/encrypted-dns>

直接下载想要的那个描述文件`.mobileconfig`即可

可能是因为UUID相同的原因导致配置文件不能共存

IOS15 在 `设置 - 通用 - 设备管理 - DNS` 启用

### 路由器

OpenWrt: https://openwrt.org/docs/guide-user/services/dns/doh_dnsmasq_https-dns-proxy

梅林: WAN - DNS Privacy Protocol

老毛子(Padavan)身边暂时没有

AdGuardHome / NextDNS 可以配置 DoH

## 检测是否正在使用 DoH 和 ECH

<https://www.cloudflare.com/zh-cn/ssl/encrypted-sni/>

<https://one.one.one.one/help/>

<https://crypto.cloudflare.com/cdn-cgi/trace/>

<https://tls-ech.dev/>

也可以使用 Wireshark 抓包查看。

- DNSSEC 测试

<https://dnscheck.tools/>
<https://wander.science/projects/dns/dnssec-resolver-test/>

- DNS Leak 测试

<https://dnsleaktest.com/>
<https://www.browserscan.net/>

WebRTC Leak Test: <https://browserleaks.com/webrtc>

## Cloudflare Workers 自部署反代 1.1.1.1 的 DoH

为了避免国内的主动扫描探测，最好要配置成：`DoH + 非标路径 + 客户端白名单(可选)`的形式。也就是说要用`/<random_str>/<secret>`，不要直接用默认的`/dns-query`。

国内目前 53(UDP) 和 853(DoT) 端口都是重点关注对象，用国外服务器搭建也会被抢答或者阻断，所以目前只推荐使用 443(DoH 或 DoQ) 的方式连接。

https://github.com/tina-hello/doh-cf-workers
