# 教育数据挖掘平台 Harness

> 基于 Claude Code Harness 思想构建的多智能体（Multi-Agent）协作开发框架
> 
> 项目部署基于 https://github.com/revfactory/harness

## 项目简介

随着大语言模型（LLM）的快速发展，AI 已逐渐从“对话工具”演化为能够自主规划、协同执行任务的智能体（Agent）。

传统的大模型开发模式通常采用：

```text
用户需求
    ↓
单个大模型
    ↓
输出结果
```

这种方式适用于简单任务，但在软件开发、系统设计、数据分析等复杂场景下，往往存在职责混乱、上下文过长、任务拆解困难等问题。

为了解决上述问题，本项目基于 Claude Code 的 Harness 思想，构建了一个面向教育数据挖掘平台（Educational Data Mining Platform）的多智能体协作框架。

---

# Harness 架构设计

项目采用 Claude Harness 的经典三层架构：

```text
CLAUDE.md
    ↓
Agents
    ↓
Skills
```

## 1. CLAUDE.md —— 全局规则层

CLAUDE.md 是整个项目的核心配置文件。

其作用类似于：

* 项目章程
* 团队规范
* 开发准则

主要负责定义：

* 项目目标
* 开发原则
* 输出格式
* Agent 调用规则
* 语言规范

示例：

```text
项目名称：
Educational Data Mining Platform

开发语言：
中文优先

目标：
构建教育数据挖掘教学平台
```

所有 Agent 在执行任务时都会共享该文件中的上下文信息。

---

## 2. Agents —— 角色层

Agents 用于模拟真实软件团队中的不同岗位。

每个 Agent 拥有：

* 独立职责
* 专业领域知识
* 工作边界
* 输出规范

### Platform Architect（平台架构师）

职责：

* 系统总体设计
* 技术选型
* 模块拆分
* 开发规划

---

# 3. Skills —— 能力层

Skills 用于存放专业知识和标准流程。

与 Agent 相比：

Agent 决定：

```text
谁来做
```

Skill 决定：

```text
怎么做
```

---

# 多 Agent 协作流程

在实际开发过程中，不同 Agent 会围绕同一个目标协同工作。

例如：

开发一个教育数据挖掘平台：

```text
用户需求
      │
      ▼
Platform Architect
      │
 ┌────┼─────┬─────┐
 ▼    ▼     ▼     ▼
前端  后端  ML   分析
 │    │     │     │
 └────┴─────┴─────┘
           │
           ▼
      QA Engineer
           │
           ▼
        最终结果
```

这种方式更接近真实的软件开发团队协作模式。

---

# 未来工作

未来计划进一步探索：

### 多模型协作

例如：

* Claude
* DeepSeek
* Qwen
* GPT

根据任务特点动态选择最优模型。

---

### 工作流编排

引入：

* AutoGen
* LangGraph
* CrewAI

实现更加复杂的 Agent 协同机制。

---

### 长期记忆

构建项目级知识库与经验库，实现持续学习。

---

### 企业级 Agent 平台

进一步实现：

* 状态管理
* 权限管理
* 审批流程
* 任务调度

逐步向企业级 AI 开发平台演进。

---

# License

仅用于学习、研究与教学交流。
