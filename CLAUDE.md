# Scientify 开发指南

本文档指导 Claude 进行 Scientify 插件的版本更新和发布工作。

## 项目概述

Scientify 是一个 OpenClaw 插件，提供 AI 驱动的科研工作流自动化功能。

**核心组件：**
- `src/tools/` - 工具实现（arxiv_search, github_search）
- `src/commands.ts` - 聊天命令处理
- `skills/` - 技能定义（随 npm 包发布）
- `index.ts` - 插件入口

## 版本更新 SOP

### 1. 代码修改

```bash
# 工作目录
cd /Users/springleaf/study/collaborator/clawdbot/extensions/scientify
```

**修改前检查：**
- [ ] 确认当前分支是 `main`
- [ ] 确认工作区干净：`git status`

### 2. 更新文档

**必须更新的文档：**
- [ ] `README.md` - 英文文档
- [ ] `README.zh.md` - 中文文档

**文档更新内容：**
- 新增 Tool：添加到 Tools 表格
- 新增 Skill：添加到 Skills 表格
- 新增 Command：添加到 Commands 表格
- 功能变更：更新相关章节
- 已知限制：更新 "Known Limitations" 章节

### 3. 构建验证

```bash
npm run build
```

确保无 TypeScript 错误。

### 4. 提交代码（重要：commit message 决定版本号）

```bash
git add -A
git commit -m "type: 描述变更

详细说明（可选）

Co-Authored-By: Claude Opus 4.5 <noreply@anthropic.com>"
```

**Commit 类型与版本号对应：**
| Commit 类型 | 版本变化 | 示例 |
|------------|---------|------|
| `fix:` | patch (1.1.0 → 1.1.1) | `fix: correct arxiv date parsing` |
| `feat:` | minor (1.1.0 → 1.2.0) | `feat: add semantic_scholar tool` |
| `feat!:` 或 `BREAKING CHANGE:` | major (1.1.0 → 2.0.0) | `feat!: change workspace structure` |
| `docs:` | patch | `docs: update README` |
| `refactor:` | patch | `refactor: simplify command handler` |
| `perf:` | patch | `perf: optimize arxiv batch download` |
| `chore:` | 不发布 | `chore: update dev dependencies` |

### 5. 推送到 GitHub（自动发布）

```bash
git push origin main
```

**CI 自动执行：**
1. 分析 commit messages
2. 计算新版本号
3. 更新 package.json
4. 生成 CHANGELOG.md
5. 发布到 npm
6. 创建 GitHub Release

### 6. 验证发布

```bash
npm view scientify versions
```

或查看 GitHub Actions 运行状态。

## 文件结构

```
scientify/
├── index.ts                 # 插件入口
├── package.json             # 包配置
├── tsconfig.json            # TypeScript 配置
├── CLAUDE.md                # 本文档
├── README.md                # 英文文档
├── README.zh.md             # 中文文档
├── src/
│   ├── commands.ts          # 聊天命令
│   ├── openclaw.d.ts        # 类型声明
│   └── tools/
│       ├── arxiv-tool.ts    # ArXiv 搜索工具
│       └── github-search-tool.ts  # GitHub 搜索工具
├── skills/
│   ├── arxiv/SKILL.md
│   ├── idea-generation/
│   │   ├── SKILL.md
│   │   └── references/idea-template.md
│   ├── literature-review/
│   │   ├── SKILL.md
│   │   └── references/note-template.md
│   ├── research-pipeline/
│   │   ├── SKILL.md
│   │   └── references/
│   │       ├── workspace-spec.md
│   │       └── prompts/{implement,plan,review,survey}.md
│   └── install-scientify/SKILL.md
├── .releaserc.json          # semantic-release 配置
└── .github/
    └── workflows/release.yml  # 自动发布 CI/CD
```

## 常见任务

### 添加新 Tool

1. 在 `src/tools/` 创建 `xxx-tool.ts`
2. 在 `index.ts` 中导入并注册：
   ```typescript
   import { createXxxTool } from "./src/tools/xxx-tool.js";
   api.registerTool(createXxxTool());
   ```
3. 更新版本号（minor bump）
4. 更新 README

### 添加新 Skill

1. 创建 `skills/xxx/SKILL.md`
2. 如需引用文件，创建 `skills/xxx/references/`
3. 确保 `package.json` 的 `files` 包含 `skills`
4. 更新版本号（minor bump）

### 添加新 Command

1. 在 `src/commands.ts` 添加 handler
2. 在 `index.ts` 注册：
   ```typescript
   api.registerCommand({
     name: "xxx",
     description: "...",
     handler: handleXxx,
   });
   ```

## 注意事项

### Sandbox 限制

当前 OpenClaw sandbox 不支持：
- GPU 访问（`--gpus`）
- 自定义共享内存（`--shm-size`）

research-pipeline skill 中的代码执行依赖用户环境配置。

### Skills vs Tools

| Skills | Tools |
|--------|-------|
| Markdown 提示词 | TypeScript 代码 |
| 指导 LLM 行为 | 执行具体操作 |
| 随包发布到 npm | 随包发布到 npm |
| 无法强制执行环境 | 可访问系统 API |

### 发布检查清单

- [ ] 构建无错误 (`npm run build`)
- [ ] README 已更新（如有新功能）
- [ ] Commit message 使用正确的类型前缀
- [ ] 推送后检查 GitHub Actions 状态
- [ ] 验证 npm 版本 (`npm view scientify versions`)

## 相关资源

- GitHub: https://github.com/tsingyuai/scientific
- npm: https://www.npmjs.com/package/scientify
- OpenClaw 插件文档: 参考 clawdbot/docs/
