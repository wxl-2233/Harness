---
name: analytics-skill
description: "构建教育用数据挖掘平台的学习分析(Learning Analytics)系统。实现学生行为追踪、学习进度分析、概念掌握度建模、个性化推荐系统、学习仪表板。请求'学习分析'、'学生建模'、'推荐系统'、'进度分析'、'掌握度'、'学习仪表板'时使用此技能。也处理推荐算法改进、分析指标追加、仪表板定制。"
---

# 学习分析技能

## 概述

构建教育用数据挖掘平台的学习分析系统。追踪、理解并优化学生的学习过程，提供个性化学习体验。

## 工作流程

### 1. 项目结构设定

```
backend/app/analytics/
├── models/                  # 数据模型
│   ├── student_model.py     # 学生状态模型
│   ├── knowledge_state.py   # 知识状态模型
│   └── activity_log.py      # 活动日志模型
├── analysis/                # 分析逻辑
│   ├── progress.py          # 进度分析
│   ├── mastery.py           # 掌握度建模
│   ├── engagement.py        # 参与度分析
│   └── difficulty.py        # 难度分析
├── recommendations/         # 推荐系统
│   ├── engine.py            # 推荐引擎
│   ├── content_based.py     # 内容基础推荐
│   ├── collaborative.py     # 协同过滤
│   └── knowledge_based.py   # 知识基础推荐
├── dashboards/              # 仪表板数据
│   ├── student_dashboard.py # 学生仪表板
│   ├── teacher_dashboard.py # 教师仪表板
│   └── admin_dashboard.py   # 管理员仪表板
└── reports/                 # 报告生成
    ├── progress_report.py
    └── recommendation_report.py
```

### 2. 学生建模

#### 2.1 知识状态模型（Knowledge State）

```python
# backend/app/analytics/models/knowledge_state.py
from typing import Dict, List
from datetime import datetime
from enum import Enum

class ConceptMastery:
    """各概念掌握水平"""

    def __init__(self, concept_id: str):
        self.concept_id = concept_id
        self.mastery_level = 0.0  # 0.0 ~ 1.0
        self.attempt_count = 0
        self.correct_count = 0
        self.last_practice = None
        self.practice_history: List[Dict] = []

    def update(self, correct: bool, timestamp: datetime):
        """根据学习结果更新掌握度"""
        self.attempt_count += 1
        if correct:
            self.correct_count += 1

        # 指数加权移动平均更新掌握度
        alpha = 0.3  # 学习率
        self.mastery_level = alpha * (1.0 if correct else 0.0) + (1 - alpha) * self.mastery_level
        self.last_practice = timestamp

        self.practice_history.append({
            'timestamp': timestamp.isoformat(),
            'correct': correct,
            'mastery_after': self.mastery_level,
        })

    def get_accuracy(self) -> float:
        if self.attempt_count == 0:
            return 0.0
        return self.correct_count / self.attempt_count

    def to_dict(self) -> Dict:
        return {
            'concept_id': self.concept_id,
            'mastery_level': round(self.mastery_level, 3),
            'attempt_count': self.attempt_count,
            'correct_count': self.correct_count,
            'accuracy': round(self.get_accuracy(), 3),
            'last_practice': self.last_practice.isoformat() if self.last_practice else None,
        }


class StudentKnowledgeModel:
    """学生整体知识状态模型"""

    def __init__(self, student_id: str):
        self.student_id = student_id
        self.concepts: Dict[str, ConceptMastery] = {}

    def get_or_create_concept(self, concept_id: str) -> ConceptMastery:
        if concept_id not in self.concepts:
            self.concepts[concept_id] = ConceptMastery(concept_id)
        return self.concepts[concept_id]

    def update_concept(self, concept_id: str, correct: bool, timestamp: datetime):
        concept = self.get_or_create_concept(concept_id)
        concept.update(correct, timestamp)

    def get_weak_concepts(self, threshold: float = 0.6) -> List[str]:
        """掌握水平低于阈值的概念"""
        return [
            cid for cid, cm in self.concepts.items()
            if cm.mastery_level < threshold
        ]

    def get_strong_concepts(self, threshold: float = 0.8) -> List[str]:
        """掌握水平高于阈值的概念"""
        return [
            cid for cid, cm in self.concepts.items()
            if cm.mastery_level >= threshold
        ]

    def get_overall_mastery(self) -> float:
        """整体掌握度平均值"""
        if not self.concepts:
            return 0.0
        return sum(cm.mastery_level for cm in self.concepts.values()) / len(self.concepts)

    def to_dict(self) -> Dict:
        return {
            'student_id': self.student_id,
            'overall_mastery': round(self.get_overall_mastery(), 3),
            'concepts': {
                cid: cm.to_dict() for cid, cm in self.concepts.items()
            },
            'weak_concepts': self.get_weak_concepts(),
            'strong_concepts': self.get_strong_concepts(),
        }
```

