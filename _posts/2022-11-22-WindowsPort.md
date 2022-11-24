---
layout: article
title: 解决 Windows11 莫名端口占用，而又找不到占用应用
date: 2022-11-22
key: P2022-11-22
tags: ["Other"]
show_author_profile: true
comment: true
sharing: true
aside:
  toc: true
---

解决 Windows11 莫名端口占用，而又找不到占用应用的奇怪问题。

<!--more-->

起初是 Clash 的 7980 端口报冲突（不建议用默认端口有可能会被扫描），后来又遇到了 qBittorrent 的 59854 端口无法连接但同一网络下的其他设备却能建立连接。

尝试用命令却找不到任何占用这个端口的程序。

```
netstat -ano | findstr 7890
```

这个问题已经有一阵子了，不过每次都是重启就可以解决也就没找修复的方案。后来在网页开发的时候提示占用端口，这就不能忍了。在网上找了一大圈子终于找到了解决办法。 <https://github.com/shadowsocks/shadowsocks-windows/issues/2171#issuecomment-603119696>

运行一下

```
netsh int ipv4 show dynamicport tcp
```

显示的输出是

```
协议 tcp 动态端口范围
---------------------------------
启动端口        : 1024
端口数          : 13977
```

启动端口是1024，这里应该是指 tcp 发起连接时使用的端口范围，不知道什么时候因为什么被改成了1024，导致中位端口很容易被发起连接占用，而应用则无法正常使用这些端口。

管理员模式下使用

```
netsh int ipv4 set dynamic tcp start=49152 num=16384
```

可以重置为Windows的默认值，一般一万多个并发连接也就够用了，毕竟不是服务器环境。然后重启即可~
