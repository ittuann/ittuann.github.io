---
layout: article
title: 使用 Rclone/Restic 进行定期备份
date: 2024-10-21
key: P2024-10-21
tags: Linux
show_author_profile: true
comment: true
sharing: true
aside:
  toc: true
---

使用 Rclone  对文件夹进行简易每日备份，以及使用 Restic 备份工具。

<!--more-->

# Rclone 

Sync同步。Rclone它更多的是作为云同步工具。没有Snapshots快照版本管理功能。

简易备份的方式有2种，一种是用增量同步（只上传改变的文件）待备份文件夹，另一种是先将目标文件夹压缩为zip再上传压缩文件夹。

## 安装

```bash
sudo apt update
sudo apt install rclone
```

## 配置 rclone webdav

```bash
rclone config
```

然后根据提示，完成配置remote的webdav设置。

测试配置：

```bash
rclone ls <remote_name>:</path/to/remote/>
```

## 配置 rclone sftp/ssh

没有选择用sftp而是用了webdav，是处于安全性的考虑。

rclone支持3种身份验证方法：Password、Key file、ssh-agent

用rclone本身加载Key file时，仅能加载未加密（没有密码）的key。而加载加密 OpenSSH 密钥的唯一方法是使用 ssh-agent。 <https://rclone.org/sftp/#ssh-authentication>

因为rclone不能在工具内部安全的保存密钥的密码，而创建未加密的密钥并不安全，此外为了一个全自动定时脚本而让ssh-agent始终存储密钥也不够安全。

## 压缩备份同步脚本

步骤是先将目标文件夹压缩为带密码的zip文件，再将整个压缩文件夹复制到远程。

先创建一下ZIP压缩文件密码:

```bash
echo "your_password_here" > /path/to/password_file
chmod 600 /path/to/password_file
```

每日同步`zip_rclone_backup.sh`脚本。将`<>`尖括号的部分替换为实际路径即可。

```bash
#!/bin/bash
set -euo pipefail

# Define source directory
SOURCE_DIR="</path/to/your/files>"

# Define zip
BACKUP_ZIP_DIR="</path/to/backup_local>"
ZIP_PASSWORD=$(cat "</path/to/password_file>")

# Define remote destination
REMOTE_NAME="<remote>"
REMOTE_DIR="</path/to/remote/directory>"

DATE_STR=$(date +'%Y-%m-%d_%H-%M')
BACKUP_ZIP_FILE="${BACKUP_ZIP_DIR}/backup_${DATE_STR}.zip"
mkdir -p "${BACKUP_ZIP_DIR}/logs"

# Zip
zip -P "${ZIP_PASSWORD}" -r ""${BACKUP_ZIP_FILE}" "${SOURCE_DIR}" > "${BACKUP_ZIP_DIR}/logs/zip_${DATE_STR}.log" 2>&1  || { echo "Zip failed"; exit 1; }

# Rclone Upload
rclone sync "${BACKUP_ZIP_DIR}" "${REMOTE_NAME}":"${REMOTE_DIR}" --verbose --progress --log-file="${BACKUP_ZIP_DIR}/logs/rclone_${DATE_STR}.log" || { echo "Rclone failed"; exit 1; }

# (Optionally) clean up locally old zip files 
find "${BACKUP_ZIP_DIR}" -name "*.zip" -type f -mtime +7 -exec rm {} \; || { echo "Clean failed"; exit 1; }
```

如果只想要上传，而不想删除远程的历史备份版本，就用`rclone copy`命令。

使用 `chmod +x backup_script.sh` 来赋予脚本执行权限。

## 定时执行

打开 `crontab` 编辑器：

```bash
crontab -e
```

将其添加到 `cron` 中。例如每天凌晨4点执行一次备份。

```
0 4 * * * /path/to/your/script.sh
```

# Restic 

使用 Restic 进行备份。TODO待补充



# 其他选择

- Duplicati

开源项目，是Web页面对操作更友好。但CLI命令行的支持在官方文档中就写了一页，感觉软件会更侧重于GUI而不是用于服务器的CLI。有Docker镜像可以直接拉。