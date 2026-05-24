# AGENTS.md — Hexo Blog Project (myblogs)

> 本文件供 AI coding agent 阅读。它总结了这个基于 Hexo 与 NexT 主题的静态博客项目的架构、构建流程与开发惯例。

---

## 项目概述

本项目是一个使用 [Hexo](https://hexo.io/)（v8.1.2）搭建的静态博客站点，主题为 [NexT](https://theme-next.js.org/)（v8.27.0，Muse 配色方案，已启用 Dark Mode）。博客内容以 Markdown 撰写，主要通过 GitHub Actions 自动构建并部署到 GitHub Pages。

- **项目名称**：`hexo-site`（`myblogs` 目录）
- **作者**：个人博客，博主为嵌入式软件开发从业人员
- **博客名称**：NOA Tailor（取自博主对持续学习的期望）
- **部署目标**：`rRaphae1/rRaphae1.github.io`（GitHub Pages）

---

## 技术栈

| 层级 | 技术 |
|------|------|
| 静态站点生成器 | Hexo 8.x（Node.js） |
| 主题 | NexT 8.27.0（`themes/next/`） |
| 模板引擎 | EJS（`.njk` 布局文件） |
| 样式 | Stylus |
| 标记语言 | Markdown（`hexo-renderer-marked`） |
| 代码高亮 | highlight.js |
| 部署 | GitHub Actions + `peaceiris/actions-gh-pages` |
| 包管理 | npm |

---

## 项目目录结构

```
myblogs/
├── _config.yml              # Hexo 站点主配置
├── _config.landscape.yml    # Landscape 主题配置（本项目未使用，为空）
├── package.json             # Node.js 依赖与 npm 脚本
├── package-lock.json
├── db.json                  # Hexo 数据库缓存
├── .gitignore
│
├── scaffolds/               # 新建文章/页面/草稿的模板
│   ├── draft.md
│   ├── page.md
│   └── post.md
│
├── source/                  # 源内容目录
│   └── _posts/              # Markdown 博文（唯一的内容来源）
│       ├── blog-introduce.md
│       └── hello-world.md
│
├── themes/                  # 主题目录
│   └── next/                # NexT 主题（含 _config.yml、layout、source、scripts 等）
│
├── public/                  # Hexo 生成的静态站点（不应提交到 Git）
├── .deploy_git/             # hexo-deployer-git 的部署缓存（不应提交）
├── tags/                    # 生成的标签页目录
│
└── .github/
    ├── workflows/
    │   └── deploy.yml       # GitHub Actions 自动部署工作流
    └── dependabot.yml       # Dependabot 配置（npm，每日检查）
```

> **注意**：`public/` 和 `.deploy_git/` 已被 `.gitignore` 排除。

---

## 构建与常用命令

所有命令均通过 npm scripts 或 Hexo CLI 执行：

| 命令 | 作用 |
|------|------|
| `npm install` | 安装项目依赖（含 Hexo 与 NexT 主题所需包） |
| `npm run build` / `hexo generate` | 生成静态站点到 `public/` |
| `npm run clean` / `hexo clean` | 清除 `public/`、`db.json` 与缓存 |
| `npm run server` / `hexo server` | 启动本地预览服务器（默认 `http://localhost:4000`） |
| `npm run deploy` / `hexo deploy` | 使用 `hexo-deployer-git` 部署到配置的 Git 仓库 |

本地开发流程：

```bash
npm install
npm run server
```

---

## 部署流程

### GitHub Actions 自动部署（主要方式）

文件：`.github/workflows/deploy.yml`

- **触发条件**：`main` 分支的 `push`
- **运行环境**：`ubuntu-latest`，Node.js 24
- **步骤**：
  1. `actions/checkout@v4` 拉取代码
  2. `actions/setup-node@v4` 设置 Node.js
  3. `npm ci` 安装依赖
  4. `npm run clean && npm run build` 生成站点
  5. `peaceiris/actions-gh-pages@v3` 将 `public/` 推送到 `page` 分支
- **认证**：使用 `secrets.RRAPHAE1_BLOG_PAGE_PERSONAL_TOKEN`（Personal Access Token）

### hexo-deployer-git（备用方式）

在 `_config.yml` 中配置了 `deploy` 字段：

- type: `git`
- repo: `git@github.com:rRaphae1/rRaphae1.github.io.git`
- branch: `main`

运行 `npm run deploy` 时，Hexo 会将 `public/` 推送到上述仓库的 `main` 分支。

---

## 代码与内容组织惯例

### 博文（Posts）

- 存放位置：`source/_posts/`
- 文件格式：Markdown（`.md`），顶部包含 YAML front matter
- 默认 front matter 模板（`scaffolds/post.md`）：

  ```yaml
  ---
  title: {{ title }}
  date: {{ date }}
  tags:
  ---
  ```

- 创建新文章：
  ```bash
  hexo new "文章标题"
  ```

### 页面（Pages）与草稿（Drafts）

- `scaffolds/page.md` — 自定义页面模板
- `scaffolds/draft.md` — 草稿模板

### 分类与标签

- 分类（categories）与标签（tags）在 front matter 中声明
- Hexo 会自动生成分类页（`categories/`）与标签页（`tags/`）
- 博主已创建 `introduce` 标签与分类

### 主题配置

- **不要直接修改** `themes/next/_config.yml` 中的内容（会导致升级冲突）。
- NexT 官方推荐通过 **Alternate Theme Config**（在站点根目录创建 `_config.next.yml` 或通过 `source/_data/` 自定义文件）来覆盖主题设置。
- 当前主题使用的关键配置：
  - `scheme: Muse`
  - `darkmode: true`
  - `creative_commons: by-nc-sa`
  - `vendors.internal: local`
  - `vendors.plugins: cdnjs`

---

## 依赖管理

- `package.json` 中定义了 Hexo 核心、生成器、渲染器及 `hexo-deployer-git`。
- `themes/next/package.json` 是主题自身的开发依赖（eslint、mocha、stylelint 等），通常不需要在站点根目录执行其脚本。
- Dependabot（`.github/dependabot.yml`）每日检查 npm 依赖更新，最多保留 20 个 open PR。

---

## 安全与敏感信息注意事项

- **个人访问令牌（PAT）**：`.github/workflows/deploy.yml` 使用了名为 `RRAPHAE1_BLOG_PAGE_PERSONAL_TOKEN` 的 secret。该令牌具有推送 GitHub Pages 仓库的权限，切勿在代码中硬编码或泄露。
- **`.gitignore`** 已正确排除了 `node_modules/`、`public/`、`.deploy*/`、`.env` 等敏感或生成目录。
- 博客内容中没有密码、API key 等硬编码凭证。

---

## 国际化与语言

- Hexo 主配置 `_config.yml` 中 `language: en`。
- NexT 主题支持多语言切换，但当前未开启 `language_switcher`。
- 博客实际内容以 **中文** 为主（如 `blog-introduce.md`），因此 front matter 中的 `title`、`tags`、`categories` 等字段也常使用中文。

---

## 对 Agent 的操作建议

1. **新增博文**：在 `source/_posts/` 下新建 `.md` 文件，确保包含正确的 YAML front matter（`title`、`date`、`tags`、`categories`）。
2. **修改站点配置**：编辑根目录 `_config.yml`（站点级），而非主题目录内的文件。
3. **修改主题外观**：优先在根目录创建 `_config.next.yml` 或 `source/_data/` 下的自定义文件来覆盖 NexT 配置，避免直接改动 `themes/next/_config.yml`。
4. **部署验证**：本地使用 `npm run server` 预览无误后再推送；GitHub Actions 会在 `push` 到 `main` 后自动完成构建与部署。
5. **不要提交生成目录**：`public/`、`.deploy_git/`、`node_modules/` 已被 `.gitignore` 忽略，无需手动处理。
