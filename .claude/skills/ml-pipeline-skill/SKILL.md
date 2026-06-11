---
name: ml-pipeline-skill
description: "构建教育用数据挖掘平台的机器学习流水线。实现基于 scikit-learn、PyTorch 的算法（线性回归、决策树、随机森林、K-Means、PCA、神经网络等），模型训练/评估、超参数调优、实验管理。请求'ML 算法'、'模型训练'、'实验'、'算法实现'、'评估指标'时使用此技能。也处理算法追加、性能优化、实验复现。"
---

# ML 流水线技能

## 概述

构建教育用数据挖掘平台的机器学习流水线。提供教育性清晰、可复现的 ML 流水线，让学生能够直接实验和理解数据挖掘算法。

## 工作流程

### 1. 项目结构设定

```
ml/
├── algorithms/              # 算法实现
│   ├── __init__.py
│   ├── base.py              # BaseAlgorithm 抽象类
│   ├── regression/
│   │   ├── linear_regression.py
│   │   ├── polynomial_regression.py
│   │   └── ridge_lasso.py
│   ├── classification/
│   │   ├── logistic_regression.py
│   │   ├── decision_tree.py
│   │   ├── random_forest.py
│   │   ├── svm.py
│   │   └── knn.py
│   ├── clustering/
│   │   ├── kmeans.py
│   │   ├── dbscan.py
│   │   └── hierarchical.py
│   ├── dimensionality/
│   │   ├── pca.py
│   │   └── tsne.py
│   └── neural/
│       ├── mlp.py
│       ├── cnn.py
│       └── rnn.py
├── preprocessing/           # 数据预处理
│   ├── __init__.py
│   ├── scaler.py            # 归一化、标准化
│   ├── encoder.py           # 类别编码
│   ├── imputer.py           # 缺失值处理
│   └── feature_selection.py
├── evaluation/              # 评估模块
│   ├── __init__.py
│   ├── metrics.py           # 准确率、精确率、召回率、F1、RMSE 等
│   ├── cross_validation.py  # 交叉验证
│   └── comparison.py        # 模型比较
├── training/                # 训练管理
│   ├── __init__.py
│   ├── trainer.py           # 训练编排器
│   ├── callback.py          # 回调（日志、早停）
│   └── experiment.py        # 实验管理
├── experiments/             # 实验配置
│   ├── configs/             # YAML 配置文件
│   └── results/             # 实验结果
└── utils/
    ├── __init__.py
    ├── data_loader.py       # 数据加载
    ├── visualization.py     # 可视化辅助
    └── seed.py              # 种子固定
```

### 2. 算法实现原则

#### 2.1 BaseAlgorithm 抽象类

所有算法继承此类实现。为教育一致性提供统一接口。

```python
# ml/algorithms/base.py
from abc import ABC, abstractmethod
from typing import Any, Dict, Optional
import numpy as np

class BaseAlgorithm(ABC):
    """所有 ML 算法的基础类"""

    def __init__(self, **hyperparameters):
        self.hyperparameters = hyperparameters
        self.model = None
        self.is_fitted = False
        self.training_history = []  # 训练过程记录（可视化用）

    @abstractmethod
    def fit(self, X: np.ndarray, y: np.ndarray, **kwargs) -> 'BaseAlgorithm':
        """模型训练"""
        pass

    @abstractmethod
    def predict(self, X: np.ndarray) -> np.ndarray:
        """预测"""
        pass

    def score(self, X: np.ndarray, y: np.ndarray) -> float:
        """评估分数（各算法使用不同指标）"""
        predictions = self.predict(X)
        return self._default_metric(y, predictions)

    @abstractmethod
    def _default_metric(self, y_true: np.ndarray, y_pred: np.ndarray) -> float:
        """默认评估指标"""
        pass

    def get_params(self) -> Dict[str, Any]:
        """返回超参数"""
        return self.hyperparameters

    def set_params(self, **params) -> 'BaseAlgorithm':
        """设置超参数"""
        self.hyperparameters.update(params)
        return self

    def get_visualization_data(self) -> Dict[str, Any]:
        """返回可视化数据（子类中重写）"""
        return {}
```

