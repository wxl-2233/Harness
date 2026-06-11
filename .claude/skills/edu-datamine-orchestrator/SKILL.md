---
name: edu-datamine-orchestrator
description: "编排教育用数据挖掘平台开发的元技能。协调7名专业代理团队（架构师、前端、后端、ML工程师、可视化工程师、分析工程师、QA工程师）构建平台。请求'数据挖掘平台'、'教育用ML平台'、'算法可视化平台'、'学习分析系统'相关工作，或'平台构建'、'启动Harness'、'启动团队'时使用此技能。也处理部分重执行（'仅前端重做'、'仅修改ML模块'）、更新（'更新平台'）、改进（'改进结果'）请求。"
---

# 教育用数据挖掘平台编排器

## 概述

编排教育用数据挖掘平台开发的7人代理团队。**执行模式：代理团队（Agent Team）**。编排器通过 TeamCreate 组建团队，通过 TaskCreate 分配任务，团队成员通过 SendMessage 直接通信并自行协调。

## 团队构成

| 代理 | 角色 | 主要职责 |
|------|------|----------|
| **platform-architect** | 架构师/领导者 | 系统设计、技术决策、团队协调 |
| **frontend-dev** | 前端开发者 | React UI、交互式组件、响应式设计 |
| **backend-dev** | 后端开发者 | REST API、WebSocket、认证、实验编排 |
| **ml-engineer** | ML 工程师 | 算法实现、训练流水线、评估、实验管理 |
| **viz-engineer** | 可视化工程师 | D3.js/Plotly 图表、算法动画、数据探索 |
| **analytics-engineer** | 分析工程师 | 学习分析、学生建模、推荐系统、仪表板 |
| **qa-engineer** | QA 工程师 | 测试策略、E2E 测试、性能/无障碍验证 |

## 工作流

### Phase 0：上下文确认

工作流开始前，确认现有产出物是否存在，决定执行模式。

1. 确认 `_workspace/` 目录是否存在
2. 判断用户请求类型：
   - `_workspace/` 存在 + 部分修改请求 → **部分重执行**（仅重新调用对应代理）
   - `_workspace/` 存在 + 新输入提供 → **新执行**（将现有 `_workspace/` 移动到 `_workspace_prev/`）
   - `_workspace/` 不存在 → **初始执行**
3. 若存在现有产出物，读取并了解当前状态

### Phase 1：需求分析及架构设计

**执行模式：** platform-architect（独立分析后团队共享）

1. platform-architect 分析用户需求
2. 决定技术栈
3. 设计系统架构
4. 定义模块间接口
5. 制定任务分配计划

**产出物：**
- `_workspace/01_architect_design.md` — 系统架构设计书
- `_workspace/01_tech_stack.md` — 技术栈决定书
- `_workspace/01_task_plan.md` — 任务分配计划（含优先级、依赖关系）
- `_workspace/01_api_contract.md` — 模块间 API 契约书

### Phase 2：团队组建及任务分配

**执行模式：** 代理团队

1. 通过 `TeamCreate` 组建7人团队
2. 基于 Phase 1 的任务分配计划通过 `TaskCreate` 分配任务
3. 明确任务间依赖关系（如：前端在完成后端 API 规范后开始）

**任务分配示例：**
```
[后端] API 设计及基础结构 → [前端] API 集成 UI
                           → [ML] 算法实现
                           → [分析] 数据模型设计
[ML] 算法实现 → [可视化] 算法可视化
[前端] UI 组件 → [可视化] 图表集成
[全体] 模块完成 → [QA] 测试及验证
```

### Phase 3：并行开发（扇出）

**执行模式：** 代理团队（自行协调）

团队成员各自并行执行任务。按照团队通信协议直接通信协调。

**核心协作点：**

| 协作 | 参与者 | 产出物 |
|------|--------|--------|
| API 设计 | backend + frontend + ml | `_workspace/03_api_spec.yaml` |
| 数据模型 | backend + analytics | `_workspace/03_data_model.sql` |
| 可视化数据格式 | ml + viz | `_workspace/03_viz_schema.json` |
| 分析数据流水线 | backend + analytics | `_workspace/03_analytics_pipeline.md` |

