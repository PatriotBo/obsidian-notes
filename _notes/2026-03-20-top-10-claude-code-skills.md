---
title: "实测了上百个 Claude Code Skills，最后只留下这10个"
source: "https://x.com/cnyzgkc/status/2034432870455140746"
author: "木马人 (@cnyzgkc)"
date: 2026-03-20
tags: [Claude-Code, AI-编程, Skills, Plugin, Anthropic, 开发工具, 效率提升, TDD, Code-Review, MCP]
category: "01-AI与技术"
---

# 实测了上百个 Claude Code Skills，最后只留下这10个

> 来源：[X 推文](https://x.com/cnyzgkc/status/2034432870455140746) | [微信公众号原文](http://mp.weixin.qq.com/s?__biz=MzYyNDE4Njc2Mw==&mid=2247487687&idx=1&sn=de61a0d8d5de5feffdbcf3bd76e30508)
> 作者：木马人 (@cnyzgkc) | 阅读日期：2026-03-20

## 📌 核心要点

- **Skill 不在多，在合适**：装太多会占满上下文，导致 Claude 变慢、思路飘、忘记上下文。作者装了 30+ 个后 Claude 直接罢工
- **按使用频率分三档**：天天用 3 个（Superpowers / Planning with Files / Ralph Loop）、项目常驻 3 个（Code Review / Code Simplifier / Webapp Testing）、按需使用 4 个
- **Skill vs Plugin**：Skill 是带 SKILL.md 的文件夹教 Claude 做事；Plugin 能额外打包命令和 MCP 服务器，功能更全。使用时无需区分
- **项目绑定策略**：跟项目相关的 Skill 放项目目录、提交 Git，团队共享且不污染其他项目上下文
- **这 10 个覆盖了 90% 的使用场景**，先熟练掌握手头的，不要盲目追新

## 📝 详细笔记

### 第一档：天天在用（离了它们不会写代码了）

#### 1. Superpowers ⭐⭐⭐⭐⭐

> 如果只能装一个，选这个。

- **功能**：20+ 个可组合子 Skill，覆盖软件开发全流程
- **核心模式**：
  - **Brainstorming**：开工前 Claude 先提问（并发处理？数据库选型？接口幂等性？），讨论后生成设计文档存到本地，减少返工
  - **TDD**：强制先写测试再写实现，测试不绿不算完
- **⚠️ 避坑**：20 多个子 Skill 别全开！只常开 brainstorming 和 TDD，其他按需加载
- 🔗 https://github.com/obra/superpowers

#### 2. Planning with Files

- **解决痛点**：Claude 自带 Plan Mode 在上下文压缩后会丢失规划
- **核心思路**：将计划持久化到 Markdown 文件。每完成一步打勾，即使上下文清空也能从文件恢复
- **类比**：思路类似 Manus，所有中间状态落盘，长任务更稳
- **安装**：
  ```bash
  claude plugin marketplace add OthmanAdi/planning-with-files
  claude plugin install planning-with-files
  ```
- 🔗 https://github.com/OthmanAdi/planning-with-files

#### 3. Ralph Loop

- **来源**：Anthropic 工程师 Daisy Hollman 制作，灵感来自《辛普森一家》的 Ralph Wiggum
- **功能**：充当"监工"，通过 Stop Hook 拦截 Claude 退出，检查完成标准，没做完就打回去继续干
- **💡 关键技巧**：完成标准必须写得非常具体！例如："JWT 登录注册可用 + 测试覆盖率 80%+ + README 包含 API 文档"
- **使用示例**：
  ```bash
  /ralph-loop:ralph-loop "实现用户认证模块。完成标准：JWT 登录注册、测试通过、README 更新。完成后输出 COMPLETE" --max-iterations 20 --completion-promise "COMPLETE"
  ```
- **安装**：`claude plugin install ralph-loop`
- 🔗 https://awesomeclaude.ai/ralph-wiggum

---

### 第二档：项目里经常开（写代码时常驻）

#### 4. Code Review（官方插件）

- **功能**：多 Agent 并行审查同一个 PR（逻辑/安全/风格各一个）
- **亮点**：每条反馈带置信度评分，只保留高分建议。实测 200 行代码从 17 条建议过滤到 5 条高质量建议
- **⚠️ 注意**：大 PR 消耗 Token 很多（多 Agent 并行），建议先拆分
- **安装**：`claude plugin install code-review`
- 🔗 https://github.com/anthropics/claude-plugins-official/tree/main/plugins/code-review

#### 5. Code Simplifier（官方插件）

- **功能**：不改功能，只精简代码。检查重复逻辑、多余中间变量、可合并条件分支
- **实战**：作者用于数据清洗脚本，合并三段重复异常处理，文件减少 40 行
- **安装**：`claude plugin install code-simplifier`
- 🔗 https://github.com/anthropics/claude-plugins-official/tree/main/plugins/code-simplifier

#### 6. Webapp Testing

- **功能**：自动写 Playwright 测试脚本 → 启动浏览器 → 跑测试 → 截屏 → 发现 bug 自动修
- **工作流**：写前端 → 跑 Webapp Testing → 看截图 → 让它修。全程不用手动开浏览器
- **安装**：
  ```bash
  claude plugin marketplace add anthropic/skills
  claude plugin install example-skills@anthropic-agent-skills
  ```
- 🔗 https://github.com/anthropics/skills/tree/main/skills/webapp-testing

---

### 第三档：按需使用（不常驻）

#### 7. UI UX Pro Max

- **解决痛点**：Claude 生成的前端千篇一律（紫色渐变 + 圆角卡片 = AI 味十足）
- **功能**：内置 67 种设计风格 + 161 套行业配色，支持 React/Vue/SwiftUI/Flutter 等
- **安装**：
  ```bash
  claude plugin marketplace add nextlevelbuilder/ui-ux-pro-max-skill
  claude plugin install ui-ux-pro-max@ui-ux-pro-max-skill
  ```
- 🔗 https://github.com/nextlevelbuilder/ui-ux-pro-max-skill

#### 8. MCP Builder

- **功能**：辅助构建 MCP Server，四步走流程（搞清 API → 设计接口 → 写代码 → 跑测试）
- **亮点**：主动提醒边界场景（限流、token 刷新），比直接让 Claude 从头写更可靠
- **安装**：
  ```bash
  claude plugin marketplace add anthropic/skills
  claude plugin install example-skills@anthropic-agent-skills
  ```
- 🔗 https://github.com/anthropics/skills/tree/main/skills/mcp-builder

#### 9. PPTX

- **功能**：直接生成 .pptx 文件，支持母版、图表、动画
- **定位**：解决"从零开始做 PPT"的痛苦，适合生成初稿，直接汇报还需手动润色
- **安装**：
  ```bash
  claude plugin marketplace add anthropic/skills
  claude plugin install document-skills@anthropic-agent-skills
  ```
- 🔗 https://github.com/anthropics/skills/tree/main/skills/pptx

#### 10. Skill Creator（官方 Meta 工具）

- **功能**：用来造 Skill 的 Skill。自带 eval 测试框架，支持 A/B 对比（有 Skill vs 无 Skill）
- **定位**：当前 9 个不够用时，自己动手定制
- **安装**：`claude plugin install skill-creator`
- 🔗 https://github.com/anthropics/claude-plugins-official/tree/main/plugins/skill-creator

---

### 避坑指南速查

| 坑 | 建议 |
|---|---|
| Skill 装太多 | 常驻只留 3 个，其他按需开启 |
| 上下文被吃满 | Skill 描述符占空间，10 个可能吃掉几千 token |
| 大 PR 用 Code Review | 先拆分再审，否则多 Agent 消耗大量 Token |
| Ralph Loop 太早停 | 完成标准写具体，不给 Claude 找借口的空间 |
| Superpowers 全开 | 只开 brainstorming + TDD，其余按需 |
| Skill 散落各处 | 项目绑定的放项目目录，提交 Git，团队共享 |

## 💡 个人思考

1. **与我的 WorkBuddy Skills 体系对照**：文章里的很多理念和我现在使用的 WorkBuddy skill 体系（note-assistant、finance-data-retrieval 等）异曲同工。Skill 的核心是「教 AI 标准化地完成特定任务」，不管是 Claude Code 还是 WorkBuddy，精选胜过贪多

2. **Superpowers 的 Brainstorming + TDD 模式值得借鉴**：对我正在做的"吵架帮手"应用开发很有启发——可以设计一个类似的 Skill，让 AI 在生成训练场景前先做需求分析（用户画像、场景复杂度、情绪级别），而不是直接开写

3. **Planning with Files 的持久化思路**：这和上次学到的 ADK 5 大设计模式中的 Pipeline 模式理念一致——中间状态必须落盘，长流程才稳。这个原则适用于所有 Agent 开发

4. **Ralph Loop 的"监工"机制**：可以直接应用到独立开发流程中——给 AI 设定明确的完成标准（功能可用 + 测试通过 + 文档更新），防止半成品交付

5. **MCP Builder 可以帮助我快速构建自定义工具**：作为想转型 AI 应用开发的后端工程师，掌握 MCP Server 开发能力是核心技能之一

## 🏷️ 关键词

`Claude Code` `Skills` `Plugin` `Superpowers` `TDD` `Planning with Files` `Ralph Loop` `Code Review` `MCP Builder` `AI编程` `效率工具`

## 🔗 相关笔记

- 2026-03-20-5-agent-skill-design-patterns-for-adk — ADK 5大 Skill 设计模式，与本文的 Skill 设计理念互相印证
