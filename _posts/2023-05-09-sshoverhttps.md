---
layout: article
title: 通过 HTTPS 端口使用 SSH
date: 2023-05-09
key: P2023-05-09
tags: ["Linux"]
show_author_profile: false
comment: true
sharing: true
aside:
  toc: true
---

通过 HTTPS 端口使用 SSH，解决国内网络下GitHub无法使用22端口拉取代码的问题

<!--more-->

有时防火墙会完全拒绝允许SSH连接。如果不能使用启用凭证缓存的HTTPS克隆选项，则可以尝试使用在HTTPS端口上建立的SSH连接进行克隆。大多数防火墙规则应该允许此操作，但代理服务器可能会干扰。

# 测试

要测试 SSH HTTPS 端口是否可行，请运行 SSH 命令：

```shell
ssh -T -p 443 git@ssh.github.com
```

如果输出以下内容，则说明成功：

```
> Hi USERNAME! You've successfully authenticated, but GitHub does not provide shell access.
```

注意：主机名为 [ssh.github.com](http://ssh.github.com/) 或 [github.com](http://github.com/) 的端口为 443。

在切换到端口443后，第一次与GitHub交互时，会收到一个警告消息，指出该主机未在known_hosts中找到或以其他名称找到。

```
> The authenticity of host '[ssh.github.com]:443 ([140.82.112.36]:443)' can't be established.
> ED25519 key fingerprint is SHA256:+DiY3wvvV6TuJJhbpZisF/zLDA0zPMSvHdkr4UvCOqU.
> This host key is known by the following other names/addresses:
>     ~/.ssh/known_hosts:32: github.com
> Are you sure you want to continue connecting (yes/no/[fingerprint])?
```

如果SSH指纹与GitHub发布的指纹之一匹配，则回答“是”是安全的。有关指纹列表，请参见 GitHub的SSH密钥指纹 https://docs.github.com/en/authentication/keeping-your-account-and-data-secure/githubs-ssh-key-fingerprints

您可以将以下 SSH 密钥条目添加到 `~/.ssh/known_hosts` 文件中，以避免手动验证 GitHub 主机

```
github.com ssh-ed25519 AAAAC3NzaC1lZDI1NTE5AAAAIOMqqnkVzrm0SdG6UOoqKLsabgH5C9okWi0dh2l9GKJl
github.com ecdsa-sha2-nistp256 AAAAE2VjZHNhLXNoYTItbmlzdHAyNTYAAAAIbmlzdHAyNTYAAABBBEmKSENjQEezOmxkZMy7opKgwFB9nkt5YRrYMjNuG5N87uRgg6CLrbo5wAdT/y6v0mKV0U2w0WZ2YB/++Tpockg=
github.com ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABgQCj7ndNxQowgcQnjshcLrqPEiiphnt+VTTvDP6mHBL9j1aNUkY4Ue1gvwnGLVlOhGeYrnZaMgRK6+PKCUXaDbC7qtbW8gIkhL7aGCsOr/C56SJMy/BCZfxd1nWzAOxSDPgVsmerOBYfNqltV9/hWCqBywINIR+5dIg6JTJ72pcEpEjcYgXkE2YEFXV1JHnsKgbLWNlhScqb2UmyRkQyytRLtL+38TGxkxCflmO+5Z8CSSNY7GidjMIZ7Q4zMjA2n1nGrlTDkzwDCsw+wqFPGQA179cnfGWOWRVruj16z6XyvxvjJwbz0wQZ75XK5tKSb7FNyeIEs4TT4jk+S4dhPeAUC5y+bDYirYgM4GC7uEnztnZyaVWQ7B381AK4Qdrwt51ZqExKbQpTUNn+EjqoTwvqNj4kqx5QUCI0ThS/YkOxJCXmPUWZbhjpCg56i+2aB6CmK2JGhn57K5mj0MNdBXA4/WnwH6XoPWJzK5Nyu2zB3nAZp+S5hpQs+p1vN1/wsjk=
```

现在，要克隆存储库，您可以运行以下命令：

```shell
git clone ssh://git@ssh.github.com:443/YOUR-USERNAME/YOUR-REPOSITORY.git
```

# 通过HTTPS启用SSH连接

如果能够通过端口443 SSH到 git@ssh.github.com ，您可以覆盖SSH设置以强制任何连接都通过该服务器和端口运行到GitHub.com

要在SSH配置文件中设置此项，请编辑位于`~/.ssh/config`的文件，并添加以下部分：

```
Host github.com
Hostname ssh.github.com
Port 443
User git
```

您可以通过连接一次 GitHub.com 来测试它是否有效：

```shell
$ ssh -T git@github.com
```

如果输出以下内容，则说明成功：

```
> Hi USERNAME! You've successfully authenticated, but GitHub does not provide shell access.
```



参考链接：

> Using SSH over the HTTPS port https://docs.github.com/en/authentication/troubleshooting-ssh/using-ssh-over-the-https-port