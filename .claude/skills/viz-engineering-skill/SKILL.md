---
name: viz-engineering-skill
description: "使用 D3.js、Plotly、Recharts 构建教育用数据挖掘平台的数据可视化及算法动画。实现交互式图表、算法动作可视化（回归拟合、决策边界、聚类形成）、数据探索 UI、学习分析仪表板。请求'可视化'、'图表'、'算法动画'、'数据探索'、'D3.js'、'Plotly'时使用此技能。也处理图表追加、动画改进、大数据优化。"
---

# 可视化工程技能

## 概述

构建教育用数据挖掘平台的数据可视化及算法动画。使用 D3.js、Plotly.js、Recharts 提供让学生直观理解数据挖掘算法运行过程的交互式可视化。

## 工作流程

### 1. 项目结构设定

```
frontend/src/viz/
├── components/              # 可复用的可视化组件
│   ├── charts/              # 基础图表
│   │   ├── ScatterPlot.tsx
│   │   ├── LineChart.tsx
│   │   ├── BarChart.tsx
│   │   ├── Heatmap.tsx
│   │   └── BoxPlot.tsx
│   ├── algorithms/          # 算法可视化
│   │   ├── RegressionFit.tsx
│   │   ├── DecisionBoundary.tsx
│   │   ├── TreeVisualization.tsx
│   │   ├── KMeansAnimation.tsx
│   │   └── PCAProjection.tsx
│   └── dashboards/          # 仪表板组件
│       ├── LearningProgress.tsx
│       ├── ConceptMastery.tsx
│       └── ActivityHeatmap.tsx
├── animations/              # 动画逻辑
│   ├── useAnimation.ts      # 动画 Hook
│   ├── transition.ts        # 过渡效果
│   └── steps.ts             # 分步动画
├── themes/                  # 图表主题
│   ├── index.ts
│   ├── light.ts
│   └── dark.ts
├── transforms/              # 数据转换工具
│   ├── format.ts            # 格式化
│   ├── aggregate.ts         # 聚合
│   └── downsample.ts        # 降采样
├── hooks/                   # 自定义 Hook
│   ├── useChart.ts
│   └── useResponsive.ts
└── utils/
    ├── colors.ts            # 颜色调色板
    └── scales.ts            # 比例尺函数
```

### 2. 核心可视化组件

**基础图表：**
- 散点图（Scatter Plot）：数据探索、显示回归线
- 折线图（Line Chart）：学习进度、时间序列数据
- 柱状图（Bar Chart）：类别比较
- 热力图（Heatmap）：相关性、2D 分布
- 箱线图（Box Plot）：数据分布、异常值

**算法可视化：**
- 回归拟合（Regression Fit）：拟合过程动画
- 决策边界（Decision Boundary）：分类边界可视化
- 树可视化（Tree Visualization）：决策树结构
- K-Means 动画：聚类形成过程
- PCA 投影：降维结果

**仪表板：**
- 学习进度图表
- 概念掌握度雷达图
- 活动热力图

> 详细组件实现：参考 `references/component-examples.md`

### 3. 大数据优化

#### 3.1 降采样

```typescript
// viz/transforms/downsample.ts
import { LTTB } from 'downsample';

/**
 * 使用 LTTB（最大三角形三桶）算法降采样
 * 在保留视觉特征的同时减少数据点数量
 */
export function downsample(
  data: Array<{ x: number; y: number }>,
  targetPoints: number = 1000
): Array<{ x: number; y: number }> {
  if (data.length <= targetPoints) return data;

  return LTTB(data as any, targetPoints) as any;
}

/**
 * 2D binning：按网格聚合数据
 * 用于热力图、2D 直方图
 */
export function bin2D(
  data: Array<{ x: number; y: number }>,
  xBins: number = 50,
  yBins: number = 50
): Array<{ x: number; y: number; count: number }> {
  const xMin = Math.min(...data.map(d => d.x));
  const xMax = Math.max(...data.map(d => d.x));
  const yMin = Math.min(...data.map(d => d.y));
  const yMax = Math.max(...data.map(d => d.y));

  const xStep = (xMax - xMin) / xBins;
  const yStep = (yMax - yMin) / yBins;

  const bins = new Map<string, { x: number; y: number; count: number }>();

  for (const point of data) {
    const xBin = Math.floor((point.x - xMin) / xStep);
    const yBin = Math.floor((point.y - yMin) / yStep);
    const key = `${xBin},${yBin}`;

    if (!bins.has(key)) {
      bins.set(key, {
        x: xMin + (xBin + 0.5) * xStep,
        y: yMin + (yBin + 0.5) * yStep,
        count: 0,
      });
    }
    bins.get(key)!.count++;
  }

  return Array.from(bins.values());
}
```

#### 3.2 WebGL 渲染（Deck.gl）

