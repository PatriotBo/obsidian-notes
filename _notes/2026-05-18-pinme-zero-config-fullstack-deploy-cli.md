---
title: "PinMe — 零配置全栈部署 CLI 工具"
source: "https://github.com/glitternetwork/pinme"
author: "Glitter Network"
date: 2026-05-18
tags: [部署工具, CLI, IPFS, 全栈, serverless, 去中心化, claude-skills, 独立开发]
category: "01-AI与技术"
---

# PinMe — 零配置全栈部署 CLI 工具

> 来源：[GitHub](https://github.com/glitternetwork/pinme)
> 组织：Glitter Network | 阅读日期：2026-05-18
> Stars: 3.4k | License: MIT | 最新版: v2.0.4

## 📌 核心要点

- **一条命令**完成全栈项目（前端 + Worker 后端 + D1 数据库）的创建和部署
- 深度集成 **IPFS** 去中心化存储，支持 `.eth.limo` 域名访问
- 原生支持 **Claude Code Skills**，可直接作为 AI Agent 的部署工具
- 提供全栈模板开箱即用，零配置即可上线
- 支持增量更新：仅更新 Worker / 数据库 / 前端中变更的部分

## 📝 详细笔记

### 项目定位

PinMe 定位为面向前端/全栈开发者的**去中心化部署平台 CLI**，标语是 *"Create and deploy your web in one command"*。它通过 IPFS 实现静态资源的去中心化托管，同时支持 Serverless Worker 后端和 D1 数据库。

### 核心功能

| 功能 | 命令 | 说明 |
|------|------|------|
| 创建项目 | `pinme create my-app` | 基于官方模板创建全栈项目 |
| 完整部署 | `pinme save` | 一键部署 Worker + SQL + 前端 |
| 更新 Worker | `pinme update-worker` | 仅更新后端代码 |
| 更新数据库 | `pinme update-db` | 仅执行 SQL 迁移 |
| 更新前端 | `pinme update-web` | 仅更新静态资源 |
| 静态站点上传 | `pinme upload dist` | 上传目录到 IPFS |
| 域名绑定 | `pinme upload ./dist --domain my-site` | 上传并绑定子域名 |
| 导入/导出 CAR | `pinme import` / `pinme export` | IPFS 内容归档 |
| AI Agent 集成 | `npx skills add glitternetwork/pinme` | 安装为 Claude Skill |

### 技术架构

- **主语言**: TypeScript (79.6%)
- **构建工具**: Rollup
- **运行环境**: Node.js >= 16.13.0
- **存储后端**: IPFS (静态资源) + D1 (数据库)
- **计算后端**: Serverless Worker
- **支持格式**: CAR 文件 (IPFS Content Archive)

### 使用限制

| 限制 | 说明 |
|------|------|
| 单文件上传 | 最大 100MB |
| 目录上传 | 最大 500MB |
| SQL 负载 | 每次 10MB |
| 域名绑定 | 需要钱包余额 |
| 认证 | upload/import 和项目命令均需登录 |

### AI Agent 安全注意事项

- ❌ 不要上传源码目录（`src/`）
- ❌ 不要上传 `node_modules`、`.git`、`.env`
- ❌ 不要在非 PinMe 项目根目录运行 `update-*` 命令

## 💡 个人思考

1. **对独立开发者极友好** — 一条命令就能把全栈项目上线，非常适合 MVP 快速验证。如果做「吵架帮手」之类的轻量应用，这种工具可以显著降低运维成本。
2. **去中心化方向有意思** — 通过 IPFS + `.eth.limo` 域名，内容不依赖单一云商，不会被随意下线，适合需要长期稳定的个人项目。
3. **AI Agent 原生支持** — 这是新趋势：工具不仅给人用，也给 AI 用。作为 Claude Code Skill 使用意味着可以在 AI 编码工作流中无缝完成部署。
4. **和 Vercel/Netlify 的区别** — PinMe 更偏去中心化 + 全栈（含数据库），不过生态成熟度和稳定性需要进一步观察。
5. **可作为 WorkBuddy skill 参考** — 它的 Claude Skill 集成方式值得借鉴，我们的 skill 体系也可以加入类似的部署能力。

## 🏷️ 关键词

`零配置部署` `IPFS` `全栈CLI` `Claude Skills` `Serverless` `去中心化托管` `独立开发工具`

## 🔗 相关笔记

- 2026-03-20-top-10-claude-code-skills
