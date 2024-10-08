---
layout: article
title: Windows包管理器Scoop&Winget
date: 2023-02-04
key: P2023-02-04
tags: ["Other"]
show_author_profile: false
comment: true
sharing: true
aside:
  toc: true
---

Windows 平台下的包管理器 Scoop, Chocolatey, Winget. 使用下来我个人其实更喜欢的是 Scoop 的理念。

<!--more-->

# Scoop

Scoop 官网 <https://scoop.sh/>

Scoop 本身和 Scoop 安装过程参考的配置文件都是开源的，要安装什么一目了然。

## 设置代理

安装参考官网就可以。

安装时可能会出现特色的网络问题，给PowerShelll挂上代理即可。但是PowerShelll原生并不支持socks5代理。

### 可选：将SOCKS5代理转换为HTTP代理

需要用到额外的第三方软件 **Privoxy** https://www.privoxy.org/

Privoxy 是一个跨平台的非缓存的 Web 代理软件，它可以用来修改 Web 页面和 HTTP 头部、控制访问等。这里我们只需要用到HTTP 代理转发和 SOCKS 代理转发，把收到的请求转给另一个代理。

安装完成后，修改 C:\Program Files (x86)\Privoxy\config.txt 配置文件。在文件的最后追加

```
forward-socks5 / 127.0.0.1:3389 .
```

注意这一行里面有`/`和`.`不要忘记输入了

在 Privoxy 默认的配置文件中已经设置了 `listen-address  127.0.0.1:8118` 表示监听的端口为本地8118端口。

配置完成后重启Privoxy即可。然后本地8118端口即为HTTP代理。

另外，如果用Clash的情况是不需要额外把SOCKS5代理转换为HTTP代理的。

然后继续配置代理：

- 配置临时环境变量：

```shell
$env:HTTP_PROXY="http://127.0.0.1:7890"
$env:HTTPS_PROXY="http://127.0.0.1:7890"
```

这组命令将`HTTP_PROXY`和`HTTPS_PROXY`环境变量设置为指定的代理服务器。

这种方法仅对当前PowerShell会话中运行的命令生效，不影响其他应用程序或Windows系统级别的代理设置。这些设置在PowerShell会话结束后将不再有效。

（如果使用cmd的话则对应：

```shell
set http_proxy=http://127.0.0.1:7890
set https_proxy=http://127.0.0.1:7890
set all_proxy=socks5://127.0.0.1:7890
```

验证代理是否配置成功

```bash
curl https://www.google.com/
```

- 或是可以使用`netsh`工具来设置Windows系统级别的代理（不建议）：

```shell
netsh winhttp show proxy				# 查看代理
netsh winhttp set proxy 127.0.0.1:7890	# 设置代理
netsh winhttp reset proxy				# 恢复默认取消代理（直连
```

这种方法会影响整个系统和所有支持使用系统代理设置的应用程序。这些设置将一直有效，直到手动更改或清除代理设置。

## 语法

Scoop 的命令非常语义化。它命令的设计很简单，基本为「scoop + 动作 + 对象」的语法（其中「对象」是可省略的

最常用的几个基础动作：

| 命令      | 动作         |
| --------- | ------------ |
| search    | 搜索软件名   |
| install   | 安装软件     |
| update    | 更新软件     |
| status    | 查看软件状态 |
| uninstall | 卸载软件     |
| info      | 查看软件详情 |
| home      | 打开软件主页 |

- Scoop 在你的用户根目录（一般是 C:\Users\用户名）下创建了一个名为 scoop 的文件夹，并默认将软件下载安装到这个文件夹下。其中 `apps` 存放有安装的所有应用。

## 配置软件仓库

Scoop 默认软件仓库（main bucket），收录条件比较严格和苛刻，其中一条就是不可以有 GUI。所以为了日常方便使用，可以添加其他的bucket

添加extras这个 bucket。地址: https://github.com/ScoopInstaller/Extras/tree/master/bucket

```shell
scoop bucket add extras
```

## 代理

```shell
# 使用代理
set scoop_proxy=socks5://127.0.0.1:7890

# 停止使用代理
set scoop_proxy=
```

## 安装软件包

```shell
scoop update
scoop cache clear
scoop install jq
```

# Chocolatey

官网: <https://chocolatey.org>

choco的好处就是收录的软件更多。

我个人其实更喜欢的是 Scoop 的理念。

# Winget

Winget是微软的Windows平台软件包管理器。现在应该是Win11都会自带了。

安装的时候依旧会弹出界面，不会完全在CLI下安装。

注意，Winget 第一次使用时会要求上传用户所在地的国家信息。

## 配置winget

- 更改 winget 显示的进度条视觉效果

先输入 `winget settings` 打开 setting.json

然后增加

```json
    "visual": {
        "progressBar": "rainbow"
        // 三种样式可选: accent(默认值), retro, rainbow
    },
```

这样子进度条就会变成彩虹色了

- 日志

共有5个等级可选： "verbose", "info", "warning", "error", "critical"

winget --verbose-logs 将覆盖这一设置，并始终创建一个verbose日志。

```json
    "logging": {
		"level": "verbose"
    },
```

- 代理

Winget配置代理同样可以使用上面提到的[环境变量](#安装时的代理)来进行配置。

更多的参数，在Github文档有描述
WinGet CLI Settings https://github.com/microsoft/winget-cli/blob/master/doc/Settings.md

# WingetUI

<https://github.com/marticliment/WingetUI>

这是个很棒的软件。它能够为常见的 CLI 包管理器（如 Winget、Scoop 和 Chocolatey）创建一个直观的 GUI。通过这个应用，可以轻松地下载、安装、更新和卸载任何支持包管理器发布的软件，以及更多功能。

---

参考链接：

> Scoop https://github.com/ScoopInstaller/Scoop/wiki

> 给 Scoop 加上这些软件仓库，让它变成强大的 Windows 软件管理器 https://sspai.com/post/52710

> 使用 winget 工具安装和管理应用程序 https://learn.microsoft.com/zh-cn/windows/package-manager/winget/

> 这或许是 Windows 上最好的包管理工具：Windows Package Manager 1.0 https://sspai.com/post/67005