```tsx
// viz/components/charts/ScatterPlotWebGL.tsx
import { ScatterplotLayer } from '@deck.gl/layers';
import { DeckGL } from '@deck.gl/react';
import { OrthographicView } from '@deck.gl/core';

interface ScatterPlotWebGLProps {
  data: Array<{ x: number; y: number; category?: string }>;
}

export function ScatterPlotWebGL({ data }: ScatterPlotWebGLProps) {
  const layer = new ScatterplotLayer({
    id: 'scatter-plot',
    data,
    getPosition: (d: any) => [d.x, d.y],
    getRadius: 5,
    getFillColor: (d: any) => {
      const colors: Record<string, [number, number, number]> = {
        'A': [37, 99, 235],
        'B': [124, 58, 237],
        'C': [16, 185, 129],
      };
      return colors[d.category] || [100, 100, 100];
    },
    opacity: 0.8,
    pickable: true,
    radiusScale: 1,
    radiusMinPixels: 2,
    radiusMaxPixels: 10,
  });

  return (
    <DeckGL
      views={new OrthographicView()}
      initialViewState={{
        target: [0, 0, 0],
        zoom: 0,
      }}
      controller={true}
      layers={[layer]}
    />
  );
}
```

### 4. 颜色调色板

```typescript
// viz/utils/colors.ts

/**
 * 基于 ColorBrewer 的颜色调色板
 * 色盲友好、可打印
 */
export const categoricalColors = {
  // 定性（类别型）
  Set2: ['#66c2a5', '#fc8d62', '#8da0cb', '#e78ac3', '#a6d854', '#ffd92f', '#e5c494', '#b3b3b3'],
  Dark2: ['#1b9e77', '#d95f02', '#7570b3', '#e7298a', '#66a61e', '#e6ab02', '#a6761d', '#666666'],
  Pastel1: ['#fbb4ae', '#b3cde3', '#ccebc5', '#decbe4', '#fed9a6', '#ffffcc', '#e5d8bd', '#fddaec'],

  // 顺序型
  Blues: ['#f7fbff', '#deebf7', '#c6dbef', '#9ecae1', '#6baed6', '#4292c6', '#2171b5', '#08519c', '#08306b'],
  Reds: ['#fff5f0', '#fee0d2', '#fcbba1', '#fc9272', '#fb6a4a', '#ef3b2c', '#cb181d', '#a50f15', '#67000d'],
  Viridis: ['#440154', '#482777', '#3f4a8a', '#31678e', '#26838f', '#1f9d8a', '#6cce5a', '#b6de2b', '#fee825'],

  // 发散型
  RdBu: ['#67001f', '#b2182b', '#d6604d', '#f4a582', '#fddbc7', '#f7f7f7', '#d1e5f0', '#92c5de', '#4393c3', '#2166ac', '#053061'],
};

/**
 * 状态颜色
 */
export const statusColors = {
  success: '#10B981',
  warning: '#F59E0B',
  error: '#EF4444',
  info: '#3B82F6',
  neutral: '#6B7280',
};

/**
 * 颜色无障碍检查（色盲模拟）
 */
export function isColorBlindSafe(palette: string[]): boolean {
  // 简单启发式：检查亮度差异是否足够
  // 实际实现使用色盲模拟库
  return true;
}
```

### 5. 响应式图表

```tsx
// viz/hooks/useResponsive.ts
import { useState, useEffect } from 'react';

export function useResponsive() {
  const [size, setSize] = useState({ width: 800, height: 600 });

  useEffect(() => {
    const handleResize = () => {
      setSize({
        width: window.innerWidth,
        height: window.innerHeight,
      });
    };

    handleResize();
    window.addEventListener('resize', handleResize);
    return () => window.removeEventListener('resize', handleResize);
  }, []);

  return size;
}

// 使用示例
export function ResponsiveChart({ children }: { children: (size: { width: number; height: number }) => React.ReactNode }) {
  const { width, height } = useResponsive();

  return (
    <div style={{ width: '100%', height: '100%' }}>
      {children({ width, height })}
    </div>
  );
}
```

### 6. 无障碍访问

**图表无障碍检查清单：**
- [ ] 不依赖颜色区分（同时使用图案、文本标签）
- [ ] 使用色盲友好调色板（ColorBrewer）
- [ ] 支持键盘导航图表（Tab、箭头键）
- [ ] 为屏幕阅读器提供替代文本
- [ ] 提供数据表格视图（图表 ↔ 表格切换）

```tsx
// 无障碍图表组件示例
export function AccessibleChart({ data, children }: { data: any[]; children: React.ReactNode }) {
  const [viewMode, setViewMode] = useState<'chart' | 'table'>('chart');

  return (
    <div>
      <div role="tablist">
        <button
          role="tab"
          aria-selected={viewMode === 'chart'}
          onClick={() => setViewMode('chart')}
        >
          Chart View
        </button>
        <button
          role="tab"
          aria-selected={viewMode === 'table'}
          onClick={() => setViewMode('table')}
        >
          Table View
        </button>
      </div>

      {viewMode === 'chart' ? (
        <div role="img" aria-label="Data visualization chart">
          {children}
        </div>
      ) : (
        <table>
          <thead>
            <tr>
              {Object.keys(data[0] || {}).map(key => (
                <th key={key}>{key}</th>
              ))}
            </tr>
          </thead>
          <tbody>
            {data.map((row, idx) => (
              <tr key={idx}>
                {Object.values(row).map((val, i) => (
                  <td key={i}>{val as any}</td>
                ))}
              </tr>
            ))}
          </tbody>
        </table>
      )}
    </div>
  );
}
```

## 参考

- D3.js: https://d3js.org
- Plotly.js: https://plotly.com/javascript
- Recharts: https://recharts.org
- Deck.gl: https://deck.gl
- ColorBrewer: https://colorbrewer2.org
