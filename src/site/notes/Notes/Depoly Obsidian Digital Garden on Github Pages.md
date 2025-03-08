---
{"dg-publish":true,"dg-path":"Docs/Depoly-Obsidian-Digital-Garden-on-Github-Pages.md","permalink":"/Docs/Depoly-Obsidian-Digital-Garden-on-Github-Pages/","title":"Depoly Obsidian Digital Garden on Github Pages","tags":["clippings","github-pages","github","11ty","ODG","nodejs"]}
---


> [!NOTE]
> #ODG : Obsidian Digital Garden, in short.

# Depoly Obsidian Digital Garden on Github Pages

在ChatGPT协助下，成功在Github Pages上直接部署了Obsidian Digital Garden插件的网站。

该页面提供了一个基于 GitHub Actions 的部署策略示例，用于将 Obsidian 文档自动构建并发布到 GitHub Pages 的 gh-pages 分支上，无需依赖 Vercel 或 Netlify。

- 自动触发构建：当 `src/site/**` 目录下有新增或修改文件时。
- 部署目标：gh-pages 分支
- 使用 Node.js 16 运行构建命令
- 部署工具：`peaceiris/actions-gh-pages`、`node.js`
- 尽管ODG是基于 #11ty 构建的，但是不要从11ty安装，要用 node.js包安装。这样才能保证部署了全部的插件。

---

下面给出一个基于 GitHub Actions 的最新部署策略示例，满足如下要求：

1. 生成器代码保留在 main 分支；
2. 当 src 目录下有新增或修改文件时，自动触发构建并将生成的静态文件发布到 gh-pages 分支；
3. 完全基于 GitHub Pages 部署，不依赖 Vercel 或 Netlify。


部署摘要

- 一般情况下，不必自己创建gh-pages分支，代码中有Action的workflow脚本，部署之后可成功执行。
- 如果在本地的Obsidian里移动了笔记文件的位置，在发布后，老的md文件很可能仍然会存在github上/src/site/notes里原有的位置。当定义了“永久链接”会引发11ty的重名文件冲突，导致Action的deploy动作失败。通常情况下一定之后要去github上找到老的删掉。
- 


## 示例 GitHub Actions Workflow

在仓库 main 分支下创建目录 `.github/workflows`，并新建文件（例如 `deploy.yml`），文件内容如下：

```yaml
name: Deploy ODG Site to GitHub Pages

permissions:
  contents: write

on:
  push:
    branches:
      - main
    paths:
      - 'src/site/**'

jobs:
  deploy:
    runs-on: ubuntu-22.04

    steps:
      # 1. 检出代码
      - name: Checkout code
        uses: actions/checkout@v3
        with:
          fetch-depth: 0

      # 2. 设置 Node.js 环境（可根据需要调整版本）
      - name: Setup Node.js
        uses: actions/setup-node@v3
        with:
          node-version: '22'

      # 3. 安装依赖
      - name: Install dependencies
        run: npm install

      # 4. 构建静态站点（不要用11ty部署，要用npm部署整个包，因为ODG有自定义的打包插件）
      - name: Build ODG Site with npm
        run: npm run build --if-present

      # 5. 部署生成的 dist 目录到 gh-pages 分支
      - name: Deploy to GitHub Pages
        uses: peaceiris/actions-gh-pages@v3
        with:
          github_token: ${{ secrets.GITHUB_TOKEN }}
          publish_dir: ./dist
          publish_branch: gh-pages

```

### 说明

- **触发条件**  
上述工作流配置在 `main` 分支中监控 `src` 目录的所有变化（`src/**`）。每当有文件添加或修改时，工作流将自动触发。

- **构建步骤**

1. 使用 `actions/checkout` 检出最新代码。
2. 设置 Node.js 环境（此处选择了 Node.js 16，可根据实际需求调整）。
3. 通过 `npm install` 安装依赖，并运行 `npm run build` 生成静态文件，假设生成目录为 `dist`。如果你的项目生成目录不是 `dist`，请相应调整 `publish_dir` 的路径。

- **部署步骤**  
使用 [peaceiris/actions-gh-pages](https://github.com/peaceiris/actions-gh-pages) 部署生成的文件到 `gh-pages` 分支。部署时利用内置的 `${{ secrets.GITHUB_TOKEN }}` 自动进行认证，无需额外设置。

- **仓库配置**  
部署成功后，请确保在 GitHub 仓库设置中，GitHub Pages 的来源选择为 `gh-pages` 分支，以便正确显示你的 Obsidian Digital Garden。

通过以上配置，Obsidian 文档将从 main 分支的 src 目录自动构建，并利用最新的 GitHub Actions 部署策略发布到 GitHub Pages 的 gh-pages 分支上。这样就实现了一个纯 GitHub Pages 的自动部署流程，无需依赖第三方平台。


## 404问题解决

出现 404 页面可能有以下几个原因，你可以逐项排查和解决：

1. **GitHub Pages 源设置**  
    请确认仓库的 GitHub Pages 配置中，Source 选项已正确设置为 gh-pages 分支的根目录。如果曾经部署过其他目录（例如 /src/site），可能会导致配置残留。你可以进入仓库设置 → Pages，检查并确保“发布来源”设置为 gh-pages 分支的根目录。
    
2. **分支内容是否正确**  
    根据最新配置，Eleventy 构建生成的静态文件位于 dist 目录，并通过 Action 部署到 gh-pages 分支。请确认 gh-pages 分支根目录下是否存在正确的 index.html 文件以及其他必要的资源。你可以直接在 GitHub 仓库的 gh-pages 分支中查看构建输出文件。如果旧的部署曾将整个 main 分支内容推送过来，可能需要清理 gh-pages 分支后再重新部署。
    
    - 建议先备份当前内容，然后删除 gh-pages 分支，再通过 GitHub UI 或命令行创建一个空的 gh-pages 分支，再重新触发 Action 部署。
3. **浏览器缓存问题**  
    有时浏览器缓存可能导致显示 404 页面，建议尝试强制刷新页面（例如使用 Ctrl+F5 或清除缓存）来查看最新状态。
    
4. **自定义域名及 DNS 配置**  
    如果你设置了自定义域名，也请确认 DNS 记录正确，并且 GitHub Pages 的 CNAME 文件（如果使用）已正确配置。
    

综上建议：

- 检查 GitHub Pages 设置，确保使用 gh-pages 分支根目录。
- 确认 gh-pages 分支内容为 Eleventy 生成的 dist 文件内容（包含 index.html）。
- 清理可能的旧缓存，必要时重新创建 gh-pages 分支再触发部署。
- 如果使用自定义域名，请检查 DNS 配置。

按照这些步骤排查后，再观察前端页面是否能正确加载。