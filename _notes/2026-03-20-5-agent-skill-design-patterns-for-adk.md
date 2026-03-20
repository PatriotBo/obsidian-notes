---
title: "5 Agent Skill Design Patterns Every ADK Developer Should Know"
source: "https://x.com/googlecloudtech/status/2033953579824758855"
author: "Shubham Saboo & Lavi Nigam (Google Cloud Tech)"
date: 2026-03-20
tags: [AI-Agent, Google-ADK, Skill-Design, Design-Patterns, SKILL-md, Prompt-Engineering]
category: "01-AI与技术"
---

# 5 Agent Skill Design Patterns Every ADK Developer Should Know

> 来源：[Google Cloud Tech 推文](https://x.com/googlecloudtech/status/2033953579824758855) | [原文博客](https://lavinigam.com/posts/adk-skill-design-patterns/)
> 作者：Shubham Saboo & Lavi Nigam | 阅读日期：2026-03-20

## 📌 核心要点

- SKILL.md 格式已被 30+ Agent 工具标准化，真正的挑战不是格式而是内部逻辑设计
- 5 种设计模式：**Tool Wrapper**（工具包装器）、**Generator**（生成器）、**Reviewer**（审查器）、**Inversion**（控制反转）、**Pipeline**（流水线）
- 模式之间可以像乐高一样自由组合，Pipeline 可以嵌套其他任何模式
- 核心思想：从"调 Prompt"转向"设计架构"，让 Agent 行为可预测、可维护
- Google ADK 的 `SkillToolset` 支持渐进式加载，只在触发时才消耗 Token

## 📝 详细笔记

### 背景：格式已标准化，挑战在于逻辑设计

**English**:
When it comes to SKILL.md, developers tend to obsess over the format — YAML frontmatter fields, directory structure, file naming. But the format has already been standardized. Over 30 agent tools now share the same SKILL.md layout. The real challenge isn't how to write a Skill document — it's what logic to put inside it.

**中文**:
谈到 SKILL.md 时，开发者往往会纠结于格式——YAML frontmatter 字段、目录结构、文件命名。但格式早已标准化，目前已有超过 30 种 Agent 工具共享相同的 SKILL.md 布局。真正的挑战不是如何写一份 Skill 文档——而是往里面放什么逻辑。

### 标准三层架构

| 层级 | English | 中文 | 作用 |
|------|---------|------|------|
| **L1** | Metadata (YAML frontmatter) | 元数据 | 用于发现和触发 |
| **L2** | Instructions (SKILL.md body) | 指令正文 | Agent 触发时加载 |
| **L3** | Resources (references/, assets/) | 扩展资源 | 按需引用，节省 Token |

标准目录结构：

```
skill-name/
├── SKILL.md          ← YAML frontmatter + markdown 指令 (Required)
├── references/       ← 风格指南、检查清单、约定 (Optional)
├── assets/           ← 模板和输出格式 (Optional)
└── scripts/          ← 可执行脚本 (Optional)
```

---

### 模式 1: Tool Wrapper（工具包装器）

> 🔧 一句话：把库级知识封装成 Skill / Wrap library-level knowledge into a Skill

**English**:
The Tool Wrapper pattern turns a Skill into a domain expert for a specific library, SDK, or internal system. The Skill contains conventions, common pitfalls, and code examples that the agent applies whenever it works with that tool. No templates or scripts needed — just instructions and reference documentation.

**中文**:
Tool Wrapper 模式将 Skill 转变为针对特定库、SDK 或内部系统的领域专家。Skill 包含约定、常见陷阱和代码示例，Agent 在使用该工具时会自动应用。不需要模板或脚本——只需指令和参考文档。

**核心要素**：
- 📁 目录：`references/`
- 🎯 场景：Agent 需要调用特定 SDK/API（如 Stripe、FastAPI）
- 💡 价值：提炼易错规则，避免写出过时或参数错误的代码，仅在需要时加载上下文

**SKILL.md 示例**:

```markdown
---
name: api-expert
description: FastAPI development best practices and conventions.
metadata:
  pattern: tool-wrapper
  domain: fastapi
---
You are an expert in FastAPI development.

## When Reviewing Code
1. Load the conventions reference
2. Check the user's code against each convention
3. For each violation, cite the specific rule and suggest the fix

## When Writing Code
1. Follow every convention exactly
2. Add type annotations to all function signatures
3. Use Annotated style for dependency injection
```

---

### 模式 2: Generator（生成器）

> 📄 一句话：模板填充式生成 / Template-based structured generation

**English**:
The Generator pattern is for when the output must follow a fixed template every time — consistency matters more than creativity. Instructions orchestrate the flow, `assets/` holds the output template, and `references/` holds the style guide. The agent becomes a "form filler" that collects variables and populates the template.

**中文**:
Generator 模式适用于输出必须每次都遵循固定模板的场景——一致性比创造力更重要。指令编排流程，`assets/` 存放输出模板，`references/` 存放风格指南。Agent 变成一个"填表员"，收集变量并填充模板。

**核心要素**：
- 📁 目录：`assets/` + `references/`
- 🎯 场景：需要一致的文档/代码输出（技术报告、API 文档等）
- 💡 价值：输出格式高度一致，更换模板即可改变输出类型

**SKILL.md 示例**:

```markdown
---
name: report-generator
description: Generates structured technical reports in Markdown.
metadata:
  pattern: generator
  output-format: markdown
---
You are a technical report generator. Follow these steps exactly:

Step 1: Load 'references/style-guide.md' for tone and formatting rules.
Step 2: Load 'assets/report-template.md' for the required output structure.
Step 3: Ask the user for any missing information.
Step 4: Fill the template following the style guide rules.
Step 5: Return the completed report as a single Markdown document.
```

---

### 模式 3: Reviewer（审查器）

> ✅ 一句话：按 checklist 打分审查 / Evaluate against a structured checklist

**English**:
The Reviewer pattern separates "what to check" (a checklist in `references/`) from "how to check" (the review protocol in instructions). This makes the review criteria a maintainable configuration — swap the checklist, and the same Skill becomes a security auditor or a style checker.

**中文**:
Reviewer 模式将"检查什么"（`references/` 中的清单）与"如何检查"（指令中的审查协议）分离。这使得审查标准成为可维护的配置——更换清单，同一个 Skill 就能变成安全审计器或风格检查器。

**核心要素**：
- 📁 目录：`references/`
- 🎯 场景：代码审查、内容审核、安全审计
- 💡 价值：将评审标准从"感觉"转化为"可维护的配置"，确保标准统一

**SKILL.md 示例**:

```markdown
---
name: code-reviewer
description: Reviews Python code for quality, style, and common bugs.
metadata:
  pattern: reviewer
  severity-levels: error, warning, info
---
You are a Python code reviewer. Follow this review protocol exactly:

Step 1: Load 'references/review-checklist.md' for review criteria.
Step 2: Read the user's code carefully. Understand its purpose.
Step 3: Apply each rule. For every violation:
  - Note line number, classify severity, explain WHY, suggest fix
Step 4: Produce structured review:
  - Summary → Findings → Score (1-10) → Top 3 Recommendations
```

---

### 模式 4: Inversion（控制反转）

> 🔄 一句话：Agent 先采访你再干活 / The Skill interviews YOU before acting

**English**:
The Inversion pattern flips the control flow: instead of the agent guessing what to build, the Skill forces the agent to interview the user first. A gate instruction (`DO NOT start building...`) prevents premature generation. Only after collecting all necessary context does the agent proceed to execution.

**中文**:
Inversion 模式反转了控制流：Agent 不再猜测要构建什么，而是 Skill 强制 Agent 先采访用户。一条门控指令（`DO NOT start building...`）防止过早生成。只有在收集完所有必要上下文后，Agent 才会进入执行阶段。

**核心要素**：
- 📁 目录：`assets/`
- 🎯 场景：需求不明确、防止 Agent 瞎猜
- 💡 价值：避免在信息不全时做无效猜测，特别适合模糊需求

**SKILL.md 示例**:

```markdown
---
name: project-planner
description: Plans a new software project by gathering requirements
  through structured questions before producing a plan.
metadata:
  pattern: inversion
  interaction: multi-turn
---
DO NOT start building or designing until all phases are complete.

## Phase 1 — Problem Discovery
- Q1: "What problem does this project solve?"
- Q2: "Who are the primary users?"
- Q3: "What is the expected scale?"

## Phase 2 — Technical Constraints (only after Phase 1)
- Q4: "What deployment environment?"
- Q5: "Any technology stack preferences?"
- Q6: "Non-negotiable requirements?"

## Phase 3 — Synthesis (only after all questions answered)
1. Load 'assets/plan-template.md', fill using gathered info
2. Present plan, iterate on feedback until confirmed
```

---

### 模式 5: Pipeline（流水线）

> ⛓️ 一句话：多步骤串行 + checkpoint / Multi-step workflow with gate conditions

**English**:
The Pipeline pattern enforces a strict multi-step workflow with gate conditions between steps. The agent must complete each step and pass its checkpoint before proceeding. This prevents skipping steps or missing critical validation. Pipelines can nest other patterns — for example, embedding a Reviewer as the final quality check.

**中文**:
Pipeline 模式强制执行严格的多步骤工作流，步骤之间设有门控条件。Agent 必须完成每一步并通过其检查点才能继续。这防止了跳步骤或遗漏关键验证。Pipeline 可以嵌套其他模式——例如将 Reviewer 作为最终质量检查嵌入。

**核心要素**：
- 📁 目录：`references/` + `assets/` + `scripts/`（全用）
- 🎯 场景：复杂工作流、需要质量门禁（文档生成、数据处理）
- 💡 价值：防止 Agent 跳步骤或漏掉关键环节

**SKILL.md 示例**:

```markdown
---
name: doc-pipeline
description: Generates API documentation from Python source code
  through a multi-step pipeline.
metadata:
  pattern: pipeline
  steps: "4"
---
Execute each step in order. Do NOT skip steps or proceed if a step fails.

## Step 1 — Parse & Inventory
Analyze code, extract public API, present checklist for confirmation.

## Step 2 — Generate Docstrings
Load 'references/docstring-style.md', generate docstrings.
Do NOT proceed to Step 3 until user confirms.

## Step 3 — Assemble Documentation
Load 'assets/api-doc-template.md', compile into API reference.

## Step 4 — Quality Check
Review against 'references/quality-checklist.md'.
Fix issues before presenting the final document.
```

---

### 决策树：如何选模式？

```
你的 Skill 需要什么？
│
├─ Agent 需要领域知识？ ──────────→ 🔧 Tool Wrapper
├─ 输出需要一致的结构？ ──────────→ 📄 Generator
├─ 需要检查质量？ ────────────────→ ✅ Reviewer
├─ 需求不够明确？ ────────────────→ 🔄 Inversion
└─ 任务有多个顺序步骤？ ──────────→ ⛓️ Pipeline
```

### 模式组合示例

一个"生成技术方案"的 Skill 可以组合三个模式，包裹在 Pipeline 中：

```
Pipeline
  ├── Step 1: 🔄 Inversion → 收集需求
  ├── Step 2: 📄 Generator → 填充模板
  └── Step 3: ✅ Reviewer  → 自检质量
```

### 关键数据/案例

**模式对比速查表**：

| 模式 | 适用场景 | 目录 | 复杂度 |
|:---|:---|:---|:---|
| **Tool Wrapper** | 特定库的专家知识 | `references/` | ⭐ 低 |
| **Generator** | 固定模板输出 | `assets/` + `references/` | ⭐⭐ 中 |
| **Reviewer** | 清单式评估 | `references/` | ⭐⭐ 中 |
| **Inversion** | 先收集再执行 | `assets/` | ⭐⭐ 中 |
| **Pipeline** | 有序步骤 + 门控 | 全部三个 | ⭐⭐⭐ 高 |

**ADK 代码集成示例**：

```python
from google.adk import Agent
from google.adk.skills import load_skill_from_dir
from google.adk.tools.skill_toolset import SkillToolset

skill_toolset = SkillToolset(
    skills=[
        load_skill_from_dir(SKILLS_DIR / "api-expert"),       # Tool Wrapper
        load_skill_from_dir(SKILLS_DIR / "report-generator"), # Generator
        load_skill_from_dir(SKILLS_DIR / "code-reviewer"),    # Reviewer
        load_skill_from_dir(SKILLS_DIR / "project-planner"),  # Inversion
        load_skill_from_dir(SKILLS_DIR / "doc-pipeline"),     # Pipeline
    ],
)

root_agent = Agent(
    model="gemini-2.5-flash",
    name="pattern_demo_agent",
    instruction="Load relevant skills before acting on any user request.",
    tools=[skill_toolset],
)
```

## 💡 个人思考

1. **从调 Prompt 到设计架构**：这套模式让我意识到，构建可靠 Agent 不是靠调优超长 Prompt，而是要像软件工程一样做模块化设计。这与我从后端开发转型 AI 应用开发的路径高度契合——后端的架构思维在这里依然适用。

2. **Inversion 模式最实用**：很多时候 Agent 产出质量不好，不是能力不行，而是上下文太模糊。强制"先问再做"这个简单规则，就能大幅提升产出质量。

3. **Pipeline 可以用在"吵架帮手"**：我正在开发的首个独立应用，其训练流程天然就是一个 Pipeline——场景选择 → 模拟对话 → 总结评分 → 经验反馈。每一步之间加上 checkpoint 可以确保训练质量。

4. **可组合性是核心竞争力**：5 种模式不互斥，一个复杂 Skill 可以像乐高积木一样组合。这和微服务架构的理念如出一辙，作为后端工程师特别有共鸣。

## 🏷️ 关键词

`Agent Skill` `Google ADK` `SKILL.md` `Design Patterns` `Tool Wrapper` `Generator` `Reviewer` `Inversion` `Pipeline` `Prompt Engineering`

## 🔗 相关笔记

- 相关链接：[Google ADK 官方文档](https://google.github.io/adk-docs/)
- 相关链接：[中文解读 (ofox.ai)](https://ofox.ai/zh/blog/google-adk-agent-skill-design-patterns-2026/)
