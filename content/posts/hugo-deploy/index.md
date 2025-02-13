---
title: '使用 Hugo 搭建个人博客网站并自动部署至 GitHub Pages'
date: '2025-02-12'
tags: ['Hugo']
draft: false
---

## 1. 前言

[Hugo](https://github.com/gohugoio/hugo)是一个用 Go 语言编写的静态网站生成器，
它可以将 Markdown 文件转换为 HTML 文件，支持主题和插件，
生成的网站可以直接部署到 GitHub Pages 、云服务器等。

本文将介绍如何使用 Hugo 搭建个人博客并使用 GitHub Actions 自动部署到 GitHub Pages 上。

实现的效果是，当我们将博客的 Markdown 文件推送到 GitHub 仓库时，
GitHub Actions 会自动构建 Hugo 网站，并将生成的 HTML 文件推送到 GitHub Pages 上，
这样我们就可以通过`https://<username>.github.io`访问我们的博客。

以后有机会再介绍如何将 Hugo 部署到云服务器上。

## 2. 安装 Hugo

具体安装方法可以参考[Hugo 官方文档](https://gohugo.io/getting-started/installing)。

以我使用的 Arch Linux 为例，可以通过以下命令安装 Hugo ：
```bash
sudo pacman -S hugo
```

## 3. 创建 Hugo 网站

初学者可以参考[Hugo 官方教程](https://gohugo.io/getting-started/quick-start)。
由于我使用的是[PaperMod](https://github.com/adityatelange/hugo-PaperMod)主题，
如果你也想使用这个主题，
可以参考[PaperMod 安装教程](https://github.com/adityatelange/hugo-PaperMod/wiki/Installation)。

以下我列出使用 git submodule 方法（推荐）创建 PaperMod 主题的 Hugo 网站的命令：
```bash
hugo new site blog --format yaml  # 可以替换 blog 为其他目录名
cd blog
git submodule add --depth=1 https://github.com/adityatelange/hugo-PaperMod.git themes/PaperMod
echo "theme: [\"PaperMod\"]" >> hugo.yaml
```

如果你未来想要更新 PaperMod 主题，可以使用以下命令：
```bash
git submodule update --remote --merge
```

## 4. 编写博客

可以参考[如何新建文章](https://gohugo.io/getting-started/quick-start/#add-content)，
以及[如何组织目录结构](https://gohugo.io/content-management/organization)。

1. 新建文章
```bash
hugo new content content/posts/hello-world/index.md
```

`content/posts/hello-world/index.md`就是新建的文章的路径，
你可以在这个文件中使用 Markdown 语法编辑你的博客内容。

其中`draft`字段决定了文章是否为草稿，
如果你想要发布这篇文章，可以将`draft`字段设置为`false`。

2. 预览 Hugo 网站

当你完成了博客的编辑，可以使用以下命令在本地运行 Hugo 网站：
```bash
hugo server -D  # -D 参数表示需要显示草稿
```

你可以在浏览器中访问`http://localhost:1313`（终端中输出的地址）从而在本地浏览你的博客。

网站具有实时预览效果，每当你修改了博客内容后，浏览器会自动刷新。

3. 配置 Hugo 网站（可选）

具体网站的配置可以参考[Hugo 官方文档](https://gohugo.io/getting-started/configuration)以及[PaperMod 文档](https://github.com/adityatelange/hugo-PaperMod/wiki/Features)。
也可以参考我个人博客网站的[配置](https://github.com/m1dsolo/blog)。

## 5. 使用 Github 管理博客

使用版本控制工具可以方便我们管理博客的历史版本，同时也算是一种备份。

1. 创建 GitHub 仓库

具体如何创建可以搜网上的教程，这里不再赘述。
创建并提交后的仓库类似于[github.com/m1dsolo/blog](https://github.com/m1dsolo/blog)。

## 6. 部署到 GitHub Pages

到目前为止，我们已经完成了本地 Hugo 网站的搭建和博客的编写，
接下来我们将介绍如何使用 GitHub Actions 自动部署到 GitHub Pages 上。

1. GitHub Actions 配置

首先对于我们刚刚创建好的 GitHub 仓库，我们需要配置 GitHub Actions ，
这样每当我们将博客的 Markdown 文件推送到 GitHub 仓库时，
GitHub Actions 会自动构建 Hugo 网站，并将生成的 HTML 文件推送到 GitHub Pages 上。

在博客仓库的根目录下创建`.github/workflows`目录，
然后在该目录下创建`deploy.yml`文件，填入：
```yaml
name: Deploy with Hugo
 
on:
  push:
    branches:
      - main
 
jobs:
  build-deploy:
    runs-on: ubuntu-latest

    steps:
    - uses: actions/checkout@v4
      with:
        fetch-depth: 0

    - name: Setup Hugo
      uses: peaceiris/actions-hugo@v3
      with:
        hugo-version: 'latest'
        extended: true

    - name: Build
      run: hugo --minify

    - name: Deploy Github
      uses: peaceiris/actions-gh-pages@v4
      if: github.ref == 'refs/heads/main'
      with:
        personal_token: ${{ secrets.PERSONAL_TOKEN }}
        external_repository: m1dsolo/m1dsolo.github.io
        publish_branch: main
        publish_dir: ./public
```

2. 创建 Personal Token

因为 GitHub Actions 需要推送生成的 HTML 文件到 GitHub Pages 上，
也就是需要访问另外一个仓库，通过配置 Personal Token 可以赋予当前仓库操控另外一个仓库的权限。
可以参考[peaceiris/actions-gh-pages](https://github.com/peaceiris/actions-gh-pages?tab=readme-ov-file#%EF%B8%8F-set-personal-access-token-personal_token)。

如果之前没有创建过 Personal Token ，可以在[github tokens](https://github.com/settings/tokens)页面创建一个 token ，

3. 配置 Secrets

在 GitHub 博客仓库的 Settings -> Secrets 页面中，添加一个名为`PERSONAL_TOKEN`的 secret ，
值为你刚刚创建的 Personal Token 。（`PERSONAL_TOKEN`这个变量名与`deploy.yml`中变量名保持一致）。

4. 创建 GitHub Pages 仓库

创建 GitHub 仓库`<username>.github.io`，其中`<username>`为你的 GitHub 用户名。
这个仓库将用于存放 Hugo 的生成的 html 文件，
github 会自动将这些文件部署到`https://<username>.github.io`。
以我的 GitHub 为例，我的仓库为[m1dsolo.github.io](https://github.com/m1dsolo/m1dsolo.github.io)。

## 7. 结语

通过本文的步骤，我们成功地使用 Hugo 搭建了个人博客网站，并通过 GitHub Actions 实现了自动部署到 GitHub Pages 的功能。

现在，每次你完成博客的撰写，只需将仓库推送到 GitHub，
更新的内容就会自动部署到你的个人博客网站 `https://<username>.github.io` 上。
（别忘了将博客内的 `draft` 字段改为 `false`）