### 2.2 算法实现示例

**线性回归（Linear Regression）：**
- 最小二乘法训练：w = (X^T X)^(-1) X^T y
- 可视化数据：回归方程、权重、偏置
- 评估指标：R² 分数

**K-Means 聚类：**
- 迭代更新 K 个中心点
- 记录每次迭代状态（动画用）
- 计算 Inertia（簇内平方和）

> 完整算法实现：参考 `references/algorithm-implementations.md`

### 3. 数据预处理

```python
# ml/preprocessing/scaler.py
import numpy as np

class StandardScaler:
    """标准化：转换为均值0、方差1"""

    def __init__(self):
        self.mean_ = None
        self.std_ = None
        self.is_fitted = False

    def fit(self, X: np.ndarray) -> 'StandardScaler':
        self.mean_ = np.mean(X, axis=0)
        self.std_ = np.std(X, axis=0)
        self.std_[self.std_ == 0] = 1  # 防止方差为0
        self.is_fitted = True
        return self

    def transform(self, X: np.ndarray) -> np.ndarray:
        if not self.is_fitted:
            raise RuntimeError("Scaler must be fitted first")
        return (X - self.mean_) / self.std_

    def fit_transform(self, X: np.ndarray) -> np.ndarray:
        return self.fit(X).transform(X)

    def inverse_transform(self, X: np.ndarray) -> np.ndarray:
        return X * self.std_ + self.mean_
```

### 4. 评估模块

```python
# ml/evaluation/metrics.py
import numpy as np
from typing import Literal

def accuracy(y_true: np.ndarray, y_pred: np.ndarray) -> float:
    """准确率"""
    return np.mean(y_true == y_pred)

def precision(y_true: np.ndarray, y_pred: np.ndarray, average: str = 'binary') -> float:
    """精确率"""
    # 二分类
    tp = np.sum((y_true == 1) & (y_pred == 1))
    fp = np.sum((y_true == 0) & (y_pred == 1))
    return tp / (tp + fp) if (tp + fp) > 0 else 0

def recall(y_true: np.ndarray, y_pred: np.ndarray, average: str = 'binary') -> float:
    """召回率"""
    tp = np.sum((y_true == 1) & (y_pred == 1))
    fn = np.sum((y_true == 1) & (y_pred == 0))
    return tp / (tp + fn) if (tp + fn) > 0 else 0

def f1_score(y_true: np.ndarray, y_pred: np.ndarray, average: str = 'binary') -> float:
    """F1 分数"""
    p = precision(y_true, y_pred, average)
    r = recall(y_true, y_pred, average)
    return 2 * p * r / (p + r) if (p + r) > 0 else 0

def mean_squared_error(y_true: np.ndarray, y_pred: np.ndarray) -> float:
    """均方误差（MSE）"""
    return np.mean((y_true - y_pred) ** 2)

def root_mean_squared_error(y_true: np.ndarray, y_pred: np.ndarray) -> float:
    """均方根误差（RMSE）"""
    return np.sqrt(mean_squared_error(y_true, y_pred))

def r2_score(y_true: np.ndarray, y_pred: np.ndarray) -> float:
    """R² 分数（决定系数）"""
    ss_res = np.sum((y_true - y_pred) ** 2)
    ss_tot = np.sum((y_true - np.mean(y_true)) ** 2)
    return 1 - (ss_res / ss_tot) if ss_tot > 0 else 0
```

### 5. 实验管理

