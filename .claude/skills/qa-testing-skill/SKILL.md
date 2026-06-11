---
name: qa-testing-skill
description: "教育用数据挖掘平台的质量保证。构建单元测试（Vitest、pytest）、集成测试、E2E 测试（Playwright）、性能测试（k6）、无障碍验证（axe-core）。请求'测试'、'QA'、'缺陷'、'质量'、'无障碍'、'性能测试'时使用此技能。也处理测试用例追加、缺陷修复验证、CI/CD 流水线构建。"
---

# QA 测试技能

## 概述

保证教育用数据挖掘平台的质量。按照测试金字塔策略，系统性地构建单元、集成、E2E、性能、无障碍测试。

## 工作流程

### 1. 测试目录结构

```
tests/
├── unit/                    # 单元测试
│   ├── frontend/            # React 组件、Hook、工具
│   │   ├── components/
│   │   ├── hooks/
│   │   └── utils/
│   ├── backend/             # Python 服务、工具
│   │   ├── services/
│   │   └── utils/
│   └── ml/                  # ML 算法
│       ├── algorithms/
│       └── preprocessing/
├── integration/             # 集成测试
│   ├── api/                 # API 端点
│   ├── database/            # DB 查询、迁移
│   └── workflows/           # 核心工作流
├── e2e/                     # E2E 测试（Playwright）
│   ├── specs/
│   │   ├── auth.spec.ts
│   │   ├── dataset.spec.ts
│   │   ├── experiment.spec.ts
│   │   └── analytics.spec.ts
│   ├── fixtures/
│   └── pages/               # 页面对象模型
├── performance/             # 性能测试
│   ├── api-load.k6.js
│   └── frontend-lighthouse.js
├── accessibility/           # 无障碍测试
│   └── axe-audit.js
├── fixtures/                # 测试 Fixture
│   ├── datasets/
│   └── mock-data/
└── utils/                   # 测试工具
    ├── api-client.ts
    └── test-helpers.ts
```

### 2. 单元测试

**前端（Vitest + React Testing Library）：**
- 组件渲染、用户交互、错误状态测试
- 自定义 Hook 的状态变更测试

**后端（pytest）：**
- 服务方法成功/失败用例
- 使用 Mock 分离 DB 依赖
- 异常处理验证

**ML 算法：**
- fit/predict 基本行为
- 权重准确性验证
- 可视化数据格式确认

> 详细测试模板：参考 `references/test-templates.md`

### 3. 集成测试

测试 API 端点、DB 查询、核心工作流。

**主要测试场景：**
- 数据集上传/查询/删除
- 实验创建/执行/结果查询
- 认证/授权（JWT 令牌）
- WebSocket 实时更新

> 详细测试模板：参考 `references/test-templates.md`

### 4. E2E 测试（Playwright）

通过自动化测试验证核心学生旅程(Student Journey)。

**主要场景：**
1. 注册 → 登录 → 确认仪表板
2. 选择数据集 → 探索 → 预处理 → 确认可视化
3. 选择算法 → 训练 → 评估 → 解读结果
4. 保存实验 → 比较 → 编写报告
5. 确认学习分析 → 获取推荐 → 继续学习

> 详细测试模板：参考 `references/test-templates.md`

### 5. 性能测试（k6）

```javascript
// tests/performance/api-load.k6.js
import http from 'k6/http';
import { check, sleep } from 'k6';

export const options = {
  stages: [
    { duration: '1m', target: 50 },   // 升温到50用户
    { duration: '3m', target: 50 },   // 50用户保持3分钟
    { duration: '1m', target: 100 },  // 升温到100用户
    { duration: '3m', target: 100 },  // 100用户保持3分钟
    { duration: '1m', target: 0 },    // 降温
  ],
  thresholds: {
    http_req_duration: ['p(95)<500'], // 95%请求在500ms以下
    http_req_failed: ['rate<0.01'],   // 失败率1%以下
  },
};

const BASE_URL = 'http://localhost:8000/api/v1';
const TOKEN = 'test-token';

export default function () {
  const headers = {
    'Authorization': `Bearer ${TOKEN}`,
    'Content-Type': 'application/json',
  };

  // 数据集列表查询
  let res = http.get(`${BASE_URL}/datasets`, { headers });
  check(res, {
    'datasets status is 200': (r) => r.status === 200,
    'datasets response time < 200ms': (r) => r.timings.duration < 200,
  });

  sleep(1);

  // 创建实验
  const payload = JSON.stringify({
    name: 'Load Test Experiment',
    algorithm: 'linear_regression',
    parameters: { learning_rate: 0.01 },
    dataset_id: 'test-dataset-id',
  });

  res = http.post(`${BASE_URL}/experiments`, payload, { headers });
  check(res, {
    'experiment created': (r) => r.status === 201,
  });

  sleep(2);
}
```

### 6. 无障碍测试

```javascript
// tests/accessibility/axe-audit.js
import { axe, toHaveNoViolations } from 'jest-axe';
import { render } from '@testing-library/react';
import { Dashboard } from '@/pages/Dashboard';

expect.extend(toHaveNoViolations);

describe('Accessibility', () => {
  it('Dashboard has no accessibility violations', async () => {
    const { container } = render(<Dashboard />);
    const results = await axe(container);
    expect(results).toHaveNoViolations();
  });

  it('DataTable is keyboard navigable', async () => {
    const { container } = render(<DataTable data={mockData} columns={columns} />);

    // Tab 键移动焦点
    const firstCell = container.querySelector('td');
    firstCell.focus();
    expect(document.activeElement).toBe(firstCell);

    // 箭头键移动
    fireEvent.keyDown(firstCell, { key: 'ArrowRight' });
    // ... 追加键盘导航测试
  });

  it('Charts have text alternatives', async () => {
    const { container } = render(<ScatterPlot data={data} />);

    const chart = container.querySelector('[role="img"]');
    expect(chart).toHaveAttribute('aria-label');
  });
});
```

### 7. 缺陷报告模板

```markdown
## Bug Report

**严重级别:** 🔴 Critical / 🟠 High / 🟡 Medium / 🟢 Low
**发现日期:** 2026-06-10
**发现者:** QA Engineer

### 描述
[缺陷的简要描述]

### 复现步骤
1. [第一步]
2. [第二步]
3. [第三步]

### 期望结果
[期望的行为]

### 实际结果
[实际的行为]

### 环境
- 浏览器: Chrome 120
- 操作系统: Windows 11
- 截图/视频: [链接]

### 日志
```
[相关日志]
```

### 影响代理
- [ ] frontend-dev
- [ ] backend-dev
- [ ] ml-engineer
- [ ] viz-engineer
- [ ] analytics-engineer
```

## 参考

- Vitest: https://vitest.dev
- Playwright: https://playwright.dev
- k6: https://k6.io
- axe-core: https://github.com/dequelabs/axe-core
