---
title: "9Router - 免费 AI 路由器 & Token 节省工具"
source: "https://github.com/decolua/9router"
author: "decolua"
date: 2026-05-12
tags: [AI工具, 路由代理, Token优化, Claude-Code, 开源, 独立开发]
category: "01-AI与技术"
---

# 9Router - 免费 AI 路由器 & Token 节省工具

> 来源：[GitHub 仓库](https://github.com/decolua/9router)
> 作者：decolua | 阅读日期：2026-05-12
> ⭐ 8.9K Stars | 🍴 1.4K Forks | 📦 v0.4.31 | MIT License

## 📌 核心要点

- **本地 AI 路由代理**：一个端点统一连接 40+ AI 提供商和 100+ 模型，支持 Claude Code、Codex、Cursor、Cline 等 12 种 CLI 工具
- **三层智能降级**：订阅（Claude/Codex）→ 便宜（GLM $0.6/1M、MiniMax $0.2/1M）→ 免费（Kiro/OpenCode Free），编码永不中断
- **RTK Token 节省**：移植自 RTK（⭐40K）的压缩管线，自动检测并压缩 tool_result 内容（git diff、grep 等），节省 20-40% 输入 Token
- **Caveman 模式**：注入简洁风格提示，最高节省 65% 输出 Token
- **完全可零成本运行**：RTK + Kiro AI + OpenCode Free = $0 成本 + Token 节省

## 📝 详细笔记

### 项目定位与解决的问题

9Router 本质是一个**本地运行的 AI API 网关/路由器**，定位于解决 AI 编程工具用户的痛点：

| 痛点 | 9Router 方案 |
|------|-------------|
| 订阅配额浪费 | 智能追踪配额利用率 |
| 速率限制中断编码 | 自动降级到备选提供商 |
| 工具输出吃 Token | RTK 自动压缩（节省 20-40%） |
| API 费用高昂 | 多层降级：订阅→便宜→免费 |
| 手动切换提供商 | 智能自动路由 |

### 架构设计

```
┌─────────────┐
│  CLI 工具   │  (Claude Code, Codex, Cursor, Cline...)
└──────┬──────┘
       │ http://localhost:20128/v1
       ↓
┌─────────────────────────────────────────────┐
│           9Router (智能路由器)               │
│  1. RTK Token 压缩                          │
│  2. 格式翻译 (OpenAI ↔ Claude ↔ Gemini)    │
│  3. 配额追踪 + 自动降级                     │
│  4. OAuth Token 自动刷新                    │
└──────┬──────────────────────────────────────┘
       │
       ├─→ [第1层: 订阅] Claude Code, Codex, GitHub Copilot
       ├─→ [第2层: 便宜] GLM ($0.6/1M), MiniMax ($0.2/1M)
       └─→ [第3层: 免费] Kiro, OpenCode Free, Vertex ($300)
```

**核心设计理念**：用户只需配置一个端点 `localhost:20128/v1`，9Router 处理所有路由、降级、格式转换和 Token 优化。

### 核心功能详解

#### 1. RTK Token 节省器

- 基于 [RTK](https://github.com/rtk-ai/rtk)（Rust 实现，⭐40K），移植到 JS
- **工作原理**：自动检测 `tool_result` 中的内容类型（git diff、grep、find、ls、tree 等），应用对应压缩过滤器
- 支持 10 种过滤器：`git-diff`、`git-status`、`grep`、`find`、`ls`、`tree`、`dedup-log`、`smart-truncate`、`read-numbered`、`search-list`
- **自动检测**：分析前 1KB 内容识别类型，无需手动配置
- **安全保证**：过滤器失败时静默保留原文
- 默认开启，效果实测：47K tokens → 28K tokens（节省 40%）

#### 2. Caveman 模式

- 灵感来自 [Caveman](https://github.com/JuliusBrussee/caveman)（⭐52K）
- 注入"穴居人式"系统提示，让 LLM 回复更简洁但保留技术实质
- 最高节省 65% 输出 Token
- 适合不需要详细解释的场景

#### 3. 格式翻译（关键能力）

支持多种 API 格式互转：
- OpenAI ↔ Claude ↔ Gemini ↔ Cursor ↔ Kiro ↔ Vertex ↔ Antigravity ↔ Ollama ↔ OpenAI Responses
- CLI 工具发送 OpenAI 格式 → 9Router 自动翻译 → 目标提供商接收原生格式

#### 4. 多账户与自动刷新

- 每个提供商支持多账户，自动轮询
- OAuth Token 过期前自动刷新，无需手动干预

### 支持的工具与提供商

**CLI 工具（12 种）**：Claude Code、OpenClaw、Codex、OpenCode、Cursor、Antigravity、Cline、Continue、Droid、Roo、Copilot、Kilo Code

**模型前缀规则**：

| 前缀 | 提供商 | 代表模型 | 成本 |
|------|--------|---------|------|
| `cc/` | Claude Code | claude-opus-4-7, sonnet-4-6 | $20-200/月 |
| `cx/` | Codex | gpt-5.5, gpt-5.4 | $20-200/月 |
| `gh/` | GitHub Copilot | gpt-5.4, claude-opus-4.7 | $10-19/月 |
| `glm/` | GLM | glm-5.1, glm-5 | $0.6/1M tokens |
| `minimax/` | MiniMax | M2.7, M2.5 | $0.2/1M tokens |
| `kr/` | Kiro | claude-sonnet-4.5, glm-5 | **免费无限** |
| `oc/` | OpenCode Free | 自动获取 | **免费无认证** |
| `vertex/` | Vertex AI | gemini-3.1-pro | $300 额度 |

### 典型使用场景

| 场景 | 推荐 Combo | 月成本 |
|------|-----------|--------|
| 已有 Claude Pro | cc/ → glm/ → kr/ | ~$25 |
| **零成本编码** | kr/ → kr/glm-5 → oc/ | **$0** |
| 24/7 不中断 | cc → cx → glm → minimax → kr | $30-220 |

### 技术栈

| 组件 | 技术 |
|------|------|
| 运行时 | Node.js 20+ |
| 框架 | **Next.js 16** |
| UI | React 19 + Tailwind CSS 4 |
| 数据库 | LowDB（JSON 文件存储） |
| 流式传输 | SSE |
| 认证 | OAuth 2.0 (PKCE) + JWT + API Keys |

### 快速上手

```bash
# 一行安装启动
npm install -g 9router
9router
# 仪表板：http://localhost:20128

# Claude Code 配置
# ~/.claude/config.json
{
  "anthropic_api_base": "http://localhost:20128/v1",
  "anthropic_api_key": "your-9router-api-key"
}
```

### 部署方式

- **本地**：`npm install -g 9router && 9router`
- **VPS**：clone + build + PM2 守护
- **Docker**：`docker build` + volume 挂载数据目录
- **Cloudflare Workers**：边缘部署

## 💡 个人思考

### 对独立开发的启示

1. **产品思路值得学习**：9Router 解决了 AI 编程工具用户的真实痛点（配额限制、费用高、切换麻烦），切入点非常精准。8.9K Star 说明需求确实存在
2. **技术选型参考**：Next.js + LowDB 的轻量组合适合工具类产品，不需要重量级数据库，JSON 文件存储对本地工具来说足够
3. **开源商业化路径**：免费开源 + 云同步服务，是典型的开源工具变现模式

### 实际应用价值

- 如果我同时用 Claude Code 和其他 AI 工具，这个路由器可以帮我**统一管理所有 API 调用**，在配额耗尽时自动切换
- RTK 的 Token 压缩思路很有意思——AI 编程场景中 tool output（git diff、文件列表等）确实占大量 Token，压缩这部分不影响理解质量
- **零成本方案**（Kiro + OpenCode Free）值得在个人项目中尝试

### 技术架构亮点

- **格式翻译层**是核心竞争力——连接上下游的"胶水层"，这也是 API 网关的经典价值
- **降级策略**设计清晰：订阅 → 便宜 → 免费，优先用已付费资源，耗尽后平滑过渡
- **RTK 移植**：将 Rust 压缩管线移植到 JS，保持了跨平台兼容性

### 与 Union 版本管理服务的类比

有意思的是，9Router 的路由降级逻辑和我们 Union 服务中的多版本降级有相似之处：
- 都是"优先用最优资源，不可用时降级"
- 都需要健康检查/配额检测来触发降级
- 都依赖配置驱动的灵活路由策略

## 🏷️ 关键词

`AI路由代理` `Token压缩` `RTK` `智能降级` `Claude Code` `开源工具` `API网关` `格式翻译` `零成本编码`

## 🔗 相关链接

- [RTK - Rust Token Kompressor](https://github.com/rtk-ai/rtk) ⭐40K
- [Caveman - 穴居人模式](https://github.com/JuliusBrussee/caveman) ⭐52K
- [OmniRoute - TypeScript 重写 Fork](https://github.com/diegosouzapw/OmniRoute)