**进度管理：**
- 各代理完成任务时通过 `TaskUpdate` 变更状态
- 发生阻塞时通过 `SendMessage` 直接与相关代理沟通
- 架构师监控进度，必要时介入

### Phase 4：集成及可视化

**执行模式：** 代理团队

1. 可视化工程师集成 ML 算法可视化组件
2. 前端连接后端 API 和可视化组件
3. 分析工程师集成学习分析仪表板

**集成顺序：**
```
ML 算法 → 可视化组件 → 前端集成 → 分析仪表板
```

**产出物：**
- `_workspace/04_integration_log.md` — 集成过程及问题记录
- `_workspace/04_ui_screenshots.md` — 主要界面截图及说明

### Phase 5：QA 及验证

**执行模式：** 代理团队（qa-engineer 主导）

QA 工程师执行渐进式 QA。每个模块完成后立即开始测试。

**测试阶段：**

1. **单元测试** — 各模块的函数/组件测试
2. **集成测试** — API + DB + 服务集成测试
3. **E2E 测试** — 核心学生旅程测试
4. **性能测试** — 大数据集渲染、API 响应时间
5. **无障碍测试** — WCAG 2.1 AA 合规验证

**产出物：**
- `_workspace/05_test_report.md` — 测试结果报告
- `_workspace/05_bugs.md` — 发现的缺陷列表及严重级别
- `_workspace/05_quality_score.md` — 质量评分（功能、性能、无障碍）

### Phase 6：汇总及报告

**执行模式：** platform-architect（主导）+ 团队（反馈）

1. platform-architect 汇总整体结果
2. 从各代理收集最终报告
3. 计算质量评分
4. 编写改进建议
5. 生成最终报告

**最终产出物：**
- `output/platform_design.md` — 最终平台设计书
- `output/architecture_diagram.md` — 架构图
- `output/api_documentation.md` — API 文档
- `output/user_guide.md` — 用户指南（学生/教师）
- `output/quality_report.md` — 质量报告
- `output/next_steps.md` — 下一步建议

## 数据传递协议

### 策略组合
- **任务基础**（协调）：通过 `TaskCreate`/`TaskUpdate` 共享任务状态
- **文件基础**（产出物）：在 `_workspace/` 中保存中间产出物
- **消息基础**（实时）：通过 `SendMessage` 实时沟通

### 文件规范
- 工作目录：`_workspace/`
- 文件名：`{phase}_{agent}_{artifact}.{ext}`
  - 例：`01_architect_design.md`, `03_api_spec.yaml`
- 最终产出物：输出到 `output/` 目录

## 错误处理

| 错误类型 | 策略 |
|----------|------|
| 代理间意见冲突 | 架构师以教育效果 + 技术实用性为基准决定 |
| API 规范不一致 | 后端优先更新 OpenAPI 规范，前端基于规范重做 |
| ML 实验失败 | ML 工程师重试，3次失败后报告架构师 |
| 可视化渲染失败 | 可视化工程师提供 SVG 回退方案 |
| 测试失败 | QA 工程师编写缺陷报告，对应代理修改 |
| 进度延迟 | 架构师重新调整关键路径任务优先级 |

## 测试场景

### 正常流程
1. 用户请求"创建数据挖掘平台"
2. Phase 0-6 顺序执行
3. 最终产出物：设计书、API 文档、用户指南、质量报告

### 部分重执行
1. 用户请求"仅重新构建 ML 模块"
2. Phase 0 检测到部分重执行
3. 仅重新调用 ML 工程师 + 可视化工程师
4. 其余模块使用现有产出物

### 错误流程
1. 后端 API 设计时与前端意见冲突
2. 架构师收集双方意见
3. 以教育效果为基准决定
4. 将决定记录到 `_workspace/01_decisions.md`
5. 继续推进

## 触发关键词

**初始执行：**
- "数据挖掘平台"、"教育用 ML 平台"、"算法可视化平台"
- "学习分析系统"、"平台构建"、"启动 Harness"、"启动团队"

**部分重执行：**
- "仅前端重做"、"仅后端修改"、"仅 ML 模块更新"
- "追加可视化"、"补充分析功能"

**后续任务：**
- "平台更新"、"改进结果"、"基于前次结果"
- "重新执行"、"重执行"、"修改"、"补充"
