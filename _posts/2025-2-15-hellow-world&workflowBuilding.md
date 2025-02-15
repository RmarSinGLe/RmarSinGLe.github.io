---
title: "{{你好世界和工作流建设}}"
date: {{date:2025-02-15 12:00:00 +0800}}
categories: [开始]
tags: [工作流]
tags: [笔记]
---

# 你好世界和工作流建设

## 工作流建设

1. 标准化Obsidian笔记格式
确保笔记符合Jekyll的Front Matter格式要求：

在Obsidian中设置模板：
使用插件（如 Templater）生成包含必要Front Matter的模板：

·title: "{{title}}"
date: {{date:YYYY-MM-DD}}
categories: [分类]
tags: [标签]·

注：Chirpy可能需要额外字段（如layout），需根据主题文档调整。

2. 处理文件名与路径
Jekyll要求文章存储在_posts目录，且文件名为YYYY-MM-DD-title.md：

手动或脚本重命名：
保存笔记时，在Obsidian中手动添加日期前缀（如2023-10-01-My-Post.md）。

自动化脚本（推荐）：
编写脚本自动从Front Matter的date字段提取日期并重命名文件：

bash
复制
### 示例：Python脚本提取日期并重命名
import os
import frontmatter

obsidian_dir = "/path/to/obsidian/notes"
jekyll_posts_dir = "/path/to/chirpy/_posts"

for filename in os.listdir(obsidian_dir):
    if filename.endswith(".md"):
        post = frontmatter.load(os.path.join(obsidian_dir, filename))
        date = post.metadata.get("date", datetime.now().strftime("%Y-%m-%d"))
        new_name = f"{date}-{filename}"
        os.rename(os.path.join(obsidian_dir, filename), os.path.join(jekyll_posts_dir, new_name))
3. 同步笔记到Chirpy仓库
方案一：手动复制+Git推送
将处理后的Markdown文件复制到Chirpy的_posts目录。

使用Git命令提交并推送：

bash
复制
cd /path/to/chirpy
git add _posts/*
git commit -m "Add new post"
git push origin main
方案二：自动化同步脚本
监控Obsidian目录：
使用文件监控工具（如Python的watchdog）自动触发同步：

python
复制
from watchdog.observers import Observer
from watchdog.events import FileSystemEventHandler

class SyncHandler(FileSystemEventHandler):
    def on_modified(self, event):
        # 调用重命名和复制脚本
        os.system("python rename_and_copy.py")

observer = Observer()
observer.schedule(SyncHandler(), path="/path/to/obsidian/notes", recursive=True)
observer.start()
Git自动化：
在脚本中添加自动提交和推送逻辑（需配置SSH密钥免密推送）。

4. 处理图片和资源
统一资源路径：
在Obsidian中设置图片保存到assets/img（与Chirpy的图片路径一致），或在脚本中替换路径：

markdown
复制
![](assets/img/image.png)  # Chirpy中的路径
同步图片：
将Obsidian中的图片复制到Chirpy的assets/img目录。

5. 使用GitHub Actions自动部署（可选）
在Chirpy仓库配置GitHub Actions，推送后自动构建部署：

yaml
复制
# .github/workflows/deploy.yml
name: Deploy
on: [push]
jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: ruby/setup-ruby@v1
        with:
          ruby-version: 3.0
          bundler-cache: true
      - run: bundle exec jekyll build
      - uses: peaceiris/actions-gh-pages@v3
        with:
          github_token: ${{ secrets.GITHUB_TOKEN }}
          publish_dir: ./_site
工具推荐
Obsidian插件：

Templater：管理Front Matter模板

Obsidian Git：定时提交到Git仓库（需配合脚本过滤_posts目录）

文件同步工具：

FreeFileSync（图形化界面同步文件）

Rsync（命令行批量同步）

常见问题
Front Matter格式错误：
使用Jekyll Front Matter校验工具检查。

构建失败：
本地运行bundle exec jekyll serve调试后再推送。

图片不显示：
确保路径为相对路径（如/assets/img/image.png）。