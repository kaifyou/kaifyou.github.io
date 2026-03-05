---
title: Hexo博客配置部署GitHub指南
date: 2026-03-05 23:25:31
tags:
---
# 📝 Hexo博客完整配置指南

基于我们之前的沟通，以下是你在Ubuntu系统上配置Hexo博客并部署到GitHub Pages的完整记录。这份指南包含了所有成功的关键步骤。

---

## 📦 一、环境准备

### 1.1 Node.js环境配置

```bash
# 检查Node.js版本（确保使用v22）
node -v  # v22.x.x

# 如果系统中有多个npm，确保使用正确的路径
export PATH=/usr/local/node-v22/bin:$PATH  # 临时生效
# 永久生效：添加到 ~/.bashrc
echo 'export PATH=/usr/local/node-v22/bin:$PATH' >> ~/.bashrc
source ~/.bashrc

# 验证
which npm  # 应显示 /usr/local/node-v22/bin/npm
npm -v
```

### 1.2 安装Hexo CLI

```bash
# 全局安装hexo命令行工具
npm install -g hexo-cli

# 验证安装
hexo version
```

---

## 🏗️ 二、初始化博客

### 2.1 创建博客目录

```bash
# 进入你的博客工作目录
cd /local/Develop/myBlog/

# 备份已有的GitHub Pages仓库（如果有）
cp -r kaifyou.github.io kaifyou.github.io.bak

# 进入仓库目录
cd kaifyou.github.io
```

### 2.2 由于目录非空，使用临时目录法初始化

```bash
# 返回上级目录
cd /local/Develop/myBlog/

# 创建临时目录
mkdir hexo-temp
cd hexo-temp

# 在空目录初始化Hexo
hexo init .
npm install  # 安装依赖
```

### 2.3 复制文件到博客目录

```bash
# 复制所有Hexo文件（包括隐藏文件）到博客目录
cp -r . ../kaifyou.github.io/
cd ../kaifyou.github.io/

# 确认.git目录还在（保留Git历史）
ls -la | grep .git

# 清理临时目录
cd /local/Develop/myBlog/
rm -rf hexo-temp
```

---

## ⚙️ 三、配置文件

### 3.1 博客配置文件 `_config.yml`

```bash
vim _config.yml
```

修改以下关键配置：

```yaml
# Site
title: 你的博客标题
subtitle: 副标题
description: 博客描述
author: 你的名字
language: zh-CN
timezone: Asia/Shanghai

# URL
url: https://kaifyou.github.io
root: /
permalink: :year/:month/:day/:title/

# Deployment
deploy:
  type: git
  repo: https://github.com/kaifyou/kaifyou.github.io.git
  branch: main
  message: "Site updated: {{ now('YYYY-MM-DD HH:mm:ss') }}"
```

### 3.2 安装部署插件

```bash
npm install hexo-deployer-git --save
```

### 3.3 确认 `package.json` 中的脚本

```json
{
  "scripts": {
    "build": "hexo generate",
    "clean": "hexo clean",
    "deploy": "hexo deploy",
    "server": "hexo server"
  }
}
```

---

## 🤖 四、GitHub Actions自动部署配置

### 4.1 创建工作流目录和文件

```bash
mkdir -p .github/workflows
vim .github/workflows/pages.yml
```

### 4.2 完整的工作流配置（确保缩进正确！）

```yaml
name: Pages

on:
  push:
    branches:
      - main

jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      
      - name: Use Node.js
        uses: actions/setup-node@v4
        with:
          node-version: "22"
          cache: 'npm'
      
      - name: Install Dependencies
        run: npm ci
      
      - name: Build
        run: npx hexo generate
      
      - name: Upload Pages artifact
        uses: actions/upload-pages-artifact@v3
        with:
          path: ./public

  deploy:
    needs: build
    permissions:
      pages: write
      id-token: write
    environment:
      name: github-pages
      url: ${{ steps.deployment.outputs.page_url }}
    runs-on: ubuntu-latest
    steps:
      - name: Deploy to GitHub Pages
        id: deployment
        uses: actions/deploy-pages@v4
```

