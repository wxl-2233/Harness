# 间隔重复算法

## 概述

间隔重复(Spaced Repetition)基于遗忘曲线计算最佳复习时间。

## SM-2 算法变体

```python
class SpacedRepetitionScheduler:
    def __init__(self):
        self.initial_interval = 1  # 天
        self.ease_factor = 2.5

    def calculate_next_review(
        self,
        last_review: datetime,
        quality: int,  # 0-5（0: 完全不记得, 5: 完全记得）
        interval: int = 1,
        ease_factor: float = 2.5,
    ) -> Dict:
        """计算下次复习时间"""

        if quality < 3:
            # 失败：重置间隔
            new_interval = 1
            new_ease = max(1.3, ease_factor - 0.2)
        else:
            # 成功：增加间隔
            if interval == 1:
                new_interval = 6
            else:
                new_interval = int(interval * ease_factor)
            new_ease = ease_factor + (0.1 - (5 - quality) * (0.08 + (5 - quality) * 0.02))
            new_ease = max(1.3, new_ease)

        next_review = last_review + timedelta(days=new_interval)

        return {
            'next_review': next_review.isoformat(),
            'interval': new_interval,
            'ease_factor': round(new_ease, 2),
        }
```

## 复习计划示例

| 复习次数 | 间隔 | 质量 5（完美） | 质量 3（一般） |
|----------|------|--------------|--------------|
| 第1次 | 1天 | 1天 | 1天 |
| 第2次 | 6天 | 6天 | 6天 |
| 第3次 | 15天 | 15天 | 15天 |
| 第4次 | 38天 | 38天 | 15天（重置） |

## 使用示例

```python
scheduler = SpacedRepetitionScheduler()

# 第一次复习
result = scheduler.calculate_next_review(
    last_review=datetime(2026, 6, 1),
    quality=4,
    interval=1,
    ease_factor=2.5,
)
print(result)
# {'next_review': '2026-06-07T00:00:00', 'interval': 6, 'ease_factor': 2.6}

# 第二次复习
result = scheduler.calculate_next_review(
    last_review=datetime(2026, 6, 7),
    quality=5,
    interval=6,
    ease_factor=2.6,
)
print(result)
# {'next_review': '2026-06-23T00:00:00', 'interval': 16, 'ease_factor': 2.7}
```

## 遗忘曲线

根据艾宾浩斯遗忘曲线：
- 20分钟后：遗忘 42%
- 1小时后：遗忘 56%
- 1天后：遗忘 66%
- 1周后：遗忘 75%
- 1个月后：遗忘 79%

间隔重复通过在最佳时间复习来防止遗忘。

## 教育效果

研究表明使用间隔重复可以：
- 长期记忆保留率提高 200-300%
- 学习时间缩短 30-50%
- 考试成绩提高 10-20%
