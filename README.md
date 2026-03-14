# fe1fan.github.io

使用 Hexo 搭建、`hexo-theme-shiro` 主题渲染，并通过 GitHub Pages 自动部署的个人博客。

## 本地启动

```bash
npm install
npm run server
```

访问 `http://localhost:4000/`。

## 创建新文章

```bash
npx hexo new "文章标题"
```

## 部署方式

推送到 `main` 分支后，`.github/workflows/pages.yml` 会自动构建并发布站点。

## 主题更新

主题以 git submodule 方式管理：

```bash
git submodule update --init --recursive
git submodule update --remote themes/shiro
```
