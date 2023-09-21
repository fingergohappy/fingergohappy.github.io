---
title: hexo push后自动部署github pages
date: 2023-09-20 09:31:18
tags: 
  - hexo
categories:
  - hexo
cover: https://source.unsplash.com/lzh3hPtJz9c/1200x628
abstracts: hexo 提交后自动部署github page
feature: true
---

# hexo 提交后自动部署github page

hexo有几种部署方式,一种是本地编译后,直接push `public`路径下的静态文件,一种是通过`cli`方式,向仓库提交`source`下的的`markdown`文件,触发`action`.实现自动部署

本文主要说明后一种方式.

## 创建github workflow

在`hexo`的根目录的`.github/workflows/pages.yml`路径下创建文件,文件内容:

```yaml
name: Pages

on:
  push:
    branches:
      - main  # default branch

jobs:
  pages:
    runs-on: ubuntu-latest
    permissions:
      contents: write
    steps:
      - uses: actions/checkout@v3
        with:
          token: ${{ secrets.GITHUB_TOKEN }}
          # If your repository depends on submodule, please see: https://github.com/actions/checkout
          submodules: recursive
      - name: Use Node.js 16.x
        uses: actions/setup-node@v2
        with:
        # 看下自己本地的noe版本
          node-version: '16'
      - name: Cache NPM dependencies
        uses: actions/cache@v2
        with:
          path: node_modules
          key: ${{ runner.OS }}-npm-cache
          restore-keys: |
            ${{ runner.OS }}-npm-cache
      - name: Install Dependencies
        run: npm install
      - name: Build
        run: npm run build
      - name: Deploy
        uses: peaceiris/actions-gh-pages@v3
        with:
          github_token: ${{ secrets.GITHUB_TOKEN }}
          publish_dir: ./public
```


## 创建github pages

参考 [https://docs.github.com/en/pages/getting-started-with-github-pages/creating-a-github-pages-site](https://docs.github.com/en/pages/getting-started-with-github-pages/creating-a-github-pages-site)

创建完成后,微调一下:

在`settings->Pages->Build and deployment->Branch`选项中,把`branch`改成`gh-pages`

## 初始化本地仓库

```bash
git init .
git remote add origin ${你上面创建的仓库地址}
git remote add  .
git commit  -m "init project"
```

## gitgnore

由于使用了`github`的自动部署功能,所以无需上传静态文件,在`hexo`根目录添加`.gitignore`文件

```bash
public/
.deploy*/
node_modules
```

## 提交

最后提交到`main`分支

```bash
git push -u origin main
```
提交完成后,在`github`的`action`选项中可以看到正在构建,构建完成,就可以在`https://${your-username}.github.io`路径访问你的在线博客了


## 参考

[https://hexo.io/docs/github-pages](https://hexo.io/docs/github-pages)
