---
title: "gstack：Garry Tan 的 Claude Code 虚拟工程团队系统"
source: "https://github.com/garrytan/gstack"
author: "Garry Tan (Y Combinator CEO)"
date: 2026-03-20
tags: [Claude-Code, AI-编程, gstack, Skills, Y-Combinator, 虚拟团队, 工程工作流, Garry-Tan, 敏捷开发, Slash-Command]
category: "01-AI与技术"
---

# gstack：Garry Tan 的 Claude Code 虚拟工程团队系统

> 来源：[GitHub 仓库](https://github.com/garrytan/gstack)
> 作者：Garry Tan (Y Combinator 总裁兼 CEO) | 阅读日期：2026-03-20

## 📌 核心要点

- **gstack 是 YC CEO Garry Tan 开源的 Claude Code 技能包**，将一个 AI 助手变成一支包含 15 个专家角色 + 6 个工具的虚拟工程团队
- **核心理念：角色分离**——不同任务交给不同"专家"，避免一个 AI 同时扮演多角色导致注意力分散
- **覆盖完整软件开发生命周期**：思考 → 计划 → 构建 → 审查 → 测试 → 发布 → 反思，对标敏捷开发流程
- **惊人的生产力数据**：Garry Tan 声称用这套系统，兼职每天产出 10,000-20,000 行可用代码，60 天内写了 600,000+ 行生产代码
- **30.3k Star / 3.7k Fork**（截至 2026-03-20），MIT 协议完全免费

## 📝 详细笔记

### 一、项目背景与动机

Garry Tan 是 Y Combinator 现任 CEO，与 Coinbase、Instacart、Rippling 等数千家初创公司合作过。他将自己多年的创业和工程管理经验，浓缩成了一套 Claude Code Skills 系统。

核心洞察：**AI 编程在模拟工程团队结构时效果最佳**。与其让 AI 一次性完成所有工作，不如让它分角色、分步骤地协作——就像真实团队中 CEO 负责方向、工程经理负责架构、Staff Engineer 负责代码审查一样。

### 二、15 个专家角色（按开发流程排列）

#### 🧠 思考阶段
| 命令 | 角色 | 功能 |
|------|------|------|
| `/office-hours` | YC 导师 | 通过 6 个强制性问题重新定义产品，在写代码前挑战假设 |
| `/plan-ceo-review` | CEO/创始人 | 寻找"10 星级产品"，含扩张/缩减等 4 种模式 |

#### 📐 计划阶段
| 命令 | 角色 | 功能 |
|------|------|------|
| `/plan-eng-review` | 工程经理 | 锁定架构、数据流、图表、边缘情况，让隐藏假设浮出水面 |
| `/plan-design-review` | 资深设计师 | 对每个设计维度 0-10 评分，含 AI 垃圾检测功能 |
| `/design-consultation` | 设计合伙人 | 从零构建完整设计系统，了解行业现状，提出创意风险 |

#### 🔨 构建与审查阶段
| 命令 | 角色 | 功能 |
|------|------|------|
| `/review` | Staff Engineer | 找出通过 CI 但在生产中会爆炸的 Bug，自动修复明显问题 |
| `/investigate` | 调试专家 | 系统性根因调试，铁律：没有调查就没有修复 |
| `/design-review` | 会写代码的设计师 | 设计审计 + 直接修复问题 |

#### 🧪 测试阶段
| 命令 | 角色 | 功能 |
|------|------|------|
| `/qa` | QA Lead | 测试应用、发现 Bug、原子提交修复、自动生成回归测试 |
| `/qa-only` | QA 报告员 | 同 `/qa` 方法论但只报告不修改代码 |
| `/browse` | QA 工程师 | 用真实 Chromium 浏览器点击和截图测试 |
| `/setup-browser-cookies` | Session Manager | 导入真实浏览器 Cookies 测试需要认证的页面 |

#### 🚀 发布与回顾
| 命令 | 角色 | 功能 |
|------|------|------|
| `/ship` | 发布工程师 | 同步主分支、跑测试、审计覆盖率、推送、开 PR |
| `/document-release` | 技术写手 | 自动更新所有项目文档匹配最新发布内容 |
| `/retro` | 工程经理 | 每周回顾：每人细分、交付记录、测试健康趋势 |

### 三、6 个强力工具

| 命令 | 功能 | 说明 |
|------|------|------|
| `/codex` | 第二意见 | 调用 OpenAI Codex CLI 进行独立代码审查，含通过/失败门控 |
| `/careful` | 安全护栏 | 拦截危险命令（rm -rf, DROP TABLE 等），说"be careful"激活 |
| `/freeze` | 编辑锁 | 将编辑限制在指定目录内，防止调试时误改其他代码 |
| `/guard` | 完整安全 | 同时激活 careful + freeze，最大生产安全性 |
| `/unfreeze` | 解锁 | 解除 freeze 限制 |
| `/gstack-upgrade` | 自动升级 | 升级 gstack 到最新版本 |

### 四、典型工作流

```
/office-hours → 重新定义问题
      ↓
/plan-ceo-review → CEO 级别审查
      ↓
/plan-eng-review → 锁定工程架构
      ↓
  编码实现
      ↓
/review → Staff Engineer 代码审查
      ↓
/qa → 测试 + 修复 + 回归测试
      ↓
/ship → 发布代码
      ↓
/document-release → 更新文档
      ↓
/retro → 每周回顾
```

### 五、并行开发能力

配合 [Conductor](https://conductor.build)，gstack 支持运行 **10-15 个并行冲刺**。每个 Claude Code 会话都在隔离的工作空间中运行，可以同时进行头脑风暴、审查、实现、测试等操作。

### 六、安装与配置

**要求**：Claude Code + Git + Bun v1.0+

```bash
git clone https://github.com/garrytan/gstack.git ~/.claude/skills/gstack && cd ~/.claude/skills/gstack && ./setup
```

技术栈：TypeScript (71%) + Go Template (24.5%) + Shell (4.5%)

### 七、业界评价（优缺点）

**✅ 优点：**
- **结构化**：将隐式的工程最佳实践转化为显式的、可复用的系统
- **角色分离**：避免 AI 同时扮演多角色，提高专业度
- **可组合性**：技能间有清晰的输入输出，可串联为自动化工作流
- **团队共享**：便于团队统一开发规范和思维模式

**❌ 批评：**
- **本质仍是提示词**：有经验的 Claude Code 用户可能早已在用类似方法，并无突破性创新
- **非万能**：虽然有人声称能发现团队未察觉的漏洞，但本质上是复杂的提示工作流

### 八、6 个最佳实践技巧

1. **建立角色分离的工作流**——为不同角色创建独立技能目录
2. **设计可组合的技能链**——确保一个技能的输出格式匹配下一个的输入
3. **定义明确的输出格式**——强制 JSON 格式，含 status / details / next_steps
4. **创建领域专用技能**——针对特定技术栈定制（如 React 前端、API 安全等）
5. **建立技能版本控制**——用目录管理版本 + CHANGELOG.md 记录变更
6. **团队技能共享**——项目根目录 `.claude-code/skills/` + Git 同步

## 💡 个人思考

- **对我的直接价值**：作为正在转型 AI 应用开发的工程师，gstack 展示了"一个人 = 一个团队"的可能性。结合之前学习的 2026-03-20-top-10-claude-code-skills，可以有选择地将 gstack 中的部分角色（特别是 `/review`、`/qa`、`/ship`）融入自己的工作流
- **角色分离的理念值得借鉴**：即使不直接使用 gstack，"不同任务用不同人格"这个思路可以应用到自己的 Skill 设计中。这与 2026-03-20-5-agent-skill-design-patterns-for-adk 中提到的 Agent 设计模式是相通的
- **关于"每天 10K-20K 行代码"的说法**：需要理性看待，代码量≠价值量。但不可否认，结构化的 AI 工作流确实能大幅提升产出效率
- **独立开发者的福音**：对于没有团队的独立开发者来说，gstack 提供了一种"模拟完整团队"的方式。开发"吵架帮手"App 时可以考虑用类似流程
- **隐私做得不错**：默认关闭遥测、不发送代码/路径/提示词，MIT 协议，对独立开发者友好

## 🏷️ 关键词

`gstack` `Claude-Code` `Garry-Tan` `Y-Combinator` `虚拟工程团队` `角色分离` `AI-工作流` `Slash-Command` `敏捷开发` `独立开发`

## 🔗 相关笔记

- 2026-03-20-top-10-claude-code-skills — 同主题：Claude Code Skills 精选与实测
- 2026-03-20-5-agent-skill-design-patterns-for-adk — 相关理念：Agent Skill 设计模式
