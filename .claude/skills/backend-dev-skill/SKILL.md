---
name: backend-dev-skill
description: "使用 FastAPI 构建教育用数据挖掘平台的后端。实现 RESTful API、WebSocket 实时通信、ML 实验编排、JWT 认证、PostgreSQL 数据库、Celery 异步任务。请求'后端'、'API 服务器'、'数据库'、'认证'、'实验执行引擎'时使用此技能。也处理 API 追加、Schema 变更、性能优化。"
---

# 后端开发技能

## 概述

使用 FastAPI(Python) 构建教育用数据挖掘平台的后端服务。负责 RESTful API、WebSocket 实时通信、ML 实验编排、用户认证/授权。

## 工作流程

### 1. 项目结构设定

```
backend/
├── app/
│   ├── main.py              # FastAPI 应用入口
│   ├── config.py            # 配置管理
│   ├── database.py          # DB 连接
│   ├── models/              # SQLAlchemy 模型
│   │   ├── user.py
│   │   ├── dataset.py
│   │   ├── experiment.py
│   │   └── analytics.py
│   ├── schemas/             # Pydantic Schema（请求/响应）
│   │   ├── user.py
│   │   ├── dataset.py
│   │   ├── experiment.py
│   │   └── analytics.py
│   ├── api/                 # API 路由
│   │   ├── v1/
│   │   │   ├── auth.py
│   │   │   ├── users.py
│   │   │   ├── datasets.py
│   │   │   ├── experiments.py
│   │   │   └── analytics.py
│   │   └── deps.py          # 依赖注入
│   ├── services/            # 业务逻辑
│   │   ├── auth_service.py
│   │   ├── dataset_service.py
│   │   ├── experiment_service.py
│   │   └── analytics_service.py
│   ├── workers/             # Celery 异步任务
│   │   ├── celery_app.py
│   │   ├── experiment_worker.py
│   │   └── data_processing_worker.py
│   ├── ws/                  # WebSocket 处理器
│   │   ├── experiment_ws.py
│   │   └── notification_ws.py
│   └── utils/               # 工具
│       ├── security.py      # JWT、密码哈希
│       ├── pagination.py
│       └── exceptions.py
├── alembic/                 # DB 迁移
│   └── versions/
├── tests/
├── requirements.txt
└── Dockerfile
```

### 2. 数据库模型

#### 2.1 用户（User）
```python
class User(Base):
    __tablename__ = "users"

    id = Column(UUID, primary_key=True, default=uuid4)
    email = Column(String, unique=True, index=True, nullable=False)
    hashed_password = Column(String, nullable=False)
    full_name = Column(String)
    role = Column(Enum("student", "teacher", "admin"), default="student")
    created_at = Column(DateTime, default=datetime.utcnow)
    last_login = Column(DateTime)

    # 关系
    experiments = relationship("Experiment", back_populates="user")
    datasets = relationship("Dataset", back_populates="user")
```

#### 2.2 数据集（Dataset）
```python
class Dataset(Base):
    __tablename__ = "datasets"

    id = Column(UUID, primary_key=True, default=uuid4)
    name = Column(String, nullable=False)
    description = Column(Text)
    file_path = Column(String, nullable=False)
    file_type = Column(Enum("csv", "json", "excel", "parquet"))
    row_count = Column(Integer)
    column_count = Column(Integer)
    metadata_json = Column(JSON)  # 列类型、统计等
    is_public = Column(Boolean, default=False)
    uploaded_by = Column(UUID, ForeignKey("users.id"))
    created_at = Column(DateTime, default=datetime.utcnow)

    # 关系
    uploader = relationship("User", back_populates="datasets")
```

#### 2.3 实验（Experiment）
```python
class Experiment(Base):
    __tablename__ = "experiments"

    id = Column(UUID, primary_key=True, default=uuid4)
    name = Column(String, nullable=False)
    algorithm = Column(String, nullable=False)  # "linear_regression", "random_forest" 等
    parameters = Column(JSON, nullable=False)    # 超参数
    dataset_id = Column(UUID, ForeignKey("datasets.id"))
    status = Column(Enum("pending", "running", "completed", "failed"), default="pending")
    progress = Column(Integer, default=0)        # 0-100
    result_json = Column(JSON)                   # 性能指标、预测结果
    error_message = Column(Text)
    started_at = Column(DateTime)
    completed_at = Column(DateTime)
    user_id = Column(UUID, ForeignKey("users.id"))
    created_at = Column(DateTime, default=datetime.utcnow)

    # 关系
    user = relationship("User", back_populates="experiments")
    dataset = relationship("Dataset")
```

