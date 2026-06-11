# ML 算法实现

## 线性回归（Linear Regression）

### 数学公式
```
y = Xw + b
w = (X^T X)^(-1) X^T y
```

### 实现
```python
class LinearRegression(BaseAlgorithm):
    def __init__(self, fit_intercept: bool = True, **kwargs):
        super().__init__(fit_intercept=fit_intercept, **kwargs)
        self.weights = None
        self.bias = None

    def fit(self, X: np.ndarray, y: np.ndarray, **kwargs) -> 'LinearRegression':
        X = np.asarray(X)
        y = np.asarray(y)

        if self.hyperparameters['fit_intercept']:
            X_bias = np.c_[np.ones(X.shape[0]), X]
        else:
            X_bias = X

        try:
            weights = np.linalg.inv(X_bias.T @ X_bias) @ X_bias.T @ y
        except np.linalg.LinAlgError:
            weights = np.linalg.lstsq(X_bias, y, rcond=None)[0]

        if self.hyperparameters['fit_intercept']:
            self.bias = weights[0]
            self.weights = weights[1:]
        else:
            self.bias = 0
            self.weights = weights

        self.is_fitted = True
        return self

    def predict(self, X: np.ndarray) -> np.ndarray:
        if not self.is_fitted:
            raise RuntimeError("Model must be fitted before prediction")
        return X @ self.weights + self.bias
```

---

## K-Means 聚类

### 算法
1. 随机初始化 K 个中心点
2. 将各数据分配到最近的中心点
3. 将中心点更新为已分配数据的均值
4. 重复步骤 2-3 直到收敛

### 实现
```python
class KMeans(BaseAlgorithm):
    def __init__(self, n_clusters: int = 3, max_iter: int = 300, **kwargs):
        super().__init__(n_clusters=n_clusters, max_iter=max_iter, **kwargs)
        self.centroids = None
        self.labels_ = None

    def fit(self, X: np.ndarray, y: np.ndarray = None, **kwargs) -> 'KMeans':
        X = np.asarray(X)
        n_samples = X.shape[0]

        # 初始中心：随机选择
        indices = np.random.choice(n_samples, self.hyperparameters['n_clusters'], replace=False)
        self.centroids = X[indices].copy()

        self.training_history = []

        for iteration in range(self.hyperparameters['max_iter']):
            # 1. 将各数据分配到最近的中心
            distances = self._compute_distances(X, self.centroids)
            self.labels_ = np.argmin(distances, axis=1)

            # 2. 更新中心
            new_centroids = self._update_centroids(X, self.labels_)

            # 3. 检查收敛
            centroid_shift = np.linalg.norm(new_centroids - self.centroids)
            self.training_history.append({
                'iteration': iteration,
                'centroids': self.centroids.tolist(),
                'labels': self.labels_.tolist(),
                'inertia': self._compute_inertia(X, self.labels_),
            })

            self.centroids = new_centroids

            if centroid_shift < 1e-4:
                break

        self.is_fitted = True
        return self
```

---

## 决策树（Decision Tree）

### 分裂标准：基尼不纯度
```
Gini(S) = 1 - Σ p_i²
```

### 信息增益
```
Gain(S, A) = Gini(S) - Σ (|S_v| / |S|) * Gini(S_v)
```

### 实现要点
- 限制最大深度（防止过拟合）
- 限制最小样本数
- 剪枝（pruning）
- 返回树结构用于可视化

---

## 随机森林（Random Forest）

### 集成策略
- Bagging：自举采样
- 特征随机选择：每次分裂使用 √d 个特征
- 投票(Voting)：多数表决决定最终预测

### 超参数
- `n_estimators`：树的数量（默认 100）
- `max_depth`：最大深度
- `max_features`：每棵树使用的特征数
- `min_samples_split`：分裂所需的最小样本数

---

## PCA（主成分分析）

### 算法
1. 数据标准化
2. 计算协方差矩阵
3. 特征值分解
4. 按特征值从大到小选择主成分
5. 投影到主成分

### 实现
```python
class PCA(BaseAlgorithm):
    def __init__(self, n_components: int = 2, **kwargs):
        super().__init__(n_components=n_components, **kwargs)
        self.components_ = None
        self.explained_variance_ = None

    def fit(self, X: np.ndarray, y: np.ndarray = None, **kwargs) -> 'PCA':
        # 1. 标准化
        X_centered = X - np.mean(X, axis=0)

        # 2. 协方差矩阵
        cov_matrix = np.cov(X_centered.T)

        # 3. 特征值分解
        eigenvalues, eigenvectors = np.linalg.eigh(cov_matrix)

        # 4. 从大到小排序
        idx = np.argsort(eigenvalues)[::-1]
        eigenvalues = eigenvalues[idx]
        eigenvectors = eigenvectors[:, idx]

        # 5. 选择前 n_components 个
        self.components_ = eigenvectors[:, :self.hyperparameters['n_components']]
        self.explained_variance_ = eigenvalues[:self.hyperparameters['n_components']]

        self.is_fitted = True
        return self

    def transform(self, X: np.ndarray) -> np.ndarray:
        X_centered = X - self.mean_
        return X_centered @ self.components_
```

---

## 神经网络（MLP）

### 前向传播
```
z^(l) = W^(l) a^(l-1) + b^(l)
a^(l) = σ(z^(l))
```

### 反向传播
```
δ^(L) = ∇_a C ⊙ σ'(z^(L))
δ^(l) = (W^(l+1))^T δ^(l+1) ⊙ σ'(z^(l))
∇_w C = δ^(l) (a^(l-1))^T
∇_b C = δ^(l)
```

### 激活函数
- **ReLU**: f(x) = max(0, x)
- **Sigmoid**: f(x) = 1 / (1 + e^(-x))
- **Tanh**: f(x) = tanh(x)
- **Softmax**: f(x_i) = e^(x_i) / Σ e^(x_j)
