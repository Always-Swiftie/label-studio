# Label Studio 源码学习指南（5天计划）

> 作为资深程序员给后端开发实习生的建议：如何系统地学习 Label Studio 这个数据标注平台项目

## 目录

1. [项目概述](#项目概述)
2. [5天学习计划](#5天学习计划)
3. [核心架构解析](#核心架构解析)
4. [关键模块详解](#关键模块详解)
5. [开发环境搭建](#开发环境搭建)
6. [代码导航技巧](#代码导航技巧)
7. [实战练习](#实战练习)
8. [学习资源](#学习资源)

---

## 项目概述

### Label Studio 是什么？

Label Studio 是一个开源的数据标注工具，支持多种数据类型（图像、文本、音频、视频、时间序列等）的标注。它可以用于准备训练数据或改进现有的机器学习模型训练数据。

### 技术栈

**后端：**
- **Python 3.10+**
- **Django 5.1+** - Web框架
- **Django REST Framework** - API框架
- **PostgreSQL/SQLite** - 数据库
- **Redis** - 缓存和任务队列
- **RQ (Redis Queue)** - 后台任务处理

**前端：**
- **React** - UI框架
- **MobX State Tree** - 状态管理
- **TypeScript/JavaScript**
- **Nx** - 单体仓库工具

### 项目规模

- 约 **42,000+ 行** Python 代码（不含测试和迁移）
- 主要模块：20+ 个独立的 Django 应用
- API 端点：100+ 个 RESTful 接口

---

## 5天学习计划

### 第1天：项目结构与环境搭建

**目标：** 了解项目整体结构，搭建开发环境，运行起来

#### 上午（3-4小时）

1. **克隆并浏览项目结构**
   ```bash
   git clone https://github.com/HumanSignal/label-studio.git
   cd label-studio
   ```

2. **阅读核心文档**
   - `README.md` - 项目介绍
   - `CONTRIBUTING.md` - 贡献指南
   - `docs/source/guide/` - 用户文档

3. **理解目录结构**
   ```
   label-studio/
   ├── label_studio/          # 主应用代码
   │   ├── core/             # 核心功能（配置、权限、中间件）
   │   ├── projects/         # 项目管理
   │   ├── tasks/            # 任务管理
   │   ├── users/            # 用户管理
   │   ├── organizations/    # 组织管理
   │   ├── data_import/      # 数据导入
   │   ├── data_export/      # 数据导出
   │   ├── data_manager/     # 数据管理器
   │   ├── ml/               # 机器学习集成
   │   ├── io_storages/      # 存储后端（S3, GCS等）
   │   └── ...
   ├── web/                  # 前端代码
   │   ├── apps/labelstudio/ # 主应用
   │   ├── libs/editor/      # 标注编辑器
   │   └── libs/datamanager/ # 数据管理器UI
   ├── docs/                 # 文档
   └── deploy/               # 部署配置
   ```

#### 下午（3-4小时）

4. **搭建开发环境**
   ```bash
   # 安装依赖
   pip install poetry
   poetry install
   
   # 运行数据库迁移
   python label_studio/manage.py migrate
   
   # 收集静态文件
   python label_studio/manage.py collectstatic --noinput
   
   # 创建超级用户
   python label_studio/manage.py createsuperuser
   
   # 启动开发服务器
   python label_studio/manage.py runserver
   ```

5. **体验功能**
   - 访问 http://localhost:8080
   - 创建账号并登录
   - 创建一个测试项目
   - 导入一些测试数据
   - 进行简单的标注操作

6. **阅读配置文件**
   - `pyproject.toml` - 项目依赖
   - `label_studio/core/settings/label_studio.py` - Django配置
   - `.env.development` - 环境变量示例

**第1天总结：**
- 记录项目的目录结构图
- 列出主要的 Django 应用及其作用
- 理解项目的运行流程
- 记录你在搭建环境时遇到的问题

---

### 第2天：核心模块与数据模型

**目标：** 深入理解核心数据模型和模块关系

#### 上午（3-4小时）

1. **学习核心数据模型**

   按以下顺序阅读 `models.py` 文件：

   a. **用户和组织** (`organizations/models.py`, `users/models.py`)
   - `Organization` - 组织
   - `User` - 用户（扩展 Django User）
   - 权限和角色管理

   b. **项目和任务** (`projects/models.py`, `tasks/models.py`)
   - `Project` - 项目（标注配置、成员）
   - `Task` - 任务（待标注的数据）
   - `Annotation` - 标注结果
   - `Prediction` - 预测结果（来自ML模型）

   **关键关系：**
   ```
   Organization 
     └── Project
           ├── Task
           │    ├── Annotation
           │    └── Prediction
           └── Member (User)
   ```

2. **使用Django Shell探索数据**
   ```bash
   python label_studio/manage.py shell
   ```

   ```python
   # 在shell中执行
   from projects.models import Project
   from tasks.models import Task, Annotation
   from users.models import User
   
   # 查看所有项目
   Project.objects.all()
   
   # 查看某个项目的任务
   project = Project.objects.first()
   project.tasks.all()
   
   # 查看任务的标注
   task = Task.objects.first()
   task.annotations.all()
   ```

#### 下午（3-4小时）

3. **学习标签配置（Label Config）**

   标签配置是 Label Studio 的核心，它定义了标注界面。

   - 阅读 `label_studio/core/label_config.py`
   - 理解如何解析 XML 配置
   - 查看 `label_studio/annotation_templates/` 中的模板示例

4. **数据流分析**

   跟踪一个完整的标注流程：
   ```
   1. 用户创建项目 → projects/api.py (ProjectViewSet)
   2. 配置标签 → 保存到 Project.label_config
   3. 导入数据 → data_import/api.py
   4. 创建任务 → tasks/models.py (Task)
   5. 用户标注 → tasks/api.py (AnnotationViewSet)
   6. 保存标注 → tasks/models.py (Annotation)
   7. 导出结果 → data_export/api.py
   ```

5. **理解 Django 信号**
   - `projects/signals.py` - 项目创建/更新时的钩子
   - `tasks/models.py` - 任务状态变化时的信号
   - 信号用于触发异步任务、更新统计信息等

**第2天总结：**
- 画出核心数据模型的 ER 图
- 列出各个模型的关键字段和关系
- 理解标签配置的作用
- 记录一个完整的数据流向

---

### 第3天：API 层与业务逻辑

**目标：** 掌握 REST API 的实现和业务逻辑处理

#### 上午（3-4小时）

1. **Django REST Framework 基础**

   Label Studio 大量使用 DRF，需要理解：
   - `ViewSet` - 视图集
   - `Serializer` - 序列化器
   - `Permission` - 权限类
   - `Filter` - 过滤器

2. **核心 API 模块学习**

   按顺序阅读以下文件的 ViewSet 实现：

   a. **用户 API** (`users/api.py`)
   - 用户注册、登录、信息管理
   - 理解 JWT 认证 (`jwt_auth/`)

   b. **项目 API** (`projects/api.py`)
   - `ProjectViewSet` - CRUD 操作
   - 理解分页、过滤、排序

   c. **任务 API** (`tasks/api.py`)
   - `TaskViewSet` - 任务管理
   - `AnnotationViewSet` - 标注管理
   - 批量操作实现

3. **权限系统**
   - `core/permissions.py` - 基础权限类
   - `projects/permissions.py` - 项目级权限
   - 使用 `django-rules` 实现基于规则的权限

#### 下午（3-4小时）

4. **使用 API 测试工具**

   使用 Postman 或 curl 测试 API：

   ```bash
   # 获取 token
   curl -X POST http://localhost:8080/api/auth/login/ \
     -H "Content-Type: application/json" \
     -d '{"username":"admin","password":"password"}'
   
   # 创建项目
   curl -X POST http://localhost:8080/api/projects/ \
     -H "Authorization: Token YOUR_TOKEN" \
     -H "Content-Type: application/json" \
     -d '{
       "title": "Test Project",
       "label_config": "<View>...</View>"
     }'
   
   # 获取项目列表
   curl http://localhost:8080/api/projects/ \
     -H "Authorization: Token YOUR_TOKEN"
   ```

5. **序列化器深入**
   - `projects/serializers.py`
   - `tasks/serializers.py`
   - 理解序列化器如何转换数据
   - 学习自定义字段和验证

6. **过滤和搜索**
   - `data_manager/` - 数据管理器的过滤逻辑
   - 使用 `django-filter` 实现复杂过滤
   - 全文搜索实现

**第3天总结：**
- 列出主要的 API 端点
- 理解请求-响应流程
- 记录权限检查的逻辑
- 尝试创建一个自定义 API 端点

---

### 第4天：存储、导入导出与异步任务

**目标：** 理解数据持久化、文件处理和后台任务

#### 上午（3-4小时）

1. **存储后端 (Storage Backends)**

   Label Studio 支持多种存储：
   - 本地文件系统 (`io_storages/localfiles/`)
   - Amazon S3 (`io_storages/s3/`)
   - Google Cloud Storage (`io_storages/gcs/`)
   - Azure Blob (`io_storages/azure_blob/`)
   - Redis (`io_storages/redis/`)

   **学习重点：**
   - `io_storages/models.py` - 存储配置模型
   - `io_storages/api.py` - 存储管理 API
   - 如何实现新的存储后端（查看已有实现）

2. **数据导入流程**

   - `data_import/models.py` - FileUpload 模型
   - `data_import/api.py` - 导入 API
   - `data_import/uploader.py` - 文件上传处理

   **支持格式：**
   - JSON, CSV, TSV
   - 图像文件（自动创建任务）
   - 压缩包（自动解压）

3. **数据导出流程**

   - `data_export/models.py` - Export 模型
   - `data_export/api.py` - 导出 API
   - `data_export/serializers.py` - 导出格式

   **导出格式：**
   - JSON, JSON-MIN
   - CSV, TSV
   - COCO, YOLO（通过 label-studio-converter）

#### 下午（3-4小时）

4. **异步任务系统 (RQ - Redis Queue)**

   Label Studio 使用 RQ 处理耗时任务：

   - `core/redis.py` - Redis 连接配置
   - 查找使用 `@job` 装饰器的函数
   - 理解任务队列和 worker

   **常见异步任务：**
   - 批量导入数据
   - 数据导出
   - ML 模型预测
   - 项目统计计算

   ```bash
   # 启动 RQ worker（在新终端）
   python label_studio/manage.py rqworker default
   ```

5. **缓存策略**

   - 使用 Redis 缓存
   - Django 缓存框架
   - 查看 `@cached_property` 的使用

6. **文件处理与媒体服务**

   - `core/middleware.py` - 中间件处理
   - 静态文件和媒体文件的服务
   - 使用 Nginx 反向代理（生产环境）

**第4天总结：**
- 画出数据导入导出的流程图
- 列出所有异步任务
- 理解存储后端的抽象设计
- 尝试添加一个简单的后台任务

---

### 第5天：机器学习集成与高级特性

**目标：** 理解 ML 集成、高级功能和代码质量

#### 上午（3-4小时）

1. **机器学习后端集成**

   Label Studio 可以连接 ML 模型：

   - `ml/` - ML 后端管理
   - `ml_models/models.py` - MLBackend 模型
   - `ml/api.py` - ML 后端 API

   **功能：**
   - 预标注（Pre-labeling）
   - 在线学习（Active Learning）
   - 模型预测展示

   **了解 ML Backend SDK：**
   ```bash
   # 查看示例
   # https://github.com/HumanSignal/label-studio-ml-backend
   ```

2. **Feature Flags（特性开关）**

   - `core/feature_flags/` - 特性开关实现
   - 使用 LaunchDarkly 或本地配置
   - 如何在代码中使用：
     ```python
     from core.feature_flags import flag_set
     
     if flag_set('ff_enable_new_feature', user=request.user):
         # 新特性代码
     ```

3. **Webhooks（事件钩子）**

   - `webhooks/models.py` - Webhook 配置
   - `webhooks/api.py` - Webhook 管理
   - 事件触发机制

#### 下午（3-4小时）

4. **测试框架**

   理解如何测试：

   ```bash
   # 运行测试
   cd label_studio
   DJANGO_SETTINGS_MODULE=core.settings.label_studio pytest -vv
   ```

   - `*/tests/` - 各模块的测试
   - 使用 pytest + pytest-django
   - 使用 tavern 测试 API
   - Mock 和 Fixture

5. **代码质量工具**

   - `ruff` - 代码检查和格式化
   - `blue` - 代码风格
   - `mypy` - 类型检查
   - `pre-commit` - 提交前检查

   ```bash
   # 运行代码检查
   ruff check label_studio/
   
   # 自动修复
   ruff check --fix label_studio/
   ```

6. **性能优化点**

   - 数据库查询优化（使用 `select_related`, `prefetch_related`）
   - 批量操作 (`bulk_create`, `bulk_update`)
   - 缓存使用
   - 异步任务分离

7. **安全考虑**

   - CSRF 保护
   - XSS 防护（使用 `bleach`）
   - SQL 注入防护（Django ORM）
   - 文件上传安全
   - API 认证和授权

**第5天总结：**
- 理解完整的 ML 工作流
- 列出主要的特性开关
- 写一个简单的测试用例
- 总结学习心得和改进建议

---

## 核心架构解析

### 1. 整体架构

```
┌─────────────────────────────────────────────────────────┐
│                    前端 (React + MST)                    │
│  ┌─────────────┐  ┌──────────────┐  ┌──────────────┐   │
│  │ 标注编辑器   │  │ 数据管理器    │  │ 项目管理      │   │
│  └─────────────┘  └──────────────┘  └──────────────┘   │
└─────────────────────────────────────────────────────────┘
                           ↓ REST API
┌─────────────────────────────────────────────────────────┐
│              Django + Django REST Framework              │
│  ┌─────────────────────────────────────────────────┐   │
│  │  API 层 (ViewSets + Serializers + Permissions)  │   │
│  └─────────────────────────────────────────────────┘   │
│  ┌─────────────────────────────────────────────────┐   │
│  │  业务逻辑层 (Models + Managers + Services)      │   │
│  └─────────────────────────────────────────────────┘   │
│  ┌─────────────────────────────────────────────────┐   │
│  │  核心服务 (权限, 配置, 中间件, 信号)            │   │
│  └─────────────────────────────────────────────────┘   │
└─────────────────────────────────────────────────────────┘
                           ↓
┌─────────────┐  ┌─────────────┐  ┌─────────────────────┐
│ PostgreSQL/ │  │    Redis    │  │  文件存储 (S3/GCS)  │
│   SQLite    │  │ (Cache/RQ)  │  │  或本地文件系统     │
└─────────────┘  └─────────────┘  └─────────────────────┘
```

### 2. 请求处理流程

```
1. 用户请求 → Nginx (生产环境)
              ↓
2. Django Middleware
   - CSRF 检查
   - 认证检查 (JWT/Session)
   - CORS 处理
              ↓
3. URL 路由 (urls.py)
              ↓
4. ViewSet (DRF)
   - 权限检查 (permissions)
   - 反序列化输入 (serializers)
              ↓
5. 业务逻辑
   - Model 操作
   - 信号触发
   - 异步任务触发
              ↓
6. 序列化输出
              ↓
7. JSON 响应
```

### 3. 数据模型层次

```
Organization (组织)
  └── User (用户) - 多对多关系
  └── Project (项目)
        ├── label_config (标签配置 XML)
        ├── ProjectMember (项目成员)
        └── Task (任务)
              ├── data (任务数据 JSON)
              ├── Annotation (标注) - 多个用户可标注
              │     ├── result (标注结果 JSON)
              │     └── completed_by (标注者)
              └── Prediction (预测) - 来自 ML 模型
                    └── result (预测结果 JSON)
```

---

## 关键模块详解

### 1. Core（核心模块）

**位置：** `label_studio/core/`

**职责：**
- Django 配置 (`settings/`)
- 标签配置解析 (`label_config.py`)
- 权限系统 (`permissions.py`, `api_permissions.py`)
- 中间件 (`middleware.py`)
- 通用工具 (`utils/`)
- Feature Flags (`feature_flags/`)

**关键文件：**
- `label_config.py` - 解析和验证标签配置 XML
- `permissions.py` - 基于 django-rules 的权限检查
- `utils/common.py` - 通用辅助函数

### 2. Projects（项目管理）

**位置：** `label_studio/projects/`

**职责：**
- 项目 CRUD 操作
- 标签配置管理
- 项目成员管理
- 项目统计

**关键模型：**
- `Project` - 项目主模型
- `ProjectSummary` - 项目统计信息

**API：**
- `ProjectViewSet` - 项目操作 API
- 支持导入/导出、复制、归档

### 3. Tasks（任务管理）

**位置：** `label_studio/tasks/`

**职责：**
- 任务 CRUD
- 标注管理
- 预测管理
- 任务分配和队列

**关键模型：**
- `Task` - 任务（待标注数据）
- `Annotation` - 标注结果
- `AnnotationDraft` - 标注草稿
- `Prediction` - ML 模型预测

**API：**
- `TaskViewSet` - 任务操作
- `AnnotationViewSet` - 标注操作
- 批量操作支持

### 4. Data Manager（数据管理器）

**位置：** `label_studio/data_manager/`

**职责：**
- 高级数据过滤和搜索
- 数据可视化
- 批量操作

**功能：**
- 复杂的查询构建器
- 多条件过滤
- 排序和分页
- 列自定义

### 5. IO Storages（存储后端）

**位置：** `label_studio/io_storages/`

**支持的存储：**
- `s3/` - Amazon S3
- `gcs/` - Google Cloud Storage
- `azure_blob/` - Azure Blob Storage
- `localfiles/` - 本地文件系统
- `redis/` - Redis 存储

**模式：**
- 使用抽象基类定义接口
- 每个存储实现继承基类
- 支持同步和异步操作

### 6. ML Integration（机器学习集成）

**位置：** `label_studio/ml/`, `label_studio/ml_models/`

**功能：**
- 连接外部 ML 后端
- 获取预测结果
- 触发模型训练
- 模型版本管理

**工作流：**
```
1. 配置 ML Backend URL
2. Label Studio 发送任务数据
3. ML Backend 返回预测
4. 预测显示在标注界面
5. 用户修正并保存标注
6. 触发模型重训练（可选）
```

### 7. Organizations & Users（组织与用户）

**位置：** `label_studio/organizations/`, `label_studio/users/`

**功能：**
- 多租户支持
- 用户认证和授权
- 组织成员管理
- 角色和权限

**角色层次：**
- Owner - 组织所有者
- Administrator - 管理员
- Manager - 项目管理者
- Reviewer - 审核者
- Annotator - 标注者

---

## 开发环境搭建

### 完整开发环境

```bash
# 1. 克隆仓库
git clone https://github.com/HumanSignal/label-studio.git
cd label-studio

# 2. 安装 Python 依赖
pip install poetry
poetry install --with test,build

# 3. 配置环境变量
cp .env.development .env
# 编辑 .env 设置数据库等配置

# 4. 运行迁移
poetry run python label_studio/manage.py migrate

# 5. 收集静态文件
poetry run python label_studio/manage.py collectstatic --noinput

# 6. 创建超级用户
poetry run python label_studio/manage.py createsuperuser

# 7. 启动开发服务器
poetry run python label_studio/manage.py runserver

# 8. (可选) 启动 RQ worker
poetry run python label_studio/manage.py rqworker default

# 9. (可选) 前端开发
cd web
yarn install
yarn start
```

### 使用 Docker 开发

```bash
# 使用 Docker Compose
docker-compose up

# 或使用开发版 Dockerfile
docker build -f Dockerfile.development -t label-studio:dev .
docker run -it -p 8080:8080 -v $(pwd):/label-studio label-studio:dev
```

### 数据库配置

**使用 PostgreSQL（推荐生产环境）：**

```python
# 在 .env 中设置
DJANGO_DB=default
POSTGRES_NAME=labelstudio
POSTGRES_USER=postgres
POSTGRES_PASSWORD=password
POSTGRES_HOST=localhost
POSTGRES_PORT=5432
```

**使用 SQLite（开发环境）：**

```python
# 在 .env 中设置
DJANGO_DB=sqlite
```

### 常用管理命令

```bash
# 创建迁移
python label_studio/manage.py makemigrations

# 应用迁移
python label_studio/manage.py migrate

# Django Shell
python label_studio/manage.py shell

# 创建测试数据
python label_studio/manage.py init_test_data

# 运行测试
cd label_studio
pytest -vv

# 代码检查
ruff check label_studio/

# 格式化代码
ruff format label_studio/
```

---

## 代码导航技巧

### 1. 使用 IDE 工具

**推荐 IDE：**
- **PyCharm Professional** - 最佳 Django 支持
- **VS Code** - 使用 Python 和 Django 扩展
- **Vim/Neovim** - 配合 LSP 和 ctags

**必备插件/扩展：**
- Python/Django 语法高亮
- 代码跳转（Go to Definition）
- 自动补全
- 调试器
- Git 集成

### 2. 查找代码的策略

**寻找功能实现：**

```bash
# 搜索关键字
grep -r "function_name" label_studio/

# 搜索 API 端点
grep -r "api/projects" label_studio/

# 搜索模型定义
find label_studio -name "models.py" | xargs grep "class Project"

# 搜索 URL 配置
grep -r "ProjectViewSet" label_studio/*/urls.py
```

**理解数据流：**

1. 从 URL 开始 → `urls.py`
2. 找到对应的 ViewSet → `api.py`
3. 查看序列化器 → `serializers.py`
4. 理解模型 → `models.py`
5. 检查权限 → `permissions.py`

**调试技巧：**

```python
# 在代码中添加断点
import pdb; pdb.set_trace()

# 或使用 IPython
import IPython; IPython.embed()

# 打印调试信息
import logging
logger = logging.getLogger(__name__)
logger.debug(f"Variable value: {var}")
```

### 3. 理解 Django 约定

**文件命名约定：**
- `models.py` - 数据模型
- `api.py` 或 `views.py` - 视图/API
- `serializers.py` - DRF 序列化器
- `urls.py` - URL 路由
- `permissions.py` - 权限类
- `admin.py` - Django Admin 配置
- `signals.py` - Django 信号
- `apps.py` - App 配置
- `tests/` - 测试代码

**代码组织：**
- 每个 Django app 是一个功能模块
- App 之间通过导入相互引用
- 使用 `core/` 存放共享代码

### 4. 文档和注释

**在代码中查找：**
- Docstrings - 函数和类的文档字符串
- 行内注释 - `# 注释`
- TODO 标记 - `# TODO: ...`
- FIXME 标记 - `# FIXME: ...`

**外部文档：**
- 官方文档：https://labelstud.io/guide/
- API 文档：https://labelstud.io/api/
- GitHub Issues 和 Discussions

---

## 实战练习

### 练习1：创建一个自定义 API 端点

**目标：** 创建一个 API 端点返回项目的统计信息

```python
# 在 projects/api.py 中添加
from rest_framework.decorators import action
from rest_framework.response import Response

class ProjectViewSet(viewsets.ModelViewSet):
    # ... 现有代码 ...
    
    @action(detail=True, methods=['get'])
    def statistics(self, request, pk=None):
        """获取项目详细统计信息"""
        project = self.get_object()
        
        stats = {
            'total_tasks': project.tasks.count(),
            'completed_tasks': project.tasks.filter(
                annotations__isnull=False
            ).distinct().count(),
            'total_annotations': project.annotations.count(),
            'unique_annotators': project.annotations.values(
                'completed_by'
            ).distinct().count(),
        }
        
        return Response(stats)
```

**测试：**
```bash
curl http://localhost:8080/api/projects/1/statistics/ \
  -H "Authorization: Token YOUR_TOKEN"
```

### 练习2：添加自定义导出格式

**目标：** 创建一个自定义的 JSON 导出格式

```python
# 在 data_export/serializers.py 中添加
from data_export.serializers import ExportDataSerializer

class CustomJSONExportSerializer(ExportDataSerializer):
    def serialize_task(self, task):
        """自定义任务序列化"""
        return {
            'id': task.id,
            'data': task.data,
            'annotations': [
                {
                    'result': ann.result,
                    'user': ann.completed_by.email,
                    'created_at': ann.created_at.isoformat(),
                }
                for ann in task.annotations.all()
            ]
        }
```

### 练习3：创建自定义管理命令

**目标：** 创建一个命令来批量导入任务

```python
# projects/management/commands/import_tasks.py
from django.core.management.base import BaseCommand
from projects.models import Project
from tasks.models import Task
import json

class Command(BaseCommand):
    help = 'Batch import tasks from JSON file'
    
    def add_arguments(self, parser):
        parser.add_argument('project_id', type=int)
        parser.add_argument('json_file', type=str)
    
    def handle(self, *args, **options):
        project = Project.objects.get(id=options['project_id'])
        
        with open(options['json_file'], 'r') as f:
            tasks_data = json.load(f)
        
        tasks = [
            Task(project=project, data=task_data)
            for task_data in tasks_data
        ]
        
        Task.objects.bulk_create(tasks)
        
        self.stdout.write(
            self.style.SUCCESS(
                f'Successfully imported {len(tasks)} tasks'
            )
        )
```

**使用：**
```bash
python label_studio/manage.py import_tasks 1 tasks.json
```

### 练习4：添加自定义权限

**目标：** 创建只允许项目所有者删除项目的权限

```python
# projects/permissions.py
from rest_framework import permissions

class IsProjectOwner(permissions.BasePermission):
    """只有项目所有者可以删除项目"""
    
    def has_object_permission(self, request, view, obj):
        if request.method == 'DELETE':
            return obj.created_by == request.user
        return True

# 在 ProjectViewSet 中使用
class ProjectViewSet(viewsets.ModelViewSet):
    permission_classes = [IsProjectOwner]
```

### 练习5：实现简单的 Webhook

**目标：** 当标注完成时触发 Webhook

```python
# tasks/signals.py
from django.db.models.signals import post_save
from django.dispatch import receiver
from tasks.models import Annotation
import requests

@receiver(post_save, sender=Annotation)
def trigger_webhook_on_annotation(sender, instance, created, **kwargs):
    """标注保存时触发 webhook"""
    if created:
        webhook_url = instance.task.project.webhook_url
        
        if webhook_url:
            payload = {
                'event': 'annotation_created',
                'annotation_id': instance.id,
                'task_id': instance.task.id,
                'project_id': instance.task.project.id,
            }
            
            try:
                requests.post(webhook_url, json=payload, timeout=5)
            except Exception as e:
                logger.error(f"Webhook failed: {e}")
```

---

## 学习资源

### 官方资源

1. **Label Studio 文档**
   - https://labelstud.io/guide/
   - 完整的用户和开发者文档

2. **API 文档**
   - https://labelstud.io/api/
   - REST API 参考

3. **GitHub 仓库**
   - https://github.com/HumanSignal/label-studio
   - 源码、Issues、Discussions

4. **Label Studio SDK**
   - https://github.com/HumanSignal/label-studio-sdk
   - Python SDK 文档和示例

5. **ML Backend**
   - https://github.com/HumanSignal/label-studio-ml-backend
   - 机器学习集成示例

### 社区资源

1. **Slack 社区**
   - https://slack.labelstud.io/
   - 活跃的开发者社区

2. **YouTube 教程**
   - Label Studio 官方频道
   - 视频教程和演示

3. **博客文章**
   - Label Studio 官方博客
   - 用例和最佳实践

### 相关技术文档

1. **Django 文档**
   - https://docs.djangoproject.com/
   - Django 核心概念

2. **Django REST Framework**
   - https://www.django-rest-framework.org/
   - API 开发指南

3. **React 文档**
   - https://react.dev/
   - 前端开发（如需修改 UI）

4. **PostgreSQL 文档**
   - https://www.postgresql.org/docs/
   - 数据库优化

### 推荐书籍

1. **《Two Scoops of Django》**
   - Django 最佳实践

2. **《Django for APIs》**
   - RESTful API 开发

3. **《Designing Data-Intensive Applications》**
   - 数据密集型应用设计

---

## 学习心得和最佳实践

### 学习建议

1. **循序渐进**
   - 不要试图一次理解所有代码
   - 从核心流程开始，逐步扩展
   - 先使用再研究实现

2. **动手实践**
   - 运行代码，添加日志
   - 修改参数，观察结果
   - 创建测试用例验证理解

3. **绘制图表**
   - ER 图 - 数据模型关系
   - 流程图 - 业务逻辑流程
   - 架构图 - 系统组件关系

4. **记录笔记**
   - 记录关键概念
   - 记录代码位置
   - 记录问题和解决方案

5. **提问和讨论**
   - 在 Slack 社区提问
   - 查看 GitHub Issues
   - 与团队成员讨论

### 代码阅读技巧

1. **自顶向下**
   - 先理解整体架构
   - 再深入具体实现
   - 最后关注细节

2. **自底向上**
   - 从数据模型开始
   - 理解数据如何流动
   - 追踪到 API 和 UI

3. **调试驱动**
   - 设置断点
   - 单步执行
   - 观察变量值

4. **测试驱动**
   - 阅读测试用例
   - 理解预期行为
   - 运行测试验证

### 贡献代码前的准备

1. **阅读贡献指南**
   - `CONTRIBUTING.md`
   - 代码规范
   - 提交流程

2. **运行测试**
   ```bash
   cd label_studio
   pytest -vv
   ```

3. **代码检查**
   ```bash
   ruff check label_studio/
   ruff format label_studio/
   ```

4. **创建 PR**
   - 描述清楚改动
   - 添加测试用例
   - 关联相关 Issue

### 调试技巧

1. **Django Debug Toolbar**
   ```python
   # 在 settings.py 中启用
   INSTALLED_APPS += ['debug_toolbar']
   MIDDLEWARE += ['debug_toolbar.middleware.DebugToolbarMiddleware']
   ```

2. **日志配置**
   ```python
   import logging
   logging.basicConfig(level=logging.DEBUG)
   logger = logging.getLogger(__name__)
   ```

3. **Django Shell Plus**
   ```bash
   pip install django-extensions
   python label_studio/manage.py shell_plus
   ```

4. **性能分析**
   ```python
   from django.db import connection
   print(connection.queries)  # 查看执行的 SQL
   ```

---

## 常见问题 (FAQ)

### Q1: 如何添加新的数据类型支持？

**A:** 修改标签配置模板和前端编辑器：
1. 在 `label_studio/annotation_templates/` 添加新模板
2. 在前端 `web/libs/editor/` 实现新的标注工具
3. 更新序列化器处理新的数据格式

### Q2: 如何优化大数据集的性能？

**A:** 考虑以下优化：
1. 使用数据库索引
2. 批量操作（`bulk_create`, `bulk_update`）
3. 使用 `select_related` 和 `prefetch_related`
4. 启用查询缓存
5. 使用异步任务处理耗时操作
6. 分页加载数据

### Q3: 如何实现自定义存储后端？

**A:** 参考现有实现：
1. 继承 `io_storages/models.py` 中的基类
2. 实现 `scan_and_create_links()` 方法
3. 实现文件读取和写入方法
4. 注册新的存储类型

### Q4: 如何调试前端问题？

**A:**
1. 启动前端开发服务器：`cd web && yarn start`
2. 使用浏览器开发者工具
3. 查看 React DevTools
4. 检查网络请求
5. 查看控制台错误

### Q5: 如何处理数据库迁移冲突？

**A:**
```bash
# 查看迁移状态
python manage.py showmigrations

# 回滚到特定迁移
python manage.py migrate app_name migration_name

# 创建空迁移以解决冲突
python manage.py makemigrations --empty app_name
```

---

## 总结与下一步

### 5天学习总结

通过这5天的学习，你应该已经：

✅ 理解 Label Studio 的整体架构
✅ 熟悉核心数据模型和关系
✅ 掌握 API 的实现方式
✅ 了解数据导入导出流程
✅ 理解 ML 集成机制
✅ 能够阅读和理解大部分代码
✅ 知道如何调试和测试
✅ 了解贡献代码的流程

### 持续学习建议

1. **深入特定模块**
   - 根据公司需求，重点学习相关模块
   - 例如：存储后端、ML 集成、权限系统等

2. **参与开源贡献**
   - 修复小 bug
   - 改进文档
   - 添加新特性

3. **关注项目动态**
   - 订阅 GitHub 仓库
   - 关注新版本发布
   - 阅读 Release Notes

4. **构建自己的功能**
   - 基于 Label Studio 定制功能
   - 开发插件或扩展
   - 分享经验和代码

### 公司项目应用建议

1. **评估需求**
   - 列出公司的标注需求
   - 评估 Label Studio 的适配度
   - 规划定制开发内容

2. **搭建环境**
   - 生产环境部署（Docker + PostgreSQL + Redis）
   - CI/CD 流程
   - 监控和日志

3. **定制开发**
   - 自定义标注模板
   - 集成公司的 ML 模型
   - 定制导入导出格式
   - 添加自动化流程

4. **团队培训**
   - 标注人员培训
   - 开发人员培训
   - 文档和最佳实践

---

## 附录

### A. 常用命令速查表

```bash
# 开发环境
poetry install                          # 安装依赖
poetry run python manage.py runserver   # 启动服务
poetry run python manage.py shell       # Django Shell
poetry run pytest                       # 运行测试

# 数据库
python manage.py makemigrations         # 创建迁移
python manage.py migrate                # 应用迁移
python manage.py dbshell                # 数据库 Shell

# 代码质量
ruff check label_studio/               # 代码检查
ruff format label_studio/              # 格式化
pytest --cov                           # 测试覆盖率

# Docker
docker-compose up                      # 启动服务
docker-compose down                    # 停止服务
docker-compose logs -f                 # 查看日志
```

### B. 重要文件路径

```
核心配置：
- label_studio/core/settings/label_studio.py
- pyproject.toml
- .env

数据模型：
- label_studio/projects/models.py
- label_studio/tasks/models.py
- label_studio/users/models.py
- label_studio/organizations/models.py

API：
- label_studio/projects/api.py
- label_studio/tasks/api.py
- label_studio/data_import/api.py
- label_studio/data_export/api.py

前端：
- web/libs/editor/
- web/libs/datamanager/
- web/apps/labelstudio/
```

### C. 调试环境变量

```bash
# 开启 DEBUG 模式
export DEBUG=true

# 设置日志级别
export LOG_LEVEL=DEBUG

# Django 数据库查询日志
export DJANGO_LOG_SQL=1

# 禁用缓存（调试时）
export DISABLE_CACHE=1
```

---

**祝你学习顺利！如有问题，欢迎在 Slack 社区或 GitHub 提问。**

**Good luck with your Label Studio journey! 🚀**
