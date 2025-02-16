# m1dsolo's blog

![deploy](https://github.com/m1dsolo/blog/actions/workflows/deploy.yaml/badge.svg?branch=main)

## Introduction

This repository contains the source code for m1dsolo's blog,
built with Hugo and deployed to [Github Pages](https://github.com/m1dsolo/m1dsolo.github.io) and [cloud server](https://m1dsolo.xyz)
using Github Actions.

Welcome to visit my blog: [https://m1dsolo.xyz](https://m1dsolo.xyz) ~ 🎉

If you want to learn more about how to build a blog like this,
please refer to the [blog post](https://m1dsolo.xyz/posts/hugo-blog) .

## Features

- Fast and static site generation using [Hugo](https://gohugo.io/)
- CI/CD using Github Actions
- Automatic deployment to [Github Pages](https://github.com/m1dsolo/m1dsolo.github.io) and [cloud server](https://m1dsolo.xyz)
- Comment system powered by [Giscus](https://giscus.app/)
- Page view statistics powered by [busuanzi](https://github.com/soxft/busuanzi)
- Theme: [PaperMod](https://github.com/adityatelange/hugo-PaperMod)
- [Google Analytics](https://analytics.google.com/)

## Usage

If you want to build a blog like this on your local machine, you can follow the steps below:

```bash
# Clone
git clone https://github.com/m1dsolo/blog.git
cd blog

# Download theme
git submodule update --init --recursive

# Start server
hugo server -D
```

Then you can visit `http://localhost:1313` to view the blog.

## Contributing

If you find any bugs, typos or have any suggestions,
please feel free to open an issue or create a pull request.

## License

[MIT](LICENSE) © m1dsolo