#### 2.4 学习活动（LearningActivity）
```python
class LearningActivity(Base):
    __tablename__ = "learning_activities"

    id = Column(UUID, primary_key=True, default=uuid4)
    user_id = Column(UUID, ForeignKey("users.id"), nullable=False)
    activity_type = Column(Enum("login", "dataset_view", "experiment_run", "quiz", "lesson"))
    resource_id = Column(UUID)  # 相关资源 ID
    duration_seconds = Column(Integer)
    metadata_json = Column(JSON)  # 追加信息
    created_at = Column(DateTime, default=datetime.utcnow)
```

### 3. API 端点

#### 3.1 认证（Auth）
```
POST /api/v1/auth/register    # 注册
POST /api/v1/auth/login       # 登录（签发 JWT）
POST /api/v1/auth/refresh     # 刷新令牌
POST /api/v1/auth/logout      # 登出
```

#### 3.2 用户（Users）
```
GET    /api/v1/users/me       # 我的信息
PUT    /api/v1/users/me       # 修改我的信息
GET    /api/v1/users/{id}     # 用户信息（仅教师/管理员）
```

#### 3.3 数据集（Datasets）
```
GET    /api/v1/datasets              # 数据集列表（搜索、过滤、排序）
POST   /api/v1/datasets              # 上传数据集
GET    /api/v1/datasets/{id}         # 数据集详情
GET    /api/v1/datasets/{id}/preview # 数据预览（前100行）
GET    /api/v1/datasets/{id}/stats   # 统计摘要
DELETE /api/v1/datasets/{id}         # 删除数据集
```

#### 3.4 实验（Experiments）
```
GET    /api/v1/experiments              # 实验列表
POST   /api/v1/experiments              # 创建实验（异步启动）
GET    /api/v1/experiments/{id}         # 实验详情
GET    /api/v1/experiments/{id}/status  # 实验状态（轮询用）
POST   /api/v1/experiments/{id}/cancel  # 取消实验
GET    /api/v1/experiments/{id}/result  # 实验结果
POST   /api/v1/experiments/compare      # 比较多个实验
```

#### 3.5 分析（Analytics）
```
GET /api/v1/analytics/dashboard          # 仪表板数据
GET /api/v1/analytics/progress           # 学习进度
GET /api/v1/analytics/recommendations    # 推荐学习项
GET /api/v1/analytics/activities         # 活动历史
```

### 4. 认证及授权

```python
# app/utils/security.py
from passlib.context import CryptContext
from jose import JWTError, jwt
from datetime import datetime, timedelta

SECRET_KEY = "your-secret-key"
ALGORITHM = "HS256"
ACCESS_TOKEN_EXPIRE_MINUTES = 30

pwd_context = CryptContext(schemes=["bcrypt"])

def create_access_token(data: dict):
    to_encode = data.copy()
    expire = datetime.utcnow() + timedelta(minutes=ACCESS_TOKEN_EXPIRE_MINUTES)
    to_encode.update({"exp": expire})
    return jwt.encode(to_encode, SECRET_KEY, algorithm=ALGORITHM)

def verify_password(plain_password, hashed_password):
    return pwd_context.verify(plain_password, hashed_password)
```

```python
# app/api/deps.py
from fastapi import Depends, HTTPException, status
from fastapi.security import OAuth2PasswordBearer

oauth2_scheme = OAuth2PasswordBearer(tokenUrl="api/v1/auth/login")

async def get_current_user(
    token: str = Depends(oauth2_scheme),
    db: Session = Depends(get_db)
):
    credentials_exception = HTTPException(
        status_code=status.HTTP_401_UNAUTHORIZED,
        detail="Could not validate credentials",
    )
    try:
        payload = jwt.decode(token, SECRET_KEY, algorithms=[ALGORITHM])
        user_id: str = payload.get("sub")
        if user_id is None:
            raise credentials_exception
    except JWTError:
        raise credentials_exception

    user = db.query(User).filter(User.id == user_id).first()
    if user is None:
        raise credentials_exception
    return user
```

