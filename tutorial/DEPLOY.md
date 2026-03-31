# Claude Code 源码教程 — 部署指南

## 本地预览

```bash
# 安装依赖
pip install mkdocs mkdocs-material

# 启动本地预览服务器
mkdocs serve

# 访问 http://127.0.0.1:8000
```

## 部署到 GitHub Pages

### 方式一：使用 GitHub Actions（推荐）

1. 将项目推送到 GitHub 仓库
2. 在仓库设置中启用 GitHub Pages（Settings → Pages → Source: GitHub Actions）
3. 推送代码到 `main` 分支时会自动部署（见 `.github/workflows/deploy-tutorial.yml`）

### 方式二：使用 mkdocs gh-deploy

```bash
# 初始化 Git 仓库（如果还没有）
git init
git remote add origin https://github.com/YOUR_USERNAME/claude-code-tutorial.git

# 一键部署到 gh-pages 分支
mkdocs gh-deploy --force
```

## 项目结构

```
claude-code-main/
├── mkdocs.yml          # MkDocs 配置
├── tutorial/           # 教程源文件（45 份 Markdown）
│   ├── index.md        # 首页
│   ├── 00_*.md         # 入门文档
│   ├── main.md         # 核心入口文件文档
│   ├── *.md            # 其他核心文件文档
│   └── */index.md      # 模块文档
├── site/               # 构建输出（不提交到 Git）
└── .github/workflows/  # GitHub Actions 部署
```

## 自定义

- 修改 `mkdocs.yml` 中的 `palette` 调整主题颜色
- 修改 `nav` 调整导航结构
- 在 `tutorial/` 下添加新的 `.md` 文件并在 `nav` 中注册
