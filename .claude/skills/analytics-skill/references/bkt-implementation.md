# 贝叶斯知识追踪（BKT）实现

## 概述

BKT 是一种概率化建模学生知识状态的算法。

## 参数

```python
class BayesianKnowledgeTracing:
    def __init__(
        self,
        p_learn: float = 0.3,      # 学习概率
        p_guess: float = 0.25,     # 猜测概率（不会但猜对的概率）
        p_slip: float = 0.1,       # 失误概率（会但做错的概率）
        p_transit: float = 0.1,    # 转移概率（一次尝试中学会的概率）
    ):
        self.p_learn = p_learn
        self.p_guess = p_guess
        self.p_slip = p_slip
        self.p_transit = p_transit
```

## 更新算法

```python
def update(self, prior: float, correct: bool) -> float:
    """计算后验概率"""
    if correct:
        p_correct_given_learned = 1 - self.p_slip
        p_correct_given_not_learned = self.p_guess
    else:
        p_correct_given_learned = self.p_slip
        p_correct_given_not_learned = 1 - self.p_guess

    p_correct = (
        p_correct_given_learned * prior +
        p_correct_given_not_learned * (1 - prior)
    )

    if p_correct == 0:
        return prior

    posterior = p_correct_given_learned * prior / p_correct

    # 应用转移概率
    posterior = posterior + (1 - posterior) * self.p_transit

    return min(posterior, 1.0)
```

## 使用示例

```python
bkt = BayesianKnowledgeTracing()

# 学生的学习历史
history = [True, False, True, True, True]  # 正确/错误

# 估算掌握度
mastery = bkt.estimate_mastery(history)
print(f"Mastery: {mastery:.3f}")  # 例: 0.742
```

## 数学背景

BKT 使用贝叶斯更新：

**后验概率：**
```
P(L_n | obs) = P(obs | L_n) * P(L_n) / P(obs)
```

**全概率：**
```
P(obs) = P(obs | L_n) * P(L_n) + P(obs | ¬L_n) * P(¬L_n)
```

**转移：**
```
P(L_{n+1}) = P(L_n | obs) + (1 - P(L_n | obs)) * P(T)
```

## 参数调优指南

| 参数 | 默认值 | 含义 | 调整基准 |
|------|--------|------|----------|
| P(Learn) | 0.3 | 一次尝试中学会的概率 | 根据课程难度调整 |
| P(Guess) | 0.25 | 不会但猜对的概率 | 选择题: 1/选项数, 主观题: 0.1 |
| P(Slip) | 0.1 | 会但做错的概率 | 失误可能性（笔误、计算错误） |
| P(Transit) | 0.1 | 转移概率 | 学习速度 |
