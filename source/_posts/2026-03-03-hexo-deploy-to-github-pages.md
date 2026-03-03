---
title: Hexo 博客部署到 GitHub Pages 完整指南
date: 2026-03-03 16:20:00
tags:
  - Hexo
  - GitHub Pages
  - 部署
categories:
  - 技术教程
---

## 前言

最近在整理博客时，发现博客无法正常部署到 GitHub Pages。经过排查，问题已解决，特此记录完整的部署流程，供有需要的朋友参考。

## 问题描述

博客使用 Hexo 框架搭建，之前一直可以正常访问，但某次部署时发现无法访问。经过检查发现以下问题：

1. `_config.yml` 中的 `deploy` 配置为空
2. 缺少部署依赖包 `hexo-deployer-git`

## 解决步骤

### 1. 安装部署依赖

首先需要安装 `hexo-deployer-git` 插件：

```bash
npm install hexo-deployer-git --save
```

### 2. 配置 _config.yml

在博客根目录的 `_config.yml` 文件中，修改或添加 deployment 配置：

```yaml
# Deployment
## Docs: https://hexo.io/docs/one-command-deployment
deploy:
  type: git
  repo: git@github.com:p-self/p-self.github.io.git
  branch: gh-pages
```

配置说明：

- **type**: 部署方式，这里使用 git
- **repo**: 仓库地址，注意使用 SSH 格式（git@github.com:...）而不是 HTTPS
- **branch**: 部署分支，GitHub Pages 默认使用 gh-pages

### 3. 部署到 GitHub

配置完成后，执行以下命令进行部署：

```bash
# 清理缓存并生成静态文件
hexo clean
hexo generate

# 或者直接使用一键部署
npm run deploy
```

部署成功后，会看到类似输出：

```
INFO  Deploying: git
INFO  Clearing .deploy_git folder...
INFO  Copying files from public folder...
To github.com:p-self/p-self.github.io.git
 * [new branch]      HEAD -> gh-pages
INFO  Deploy done: git
```

### 4. 启用 GitHub Pages

部署完成后，还需要在 GitHub 仓库设置中启用 GitHub Pages：

1. 进入仓库页面，点击 **Settings**
2. 在左侧菜单中找到 **Pages**
3. 在 **Build and deployment** 部分：
   - Source 选择 **Deploy from a branch**
   - Branch 选择 **gh-pages**
   - 点击 Save

等待几分钟后，网站就可以通过自定义域名访问了（本博客使用 pself.net）。

## 注意事项

### SSH vs HTTPS

配置 repo 时，有两种方式：

- SSH 格式：`git@github.com:username/repo.git`
- HTTPS 格式：`https://github.com/username/repo.git`

推荐使用 SSH 格式，避免每次部署时输入用户名和密码。但需要确保本地已配置 SSH 公钥。

### 部署分支

GitHub Pages 支持从以下分支部署：

- `gh-pages`：适用于项目仓库
- `master` / `main`：适用于用户或组织页面

本博客采用 `gh-pages` 分支进行部署。

### 自定义域名

如果需要使用自定义域名（如 pself.net），需要在仓库根目录添加 CNAME 文件，内容为你的域名。然后在域名提供商处配置 DNS 解析到 GitHub Pages。

## 总结

Hexo 部署到 GitHub Pages 的流程相对简单，关键在于正确配置 `hexo-deployer-git` 插件和仓库信息。部署完成后，GitHub 会自动处理静态页面的托管，配合自定义域名即可实现稳定的博客访问。