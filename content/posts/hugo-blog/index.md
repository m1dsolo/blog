---
title: '使用 Hugo 搭建博客并自动部署到 GitHub Pages'
date: '2026-03-10'
tags: ['Hugo']
draft: false
math: true
description: '本文介绍如何使用 Hugo + PaperMod 主题搭建静态博客，并通过 GitHub Actions 自动部署到 GitHub Pages。涵盖图床、搜索、评论、数学公式等常用功能配置，以及国内云服务器部署优化方案。'
keywords: [
  "Hugo",
  "PaperMod",
  "GitHub Pages",
  "静态博客",
  "自动部署",
]
---

## 前言

Hugo 是一个用 Go 编写的静态网站生成器，可以将 Markdown 文件转换为 HTML。它的优点是**速度快**（生成千页文章只需几秒）、**主题丰富**、**部署简单**（生成的静态文件可托管到 GitHub Pages、云服务器等）。

本文以 Arch Linux 为例，介绍如何用 Hugo + PaperMod 主题搭建博客，并通过 GitHub Actions 自动部署。其他系统用户可参考相应命令。

**最终效果：** 写 Markdown → 推送 GitHub → 自动发布 → 访问 `https://你的用户名.github.io`

---

## 1. 安装 Hugo

**Arch Linux：**
```bash
sudo pacman -S hugo
```

**验证安装：**
```bash
hugo version
```

**注意：** Hugo 分为 **extended** 和 **standard** 两个版本。extended 版本支持 SCSS/SASS 编译（PaperMod 需要）。Arch 的 `hugo` 包已包含 extended 版本，其他系统可能需要手动指定。

---

## 2. 创建 Hugo 站点

### 2.1 初始化项目

```bash
hugo new site blog --format yaml  # --format yaml 使用 YAML 配置
cd blog
git init
```

目录结构：
```
blog/
├── content/          # 文章内容
├── layouts/          # 自定义模板
├── static/           # 静态资源（图片、CSS 等）
├── themes/           # 主题目录
├── hugo.yaml         # 配置文件
└── ...
```

### 2.2 安装 PaperMod 主题

**推荐使用 Git Submodule（方便更新）：**

```bash
git submodule add --depth=1 https://github.com/adityatelange/hugo-PaperMod.git themes/PaperMod
```

在 `hugo.yaml` 添加主题：
```yaml
theme: ["PaperMod"]
```

**主题更新：**
```bash
git submodule update --remote --merge
git add themes/PaperMod
git commit -m "update PaperMod theme"
```

### 2.3 创建第一篇文章

```bash
hugo new content posts/hello-world/index.md
```

生成的文件位于 `content/posts/hello-world/index.md`，内容如下：

```markdown
---
title: "Hello World"
date: 2025-02-12T12:00:00+08:00
draft: true
tags: ["test"]
---

这是我的第一篇文章！
```

**重要字段说明：**

| 字段 | 说明 |
|------|------|
| `title` | 文章标题 |
| `date` | 发布时间（ISO 8601 格式） |
| `draft` | 是否为草稿（`true` 不发布） |
| `tags` | 标签（数组） |
| `categories` | 分类（可选） |
| `description` | 文章描述（SEO 用） |

---

## 3. 配置 PaperMod 主题

### 3.1 基础配置

编辑 `hugo.yaml`：

```yaml
baseURL: "https://m1dsolo.github.io/"
defaultContentLanguage: zh-cn
languageCode: zh-CN
title: 我的博客
theme: ["PaperMod"]

# 主题参数
params:
  env: production
  title: 我的博客
  description: "个人技术博客"
  author: 你的名字
  defaultTheme: auto  # dark, light, auto
  disableThemeToggle: false
  
  # 文章显示选项
  ShowReadingTime: true
  ShowShareButtons: true
  ShowPostNavLinks: true
  ShowBreadCrumbs: true
  ShowCodeCopyButtons: true
  ShowWordCount: true
  
  # 主页配置
  profileMode:
    enabled: true
    title: 我的博客
    subtitle: "欢迎来到我的博客～"
    imageUrl: "images/avatar.jpg"
    imageWidth: 120
    imageHeight: 120
    buttons:
      - name: 文章
        url: posts/
      - name: 标签
        url: tags/
  
  # 社交链接
  socialIcons:
    - name: github
      url: "https://github.com/你的用户名"
    - name: twitter
      url: "https://twitter.com/你的账号"

# 菜单配置
menu:
  main:
    - identifier: posts
      name: 文章
      url: /posts/
      weight: 10
    - identifier: tags
      name: 标签
      url: /tags/
      weight: 20
```