#### 2.2 活动日志模型

记录学生的所有学习活动。

**活动类型（ActivityType）：**
- `login`, `logout`：登录/登出
- `dataset_view`, `dataset_upload`：数据集相关
- `experiment_start`, `experiment_complete`, `experiment_fail`：实验相关
- `quiz_start`, `quiz_submit`：测验相关
- `lesson_view`, `lesson_complete`：课程相关
- `video_play`, `video_pause`：视频相关
- `forum_post`, `forum_reply`：论坛相关

**记录项目：**
- `student_id`：学生 ID
- `activity_type`：活动类型
- `resource_id`：相关资源 ID（数据集、实验等）
- `duration_seconds`：停留时间
- `metadata_json`：追加元数据（正确与否、分数等）
- `timestamp`：活动时间

### 3. 分析逻辑

#### 3.1 进度分析

```python
# backend/app/analytics/analysis/progress.py
from typing import List, Dict
from datetime import datetime, timedelta
from collections import defaultdict

class ProgressAnalyzer:
    def __init__(self, activity_logs: List[Dict]):
        self.logs = activity_logs

    def daily_activity(self, days: int = 30) -> List[Dict]:
        """每日活动量"""
        today = datetime.utcnow().date()
        daily_counts = defaultdict(int)

        for log in self.logs:
            log_date = datetime.fromisoformat(log['timestamp']).date()
            if (today - log_date).days <= days:
                daily_counts[log_date.isoformat()] += 1

        return [
            {'date': date, 'count': count}
            for date, count in sorted(daily_counts.items())
        ]

    def weekly_progress(self, weeks: int = 12) -> List[Dict]:
        """每周学习进度"""
        today = datetime.utcnow().date()
        weekly_data = defaultdict(lambda: {
            'experiments': 0,
            'quizzes': 0,
            'lessons': 0,
            'total_time': 0,
        })

        for log in self.logs:
            log_date = datetime.fromisoformat(log['timestamp']).date()
            if (today - log_date).days > weeks * 7:
                continue

            week_key = log_date.isocalendar()[1]  # 周数
            activity_type = log['activity_type']

            if activity_type == 'experiment_complete':
                weekly_data[week_key]['experiments'] += 1
            elif activity_type == 'quiz_submit':
                weekly_data[week_key]['quizzes'] += 1
            elif activity_type == 'lesson_complete':
                weekly_data[week_key]['lessons'] += 1

            if log.get('duration_seconds'):
                weekly_data[week_key]['total_time'] += log['duration_seconds']

        return [
            {'week': week, **data}
            for week, data in sorted(weekly_data.items())
        ]

    def completion_rate(self) -> Dict[str, float]:
        """完成率分析"""
        total_started = sum(1 for log in self.logs if log['activity_type'] in ['experiment_start', 'lesson_view'])
        total_completed = sum(1 for log in self.logs if log['activity_type'] in ['experiment_complete', 'lesson_complete'])

        if total_started == 0:
            return {'overall': 0.0}

        return {
            'overall': total_completed / total_started,
            'total_started': total_started,
            'total_completed': total_completed,
        }
```