```python
# ml/training/experiment.py
import json
import yaml
from pathlib import Path
from datetime import datetime
from typing import Dict, Any, Optional

class ExperimentManager:
    """实验管理：配置、结果保存、确保可复现性"""

    def __init__(self, experiment_dir: str = "ml/experiments"):
        self.experiment_dir = Path(experiment_dir)
        self.experiment_dir.mkdir(parents=True, exist_ok=True)

    def create_experiment(
        self,
        name: str,
        algorithm: str,
        parameters: Dict[str, Any],
        dataset_id: str,
        random_seed: int = 42
    ) -> str:
        """创建新实验"""
        experiment_id = f"{datetime.now().strftime('%Y%m%d_%H%M%S')}_{name}"

        config = {
            'experiment_id': experiment_id,
            'name': name,
            'algorithm': algorithm,
            'parameters': parameters,
            'dataset_id': dataset_id,
            'random_seed': random_seed,
            'created_at': datetime.now().isoformat(),
            'status': 'pending',
        }

        config_path = self.experiment_dir / "configs" / f"{experiment_id}.yaml"
        config_path.parent.mkdir(parents=True, exist_ok=True)
        with open(config_path, 'w') as f:
            yaml.dump(config, f)

        return experiment_id

    def save_result(
        self,
        experiment_id: str,
        result: Dict[str, Any],
        metrics: Dict[str, float],
        duration_seconds: float
    ):
        """保存实验结果"""
        result_data = {
            'experiment_id': experiment_id,
            'result': result,
            'metrics': metrics,
            'duration_seconds': duration_seconds,
            'completed_at': datetime.now().isoformat(),
        }

        result_path = self.experiment_dir / "results" / f"{experiment_id}.json"
        result_path.parent.mkdir(parents=True, exist_ok=True)
        with open(result_path, 'w') as f:
            json.dump(result_data, f, indent=2)

    def load_config(self, experiment_id: str) -> Dict[str, Any]:
        """加载实验配置"""
        config_path = self.experiment_dir / "configs" / f"{experiment_id}.yaml"
        with open(config_path, 'r') as f:
            return yaml.safe_load(f)

    def load_result(self, experiment_id: str) -> Dict[str, Any]:
        """加载实验结果"""
        result_path = self.experiment_dir / "results" / f"{experiment_id}.json"
        with open(result_path, 'r') as f:
            return json.load(f)
```

### 6. 算法列表及教学顺序

#### 6.1 基础（第1周）
| 算法 | 文件 | 教学要点 |
|------|------|----------|
| Linear Regression | `regression/linear_regression.py` | 最小二乘法、回归方程 |
| Logistic Regression | `classification/logistic_regression.py` | Sigmoid、二分类 |
| K-Nearest Neighbors | `classification/knn.py` | 基于距离、惰性学习 |

#### 6.2 中级（第2-3周）
| 算法 | 文件 | 教学要点 |
|------|------|----------|
| Decision Tree | `classification/decision_tree.py` | 信息增益、剪枝 |
| Random Forest | `classification/random_forest.py` | 集成、Bagging |
| SVM | `classification/svm.py` | 间隔、核技巧 |
| K-Means | `clustering/kmeans.py` | 迭代优化、肘部曲线 |
| DBSCAN | `clustering/dbscan.py` | 基于密度、噪声 |

#### 6.3 高级（第4-5周）
| 算法 | 文件 | 教学要点 |
|------|------|----------|
| PCA | `dimensionality/pca.py` | 特征值分解、方差保留 |
| t-SNE | `dimensionality/tsne.py` | 概率嵌入 |
| MLP | `neural/mlp.py` | 反向传播、激活函数 |
| CNN | `neural/cnn.py` | 卷积、池化 |
| RNN/LSTM | `neural/rnn.py` | 序列、长期依赖 |

### 7. 可复现性保证

```python
# ml/utils/seed.py
import numpy as np
import random
import torch

def set_seed(seed: int = 42):
    """固定所有随机种子"""
    random.seed(seed)
    np.random.seed(seed)
    torch.manual_seed(seed)
    if torch.cuda.is_available():
        torch.cuda.manual_seed(seed)
        torch.cuda.manual_seed_all(seed)
    torch.backends.cudnn.deterministic = True
    torch.backends.cudnn.benchmark = False
```

## 参考

- scikit-learn: https://scikit-learn.org
- PyTorch: https://pytorch.org
- "Hands-On Machine Learning" (Aurélien Géron)
