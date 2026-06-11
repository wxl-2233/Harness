# 测试示例及模板

## 单元测试模板（React）

```typescript
import { render, screen, fireEvent } from '@testing-library/react';
import { describe, it, expect } from 'vitest';
import { ComponentName } from '@/components/ComponentName';

describe('ComponentName', () => {
  it('renders correctly', () => {
    render(<ComponentName />);
    expect(screen.getByText('Expected Text')).toBeInTheDocument();
  });

  it('handles user interaction', () => {
    render(<ComponentName />);
    const button = screen.getByRole('button', { name: /click me/i });
    fireEvent.click(button);
    // 验证
  });

  it('displays error state', () => {
    render(<ComponentName error="Test error" />);
    expect(screen.getByText('Test error')).toBeInTheDocument();
  });
});
```

## 单元测试模板（Python）

```python
import pytest
from unittest.mock import Mock, patch
from app.services.service_name import ServiceName

@pytest.fixture
def service():
    db = Mock()
    return ServiceName(db)

def test_method_success(service):
    # Arrange
    service.db.query.return_value.filter.return_value.first.return_value = Mock()

    # Act
    result = service.method_name("param")

    # Assert
    assert result is not None
    service.db.commit.assert_called_once()

def test_method_not_found(service):
    service.db.query.return_value.filter.return_value.first.return_value = None

    with pytest.raises(NotFoundException):
        service.method_name("invalid-id")
```

## 集成测试模板（API）

```typescript
import request from 'supertest';
import { app } from '@/app';

describe('API Endpoint', () => {
  describe('POST /api/v1/resource', () => {
    it('creates resource successfully', async () => {
      const response = await request(app)
        .post('/api/v1/resource')
        .send({ name: 'Test', value: 123 })
        .set('Authorization', 'Bearer test-token')
        .expect(201);

      expect(response.body).toHaveProperty('id');
      expect(response.body.name).toBe('Test');
    });

    it('validates required fields', async () => {
      await request(app)
        .post('/api/v1/resource')
        .send({}) // 缺失字段
        .set('Authorization', 'Bearer test-token')
        .expect(422);
    });
  });
});
```

## E2E 测试模板（Playwright）

```typescript
import { test, expect } from '@playwright/test';

test.describe('Feature Name', () => {
  test.beforeEach(async ({ page }) => {
    // 登录等前置操作
    await page.goto('/login');
    await page.fill('input[name="email"]', 'test@example.com');
    await page.fill('input[name="password"]', 'password');
    await page.click('button[type="submit"]');
  });

  test('completes user journey', async ({ page }) => {
    await page.goto('/feature');
    await page.click('button:has-text("Start")');
    await page.waitForSelector('[data-testid="result"]');
    await expect(page.locator('[data-testid="result"]')).toContainText('Success');
  });
});
```

## 性能测试模板（k6）

```javascript
import http from 'k6/http';
import { check, sleep } from 'k6';

export const options = {
  stages: [
    { duration: '1m', target: 50 },
    { duration: '3m', target: 50 },
    { duration: '1m', target: 0 },
  ],
  thresholds: {
    http_req_duration: ['p(95)<500'],
    http_req_failed: ['rate<0.01'],
  },
};

export default function () {
  const res = http.get('http://localhost:8000/api/v1/endpoint', {
    headers: { 'Authorization': 'Bearer token' },
  });

  check(res, {
    'status is 200': (r) => r.status === 200,
    'response time < 200ms': (r) => r.timings.duration < 200,
  });

  sleep(1);
}
```

## 无障碍测试检查清单

- [ ] 所有图片有 `alt` 属性
- [ ] 颜色对比度 4.5:1 以上
- [ ] 支持键盘导航
- [ ] 兼容屏幕阅读器
- [ ] 表单字段关联 `label`
- [ ] 错误消息使用 `aria-describedby`
- [ ] 使用实时区域（`aria-live`）

## 缺陷报告模板

```markdown
## Bug Report

**严重级别:** 🔴 Critical / 🟠 High / 🟡 Medium / 🟢 Low
**发现日期:** YYYY-MM-DD
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

### 日志
```
[相关日志]
```
```
