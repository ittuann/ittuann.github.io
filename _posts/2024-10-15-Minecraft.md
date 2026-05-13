---
layout: article
title: Minecraft Java 开服记录
date: 2024-10-15
key: P2024-10-15
tags: Other
show_author_profile: true
comment: true
sharing: true
aside:
  toc: true
---

Ubuntu 开 Fabric 服

<!--more-->

# Server 服务端

1.14及以后版本由于Forge优化较差（主要体现在加载速度慢）及主流Mod开发者逐渐转移至Fabric，建议使用Fabric。

- Fabric <https://fabricmc.net/use/server/>
- 官方Java版本 <https://www.minecraft.net/zh-hans/download/server>

## 配置Java环境

- Java最低版本要求

| 游戏版本    | Java版本需求     |
| ----------- | ---------------- |
| 1.13 - 1.16 | Java8或更高版本  |
| 1.17        | Java16或更高版本 |
| 1.18 - 1.20 | Java17或更高版本 |
| 1.21        | Java21或更高版本 |

一般Java版本越高可能会为游戏带来更好的性能、更高的安全性和更少的漏洞。

- JDK和JRE的关系

JDK（Java Development Kit，Java开发工具包）用于开发Java程序，JRE（Java Runtime Environment，Java运行环境）用于运行Java程序；JDK包含JRE，而JRE中包含了JVM；

原版Minecraft只需使用JRE。但由于某些插件或Mod可能需要用到JDK的一些功能，所以建议安装JDK。

- 安装

```
sudo apt update
sudo apt install openjdk-21-jdk-headless

java -version
```

`headless`无头Java不会安装Java的图形界面GUI组件。

## 端口

```
sudo ufw allow 25565/tcp
```

## 启动

```shell
java -Xmx2G -Xms512M -jar fabric-server-mc.1.21.1-loader.0.16.7-launcher.1.0.1.jar nogui
```

参数:

| 参数 | 作用                   | 解释                                         |
| ---- | ---------------------- | -------------------------------------------- |
| -Xmx | 设置JVM堆内存的最大值  | 最大消耗内存                                 |
| -Xms | 设置JVM堆内存的初始值  | 只影响启动性能                               |
| -Xmn | 设置新生代堆内存的大小 | 指定年轻代（Eden区、Survivor区）的空间大小。 |

- 最大内存`-Xmx2G`对5名玩家基本够了。
- 首次运行: 需要同意elua

# 配置

- `server.properties`文件：

```
online-mode=false # 关闭正版验证
pvp=true   # PVP
difficulty=normal  # 游戏难度。默认为easy，可选peaceful/easy/normal/hard
motd=\u00a7oittuann\u00a7r \u00a72Minecraft\u00a7r Server\u2764	# 服务器描述
```

- 控制台常用命令记录：

```
/op xxx    # 给xxx玩家op权限
/deop xxx   # 去除xxx玩家op权限

/tp <玩家名> <X Y Z 坐标>

/locate biome minecraft:cherry_grove # 最近的樱花树林
/seed         # 显示当前世界种子
```

# Server-side Fabric mod

下载后直接放到服务端`mod`文件夹下即可安装。

- Fabric API <https://modrinth.com/mod/fabric-api>
- Essential Commands <https://modrinth.com/mod/essential-commands> 命令 /tpa /back /home /rtp

```
allow_back_on_death=true # 允许back回死亡地点
language=zh_cn    # 文本语言
home_limit=[5, 6, 7]  # 增加home的数量上限
```

- Lithium <https://modrinth.com/mod/lithium> 神奇的性能优化。Srats和下载量还很高
- FancyClear <https://www.curseforge.com/minecraft/mc-mods/fancyclear> 实体和掉落物清理

```
AutoClear: true    # 开启自动清理
Mob:
 clear: false
# 关闭魔物/生物清理。因为black-list.yml设置的生物清理排除名单，并不包含完整的新版生物。
```

- No Chat Reports <https://modrinth.com/mod/no-chat-reports>

## 可选mod

- EasyAuth <https://modrinth.com/mod/easyauth> 登录验证
- Fabric Tailor <https://modrinth.com/mod/fabrictailor> 皮肤
- Cadmus (Land Claiming) <https://modrinth.com/mod/cadmus> 圈地/领地
- LuckPerms <https://modrinth.com/mod/luckperms> 权限管理
- Chunky <https://modrinth.com/plugin/chunky> 预生成区块
- Simple Voice Chat <https://modrinth.com/plugin/simple-voice-chat> 游戏内语音(同时需要客户端mod)
- Fast Backups https://modrinth.com/mod/fastback 世界备份
- FallingTree <https://modrinth.com/mod/fallingtree> 砍树
- Carry On <https://modrinth.com/mod/carry-on> 搬运箱子

# 客户端推荐mod

- 信息类:

```
[模组菜单] Mod Menu
[锂] Lithium
[JEI物品管理器] Just Enough Items
    选中物品后按R键即可显示该物品配方
[JER] Just Enough Resources  # 为JEI添加生物掉落等信息
[JEI拼音搜索] Just Enough Characters
[玉] Jade 🔍
[合成辅助] Crafting Tweaks
[一键背包整理] Inventory Profiles Next
    https://www.mcmod.cn/post/2650.html
[苹果皮] appleskin
[Xaero的小地图] Xaeros_Minimap
[Xaero的世界地图] XaerosWorldMap
[附魔介绍] Enchantment Descriptions
```

- 社交类:

```
[禁用聊天举报] No Chat Reports
[更多聊天记录] More Chat History
[聊天头像] chat-heads
```

- 显示效果:

```
[落叶粒子效果] Falling Leaves
[实体模型特性] Entity Model Features(EMF)
[光影] Iris Shaders
[动态光源] LambDynamicLights
[光线追踪] Photonics: A raytracing engine
```

- 建筑

```
[投影] Litematica
```

- 其他

```
[搬运] Carry On
[地毯] Carpet
	仙人掌扳手
ReplayMod
Freecam (Modrinth Edition)
```

如果服务端没有`Essential Commands`的`/back`等命令支持，客户端单人游戏可以安装`FTB Essentials`实现类似的指令。

## 客户端推荐光影

Iris Shaders <https://modrinth.com/mod/iris> （替代OptiFine）

光影文件位置在`.minecraft/shaderpacks`文件夹中。

- BSL Shaders - Original <https://modrinth.com/shader/bsl-shaders>
- Complementary Shaders - Unbound <https://modrinth.com/shader/complementary-unbound>

## 可选材质包

- Better Leaves https://modrinth.com/resourcepack/better-leaves
- Dramatic Skys https://modrinth.com/resourcepack/dramatic-skys
- Fresh Animations https://modrinth.com/resourcepack/fresh-animations
- 红石辅助RedstoneAuxiliary <https://modrinth.com/resourcepack/redstoneauxiliary>

其他有趣的材质包：

- Roundista https://modrinth.com/resourcepack/roundista

# 参考链接

> 架设Mod服务器 <https://zh.minecraft.wiki/w/Tutorial:%E6%9E%B6%E8%AE%BEMod%E6%9C%8D%E5%8A%A1%E5%99%A8>
>
> 控制台命令 <https://minecraft.fandom.com/zh/wiki/%E5%91%BD%E4%BB%A4>
>
> 教程Fabric-Server-Mod索引 <https://www.mcmod.cn/post/2318.html>
>
> 配置文件优化 <https://mhy278.github.io/MinecraftServerHostGuideHtml/Optimization.html>