### 5. 异步任务（Celery）

```python
# app/workers/celery_app.py
from celery import Celery

celery_app = Celery(
    "worker",
    broker="redis://localhost:6379/0",
    backend="redis://localhost:6379/0"
)

celery_app.conf.update(
    task_serializer="json",
    accept_content=["json"],
    result_serializer="json",
    timezone="UTC",
    enable_utc=True,
    task_track_started=True,
    task_time_limit=3600,  # 1小时
    worker_prefetch_multiplier=1,
)
```

```python
# app/workers/experiment_worker.py
from app.workers.celery_app import celery_app
from app.services.ml_service import run_experiment

@celery_app.task(bind=True, name="run_experiment")
def run_experiment_task(self, experiment_id: str):
    try:
        # 进度更新回调
        def update_progress(progress):
            self.update_state(
                state="PROGRESS",
                meta={"progress": progress}
            )

        result = run_experiment(experiment_id, progress_callback=update_progress)
        return {"status": "completed", "result": result}
    except Exception as e:
        return {"status": "failed", "error": str(e)}
```

### 6. WebSocket 实时通信

```python
# app/ws/experiment_ws.py
from fastapi import WebSocket, WebSocketDisconnect
from app.services.experiment_service import get_experiment_status
import asyncio
import json

class ExperimentConnectionManager:
    def __init__(self):
        self.active_connections: dict[str, list[WebSocket]] = {}

    async def connect(self, websocket: WebSocket, experiment_id: str):
        await websocket.accept()
        if experiment_id not in self.active_connections:
            self.active_connections[experiment_id] = []
        self.active_connections[experiment_id].append(websocket)

    def disconnect(self, websocket: WebSocket, experiment_id: str):
        self.active_connections[experiment_id].remove(websocket)

    async def broadcast_progress(self, experiment_id: str, progress: int):
        if experiment_id in self.active_connections:
            message = json.dumps({"type": "progress", "progress": progress})
            for connection in self.active_connections[experiment_id]:
                await connection.send_text(message)

manager = ExperimentConnectionManager()

# WebSocket 端点
@router.websocket("/ws/experiments/{experiment_id}")
async def experiment_websocket(websocket: WebSocket, experiment_id: str):
    await manager.connect(websocket, experiment_id)
    try:
        while True:
            # 接收客户端消息（心跳等）
            data = await websocket.receive_text()
            # 需要时响应
    except WebSocketDisconnect:
        manager.disconnect(websocket, experiment_id)
```

### 7. 错误处理

```python
# app/utils/exceptions.py
from fastapi import HTTPException, status

class NotFoundException(HTTPException):
    def __init__(self, detail: str = "Resource not found"):
        super().__init__(status_code=status.HTTP_404_NOT_FOUND, detail=detail)

class ForbiddenException(HTTPException):
    def __init__(self, detail: str = "Forbidden"):
        super().__init__(status_code=status.HTTP_403_FORBIDDEN, detail=detail)

class ExperimentFailedException(HTTPException):
    def __init__(self, detail: str = "Experiment failed"):
        super().__init__(status_code=status.HTTP_500_INTERNAL_SERVER_ERROR, detail=detail)
```

### 8. API 文档（OpenAPI）

```python
# app/main.py
from fastapi import FastAPI
from fastapi.openapi.docs import get_swagger_ui_html

app = FastAPI(
    title="Educational Data Mining Platform API",
    description="面向学生的教育用数据挖掘平台 API",
    version="1.0.0",
    docs_url="/api/docs",
    redoc_url="/api/redoc",
)
```

## 性能优化

**数据库：**
- 索引：频繁搜索的列（email, created_at, status）
- 查询优化：使用 `joinedload` 防止 N+1 问题
- 连接池：`pool_size=20`, `max_overflow=10`

**API：**
- 响应缓存：使用 Redis 缓存频繁查询的数据
- 分页：默认20条，最多100条
- 异步 I/O：使用 `async/await` 处理并发请求

**异步任务：**
- Celery Worker：4个进程
- 任务超时：1小时
- 重试策略：最多3次，指数退避

## 参考

- FastAPI 官方文档: https://fastapi.tiangolo.com
- SQLAlchemy: https://docs.sqlalchemy.org
- Celery: https://docs.celeryq.dev
- Alembic: https://alembic.sqlalchemy.org
