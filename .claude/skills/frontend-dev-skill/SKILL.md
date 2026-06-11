---
name: frontend-dev-skill
description: "使用 React 构建教育用数据挖掘平台的前端。实现交互式 UI 组件、数据集探索界面、算法实验界面、学习仪表板、响应式布局。请求'前端'、'UI'、'React 组件'、'学生界面'、'仪表板 UI'时使用此技能。也处理前端缺陷修复、样式变更、组件追加。"
---

# 前端开发技能

## 概述

使用 React 构建教育用数据挖掘平台的前端应用。核心目标是提供让学生直观理解数据挖掘概念的交互式 UI。

## 工作流程

### 1. 项目结构设定

```
frontend/
├── src/
│   ├── components/        # 可复用的 UI 组件
│   │   ├── common/        # 按钮、输入、模态框等
│   │   ├── layout/        # 头部、侧边栏、底部
│   │   ├── data/          # 数据集相关组件
│   │   ├── experiment/    # 实验执行组件
│   │   └── analytics/     # 学习分析组件
│   ├── pages/             # 路由页面
│   ├── store/             # Zustand/Redux 状态管理
│   ├── api/               # API 调用函数
│   ├── hooks/             # 自定义 Hook
│   ├── utils/             # 工具函数
│   ├── styles/            # 全局样式、主题
│   └── types/             # TypeScript 类型定义
├── public/
├── vite.config.ts
├── tailwind.config.js
└── package.json
```

### 2. 核心界面实现

#### 2.1 仪表板（首页）
- **学习进度概览**：完成的实验、进行中的课程、获得的徽章
- **推荐学习**：下一个学习的概念、复习提醒
- **最近活动**：最后的实验、保存的数据集
- **统计**：总学习时间、完成率、平均分

#### 2.2 数据集探索
- **数据集列表**：搜索、过滤、排序功能
- **数据预览**：表格形式、统计摘要
- **列分析**：类型、分布、缺失值比例
- **可视化**：直方图、箱线图、相关性矩阵
- **下载/上传**：支持 CSV、JSON、Excel

#### 2.3 实验执行
- **算法选择**：按类别分类（回归、分类、聚类等）
- **参数设置**：通过滑块、输入框调整超参数
- **执行训练**：实时显示进度状态（WebSocket）
- **确认结果**：性能指标、可视化、比较图表
- **保存/分享**：保存实验结果、分享链接

#### 2.4 算法可视化
- **分步动画**：分步可视化算法运行过程
- **交互式操作**：速度调节、步骤移动、重置
- **说明文本**：每一步附带概念说明
- **比较模式**：并排比较多个算法

#### 2.5 学习分析
- **进度图表**：按时间学习量、按概念掌握度
- **强项/弱项**：雷达图显示各领域水平
- **推荐路径**：下一学习项、复习时间
- **同伴比较**：与匿名化平均值比较（可选）

### 3. 状态管理

```typescript
// store/useExperimentStore.ts
interface ExperimentState {
  currentAlgorithm: string | null;
  parameters: Record<string, any>;
  status: 'idle' | 'running' | 'completed' | 'error';
  progress: number; // 0-100
  result: ExperimentResult | null;
  logs: string[];

  setAlgorithm: (name: string) => void;
  setParameter: (key: string, value: any) => void;
  startExperiment: () => void;
  updateProgress: (progress: number) => void;
  setResult: (result: ExperimentResult) => void;
}
```

### 4. API 集成

```typescript
// api/client.ts
const apiClient = axios.create({
  baseURL: import.meta.env.VITE_API_BASE_URL,
  timeout: 30000,
});

// WebSocket for real-time updates
const ws = new WebSocket(import.meta.env.VITE_WS_URL);
ws.onmessage = (event) => {
  const data = JSON.parse(event.data);
  // 状态更新
};
```

### 5. 样式指南

**设计原则：**
- **简洁专业**：虽是教育平台但不过于花哨
- **信息层次**：重要信息放大，细节信息缩小
- **一致配色**：状态色（成功：绿、警告：黄、错误：红）
- **暗色模式**：支持暗色模式以减少视觉疲劳

**Tailwind 主题：**
```javascript
// tailwind.config.js
module.exports = {
  theme: {
    extend: {
      colors: {
        primary: '#2563EB',      // 蓝色：主要操作
        secondary: '#7C3AED',    // 紫色：辅助
        success: '#10B981',      // 绿色：成功
        warning: '#F59E0B',      // 黄色：警告
        error: '#EF4444',        // 红色：错误
      },
      fontFamily: {
        sans: ['Inter', 'system-ui', 'sans-serif'],
        mono: ['JetBrains Mono', 'monospace'], // 代码显示用
      },
    },
  },
};
```

### 6. 无障碍访问（A11y）

**必须遵守事项：**
- 所有图片添加 `alt` 文本
- 所有功能可通过键盘访问
- 颜色对比度 4.5:1 以上（WCAG AA）
- 表单字段关联 `label`
- 错误消息使用 `aria-describedby`
- 使用实时区域（`aria-live`）通知实时更新

### 7. 性能优化

**大数据处理：**
- **虚拟滚动**：1000行以上表格使用 react-window
- **记忆化**：使用 `useMemo`、`useCallback` 防止不必要的渲染
- **代码分割**：使用 React.lazy 按页面延迟加载
- **图片优化**：WebP 格式、延迟加载

**API 优化：**
- **缓存**：使用 React Query 缓存 API 响应
- **乐观更新**：先更新 UI 以提升用户体验
- **防抖**：搜索输入 300ms 防抖

## 测试

**单元测试：**
```typescript
// components/DataTable.test.tsx
import { render, screen } from '@testing-library/react';
import { DataTable } from './DataTable';

test('renders data rows correctly', () => {
  const data = [{ name: 'Alice', age: 25 }, { name: 'Bob', age: 30 }];
  render(<DataTable data={data} columns={['name', 'age']} />);
  expect(screen.getByText('Alice')).toBeInTheDocument();
  expect(screen.getByText('Bob')).toBeInTheDocument();
});
```

**集成测试：**
- API 模拟（MSW）后测试完整流程
- 状态管理逻辑测试
- 路由测试

## 参考

- React 18 官方文档: https://react.dev
- Tailwind CSS: https://tailwindcss.com
- Vite: https://vitejs.dev
- React Testing Library: https://testing-library.com/docs/react-testing-library/intro