### 3.2 添加头像

将头像图片放到 `static/images/avatar.jpg`，然后在配置中引用。

### 3.3 本地预览

```bash
hugo server -D  # -D 显示草稿
```

访问 `http://localhost:1313` 查看效果。

---

## 4. 使用 GitHub 管理代码

### 4.1 创建仓库

在 GitHub 创建名为 `blog` 的仓库（私有或公开均可）。

### 4.2 推送代码

```bash
git add .
git commit -m "init blog"
git branch -M main
git remote add origin git@github.com:你的用户名/blog.git
git push -u origin main
```

### 4.3 添加 .gitignore

```bash
# Hugo
public/
resources/

# IDE
.vscode/
.idea/
*.swp
*.swo
```

---

## 5. 自动部署到 GitHub Pages

### 5.1 创建 GitHub Pages 仓库

创建新仓库，命名格式：`你的用户名.github.io`

例如：`m1dsolo.github.io`

**这个仓库用于存放 Hugo 生成的 HTML 文件**，GitHub 会自动将其部署到 `https://你的用户名.github.io`。

### 5.2 创建 Personal Access Token

1. 访问 [GitHub Tokens](https://github.com/settings/tokens)
2. 点击 **Generate new token (classic)**
3. 填写备注（如 `Hugo Deploy`）
4. 勾选权限：`repo`（全部）
5. 生成并复制 token（**只展示一次，妥善保存**）

### 5.3 配置 Secrets

1. 进入 `blog` 仓库 → Settings → Secrets and variables → Actions
2. 点击 **New repository secret**
3. 添加以下 secrets：

| Name | Value |
|------|-------|
| `PERSONAL_TOKEN` | 刚才复制的 token |

### 5.4 创建 Actions 配置

在 `blog` 仓库创建 `.github/workflows/deploy.yml`：

```yaml
name: Deploy Hugo Site

on:
  push:
    branches: [main]
  workflow_dispatch:  # 允许手动触发

jobs:
  build-deploy:
    runs-on: ubuntu-latest
    steps:
    - name: Checkout
      uses: actions/checkout@v4
      with:
        submodules: true
        fetch-depth: 0

    - name: Setup Hugo
      uses: peaceiris/actions-hugo@v3
      with:
        hugo-version: 'latest'
        extended: true

    - name: Build
      run: hugo --minify

    - name: Deploy to GitHub Pages
      uses: peaceiris/actions-gh-pages@v4
      with:
        personal_token: ${{ secrets.PERSONAL_TOKEN }}
        external_repository: 你的用户名/你的用户名.github.io
        publish_branch: main
        publish_dir: ./public
        full_commit_message: "deploy: ${{ github.event.head_commit.message }}"
```

**配置说明：**

| 字段 | 说明 |
|------|------|
| `submodules: true` | 拉取主题子模块 |
| `extended: true` | 使用 extended 版本 |
| `external_repository` | 部署目标仓库 |
| `publish_dir` | 发布的目录（Hugo 输出目录） |

### 5.5 测试部署

1. 修改 `hugo.yaml` 中的 `draft: false`
2. 推送代码：
```bash
git add .
git commit -m "add first post"
git push
```
3. 进入 Actions 页面，查看构建状态
4. 构建完成后，访问 `https://你的用户名.github.io`

### 5.6 故障排查

**❌ 构建失败：**

| 问题 | 解决方案 |
|------|---------|
| `theme not found` | 检查 `git submodule update` |
| `permission denied` | 检查 PERSONAL_TOKEN 权限 |
| `404 Not Found` | 检查 `external_repository` 仓库名 |

**❌ 部署成功但页面空白：**

1. 检查 `baseURL` 是否正确
2. 浏览器控制台查看资源加载错误
3. 清除 CDN 缓存

---

## 6. 配置图床

### 6.1 为什么需要图床？

- 图片直接放 Git 仓库会导致仓库体积过大
- 图床可提供 CDN 加速
- 方便图片管理和迁移

### 6.2 方案对比

| 方案 | 优点 | 缺点 |
|------|------|------|
| **GitHub Issues** | 免费、稳定 | 需手动上传、可能被墙 |
| **PicGo + 腾讯云 COS** | 快速、可控 | 需付费（约 5 元/月） |
| **自建云服务器** | 完全可控 | 需服务器、配置 Nginx |
| **免费图床** | 免费 | 不稳定、可能跑路 |

### 6.3 推荐方案：PicGo + 腾讯云 COS

**步骤：**

1. 下载 [PicGo](https://github.com/Molunerfinn/PicGo)
2. 注册 [腾讯云 COS](https://cloud.tencent.com/product/cos)
3. 创建存储桶（选择离你近的地域）
4. 配置 PicGo（密钥在腾讯云控制台获取）
5. 在 Typora/VSCode 中配置 PicGo 插件

**Hugo 中引用：**
```markdown
![描述](https://你的存储桶.cos.地域.myqcloud.com/images/xxx.png)
```

### 6.4 方案二：自建图床脚本

```bash
#!/bin/bash
# upload_image.sh
for img in "$@"; do
    scp "$img" yang@服务器 IP:/var/www/images/
    echo "https://你的域名/images/$(basename $img)" | xclip -selection clipboard
done
```

在 Neovim 中配置自动上传插件（如 `image.nvim`）。

---

## 7. 配置搜索功能

### 7.1 启用 PaperMod 搜索

在 `hugo.yaml` 添加：

```yaml
outputs:
  home:
    - HTML
    - JSON  # 搜索需要 JSON 输出

params:
  fuseOpts:
    isCaseSensitive: false
    shouldSort: true
    location: 0
    distance: 1000
    threshold: 0.4
    minMatchCharLength: 0
    keys: ["title", "permalink", "summary", "content"]
```

### 7.2 创建搜索页面

创建 `content/search.md`：

```markdown
---
title: "搜索"
layout: "search"
summary: "搜索文章"
placeholder: "输入关键词..."
---
```

### 7.3 访问搜索

部署后访问 `https://你的域名/search/` 即可使用搜索功能。

---

## 8. 配置评论系统

### 8.1 选择评论系统

| 系统 | 优点 | 缺点 |
|------|------|------|
| **Giscus** | 开源、GitHub 登录、免费 | 需启用 Discussion |
| **Disqus** | 功能丰富 | 需翻墙、有广告 |
| **Waline** | 自托管、功能全 | 需服务器 |

**推荐使用 Giscus**（无需额外服务器，GitHub 账号登录）。

### 8.2 配置 Giscus

**步骤：**

1. 访问 [giscus.app](https://giscus.app/zh-CN)
2. 输入仓库信息（`你的用户名/你的用户名.github.io`）
3. 启用仓库的 Discussion 功能（Settings → Features → Discussions）
4. 安装 [Giscus App](https://github.com/apps/giscus)
5. 生成配置并复制

### 8.3 添加到 Hugo

**方式一：使用 PaperMod 内置支持**

在 `hugo.yaml` 添加：

```yaml
params:
  giscus:
    repo: "你的用户名/你的用户名.github.io"
    repoId: "R_kgDxxxxxx"  # 从 giscus.app 获取
    category: "Announcements"
    categoryId: "DIC_kwDOxxxxxx"
    mapping: "pathname"
    reactionsEnabled: 1
    emitMetadata: 0
    inputPosition: "bottom"
    theme: "preferred_color_scheme"
    lang: "zh-CN"
    crossorigin: "anonymous"
```

**方式二：手动添加 partial**

创建 `layouts/partials/comments.html`：

```html
{{- if .Site.Params.giscus.repo -}}
<script src="https://giscus.app/client.js"
        data-repo="{{ .Site.Params.giscus.repo }}"
        data-repo-id="{{ .Site.Params.giscus.repoId }}"
        data-category="{{ .Site.Params.giscus.category }}"
        data-category-id="{{ .Site.Params.giscus.categoryId }}"
        data-mapping="pathname"
        data-reactions-enabled="1"
        data-emit-metadata="0"
        data-input-position="bottom"
        data-theme="preferred_color_scheme"
        data-lang="zh-CN"
        crossorigin="anonymous"
        async>
</script>
{{- end }}
```

在 `layouts/_default/single.html` 中调用：
```html
{{ partial "comments.html" . }}
```

---

## 9. 配置数学公式

### 9.1 使用 MathJax

**步骤 1：创建 partial**

创建 `layouts/partials/mathjax.html`：

```html
<script>
  MathJax = {
    tex: {
      displayMath: [['\\[', '\\]'], ['$$', '$$']],
      inlineMath: [['\\(', '\\)']]
    },
    loader: { load: ['ui/safe'] }
  };
</script>
<script id="MathJax-script" async
  src="https://cdn.jsdelivr.net/npm/mathjax@3/es5/tex-chtml.js">
</script>
```

**步骤 2：在 head 中引入**

创建 `layouts/partials/extend_head.html`：

```html
{{ if or .Params.math .Site.Params.math }}
  {{ partial "mathjax.html" . }}
{{ end }}
```

**步骤 3：配置 hugo.yaml**

```yaml
params:
  math: true  # 全局启用

markup:
  goldmark:
    extensions:
      passthrough:
        delimiters:
          block:
          - - \[
            - \]
          - - $$
            - $$
          inline:
          - - \(
            - \)
        enable: true
```

### 9.2 使用示例

**行内公式：** `\(E = mc^2\)` → \(E = mc^2\)

**块级公式：**
```markdown
$$
\sum_{i=1}^{n} x_i = x_1 + x_2 + \cdots + x_n
$$
```

**复杂公式：**
```markdown
\[
\begin{aligned}
KL(\hat{y} || y) &= \sum_{c=1}^{M}\hat{y}_c \log{\frac{\hat{y}_c}{y_c}} \\
JS(\hat{y} || y) &= \frac{1}{2}(KL(y||\frac{y+\hat{y}}{2}) + KL(\hat{y}||\frac{y+\hat{y}}{2}))
\end{aligned}
\]
```

### 9.3 常见问题

**问题：公式不渲染**

- 检查 `math: true` 是否配置
- 检查 CDN 是否可访问（国内可用 `https://cdn.bootcdn.net/ajax/libs/mathjax/3.2.0/es5/tex-chtml.js`）

---

## 10. 配置访问量统计

### 10.1 使用不蒜子（推荐）

**步骤：**

在 `layouts/partials/extend_footer.html` 添加：

```html
<script async src="//busuanzi.ibruce.info/busuanzi/2.3/busuanzi.pure.mini.js"></script>
<footer>
  <p>
    本站总访问量 <span id="busuanzi_value_site_pv"></span> 次 |
    本文阅读量 <span id="busuanzi_value_page_pv"></span> 次
  </p>
</footer>
```

**说明：**
- `site_pv`：全站总访问量
- `page_pv`：当前文章阅读量

### 10.2 使用 Google Analytics

**步骤：**

1. 注册 [Google Analytics](https://analytics.google.com/)
2. 获取 Tracking ID（格式：`G-XXXXXXXXXX`）
3. 在 `hugo.yaml` 添加：

```yaml
services:
  googleAnalytics:
    id: "G-XXXXXXXXXX"
```

4. 在 `layouts/partials/extend_head.html` 添加：
```html
{{ template "_internal/google_analytics.html" . }}
```

---

## 11. 部署到云服务器（国内访问优化）

### 11.1 为什么需要云服务器？

- GitHub Pages 在国内访问慢（DNS 污染）
- 云服务器可提供更快访问速度
- 可自定义域名（如 `m1dsolo.xyz`）

### 11.2 准备工作

| 项目 | 说明 |
|------|------|
| **云服务器** | 阿里云/腾讯云（学生机约 10 元/月） |
| **域名** | 阿里云/腾讯云（`.xyz` 首年约 1 元） |
| **备案** | 国内服务器必须备案（约 10-20 天） |

### 11.3 服务器配置

**步骤 1：安装 Nginx**

以 Debian 为例：
```bash
sudo apt update
sudo apt install nginx
```

其他系统：
- **Arch Linux：** `sudo pacman -S nginx`
- **CentOS：** `sudo yum install nginx`

**步骤 2：配置站点**

创建 `/etc/nginx/sites-available/blog`：

```nginx
server {
    listen 80;
    server_name m1dsolo.xyz www.m1dsolo.xyz;

    root /var/www/blog;
    index index.html;

    location / {
        try_files $uri $uri/ =404;
    }

    # 图片目录（图床用）
    location /images/ {
        alias /var/www/images/;
    }

    # HTTPS 重定向（配置 SSL 后启用）
    # return 301 https://$host$request_uri;
}
```

启用配置：
```bash
sudo ln -s /etc/nginx/sites-available/blog /etc/nginx/sites-enabled/
sudo nginx -t  # 检查配置
sudo systemctl restart nginx
```

**步骤 3：配置 SSH 密钥**

在 GitHub 仓库 Settings → Secrets 添加：

| Name | Value |
|------|-------|
| `SERVER_IP` | 云服务器 IP |
| `SSH_PRIVATE_KEY` | SSH 私钥（`cat ~/.ssh/id_rsa`） |

### 11.4 修改部署配置

更新 `.github/workflows/deploy.yml`：

```yaml
jobs:
  build-deploy:
    runs-on: ubuntu-latest
    steps:
    # ... 前面的步骤不变 ...

    - name: Build for GitHub Pages
      run: hugo --minify -b https://你的用户名.github.io/

    - name: Deploy to GitHub Pages
      uses: peaceiris/actions-gh-pages@v4
      with:
        personal_token: ${{ secrets.PERSONAL_TOKEN }}
        external_repository: 你的用户名/你的用户名.github.io
        publish_dir: ./public

    - name: Build for Cloud Server
      run: hugo --minify -b https://你的域名/

    - name: Deploy to Cloud Server
      uses: wlixcc/SFTP-Deploy-Action@v1.2.5
      with:
        username: root
        server: ${{ secrets.SERVER_IP }}
        ssh_private_key: ${{ secrets.SSH_PRIVATE_KEY }}
        local_path: ./public/*
        remote_path: /var/www/blog
```

**关键点：**
- 两次构建使用不同 `baseURL`
- GitHub Pages 和云服务器同时部署

### 11.5 配置 HTTPS（可选）

使用 Let's Encrypt 免费证书：

以 Debian 为例：
```bash
sudo apt install certbot python3-certbot-nginx
sudo certbot --nginx -d 你的域名 -d www.你的域名
```

其他系统：
- **Arch Linux：** `sudo pacman -S certbot python-certbot-nginx`

自动续期（添加到 crontab）：
```bash
0 3 1 * * certbot renew --quiet
```

### 11.6 备案信息展示

在 `layouts/partials/extend_footer.html` 添加：

```html
<div style="text-align: center; font-size: 12px; margin-top: 20px;">
  <a href="https://beian.miit.gov.cn/" target="_blank" rel="nofollow">
    京 ICP 备 XXXXXXXX 号
  </a>
</div>
```

---

## 12. 性能优化

### 12.1 图片优化

```bash
# 压缩图片
convert input.jpg -quality 80 output.jpg

# 转换格式（WebP 更小）
convert input.jpg output.webp
```

### 12.2 启用缓存

在 `hugo.yaml` 添加：

```yaml
caches:
  getresource:
    dir: :cacheDir/:project
    maxAge: -1
```

### 12.3 使用 CDN

在 `hugo.yaml` 配置外部资源 CDN：

```yaml
params:
  assets:
    favicon: "favicon.ico"
    disableHLJS: false
    external:
      css:
        - https://cdn.example.com/style.css
      js:
        - https://cdn.example.com/script.js
```

---

## 13. 常见问题

### Q1: 本地预览正常，部署后 404

**检查清单：**
- [ ] `draft: false`
- [ ] `baseURL` 正确
- [ ] Actions 构建日志无错误
- [ ] 清除浏览器缓存

### Q2: 图片不显示

**原因：** 路径错误

**正确写法：**
```markdown
<!-- 绝对路径（推荐） -->
![描述](/images/xxx.png)

<!-- 相对路径（不推荐） -->
![描述](../images/xxx.png)
```

### Q3: 主题更新后样式错乱

**解决：**
```bash
git submodule update --remote --merge
hugo --gc --minify  # 清理缓存
git add .
git commit -m "update theme"
git push
```

### Q4: 搜索功能不工作

**检查：**
- [ ] `outputs.home` 包含 `JSON`
- [ ] `public/index.json` 文件存在
- [ ] 浏览器控制台无 JS 错误

### Q5: 国内访问 GitHub Pages 慢

**解决方案：**
1. 使用 Cloudflare CDN（免费）
2. 部署到国内云服务器
3. 使用 Gitee Pages（需实名）

---

## 14. 总结

本文介绍了从零搭建 Hugo 博客到部署上线的完整流程，包括：

- ✅ Hugo 安装与配置
- ✅ PaperMod 主题使用
- ✅ GitHub Actions 自动部署
- ✅ 图床、搜索、评论、数学公式
- ✅ 云服务器部署优化

**核心建议：**

1. **不要过度折腾**：博客是用来写的，不是用来折腾的
2. **定期备份**：Git 仓库 + 本地备份
3. **内容为王**：好的内容比好的主题更重要

**参考资源：**

- [Hugo 官方文档](https://gohugo.io/documentation/)
- [PaperMod 主题文档](https://github.com/adityatelange/hugo-PaperMod/wiki)
- [我的博客源码](https://github.com/m1dsolo/blog)

---

**最后更新：** 2026-03-10  
**本文字数：** 约 5000 字  
**阅读时间：** 约 15 分钟
