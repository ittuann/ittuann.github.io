---
layout: article
title: 降低 Microsoft Defender 的性能占用
date: 2025-11-21
key: P2025-11-21
tags: Other
show_author_profile: true
comment: true
sharing: true
aside:
  toc: true
---

降低 Microsoft Defender 的性能占用：限制CPU使用率的最大百分比、配置低CPU优先级

<!--more-->

## Microsoft Defender

Microsoft Defender Antivirus 默认CPU使用率的上限是50%，这很可能会影响系统性能。

而问题是它不能直接设置。任务管理器右键 Antimalware Service Executable 转到详细信息 MsMpEng.exe，无法设置优先级为低，也无法设置相关性至选择最下方的CPU。

调整步骤：

1. 打开 `打开本地组策略编辑器`

Win+R gpedit.msc

2. 导航至 `计算机配置/管理模板/Windows 组件/Microsoft Defender 防病毒/扫描`

3. 双击`指定扫描期间CPU使用率的最大百分比` 策略，将其设置为`已启用`，并在选项中输入`5`。

可以输入一个介于 5 到 100 之间的最大 CPU 使用率值。默认值是50。如果设置值为 0 将禁用 CPU 限制，也就是允许它使用任意多的 CPU 资源。

4. 双击`配置计划扫描的低CPU优先级`，将其设置为`已启用`。

## 参考资源

https://www.tenforums.com/tutorials/142728-set-windows-defender-antivirus-max-cpu-usage-scan-windows-10-a.html
