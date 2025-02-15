---
title: "hello world"
date: 2025-02-15 12:00:00 +0800
categories: [开始]
tags: [工作流]
---
 hello world

## obsidian chirpy workflow
1. 标准化Obsidian笔记格式
确保笔记符合Jekyll的Front Matter格式要求：
在Obsidian中设置模板：
使用插件（如 Templater）生成包含必要Front Matter的模板
2. 处理文件名与路径
Jekyll要求文章存储在_posts目录，且文件名为YYYY-MM-DD-title.md：
手动或脚本重命名：
保存笔记时，在Obsidian中手动添加日期前缀（如2023-10-01-My-Post.md）。
自动化脚本（推荐）：
编写脚本自动从Front Matter的date字段提取日期并重命名文件
3. 同步笔记到Chirpy仓库
方案一：手动复制+Git推送
将处理后的Markdown文件复制到Chirpy的_posts目录。
使用Git命令提交并推送
方案二：自动化同步脚本
监控Obsidian目录：
使用文件监控工具（如Python的watchdog）自动触发同步：
Git自动化：
在脚本中添加自动提交和推送逻辑（需配置SSH密钥免密推送）
4. 处理图片和资源
统一资源路径：
在Obsidian中设置图片保存到assets/img（与Chirpy的图片路径一致），或在脚本中替换路径：
同步图片：
将Obsidian中的图片复制到Chirpy的assets/img目录
5. 使用GitHub Actions自动部署（可选）
在Chirpy仓库配置GitHub Actions，推送后自动构建部
工具推荐
Obsidian插件：
Templater：管理Front Matter模板
Obsidian Git：定时提交到Git仓库（需配合脚本过滤_posts目录）
文件同步工具：
FreeFileSync（图形化界面同步文件）
Rsync（命令行批量同步）