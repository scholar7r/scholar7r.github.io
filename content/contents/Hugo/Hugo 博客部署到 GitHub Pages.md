---
title: "Hugo 博客部署到 GitHub Pages"
date: 2023-09-09T07:06:03+08:00
categories: Hugo
draft: false
---
GitHub 向个人和组织提供免费的静态托管服务，在 GitHub Pages 服务的加持下，用户可以直接从储存库中构建可直接访问的网站。本篇演示如果通过 GitHub Actions 服务自动部署 Hugo 网站。

<!--more-->

## 前言

本篇提到的部署方式有区别于中文 Hugo 网站的多数部署方案，并不采用在 public 文件夹下创建 Git 仓库的方法，而是将整个 Hugo 站点（包含所有的配置文件）一同上传到 GitHub。

貌似这种方法会产生一些歧义，会产生类似于「为什么同步整个站点而不同步生成的文件？」这些问题。理由是通过 GitHub Actions 可以将生成的文件直接同步到 GitHub Pages，这样也保证了无论身在何处都可以操作博客系统。

## 要求

- GitHub 标准账户
- 系统中安装了 Git

## 操作步骤

首先，在 GitHub 创建为存放 Hugo 站点的储存库，不需要刻意的使用 `username + github.io` 的形式，随意命名即可。

使用 `hugo new site sitename` 命令创建 Hugo 站点，并切换到此目录中，完成 Hugo 站点初始化和 Git 的账户设置。

此时可以在 GitHub 的储存库页面选择 Settings 选项并进入 Pages 项。在接下来的 Build and deployment 下的 Source 方法更换为 GitHub Actions。这一步是为了接下来的 Workflows 能够直接运行。

在 Hugo 目录下创建 `.github/workflows/hugo.yaml` 文件，并且写入以下内容。

```yaml
# Sample workflow for building and deploying a Hugo site to GitHub Pages
name: Deploy Hugo site to Pages

on:
  # Runs on pushes targeting the default branch
  push:
    branches:
      - main

  # Allows you to run this workflow manually from the Actions tab
  workflow_dispatch:

# Sets permissions of the GITHUB_TOKEN to allow deployment to GitHub Pages
permissions:
  contents: read
  pages: write
  id-token: write

# Allow only one concurrent deployment, skipping runs queued between the run in-progress and latest queued.
# However, do NOT cancel in-progress runs as we want to allow these production deployments to complete.
concurrency:
  group: "pages"
  cancel-in-progress: false

# Default to bash
defaults:
  run:
    shell: bash

jobs:
  # Build job
  build:
    runs-on: ubuntu-latest
    env:
      HUGO_VERSION: 0.115.4
    steps:
      - name: Install Hugo CLI
        run: |
          wget -O ${{ runner.temp }}/hugo.deb https://github.com/gohugoio/hugo/releases/download/v${HUGO_VERSION}/hugo_extended_${HUGO_VERSION}_linux-amd64.deb \
          && sudo dpkg -i ${{ runner.temp }}/hugo.deb          
      - name: Install Dart Sass
        run: sudo snap install dart-sass
      - name: Checkout
        uses: actions/checkout@v3
        with:
          submodules: recursive
          fetch-depth: 0
      - name: Setup Pages
        id: pages
        uses: actions/configure-pages@v3
      - name: Install Node.js dependencies
        run: "[[ -f package-lock.json || -f npm-shrinkwrap.json ]] && npm ci || true"
      - name: Build with Hugo
        env:
          # For maximum backward compatibility with Hugo modules
          HUGO_ENVIRONMENT: production
          HUGO_ENV: production
        run: |
          hugo \
            --gc \
            --minify \
            --baseURL "${{ steps.pages.outputs.base_url }}/"          
      - name: Upload artifact
        uses: actions/upload-pages-artifact@v1
        with:
          path: ./public

  # Deployment job
  deploy:
    environment:
      name: github-pages
      url: ${{ steps.deployment.outputs.page_url }}
    runs-on: ubuntu-latest
    needs: build
    steps:
      - name: Deploy to GitHub Pages
        id: deployment
        uses: actions/deploy-pages@v2
```

在上述的内容中，可以对某些字段做出改变，例如 HUGO_VERSION 可以参考其最新版本，保证与本地项目的一致性。

接下来，只需要同步储存库到 GitHub，GitHub Actions 将会被自动执行，生成并且托管网站到 GitHub Pages 服务中。

--- 

参考文档：

[Host on GitHub Pages | Hugo (gohugo.io)](https://gohugo.io/hosting-and-deployment/hosting-on-github/)