**⚠️ 关键点：**
- 必须用**空格**，不能用Tab
- 每层缩进**2个空格**
- `- main` 前面要有**6个空格**

---

## 🔑 五、GitHub认证配置

### 5.1 生成Personal Access Token

1. 登录GitHub → 头像 → **Settings**
2. 左侧底部 → **Developer settings**
3. **Personal access tokens** → **Tokens (classic)**
4. **Generate new token (classic)**
5. 配置：
   - **Note**: `Hexo blog`
   - **Expiration**: 90 days（或No expiration）
   - **Scopes**: 勾选 `repo` 和 `workflow`
6. 点击 **Generate token** 并**保存好token**

### 5.2 配置Git记住认证信息

```bash
# 配置credential helper
git config --global credential.helper store

# 推送一次，输入用户名和token
git push origin main
# 用户名: kaifyou
# 密码: 粘贴你的token（粘贴后不显示，直接回车）
```

---

## 🚀 六、分支管理

### 6.1 重命名分支为main（可选）

```bash
# 如果本地是master，重命名为main
git branch -m master main

# 推送main到远程
git push -u origin main

# 在GitHub网站修改默认分支为main
# Settings → Branches → Default branch → 切换到main

# 删除远程master分支
git push origin --delete master

# 清理本地缓存的远程分支信息
git remote prune origin
```

---

## 🌐 七、GitHub Pages配置

### 7.1 仓库Pages设置

1. 进入仓库 `https://github.com/kaifyou/kaifyou.github.io`
2. 点击 **Settings** → **Pages**
3. 在 "Build and deployment" 的 "Source" 中，选择 **"GitHub Actions"**

### 7.2 Environments配置

1. 进入仓库 **Settings** → **Environments**
2. 点击 **`github-pages`** 环境
3. 在 "Deployment branches" 部分：
   - 点击 **"Add deployment branch rule"**
   - 输入：`main`
   - 点击 **"Add rule"**

---

## ✍️ 八、写文章和部署流程

### 8.1 创建新文章

```bash
# 创建文章
hexo new post "我的第一篇文章"

# 编辑文章
vim source/_posts/我的第一篇文章.md
```

文章格式示例：
```markdown
---
title: 我的第一篇文章
date: 2026-03-05 20:00:00
tags:
- Hexo
- 博客
categories:
- 技术
---

这里是文章内容，支持 **Markdown** 语法。
```

### 8.2 本地预览（可选）

```bash
hexo clean
hexo generate
hexo server
# 访问 http://localhost:4000 预览
```

### 8.3 部署到GitHub

```bash
# 提交代码
git add .
git commit -m "post: 添加第一篇文章"

# 推送到GitHub（自动触发Actions部署）
git push origin main
```

### 8.4 查看部署状态

```bash
# 查看GitHub Actions状态
gh run list
gh run watch  # 监控当前运行
```

或者直接在GitHub仓库的 **Actions** 标签页查看。

---

## 🎯 九、验证结果

部署成功后访问：
```
https://kaifyou.github.io
```

---

## 📋 十、常用命令速查表

| 命令 | 说明 |
|------|------|
| `hexo new post "标题"` | 创建新文章 |
| `hexo generate` 或 `hexo g` | 生成静态文件 |
| `hexo server` 或 `hexo s` | 本地预览 |
| `hexo clean` | 清理缓存 |
| `git add .` | 添加所有修改 |
| `git commit -m "message"` | 提交修改 |
| `git push origin main` | 推送到GitHub |
| `gh run list` | 查看Actions运行状态 |
| `gh run rerun --failed` | 重新运行失败的作业 |

---

## 🎉 恭喜！

你的Hexo博客已经完全配置好，并且实现了：
- ✅ 本地写作和预览
- ✅ GitHub自动部署
- ✅ 一推送即发布

现在可以开始愉快地写博客了！有任何问题随时问我。