### 3.2 掌握度建模（BKT）

使用贝叶斯知识追踪(BKT)算法估算学生各概念的掌握水平。

**参数：**
- P(Learn)：一次尝试中学会的概率（默认 0.3）
- P(Guess)：不会但猜对的概率（默认 0.25）
- P(Slip)：会但做错的概率（默认 0.1）
- P(Transit)：转移概率（默认 0.1）

**算法流程：**
1. 设定先验概率 P(L)（默认 0.1）
2. 基于观察（正确/错误）计算后验概率
3. 应用转移概率预测下一状态
4. 重复

> 详细实现：参考 `references/bkt-implementation.md`

#### 3.3 参与度分析

```python
# backend/app/analytics/analysis/engagement.py
from typing import List, Dict
from datetime import datetime, timedelta

class EngagementAnalyzer:
    def __init__(self, activity_logs: List[Dict]):
        self.logs = activity_logs

    def engagement_score(self) -> float:
        """
        参与度分数（0-100）

        要素：
        - 活动频率（30%）
        - 停留时间（30%）
        - 完成率（20%）
        - 多样性（20%）
        """
        if not self.logs:
            return 0.0

        # 活动频率：最近7天的活动天数
        recent_logs = [
            log for log in self.logs
            if (datetime.utcnow() - datetime.fromisoformat(log['timestamp'])).days <= 7
        ]
        active_days = len(set(
            datetime.fromisoformat(log['timestamp']).date()
            for log in recent_logs
        ))
        frequency_score = min(active_days / 7, 1.0) * 100

        # 停留时间
        total_time = sum(log.get('duration_seconds', 0) for log in self.logs)
        time_score = min(total_time / 3600, 1.0) * 100  # 以1小时为基准

        # 完成率
        started = sum(1 for log in self.logs if log['activity_type'] in ['experiment_start', 'lesson_view'])
        completed = sum(1 for log in self.logs if log['activity_type'] in ['experiment_complete', 'lesson_complete'])
        completion_score = (completed / started * 100) if started > 0 else 0

        # 多样性：多种活动类型
        activity_types = set(log['activity_type'] for log in self.logs)
        diversity_score = min(len(activity_types) / 5, 1.0) * 100

        # 加权平均
        engagement = (
            frequency_score * 0.3 +
            time_score * 0.3 +
            completion_score * 0.2 +
            diversity_score * 0.2
        )

        return round(engagement, 1)

    def at_risk_students(self, threshold: float = 40.0) -> List[str]:
        """识别风险学生（参与度低于阈值）"""
        # 实际实现中按学生分组计算
        if self.engagement_score() < threshold:
            return ["at_risk"]
        return []
```

### 4. 推荐系统

#### 4.1 内容基础推荐

基于学生掌握状态推荐下一学习概念。

**推荐策略：**
1. **前置概念确认**：仅选择已掌握前置条件的概念作为候选
2. **学习需求度**：优先选择掌握度 0.5-0.7 之间的概念（最近发展区）
3. **难度适合性**：选择适合学生水平的难度

**分数计算：**
```
score = need_score * 0.6 + difficulty_score * 0.4
```

**输出格式：**
```json
{
  "concept_id": "random_forest",
  "score": 0.85,
  "current_mastery": 0.4,
  "reason": "已掌握 decision_tree，已准备好学习 random_forest。"
}
```

#### 4.2 间隔重复（Spaced Repetition）

基于遗忘曲线计算最佳复习时间。使用 SM-2 算法变体。

**核心概念：**
- 学习成功时：增加复习间隔（1天 → 6天 → 15天 → ...）
- 学习失败时：重置间隔（回到1天）
- Ease factor：根据学生学习速度自动调整

