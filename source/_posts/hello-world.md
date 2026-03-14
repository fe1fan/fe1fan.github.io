---
title: 用 Hexo 搭起第一篇博客
date: 2026-03-14 10:20:00
tags:
  - Hexo
  - GitHub Pages
  - Shiro
categories:
  - 开发
---

这个站点已经基于 Hexo 初始化，并切换到了 `hexo-theme-shiro` 主题。

<!-- more -->

## 当前包含的内容

- 使用 `shiro` 主题完成基础配置
- 已创建标签、分类、关于页面
- 已添加 GitHub Pages 自动部署工作流

## 本地开发

在项目根目录执行：

```bash
npm install
npm run server
```

默认访问地址为 `http://localhost:4000/`。

## 发布流程

代码推送到 `main` 分支后，GitHub Actions 会自动执行：

```bash
npm ci
npm run build
```

随后把生成的 `public/` 目录部署到 GitHub Pages。

## 下一步建议

你可以继续做这几件事：

1. 修改 `_config.yml` 里的站点标题、描述和 URL。
2. 编辑 `_config.shiro.yml` 调整导航、暗色模式和印章文字。
3. 用 `hexo new "文章标题"` 创建新文章。
