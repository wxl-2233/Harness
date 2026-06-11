# 可视化组件示例

## 散点图（Scatter Plot）

```tsx
import { ScatterChart, Scatter, XAxis, YAxis, CartesianGrid, Tooltip, ResponsiveContainer } from 'recharts';

interface ScatterPlotProps {
  data: Array<{ x: number; y: number; category?: string }>;
  xLabel?: string;
  yLabel?: string;
}

export function ScatterPlot({ data, xLabel, yLabel }: ScatterPlotProps) {
  const colors = ['#2563EB', '#7C3AED', '#10B981', '#F59E0B'];

  return (
    <ResponsiveContainer width="100%" height={400}>
      <ScatterChart>
        <CartesianGrid strokeDasharray="3 3" />
        <XAxis type="number" dataKey="x" name={xLabel} />
        <YAxis type="number" dataKey="y" name={yLabel} />
        <Tooltip />
        <Scatter data={data} fill={colors[0]} />
      </ScatterChart>
    </ResponsiveContainer>
  );
}
```

## K-Means 动画

```tsx
import { useState, useEffect } from 'react';
import { ScatterChart, Scatter, XAxis, YAxis, Cell } from 'recharts';

interface KMeansAnimationProps {
  iterations: Array<{
    centroids: Array<[number, number]>;
    labels: number[];
  }>;
  data: Array<{ x: number; y: number }>;
  speed?: number;
}

export function KMeansAnimation({ iterations, data, speed = 1000 }: KMeansAnimationProps) {
  const [currentStep, setCurrentStep] = useState(0);
  const [isPlaying, setIsPlaying] = useState(false);

  useEffect(() => {
    if (!isPlaying || currentStep >= iterations.length - 1) return;
    const timer = setTimeout(() => setCurrentStep(s => s + 1), speed);
    return () => clearTimeout(timer);
  }, [isPlaying, currentStep]);

  const colors = ['#2563EB', '#7C3AED', '#10B981', '#F59E0B'];
  const currentIteration = iterations[currentStep];

  const coloredData = data.map((point, idx) => ({
    ...point,
    cluster: currentIteration.labels[idx],
  }));

  return (
    <div>
      <div className="controls">
        <button onClick={() => setIsPlaying(!isPlaying)}>
          {isPlaying ? 'Pause' : 'Play'}
        </button>
        <span>Iteration: {currentStep + 1} / {iterations.length}</span>
      </div>

      <ResponsiveContainer width="100%" height={500}>
        <ScatterChart>
          <XAxis type="number" dataKey="x" />
          <YAxis type="number" dataKey="y" />
          <Scatter data={coloredData}>
            {coloredData.map((entry, index) => (
              <Cell key={index} fill={colors[entry.cluster]} />
            ))}
          </Scatter>
          <Scatter
            data={currentIteration.centroids.map(([x, y]) => ({ x, y }))}
            fill="#000"
            shape="star"
          />
        </ScatterChart>
      </ResponsiveContainer>
    </div>
  );
}
```

## 热力图（Heatmap）

```tsx
interface HeatmapProps {
  data: number[][];
  xLabels: string[];
  yLabels: string[];
  colorScale?: string;
}

export function Heatmap({ data, xLabels, yLabels, colorScale = 'Blues' }: HeatmapProps) {
  const colors = colorScales[colorScale];

  return (
    <svg width="100%" height="400" viewBox={`0 0 ${xLabels.length} ${yLabels.length}`}>
      {data.map((row, i) =>
        row.map((value, j) => (
          <rect
            key={`${i}-${j}`}
            x={j}
            y={i}
            width={1}
            height={1}
            fill={getColor(value, colors)}
          />
        ))
      )}
    </svg>
  );
}
```

## 学习进度图表

```tsx
import { LineChart, Line, XAxis, YAxis, CartesianGrid, Tooltip, Legend } from 'recharts';

interface LearningProgressProps {
  data: Array<{
    date: string;
    mastery: number;
    experiments: number;
  }>;
}

export function LearningProgress({ data }: LearningProgressProps) {
  return (
    <ResponsiveContainer width="100%" height={300}>
      <LineChart data={data}>
        <CartesianGrid strokeDasharray="3 3" />
        <XAxis dataKey="date" />
        <YAxis yAxisId="left" />
        <YAxis yAxisId="right" orientation="right" />
        <Tooltip />
        <Legend />
        <Line
          yAxisId="left"
          type="monotone"
          dataKey="mastery"
          stroke="#2563EB"
          name="Mastery %"
        />
        <Line
          yAxisId="right"
          type="monotone"
          dataKey="experiments"
          stroke="#10B981"
          name="Experiments"
        />
      </LineChart>
    </ResponsiveContainer>
  );
}
```

## 颜色调色板

### ColorBrewer 调色板

```typescript
export const colorScales = {
  // 顺序型
  Blues: ['#f7fbff', '#deebf7', '#c6dbef', '#9ecae1', '#6baed6', '#4292c6', '#2171b5', '#08519c', '#08306b'],
  Reds: ['#fff5f0', '#fee0d2', '#fcbba1', '#fc9272', '#fb6a4a', '#ef3b2c', '#cb181d', '#a50f15', '#67000d'],
  Viridis: ['#440154', '#482777', '#3f4a8a', '#31678e', '#26838f', '#1f9d8a', '#6cce5a', '#b6de2b', '#fee825'],

  // 发散型
  RdBu: ['#67001f', '#b2182b', '#d6604d', '#f4a582', '#fddbc7', '#f7f7f7', '#d1e5f0', '#92c5de', '#4393c3', '#2166ac', '#053061'],

  // 类别型
  Set2: ['#66c2a5', '#fc8d62', '#8da0cb', '#e78ac3', '#a6d854', '#ffd92f', '#e5c494', '#b3b3b3'],
};
```

### 无障碍考虑

- **色盲友好**：使用 ColorBrewer 调色板
- **亮度对比**：保持 4.5:1 以上
- **双重编码**：同时使用颜色 + 图案/纹理
- **替代文本**：提供图表说明