> 详细实现：参考 `references/spaced-repetition.md`

### 5. 仪表板数据

#### 5.1 学生仪表板

```python
# backend/app/analytics/dashboards/student_dashboard.py
from typing import Dict, Any
from ..analysis.progress import ProgressAnalyzer
from ..analysis.mastery import MasteryAnalyzer
from ..analysis.engagement import EngagementAnalyzer
from ..recommendations.content_based import ContentBasedRecommender
from ..recommendations.spaced_repetition import SpacedRepetitionScheduler

class StudentDashboard:
    def __init__(self, student_id: str, activity_logs: List[Dict], knowledge_state: Dict):
        self.student_id = student_id
        self.logs = activity_logs
        self.knowledge = knowledge_state

    def generate(self) -> Dict[str, Any]:
        """生成学生仪表板数据"""
        progress = ProgressAnalyzer(self.logs)
        engagement = EngagementAnalyzer(self.logs)
        mastery = MasteryAnalyzer()
        recommender = ContentBasedRecommender(self._get_concepts())
        scheduler = SpacedRepetitionScheduler()

        # 学生掌握状态
        mastery_results = mastery.analyze_student(self.knowledge)

        # 推荐
        recommendations = recommender.recommend(
            {cid: m['mastery'] for cid, m in mastery_results.items()}
        )

        # 需要复习的概念
        due_concepts = scheduler.get_due_concepts(self.knowledge.get('concepts', {}))

        return {
            'student_id': self.student_id,
            'overview': {
                'engagement_score': engagement.engagement_score(),
                'overall_mastery': self.knowledge.get('overall_mastery', 0),
                'total_experiments': sum(1 for log in self.logs if log['activity_type'] == 'experiment_complete'),
                'completion_rate': progress.completion_rate()['overall'],
            },
            'activity': {
                'daily': progress.daily_activity(days=30),
                'weekly': progress.weekly_progress(weeks=12),
            },
            'mastery': mastery_results,
            'weak_concepts': [cid for cid, m in mastery_results.items() if m['mastery'] < 0.6],
            'strong_concepts': [cid for cid, m in mastery_results.items() if m['mastery'] >= 0.8],
            'recommendations': recommendations,
            'due_for_review': due_concepts,
        }

    def _get_concepts(self) -> List[Dict]:
        """概念元数据（实际实现中从 DB 加载）"""
        return [
            {'id': 'linear_regression', 'prerequisites': [], 'difficulty': 0.3},
            {'id': 'logistic_regression', 'prerequisites': ['linear_regression'], 'difficulty': 0.4},
            {'id': 'decision_tree', 'prerequisites': [], 'difficulty': 0.4},
            {'id': 'random_forest', 'prerequisites': ['decision_tree'], 'difficulty': 0.6},
            {'id': 'kmeans', 'prerequisites': [], 'difficulty': 0.5},
            {'id': 'pca', 'prerequisites': ['linear_algebra_basics'], 'difficulty': 0.7},
        ]
```

### 6. 伦理数据使用

**隐私保护原则：**
1. **最小收集**：仅收集教育所需的数据
2. **匿名化**：分析时去除个人标识信息
3. **透明性**：明确告知学生收集哪些数据
4. **控制权**：提供学生查看、删除自己数据的权限
5. **安全性**：数据加密存储、访问控制

**数据使用指南：**
- ✅ 改善学习体验
- ✅ 个性化推荐
- ✅ 教育效果分析
- ❌ 学生评价/自动评分（仅作为教师辅助）
- ❌ 第三方出售/共享
- ❌ 歧视性目的使用

## 参考

- "Educational Data Mining and Learning Analytics" (Baker & Inventado)
- Bayesian Knowledge Tracing: Corbett & Anderson (1994)
- Spaced Repetition: SM-2 Algorithm (Piotr Wozniak)
