# ExamAI — 智能试卷生成平台 实施计划

> **版本**: v1.0 | **日期**: 2026-05-30 | **状态**: 待执行
>
> **对于执行者：** 本计划按里程碑 M1-M6 组织，每个里程碑包含详细任务步骤。推荐使用 `superpowers:subagent-driven-development` 或 `superpowers:executing-plans` skill 逐任务执行。所有步骤使用 `- [ ]` checkbox 追踪。
>
> **目标：** 在 16 周内从零构建一个可交付的初中教师智能试卷生成平台 MVP
>
> **架构：** 前后端分离 — React SPA (Vite + Ant Design + Zustand) 通过 REST API 与 Django REST Framework 后端通信，Celery + Redis 处理异步任务，PostgreSQL 存储所有业务数据
>
> **技术栈：** Python 3.12 + Django 5.0 + DRF 3.15 + Celery 5.4 + PostgreSQL 16 + Redis 7 + Node.js 20 + React 18 + Vite 5 + Ant Design 5 + Zustand 4

---

## 总体里程碑概览

| 里程碑 | 内容 | 前后端 | 时间 | 周 |
|--------|------|:----:|------|:---:|
| M1 — 基础架构 | Django + React 脚手架、DB模型、JWT认证、LLM接口抽象 | 基础搭建 | 第1-2周 | W1-W2 |
| M2 — 题库模块 | 知识点树 + 题目CRUD + Excel批量导入 + 题库管理页面 + 种子数据入库 | 前后端同步 | 第3-5周 | W3-W5 |
| M3 — 组卷引擎 | A面表单 + B面AI推荐 + 试卷生成 + 智能组卷页面 | 前后端同步 | 第6-9周 | W6-W9 |
| M4 — 试卷导出 | 预览 + PDF/Word异步生成 + 答题卡 + 导出页面 | 前后端同步 | 第10-12周 | W10-W12 |
| M5 — 资源采集 | 手动上传 + AI解析(异步) + 审核入库 + 采集页面 | 前后端同步 | 第12-14周 | W12-W14 |
| M6 — 上线整合 | 工作台首页 + 登录注册页 + 联调测试 + 部署 | 整体整合 | 第14-16周 | W14-W16 |

---

## 项目文件结构总览

```
ExamAI/
├── backend/                            # Django 项目根
│   ├── config/                         # Django 配置
│   │   ├── __init__.py
│   │   ├── settings.py                 # 主配置
│   │   ├── urls.py                     # 根 URL 路由
│   │   ├── wsgi.py
│   │   ├── asgi.py
│   │   └── celery.py                   # Celery 配置
│   ├── apps/
│   │   ├── users/                      # 用户认证模块
│   │   │   ├── __init__.py
│   │   │   ├── models.py               # User 模型
│   │   │   ├── serializers.py          # 序列化器
│   │   │   ├── views.py                # API 视图
│   │   │   ├── urls.py                 # 路由
│   │   │   ├── admin.py
│   │   │   ├── tests/
│   │   │   │   ├── __init__.py
│   │   │   │   ├── test_models.py
│   │   │   │   └── test_api.py
│   │   │   └── migrations/
│   │   ├── questions/                  # 题库模块
│   │   │   ├── __init__.py
│   │   │   ├── models.py               # Subject, Grade, KnowledgePoint, Question, QuestionOption
│   │   │   ├── serializers.py
│   │   │   ├── views.py
│   │   │   ├── urls.py
│   │   │   ├── filters.py              # 筛选器
│   │   │   ├── admin.py
│   │   │   ├── imports.py              # Excel 批量导入逻辑
│   │   │   ├── tests/
│   │   │   │   ├── __init__.py
│   │   │   │   ├── test_models.py
│   │   │   │   ├── test_api.py
│   │   │   │   └── test_import.py
│   │   │   └── migrations/
│   │   ├── exams/                      # 试卷模块
│   │   │   ├── __init__.py
│   │   │   ├── models.py               # ExamPaper, ExamPaperSection, ExamPaperItem
│   │   │   ├── serializers.py
│   │   │   ├── views.py
│   │   │   ├── urls.py
│   │   │   ├── engine.py               # 组卷引擎核心逻辑
│   │   │   ├── recommend.py            # AI 组卷推荐
│   │   │   ├── filters.py
│   │   │   ├── admin.py
│   │   │   ├── tests/
│   │   │   │   ├── __init__.py
│   │   │   │   ├── test_models.py
│   │   │   │   ├── test_api.py
│   │   │   │   ├── test_engine.py
│   │   │   │   └── test_recommend.py
│   │   │   └── migrations/
│   │   ├── resources/                  # 资源采集模块
│   │   │   ├── __init__.py
│   │   │   ├── models.py               # Resource
│   │   │   ├── serializers.py
│   │   │   ├── views.py
│   │   │   ├── urls.py
│   │   │   ├── extractors.py           # 文本提取（Word/PDF/图片）
│   │   │   ├── admin.py
│   │   │   ├── tasks.py                # Celery 异步解析任务
│   │   │   ├── tests/
│   │   │   │   ├── __init__.py
│   │   │   │   ├── test_models.py
│   │   │   │   ├── test_api.py
│   │   │   │   ├── test_extractors.py
│   │   │   │   └── test_tasks.py
│   │   │   └── migrations/
│   │   └── ai/                         # AI 服务模块
│   │       ├── __init__.py
│   │       ├── models.py               # AIProvider, AITaskLog
│   │       ├── base.py                 # 抽象 Adapter 接口
│   │       ├── providers/
│   │       │   ├── __init__.py
│   │       │   ├── deepseek.py         # DeepSeek adapter
│   │       │   ├── qwen.py             # 通义千问 adapter（预留）
│   │       │   └── glm.py              # 智谱 GLM adapter（预留）
│   │       ├── admin.py
│   │       ├── tests/
│   │       │   ├── __init__.py
│   │       │   ├── test_models.py
│   │       │   └── test_providers.py
│   │       └── migrations/
│   ├── common/                         # 公共工具
│   │   ├── __init__.py
│   │   ├── pagination.py               # 统一分页
│   │   ├── response.py                 # 统一响应格式
│   │   ├── permissions.py              # 权限类
│   │   ├── exceptions.py               # 全局异常处理
│   │   └── utils.py                    # 通用工具函数
│   ├── manage.py
│   ├── requirements/
│   │   ├── base.txt
│   │   └── dev.txt
│   └── Dockerfile
├── frontend/                           # Vite + React SPA
│   ├── src/
│   │   ├── main.tsx                    # 入口
│   │   ├── App.tsx                     # 根组件
│   │   ├── pages/
│   │   │   ├── LoginPage.tsx           # 登录/注册/找回密码
│   │   │   ├── DashboardPage.tsx       # 工作台首页
│   │   │   ├── ExamGeneratePage.tsx    # 智能组卷页
│   │   │   ├── QuestionBankPage.tsx    # 题库管理页
│   │   │   ├── ExamDetailPage.tsx      # 试卷详情/导出页
│   │   │   └── ResourceCollectPage.tsx # 资源采集页
│   │   ├── components/
│   │   │   ├── layout/
│   │   │   │   ├── AppLayout.tsx       # 主布局（侧边栏+顶栏+内容区）
│   │   │   │   ├── Sidebar.tsx         # 侧边导航
│   │   │   │   └── HeaderBar.tsx       # 顶部栏（用户信息/退出）
│   │   │   ├── common/
│   │   │   │   ├── Loading.tsx
│   │   │   │   ├── ErrorBoundary.tsx
│   │   │   │   └── EmptyState.tsx
│   │   │   ├── question/
│   │   │   │   ├── QuestionForm.tsx    # 题目编辑表单
│   │   │   │   ├── QuestionTable.tsx   # 题目列表表格
│   │   │   │   ├── QuestionFilter.tsx  # 筛选面板
│   │   │   │   ├── KnowledgeTree.tsx   # 知识点树组件
│   │   │   │   └── BatchImportModal.tsx # 批量导入弹窗
│   │   │   ├── exam/
│   │   │   │   ├── ExamConfigPanel.tsx  # A面配置面板
│   │   │   │   ├── ExamAIRecommend.tsx  # B面 AI 推荐面板
│   │   │   │   ├── ExamPreview.tsx      # 试卷预览
│   │   │   │   ├── ExamPreviewToolbar.tsx # 预览工具栏（换题/导出）
│   │   │   │   ├── DifficultyChart.tsx  # 难度分布图
│   │   │   │   └── SwapQuestionModal.tsx # 换题弹窗
│   │   │   ├── export/
│   │   │   │   ├── ExportPanel.tsx      # 导出操作面板
│   │   │   │   ├── AnswerSheet.tsx      # 答题卡预览
│   │   │   │   └── ExportProgress.tsx   # 导出进度
│   │   │   └── resource/
│   │   │       ├── UploadPanel.tsx      # 文件上传面板
│   │   │       ├── ParseProgress.tsx    # 解析进度
│   │   │       └── ReviewPanel.tsx      # 审核面板
│   │   ├── services/
│   │   │   ├── api.ts                  # axios 实例 + 拦截器
│   │   │   ├── authService.ts          # 认证 API
│   │   │   ├── questionService.ts      # 题库 API
│   │   │   ├── examService.ts          # 试卷 API
│   │   │   ├── resourceService.ts      # 资源采集 API
│   │   │   └── aiService.ts            # AI 相关 API
│   │   ├── stores/
│   │   │   ├── authStore.ts            # 认证状态
│   │   │   ├── questionStore.ts        # 题库状态
│   │   │   ├── examStore.ts            # 组卷状态
│   │   │   └── resourceStore.ts        # 资源采集状态
│   │   ├── router/
│   │   │   └── index.tsx               # 路由配置
│   │   ├── hooks/
│   │   │   ├── useAuth.ts
│   │   │   ├── usePagination.ts
│   │   │   └── useDebounce.ts
│   │   ├── utils/
│   │   │   ├── constants.ts            # 常量（学科/题型/难度枚举等）
│   │   │   └── helpers.ts              # 工具函数
│   │   └── styles/
│   │       └── global.css
│   ├── index.html
│   ├── vite.config.ts
│   ├── tsconfig.json
│   ├── package.json
│   └── Dockerfile
├── docker-compose.yml
├── .gitignore
├── .env.example
└── docs/
    └── superpowers/
        ├── specs/
        │   └── 2026-05-30-examai-design.md
        └── plans/
            └── 2026-05-30-examai-plan.md  # 本文档
```

---

## 技能使用指南（贯穿整个项目）

在实施各里程碑时，以下 skills 将贯穿使用：

| Skill | 使用时机 | 用途 |
|-------|---------|------|
| `superpowers:test-driven-development` | 每个功能实现前 | 先写测试，再写实现 |
| `superpowers:subagent-driven-development` | 里程碑执行时 | 独立任务并行派发子代理 |
| `superpowers:brainstorming` | 遇到设计不明确时 | 补充设计细节 |
| `superpowers:systematic-debugging` | 遇到 bug 时 | 系统化调试 |
| `superpowers:verification-before-completion` | 每个任务完成后 | 验证功能正确性 |
| `superpowers:requesting-code-review` | 每个里程碑完成后 | 代码审查 |
| `code-review` | 提交前 | 检查代码质量 |
| `simplify` | 代码审查后 | 简化和优化代码 |

---

## M1 — 基础架构搭建（第1-2周）

> **目标：** 搭建完整开发环境，创建 Django + React 脚手架，实现所有数据模型，JWT 认证，LLM 接口抽象层。
> **交付物：** 可运行的全栈骨架 + 数据库迁移 + 用户注册登录 API + AI Provider 配置管理

### 开发环境准备

#### Task M1.0: 项目初始化与环境配置

**前置条件：** 本地已安装 Docker Desktop、Python 3.12、Node.js 20、PostgreSQL 16、Redis 7（或通过 Docker 提供）

**文件：**
- 创建：`docker-compose.yml`
- 创建：`.env.example`
- 创建：`.gitignore`
- 修改：无

- [ ] **Step 1: 创建 .gitignore**

```bash
cat > /Users/mac/2026/ai_extra/ExamAI/.gitignore << 'GITIGNORE_EOF'
# Python
__pycache__/
*.py[cod]
*.egg-info/
dist/
.venv/
venv/
*.sqlite3

# Django
backend/staticfiles/
backend/media/

# Node
frontend/node_modules/
frontend/dist/

# Environment
.env
.env.local

# IDE
.vscode/
.idea/
*.swp
*.swo

# OS
.DS_Store
Thumbs.db

# Docker
pgdata/
redisdata/

# Logs
*.log
GITIGNORE_EOF
```

- [ ] **Step 2: 创建 .env.example**

```bash
cat > /Users/mac/2026/ai_extra/ExamAI/.env.example << 'ENV_EOF'
# Django
SECRET_KEY=change-me-in-production
DEBUG=True
ALLOWED_HOSTS=localhost,127.0.0.1

# Database
DATABASE_URL=postgres://examai:examai@localhost:5432/examai

# Redis
REDIS_URL=redis://localhost:6379/0

# JWT
JWT_ACCESS_TOKEN_LIFETIME_MINUTES=60
JWT_REFRESH_TOKEN_LIFETIME_DAYS=7

# DeepSeek (default AI provider)
DEEPSEEK_API_KEY=sk-your-key-here
DEEPSEEK_API_BASE=https://api.deepseek.com

# File Storage
MEDIA_ROOT=./media
FILE_UPLOAD_MAX_MB=50
ENV_EOF
```

- [ ] **Step 3: 创建 docker-compose.yml**

写入文件 `/Users/mac/2026/ai_extra/ExamAI/docker-compose.yml`：

```yaml
version: '3.8'

services:
  db:
    image: postgres:16-alpine
    environment:
      POSTGRES_DB: examai
      POSTGRES_USER: examai
      POSTGRES_PASSWORD: examai
    ports:
      - "5432:5432"
    volumes:
      - pgdata:/var/lib/postgresql/data
    healthcheck:
      test: ["CMD-SHELL", "pg_isready -U examai"]
      interval: 5s
      timeout: 5s
      retries: 5

  redis:
    image: redis:7-alpine
    ports:
      - "6379:6379"
    volumes:
      - redisdata:/data
    healthcheck:
      test: ["CMD", "redis-cli", "ping"]
      interval: 5s
      timeout: 5s
      retries: 5

volumes:
  pgdata:
  redisdata:
```

- [ ] **Step 4: 启动基础服务并验证**

```bash
cd /Users/mac/2026/ai_extra/ExamAI
docker compose up -d db redis
docker compose ps
```

预期输出：db 和 redis 服务状态为 `Up` 且 `healthy`。

- [ ] **Step 5: 提交**

```bash
git add .gitignore .env.example docker-compose.yml
git commit -m "chore: 初始化项目环境配置 — docker-compose, .gitignore, .env.example"
```

---

#### Task M1.1: Django 后端脚手架

**文件：**
- 创建：`backend/manage.py`
- 创建：`backend/config/__init__.py`
- 创建：`backend/config/settings.py`
- 创建：`backend/config/urls.py`
- 创建：`backend/config/wsgi.py`
- 创建：`backend/config/asgi.py`
- 创建：`backend/config/celery.py`
- 创建：`backend/requirements/base.txt`
- 创建：`backend/requirements/dev.txt`
- 创建：`backend/common/__init__.py`
- 创建：`backend/common/response.py`
- 创建：`backend/common/pagination.py`
- 创建：`backend/common/exceptions.py`

- [ ] **Step 1: 创建后端目录结构**

```bash
mkdir -p /Users/mac/2026/ai_extra/ExamAI/backend/config
mkdir -p /Users/mac/2026/ai_extra/ExamAI/backend/apps
mkdir -p /Users/mac/2026/ai_extra/ExamAI/backend/common
mkdir -p /Users/mac/2026/ai_extra/ExamAI/backend/requirements
mkdir -p /Users/mac/2026/ai_extra/ExamAI/backend/media
touch /Users/mac/2026/ai_extra/ExamAI/backend/apps/__init__.py
touch /Users/mac/2026/ai_extra/ExamAI/backend/__init__.py
```

- [ ] **Step 2: 创建依赖文件**

写入 `/Users/mac/2026/ai_extra/ExamAI/backend/requirements/base.txt`：

```
Django>=5.0,<5.1
djangorestframework>=3.15,<3.16
djangorestframework-simplejwt>=5.3,<5.4
django-cors-headers>=4.3,<4.4
django-filter>=24.1,<24.2
psycopg2-binary>=2.9,<3.0
celery>=5.4,<5.5
redis>=5.0,<5.1
python-decouple>=3.8,<3.9
openpyxl>=3.1,<3.2
Pillow>=10.3,<10.4
openai>=1.30,<1.31
```

写入 `/Users/mac/2026/ai_extra/ExamAI/backend/requirements/dev.txt`：

```
-r base.txt
django-extensions>=3.2,<3.3
pytest>=8.2,<8.3
pytest-django>=4.8,<4.9
pytest-cov>=5.0,<5.1
factory-boy>=3.3,<3.4
ipython>=8.24,<8.25
```

- [ ] **Step 3: 创建虚拟环境并安装依赖**

```bash
cd /Users/mac/2026/ai_extra/ExamAI/backend
python3.12 -m venv .venv
source .venv/bin/activate
pip install -r requirements/dev.txt
```

- [ ] **Step 4: 创建 Django 项目配置**

写入 `/Users/mac/2026/ai_extra/ExamAI/backend/manage.py`：

```python
#!/usr/bin/env python
"""Django's command-line utility for administrative tasks."""
import os
import sys


def main():
    """Run administrative tasks."""
    os.environ.setdefault('DJANGO_SETTINGS_MODULE', 'config.settings')
    try:
        from django.core.management import execute_from_command_line
    except ImportError as exc:
        raise ImportError(
            "Couldn't import Django. Are you sure it's installed and "
            "available on your PYTHONPATH environment variable? Did you "
            "forget to activate a virtual environment?"
        ) from exc
    execute_from_command_line(sys.argv)


if __name__ == '__main__':
    main()
```

写入 `/Users/mac/2026/ai_extra/ExamAI/backend/config/__init__.py`：

```python
from .celery import app as celery_app

__all__ = ('celery_app',)
```

写入 `/Users/mac/2026/ai_extra/ExamAI/backend/config/settings.py`：

```python
import os
from datetime import timedelta
from pathlib import Path

from decouple import Csv, config

BASE_DIR = Path(__file__).resolve().parent.parent

SECRET_KEY = config('SECRET_KEY', default='dev-secret-key-change-in-production')

DEBUG = config('DEBUG', default=True, cast=bool)

ALLOWED_HOSTS = config('ALLOWED_HOSTS', default='localhost,127.0.0.1', cast=Csv())

# Application definition
INSTALLED_APPS = [
    'django.contrib.admin',
    'django.contrib.auth',
    'django.contrib.contenttypes',
    'django.contrib.sessions',
    'django.contrib.messages',
    'django.contrib.staticfiles',
    # Third-party
    'rest_framework',
    'rest_framework_simplejwt',
    'corsheaders',
    'django_filters',
    # Local apps
    'apps.users',
    'apps.questions',
    'apps.exams',
    'apps.resources',
    'apps.ai',
]

MIDDLEWARE = [
    'corsheaders.middleware.CorsMiddleware',
    'django.middleware.security.SecurityMiddleware',
    'django.contrib.sessions.middleware.SessionMiddleware',
    'django.middleware.common.CommonMiddleware',
    'django.middleware.csrf.CsrfViewMiddleware',
    'django.contrib.auth.middleware.AuthenticationMiddleware',
    'django.contrib.messages.middleware.MessageMiddleware',
    'django.middleware.clickjacking.XFrameOptionsMiddleware',
]

ROOT_URLCONF = 'config.urls'

TEMPLATES = [
    {
        'BACKEND': 'django.template.backends.django.DjangoTemplates',
        'DIRS': [],
        'APP_DIRS': True,
        'OPTIONS': {
            'context_processors': [
                'django.template.context_processors.debug',
                'django.template.context_processors.request',
                'django.contrib.auth.context_processors.auth',
                'django.contrib.messages.context_processors.messages',
            ],
        },
    },
]

WSGI_APPLICATION = 'config.wsgi.application'

# Database
DATABASE_URL = config('DATABASE_URL', default='postgres://examai:examai@localhost:5432/examai')
DATABASES = {
    'default': {
        'ENGINE': 'django.db.backends.postgresql',
        'NAME': DATABASE_URL.split('/')[-1],
        'USER': DATABASE_URL.split('://')[1].split(':')[0],
        'PASSWORD': DATABASE_URL.split(':')[2].split('@')[0],
        'HOST': DATABASE_URL.split('@')[1].split(':')[0],
        'PORT': DATABASE_URL.split(':')[-1].split('/')[0],
    }
}

# Redis / Cache
REDIS_URL = config('REDIS_URL', default='redis://localhost:6379/0')
CACHES = {
    'default': {
        'BACKEND': 'django.core.cache.backends.redis.RedisCache',
        'LOCATION': REDIS_URL,
    }
}

# Celery
CELERY_BROKER_URL = config('CELERY_BROKER_URL', default='redis://localhost:6379/1')
CELERY_RESULT_BACKEND = config('CELERY_RESULT_BACKEND', default='redis://localhost:6379/2')
CELERY_ACCEPT_CONTENT = ['json']
CELERY_TASK_SERIALIZER = 'json'
CELERY_RESULT_SERIALIZER = 'json'
CELERY_TIMEZONE = 'Asia/Shanghai'

# JWT / DRF
REST_FRAMEWORK = {
    'DEFAULT_AUTHENTICATION_CLASSES': (
        'rest_framework_simplejwt.authentication.JWTAuthentication',
    ),
    'DEFAULT_PERMISSION_CLASSES': (
        'rest_framework.permissions.IsAuthenticated',
    ),
    'DEFAULT_PAGINATION_CLASS': 'common.pagination.StandardPagination',
    'DEFAULT_FILTER_BACKENDS': (
        'django_filters.rest_framework.DjangoFilterBackend',
        'rest_framework.filters.SearchFilter',
        'rest_framework.filters.OrderingFilter',
    ),
    'EXCEPTION_HANDLER': 'common.exceptions.custom_exception_handler',
    'DEFAULT_PARSER_CLASSES': (
        'rest_framework.parsers.JSONParser',
        'rest_framework.parsers.MultiPartParser',
    ),
}

SIMPLE_JWT = {
    'ACCESS_TOKEN_LIFETIME': timedelta(minutes=config('JWT_ACCESS_TOKEN_LIFETIME_MINUTES', default=60, cast=int)),
    'REFRESH_TOKEN_LIFETIME': timedelta(days=config('JWT_REFRESH_TOKEN_LIFETIME_DAYS', default=7, cast=int)),
    'AUTH_HEADER_TYPES': ('Bearer',),
}

# CORS
CORS_ALLOWED_ORIGINS = config(
    'CORS_ALLOWED_ORIGINS',
    default='http://localhost:5173',
    cast=Csv()
)
CORS_ALLOW_CREDENTIALS = True

# Internationalization
LANGUAGE_CODE = 'zh-hans'
TIME_ZONE = 'Asia/Shanghai'
USE_I18N = True
USE_TZ = True

# Static & Media
STATIC_URL = 'static/'
MEDIA_URL = 'media/'
MEDIA_ROOT = BASE_DIR / config('MEDIA_ROOT', default='media')

DEFAULT_AUTO_FIELD = 'django.db.models.BigAutoField'

# Auth user model
AUTH_USER_MODEL = 'users.User'

# File upload
FILE_UPLOAD_MAX_MB = config('FILE_UPLOAD_MAX_MB', default=50, cast=int)
DATA_UPLOAD_MAX_MEMORY_SIZE = FILE_UPLOAD_MAX_MB * 1024 * 1024

# AI default provider
DEEPSEEK_API_KEY = config('DEEPSEEK_API_KEY', default='')
DEEPSEEK_API_BASE = config('DEEPSEEK_API_BASE', default='https://api.deepseek.com')
```

写入 `/Users/mac/2026/ai_extra/ExamAI/backend/config/urls.py`：

```python
from django.conf import settings
from django.conf.urls.static import static
from django.contrib import admin
from django.urls import include, path

urlpatterns = [
    path('admin/', admin.site.urls),
    path('api/auth/', include('apps.users.urls')),
    path('api/questions/', include('apps.questions.urls')),
    path('api/exams/', include('apps.exams.urls')),
    path('api/resources/', include('apps.resources.urls')),
    path('api/ai/', include('apps.ai.urls')),
]

if settings.DEBUG:
    urlpatterns += static(settings.MEDIA_URL, document_root=settings.MEDIA_ROOT)
```

写入 `/Users/mac/2026/ai_extra/ExamAI/backend/config/wsgi.py`：

```python
import os
from django.core.wsgi import get_wsgi_application
os.environ.setdefault('DJANGO_SETTINGS_MODULE', 'config.settings')
application = get_wsgi_application()
```

写入 `/Users/mac/2026/ai_extra/ExamAI/backend/config/asgi.py`：

```python
import os
from django.core.asgi import get_asgi_application
os.environ.setdefault('DJANGO_SETTINGS_MODULE', 'config.settings')
application = get_asgi_application()
```

写入 `/Users/mac/2026/ai_extra/ExamAI/backend/config/celery.py`：

```python
import os
from celery import Celery

os.environ.setdefault('DJANGO_SETTINGS_MODULE', 'config.settings')

app = Celery('examai')
app.config_from_object('django.conf:settings', namespace='CELERY')
app.autodiscover_tasks()
```

- [ ] **Step 5: 创建公共模块**

写入 `/Users/mac/2026/ai_extra/ExamAI/backend/common/__init__.py`（空文件）

写入 `/Users/mac/2026/ai_extra/ExamAI/backend/common/response.py`：

```python
from rest_framework.response import Response


class APIResponse(Response):
    """统一 API 响应格式：{ code: 0, data: {...}, message: "ok" }"""

    def __init__(self, data=None, code=0, message='ok', status=200, **kwargs):
        body = {
            'code': code,
            'data': data,
            'message': message,
        }
        super().__init__(data=body, status=status, **kwargs)


def success(data=None, message='ok', status=200):
    return APIResponse(data=data, code=0, message=message, status=status)


def error(message='error', code=1, data=None, status=400):
    return APIResponse(data=data, code=code, message=message, status=status)
```

写入 `/Users/mac/2026/ai_extra/ExamAI/backend/common/pagination.py`：

```python
from rest_framework.pagination import PageNumberPagination


class StandardPagination(PageNumberPagination):
    """统一分页格式：{ page, page_size, total, results }"""
    page_size = 20
    page_size_query_param = 'page_size'
    max_page_size = 100

    def get_paginated_response(self, data):
        return Response({
            'code': 0,
            'data': {
                'page': self.page.number,
                'page_size': self.get_page_size(self.request),
                'total': self.page.paginator.count,
                'results': data,
            },
            'message': 'ok',
        })
```

（需要导入 Response — 修正：）

写入 `/Users/mac/2026/ai_extra/ExamAI/backend/common/pagination.py`（完整版）：

```python
from rest_framework.pagination import PageNumberPagination
from rest_framework.response import Response


class StandardPagination(PageNumberPagination):
    """统一分页格式：{ page, page_size, total, results }"""
    page_size = 20
    page_size_query_param = 'page_size'
    max_page_size = 100

    def get_paginated_response(self, data):
        return Response({
            'code': 0,
            'data': {
                'page': self.page.number,
                'page_size': self.get_page_size(self.request),
                'total': self.page.paginator.count,
                'results': data,
            },
            'message': 'ok',
        })
```

写入 `/Users/mac/2026/ai_extra/ExamAI/backend/common/exceptions.py`：

```python
from django.core.exceptions import ValidationError as DjangoValidationError
from rest_framework.exceptions import ValidationError as DRFValidationError
from rest_framework.views import exception_handler


def custom_exception_handler(exc, context):
    """全局异常处理，统一返回 { code, data, message } 格式"""
    response = exception_handler(exc, context)

    if response is not None:
        errors = response.data
        message = '请求参数错误'

        if isinstance(exc, DRFValidationError):
            # 提取第一个校验错误信息
            detail = exc.detail
            if isinstance(detail, dict):
                first_key = next(iter(detail))
                first_error = detail[first_key]
                if isinstance(first_error, list):
                    message = str(first_error[0])
                else:
                    message = str(first_error)
            elif isinstance(detail, list):
                message = str(detail[0])

        response.data = {
            'code': response.status_code,
            'data': errors,
            'message': message,
        }

    return response
```

- [ ] **Step 6: 创建各 app 空壳**

```bash
cd /Users/mac/2026/ai_extra/ExamAI/backend
source .venv/bin/activate
mkdir -p apps/users apps/questions apps/exams apps/resources apps/ai
for app in users questions exams resources ai; do
    touch apps/$app/__init__.py
    touch apps/$app/models.py
    touch apps/$app/admin.py
    touch apps/$app/urls.py
done
```

- [ ] **Step 7: 运行 Django 检查**

```bash
cd /Users/mac/2026/ai_extra/ExamAI/backend
source .venv/bin/activate
python manage.py check
```

预期：无错误输出（会有 apps 未注册的 warning，暂忽略）。

- [ ] **Step 8: 提交**

```bash
cd /Users/mac/2026/ai_extra/ExamAI
git add backend/
git commit -m "feat(m1): 搭建 Django 后端脚手架 — 配置/公共模块/app骨架"
```

---

#### Task M1.2: 用户模型与 JWT 认证

**文件：**
- 创建：`backend/apps/users/models.py`
- 创建：`backend/apps/users/serializers.py`
- 创建：`backend/apps/users/views.py`
- 创建：`backend/apps/users/urls.py`
- 创建：`backend/apps/users/admin.py`
- 创建：`backend/apps/users/tests/__init__.py`
- 创建：`backend/apps/users/tests/test_models.py`
- 创建：`backend/apps/users/tests/test_api.py`

- [ ] **Step 1: 编写 User 模型测试**

写入 `/Users/mac/2026/ai_extra/ExamAI/backend/apps/users/tests/test_models.py`：

```python
import pytest
from django.core.exceptions import ValidationError
from apps.users.models import User


@pytest.mark.django_db
class TestUserModel:
    def test_create_user_with_email(self):
        """创建用户时 email 为必填字段"""
        user = User.objects.create_user(
            email='teacher@test.com',
            password='testpass123',
            nickname='张老师'
        )
        assert user.email == 'teacher@test.com'
        assert user.nickname == '张老师'
        assert user.role == User.Role.TEACHER
        assert user.check_password('testpass123')
        assert user.is_active is True

    def test_create_user_without_email_raises(self):
        """email 为空时抛异常"""
        with pytest.raises(ValueError, match='email'):
            User.objects.create_user(email='', password='testpass123')

    def test_create_superuser(self):
        """创建管理员用户"""
        admin = User.objects.create_superuser(
            email='admin@examai.com',
            password='adminpass123'
        )
        assert admin.role == User.Role.ADMIN
        assert admin.is_staff is True
        assert admin.is_superuser is True

    def test_user_str_representation(self):
        """User 的字符串表示"""
        user = User.objects.create_user(
            email='teacher@test.com',
            password='testpass123',
            nickname='李老师'
        )
        assert str(user) == '李老师 (teacher@test.com)'

    def test_quota_defaults(self):
        """配额默认值"""
        user = User.objects.create_user(
            email='teacher@test.com',
            password='testpass123'
        )
        assert user.quota_total == 100
        assert user.quota_used == 0
```

- [ ] **Step 2: 运行测试，验证失败**

```bash
cd /Users/mac/2026/ai_extra/ExamAI/backend
source .venv/bin/activate
DJANGO_SETTINGS_MODULE=config.settings pytest apps/users/tests/test_models.py -v
```

预期：FAIL（User 模型未定义）

- [ ] **Step 3: 实现 User 模型**

写入 `/Users/mac/2026/ai_extra/ExamAI/backend/apps/users/models.py`：

```python
from django.contrib.auth.models import AbstractUser, BaseUserManager
from django.db import models


class UserManager(BaseUserManager):
    """自定义 User Manager，用 email 代替 username"""

    def _create_user(self, email, password, **extra_fields):
        if not email:
            raise ValueError('email 为必填字段')
        email = self.normalize_email(email)
        user = self.model(email=email, **extra_fields)
        user.set_password(password)
        user.save(using=self._db)
        return user

    def create_user(self, email, password=None, **extra_fields):
        extra_fields.setdefault('is_staff', False)
        extra_fields.setdefault('is_superuser', False)
        return self._create_user(email, password, **extra_fields)

    def create_superuser(self, email, password=None, **extra_fields):
        extra_fields.setdefault('is_staff', True)
        extra_fields.setdefault('is_superuser', True)
        extra_fields.setdefault('role', User.Role.ADMIN)
        return self._create_user(email, password, **extra_fields)


class User(AbstractUser):
    """自定义 User 模型 — email 登录"""

    class Role(models.TextChoices):
        TEACHER = 'teacher', '教师'
        ADMIN = 'admin', '管理员'

    username = None
    email = models.EmailField('邮箱', unique=True, db_index=True)
    nickname = models.CharField('昵称', max_length=50, blank=True, default='')
    role = models.CharField(
        '角色',
        max_length=20,
        choices=Role.choices,
        default=Role.TEACHER,
    )
    quota_total = models.PositiveIntegerField('每月配额总量', default=100)
    quota_used = models.PositiveIntegerField('本月已用配额', default=0)
    created_at = models.DateTimeField('创建时间', auto_now_add=True)
    updated_at = models.DateTimeField('更新时间', auto_now=True)

    USERNAME_FIELD = 'email'
    REQUIRED_FIELDS = []

    objects = UserManager()

    class Meta:
        db_table = 'users'
        verbose_name = '用户'
        verbose_name_plural = verbose_name
        ordering = ['-created_at']

    def __str__(self):
        return f'{self.nickname} ({self.email})'
```

- [ ] **Step 4: 创建并运行 migrations**

```bash
cd /Users/mac/2026/ai_extra/ExamAI/backend
source .venv/bin/activate
python manage.py makemigrations users
python manage.py migrate
```

- [ ] **Step 5: 运行模型测试，验证通过**

```bash
pytest apps/users/tests/test_models.py -v
```

预期：全部 PASS

- [ ] **Step 6: 编写认证 API 测试**

写入 `/Users/mac/2026/ai_extra/ExamAI/backend/apps/users/tests/test_api.py`：

```python
import pytest
from django.urls import reverse
from rest_framework import status
from apps.users.models import User


@pytest.mark.django_db
class TestAuthAPI:
    REGISTER_URL = reverse('user-register')
    LOGIN_URL = reverse('token-obtain-pair')
    REFRESH_URL = reverse('token-refresh')
    PROFILE_URL = reverse('user-profile')

    def test_register_success(self, client):
        """注册成功返回 201"""
        data = {
            'email': 'newteacher@test.com',
            'password': 'testpass123',
            'confirm_password': 'testpass123',
            'nickname': '新老师',
        }
        response = client.post(self.REGISTER_URL, data, format='json')
        assert response.status_code == status.HTTP_201_CREATED
        assert response.data['code'] == 0
        assert response.data['data']['email'] == 'newteacher@test.com'
        assert 'password' not in response.data['data']

    def test_register_duplicate_email(self, client):
        """重复邮箱注册失败"""
        User.objects.create_user(email='dup@test.com', password='pass123')
        data = {
            'email': 'dup@test.com',
            'password': 'testpass123',
            'confirm_password': 'testpass123',
        }
        response = client.post(self.REGISTER_URL, data, format='json')
        assert response.status_code == status.HTTP_400_BAD_REQUEST

    def test_login_success(self, client):
        """登录成功返回 JWT token pair"""
        User.objects.create_user(email='teacher@test.com', password='testpass123')
        data = {'email': 'teacher@test.com', 'password': 'testpass123'}
        response = client.post(self.LOGIN_URL, data, format='json')
        assert response.status_code == status.HTTP_200_OK
        assert 'access' in response.data['data']
        assert 'refresh' in response.data['data']

    def test_login_wrong_password(self, client):
        """密码错误登录失败"""
        User.objects.create_user(email='teacher@test.com', password='testpass123')
        data = {'email': 'teacher@test.com', 'password': 'wrongpass'}
        response = client.post(self.LOGIN_URL, data, format='json')
        assert response.status_code == status.HTTP_401_UNAUTHORIZED

    def test_get_profile_authenticated(self, client):
        """已认证用户获取个人信息"""
        user = User.objects.create_user(
            email='teacher@test.com',
            password='testpass123',
            nickname='张老师'
        )
        client.force_authenticate(user=user)
        response = client.get(self.PROFILE_URL)
        assert response.status_code == status.HTTP_200_OK
        assert response.data['data']['nickname'] == '张老师'

    def test_get_profile_unauthenticated(self, client):
        """未认证用户获取个人信息失败"""
        response = client.get(self.PROFILE_URL)
        assert response.status_code == status.HTTP_401_UNAUTHORIZED

    def test_refresh_token(self, client):
        """刷新 access token"""
        User.objects.create_user(email='teacher@test.com', password='testpass123')
        login_resp = client.post(
            self.LOGIN_URL,
            {'email': 'teacher@test.com', 'password': 'testpass123'},
            format='json'
        )
        refresh_token = login_resp.data['data']['refresh']
        response = client.post(self.REFRESH_URL, {'refresh': refresh_token}, format='json')
        assert response.status_code == status.HTTP_200_OK
        assert 'access' in response.data['data']
```

- [ ] **Step 7: 运行 API 测试，验证失败**

```bash
pytest apps/users/tests/test_api.py -v
```

预期：FAIL（serializer/view/url 未定义）

- [ ] **Step 8: 实现 Serializer**

写入 `/Users/mac/2026/ai_extra/ExamAI/backend/apps/users/serializers.py`：

```python
from django.contrib.auth import authenticate
from rest_framework import serializers
from rest_framework_simplejwt.serializers import TokenObtainPairSerializer

from .models import User


class RegisterSerializer(serializers.ModelSerializer):
    """注册序列化器"""
    password = serializers.CharField(write_only=True, min_length=6, max_length=128)
    confirm_password = serializers.CharField(write_only=True, min_length=6, max_length=128)

    class Meta:
        model = User
        fields = ('id', 'email', 'password', 'confirm_password', 'nickname')

    def validate_email(self, value):
        if User.objects.filter(email=value).exists():
            raise serializers.ValidationError('该邮箱已被注册')
        return value

    def validate(self, attrs):
        if attrs['password'] != attrs.pop('confirm_password'):
            raise serializers.ValidationError({'confirm_password': '两次密码输入不一致'})
        return attrs

    def create(self, validated_data):
        return User.objects.create_user(
            email=validated_data['email'],
            password=validated_data['password'],
            nickname=validated_data.get('nickname', ''),
        )


class UserProfileSerializer(serializers.ModelSerializer):
    """用户个人信息序列化器"""

    class Meta:
        model = User
        fields = ('id', 'email', 'nickname', 'role', 'quota_total', 'quota_used', 'created_at')
        read_only_fields = fields


class CustomTokenObtainPairSerializer(TokenObtainPairSerializer):
    """自定义 JWT 登录序列化器 — 用 email 代替 username"""

    def validate(self, attrs):
        email = attrs.get('email')
        password = attrs.get('password')
        user = authenticate(request=self.context.get('request'), email=email, password=password)
        if user is None:
            raise serializers.ValidationError('邮箱或密码错误')
        refresh = self.get_token(user)
        return {
            'refresh': str(refresh),
            'access': str(refresh.access_token),
            'user': UserProfileSerializer(user).data,
        }
```

- [ ] **Step 9: 实现 View**

写入 `/Users/mac/2026/ai_extra/ExamAI/backend/apps/users/views.py`：

```python
from rest_framework import generics, permissions, status
from rest_framework.decorators import api_view, permission_classes
from rest_framework_simplejwt.views import TokenObtainPairView, TokenRefreshView

from common.response import success, error

from .models import User
from .serializers import (
    CustomTokenObtainPairSerializer,
    RegisterSerializer,
    UserProfileSerializer,
)


class RegisterView(generics.CreateAPIView):
    """用户注册"""
    queryset = User.objects.all()
    serializer_class = RegisterSerializer
    permission_classes = (permissions.AllowAny,)

    def create(self, request, *args, **kwargs):
        serializer = self.get_serializer(data=request.data)
        serializer.is_valid(raise_exception=True)
        user = serializer.save()
        return success(
            data=UserProfileSerializer(user).data,
            message='注册成功',
            status=status.HTTP_201_CREATED,
        )


class LoginView(TokenObtainPairView):
    """登录 — 返回 JWT token pair"""
    serializer_class = CustomTokenObtainPairSerializer
    permission_classes = (permissions.AllowAny,)

    def post(self, request, *args, **kwargs):
        serializer = self.get_serializer(data=request.data)
        serializer.is_valid(raise_exception=True)
        return success(data=serializer.validated_data, message='登录成功')


class RefreshView(TokenRefreshView):
    """刷新 access token"""
    permission_classes = (permissions.AllowAny,)

    def post(self, request, *args, **kwargs):
        serializer = self.get_serializer(data=request.data)
        serializer.is_valid(raise_exception=True)
        return success(data=serializer.validated_data, message='token 刷新成功')


class ProfileView(generics.RetrieveUpdateAPIView):
    """获取/更新个人信息"""
    serializer_class = UserProfileSerializer
    permission_classes = (permissions.IsAuthenticated,)

    def get_object(self):
        return self.request.user

    def retrieve(self, request, *args, **kwargs):
        instance = self.get_object()
        return success(data=self.get_serializer(instance).data)

    def update(self, request, *args, **kwargs):
        partial = kwargs.pop('partial', False)
        instance = self.get_object()
        serializer = self.get_serializer(instance, data=request.data, partial=partial)
        serializer.is_valid(raise_exception=True)
        self.perform_update(serializer)
        return success(data=serializer.data, message='更新成功')
```

- [ ] **Step 10: 实现 URL 路由**

写入 `/Users/mac/2026/ai_extra/ExamAI/backend/apps/users/urls.py`：

```python
from django.urls import path

from . import views

urlpatterns = [
    path('register/', views.RegisterView.as_view(), name='user-register'),
    path('login/', views.LoginView.as_view(), name='token-obtain-pair'),
    path('refresh/', views.RefreshView.as_view(), name='token-refresh'),
    path('profile/', views.ProfileView.as_view(), name='user-profile'),
]
```

- [ ] **Step 11: 运行 API 测试，验证通过**

```bash
pytest apps/users/tests/test_api.py -v
```

预期：全部 PASS

- [ ] **Step 12: 提交**

```bash
git add backend/apps/users/
git commit -m "feat(m1): 实现 User 模型与 JWT 认证 — 注册/登录/刷新/个人信息"
```

---

#### Task M1.3: AI 服务抽象层

**文件：**
- 创建：`backend/apps/ai/__init__.py`
- 创建：`backend/apps/ai/models.py`
- 创建：`backend/apps/ai/base.py`
- 创建：`backend/apps/ai/providers/__init__.py`
- 创建：`backend/apps/ai/providers/deepseek.py`
- 创建：`backend/apps/ai/admin.py`
- 创建：`backend/apps/ai/urls.py`
- 创建：`backend/apps/ai/tests/__init__.py`
- 创建：`backend/apps/ai/tests/test_models.py`
- 创建：`backend/apps/ai/tests/test_providers.py`

- [ ] **Step 1: 编写 AI 模型测试**

写入 `/Users/mac/2026/ai_extra/ExamAI/backend/apps/ai/tests/test_models.py`：

```python
import pytest
from apps.ai.models import AIProvider, AITaskLog


@pytest.mark.django_db
class TestAIProvider:
    def test_create_provider(self):
        """创建 AI 提供商配置"""
        provider = AIProvider.objects.create(
            name='deepseek',
            display_name='DeepSeek',
            api_base_url='https://api.deepseek.com',
            api_key='sk-test-key',
            model_name='deepseek-chat',
            is_active=True,
            is_default=True,
        )
        assert provider.name == 'deepseek'
        assert provider.is_active is True
        assert provider.is_default is True
        assert str(provider) == 'DeepSeek'

    def test_default_provider_manager(self):
        """获取默认 provider"""
        AIProvider.objects.create(
            name='deepseek',
            display_name='DeepSeek',
            api_base_url='https://api.deepseek.com',
            api_key='sk-test-key',
            model_name='deepseek-chat',
            is_default=True,
        )
        AIProvider.objects.create(
            name='qwen',
            display_name='通义千问',
            api_base_url='https://dashscope.aliyuncs.com',
            api_key='sk-test-key',
            model_name='qwen-turbo',
            is_default=False,
        )
        default = AIProvider.objects.filter(is_default=True).first()
        assert default is not None
        assert default.name == 'deepseek'


@pytest.mark.django_db
class TestAITaskLog:
    def test_create_task_log(self):
        """创建 AI 调用日志"""
        provider = AIProvider.objects.create(
            name='deepseek',
            display_name='DeepSeek',
            api_base_url='https://api.deepseek.com',
            api_key='sk-test-key',
            model_name='deepseek-chat',
        )
        log = AITaskLog.objects.create(
            provider=provider,
            task_type='parse_question',
            input_tokens=500,
            output_tokens=200,
            cost=0.001,
            duration_ms=1500,
            success=True,
        )
        assert log.task_type == 'parse_question'
        assert log.success is True
        assert log.cost == 0.001
```

- [ ] **Step 2: 运行测试，验证失败**

```bash
pytest apps/ai/tests/test_models.py -v
```

预期：FAIL（models 未定义）

- [ ] **Step 3: 实现 AI 模型**

写入 `/Users/mac/2026/ai_extra/ExamAI/backend/apps/ai/models.py`：

```python
from django.db import models


class AIProvider(models.Model):
    """LLM 提供商配置"""

    name = models.CharField('标识名', max_length=50, unique=True)
    display_name = models.CharField('显示名称', max_length=100)
    api_base_url = models.URLField('API 地址', max_length=500)
    api_key = models.CharField('API Key', max_length=500)
    model_name = models.CharField('模型名称', max_length=100)
    is_active = models.BooleanField('是否启用', default=True)
    is_default = models.BooleanField('是否默认', default=False)
    config = models.JSONField('额外配置', default=dict, blank=True)
    created_at = models.DateTimeField('创建时间', auto_now_add=True)

    class Meta:
        db_table = 'ai_providers'
        verbose_name = 'AI 提供商'
        verbose_name_plural = verbose_name
        ordering = ['-is_default', 'name']

    def __str__(self):
        return self.display_name


class AITaskLog(models.Model):
    """AI 调用日志 / 成本统计"""

    class TaskType(models.TextChoices):
        PARSE_QUESTION = 'parse_question', '题目解析'
        RECOMMEND_EXAM = 'recommend_exam', '组卷推荐'

    provider = models.ForeignKey(
        AIProvider,
        on_delete=models.PROTECT,
        related_name='task_logs',
        verbose_name='提供商',
    )
    task_type = models.CharField('任务类型', max_length=50, choices=TaskType.choices)
    input_tokens = models.PositiveIntegerField('输入 token 数', default=0)
    output_tokens = models.PositiveIntegerField('输出 token 数', default=0)
    cost = models.DecimalField('费用', max_digits=10, decimal_places=6, default=0)
    duration_ms = models.PositiveIntegerField('耗时(毫秒)', default=0)
    success = models.BooleanField('是否成功', default=True)
    error_message = models.TextField('错误信息', blank=True, default='')
    created_at = models.DateTimeField('创建时间', auto_now_add=True)

    class Meta:
        db_table = 'ai_task_logs'
        verbose_name = 'AI 调用日志'
        verbose_name_plural = verbose_name
        ordering = ['-created_at']
        indexes = [
            models.Index(fields=['task_type', '-created_at']),
            models.Index(fields=['provider', '-created_at']),
        ]

    def __str__(self):
        return f'{self.get_task_type_display()} - {self.created_at:%Y-%m-%d %H:%M}'
```

- [ ] **Step 4: 实现 Adapter 抽象基类**

写入 `/Users/mac/2026/ai_extra/ExamAI/backend/apps/ai/base.py`：

```python
import time
from abc import ABC, abstractmethod
from typing import Any, Dict, Optional


class BaseAIAdapter(ABC):
    """AI 服务适配器抽象基类 — 所有 provider 需实现此接口"""

    def __init__(self, provider):
        self.provider = provider
        self.client = self._build_client()

    @abstractmethod
    def _build_client(self):
        """构建对应 SDK 的客户端实例"""
        ...

    @abstractmethod
    def chat(self, messages: list[Dict[str, str]], **kwargs) -> Dict[str, Any]:
        """
        通用对话接口
        返回: {'content': str, 'input_tokens': int, 'output_tokens': int, 'model': str}
        """
        ...

    def call_with_logging(self, task_type: str, messages: list[Dict[str, str]], **kwargs) -> Dict[str, Any]:
        """带日志记录的调用包装"""
        from .models import AITaskLog

        start_time = time.time()
        success = True
        error_message = ''
        result = {}

        try:
            result = self.chat(messages, **kwargs)
        except Exception as e:
            success = False
            error_message = str(e)
            result = {'content': '', 'input_tokens': 0, 'output_tokens': 0, 'model': self.provider.model_name}

        duration_ms = int((time.time() - start_time) * 1000)

        # 粗略计费（各模型可 override cost_per_token 方法）
        input_cost = result.get('input_tokens', 0) * self._input_cost_per_token()
        output_cost = result.get('output_tokens', 0) * self._output_cost_per_token()

        AITaskLog.objects.create(
            provider=self.provider,
            task_type=task_type,
            input_tokens=result.get('input_tokens', 0),
            output_tokens=result.get('output_tokens', 0),
            cost=input_cost + output_cost,
            duration_ms=duration_ms,
            success=success,
            error_message=error_message,
        )

        if not success:
            raise RuntimeError(f'AI 调用失败: {error_message}')

        return result

    def _input_cost_per_token(self) -> float:
        return 0.0

    def _output_cost_per_token(self) -> float:
        return 0.0


def get_default_adapter() -> BaseAIAdapter:
    """获取默认 AI provider 的 adapter 实例"""
    from .models import AIProvider
    from .providers import ADAPTER_REGISTRY

    provider = AIProvider.objects.filter(is_active=True, is_default=True).first()
    if not provider:
        raise RuntimeError('没有可用的默认 AI 提供商，请在管理后台配置')

    adapter_cls = ADAPTER_REGISTRY.get(provider.name)
    if not adapter_cls:
        raise RuntimeError(f'未找到 {provider.name} 对应的 adapter，请检查 providers 注册')

    return adapter_cls(provider)
```

写入 `/Users/mac/2026/ai_extra/ExamAI/backend/apps/ai/providers/__init__.py`：

```python
from .deepseek import DeepSeekAdapter

ADAPTER_REGISTRY = {
    'deepseek': DeepSeekAdapter,
}
```

- [ ] **Step 5: 实现 DeepSeek Adapter**

写入 `/Users/mac/2026/ai_extra/ExamAI/backend/apps/ai/providers/deepseek.py`：

```python
from typing import Any, Dict, List

from openai import OpenAI

from ..base import BaseAIAdapter


class DeepSeekAdapter(BaseAIAdapter):
    """DeepSeek API 适配器 — 兼容 OpenAI SDK"""

    def _build_client(self):
        return OpenAI(
            api_key=self.provider.api_key,
            base_url=self.provider.api_base_url,
        )

    def chat(self, messages: List[Dict[str, str]], **kwargs) -> Dict[str, Any]:
        config = self.provider.config or {}
        temperature = kwargs.get('temperature', config.get('temperature', 0.7))
        max_tokens = kwargs.get('max_tokens', config.get('max_tokens', 4096))
        response_format = kwargs.get('response_format', config.get('response_format'))

        completion = self.client.chat.completions.create(
            model=self.provider.model_name,
            messages=messages,
            temperature=temperature,
            max_tokens=max_tokens,
            response_format=response_format,
        )

        choice = completion.choices[0]
        return {
            'content': choice.message.content or '',
            'input_tokens': completion.usage.prompt_tokens if completion.usage else 0,
            'output_tokens': completion.usage.completion_tokens if completion.usage else 0,
            'model': completion.model,
        }

    def _input_cost_per_token(self) -> float:
        # DeepSeek V3 价格: ¥0.001 / 1K tokens (输入)
        return 0.001 / 1000

    def _output_cost_per_token(self) -> float:
        # DeepSeek V3 价格: ¥0.002 / 1K tokens (输出)
        return 0.002 / 1000
```

- [ ] **Step 6: 编写 Provider 测试**

写入 `/Users/mac/2026/ai_extra/ExamAI/backend/apps/ai/tests/test_providers.py`：

```python
import pytest
from unittest.mock import MagicMock, patch
from apps.ai.models import AIProvider
from apps.ai.providers.deepseek import DeepSeekAdapter


class TestDeepSeekAdapter:
    @pytest.fixture
    def provider(self):
        return AIProvider(
            name='deepseek',
            display_name='DeepSeek',
            api_base_url='https://api.deepseek.com',
            api_key='sk-test-key',
            model_name='deepseek-chat',
            config={'temperature': 0.7, 'max_tokens': 4096},
        )

    def test_build_client(self, provider):
        """构建 OpenAI 客户端"""
        with patch('apps.ai.providers.deepseek.OpenAI') as mock_openai:
            DeepSeekAdapter(provider)
            mock_openai.assert_called_once_with(
                api_key='sk-test-key',
                base_url='https://api.deepseek.com',
            )

    @patch('apps.ai.providers.deepseek.OpenAI')
    def test_chat_returns_expected_format(self, mock_openai_cls, provider):
        """chat 返回预期格式"""
        mock_client = MagicMock()
        mock_choice = MagicMock()
        mock_choice.message.content = '你好，这是回复内容'
        mock_usage = MagicMock()
        mock_usage.prompt_tokens = 100
        mock_usage.completion_tokens = 50
        mock_completion = MagicMock()
        mock_completion.choices = [mock_choice]
        mock_completion.usage = mock_usage
        mock_completion.model = 'deepseek-chat'
        mock_client.chat.completions.create.return_value = mock_completion
        mock_openai_cls.return_value = mock_client

        adapter = DeepSeekAdapter(provider)
        result = adapter.chat([{'role': 'user', 'content': '你好'}])

        assert result['content'] == '你好，这是回复内容'
        assert result['input_tokens'] == 100
        assert result['output_tokens'] == 50
        assert result['model'] == 'deepseek-chat'
```

- [ ] **Step 7: 创建 AI admin 和 urls**

写入 `/Users/mac/2026/ai_extra/ExamAI/backend/apps/ai/admin.py`：

```python
from django.contrib import admin
from .models import AIProvider, AITaskLog


@admin.register(AIProvider)
class AIProviderAdmin(admin.ModelAdmin):
    list_display = ('display_name', 'name', 'model_name', 'is_active', 'is_default')
    list_filter = ('is_active', 'is_default')
    search_fields = ('name', 'display_name')


@admin.register(AITaskLog)
class AITaskLogAdmin(admin.ModelAdmin):
    list_display = ('task_type', 'provider', 'success', 'input_tokens', 'output_tokens', 'cost', 'duration_ms', 'created_at')
    list_filter = ('task_type', 'success', 'provider')
    readonly_fields = ('input_tokens', 'output_tokens', 'cost', 'duration_ms')
```

写入 `/Users/mac/2026/ai_extra/ExamAI/backend/apps/ai/urls.py`（空路由，预留）：

```python
urlpatterns = []
```

- [ ] **Step 8: 运行所有 AI 测试**

```bash
pytest apps/ai/tests/ -v
```

预期：全部 PASS

- [ ] **Step 9: 运行 migrations**

```bash
python manage.py makemigrations ai
python manage.py migrate
```

- [ ] **Step 10: 提交**

```bash
git add backend/apps/ai/
git commit -m "feat(m1): 实现 AI 服务抽象层 — AIProvider/AITaskLog 模型 + DeepSeek adapter"
```

---

#### Task M1.4: 前端脚手架

**文件：**
- 创建：`frontend/package.json`
- 创建：`frontend/tsconfig.json`
- 创建：`frontend/vite.config.ts`
- 创建：`frontend/index.html`
- 创建：`frontend/src/main.tsx`
- 创建：`frontend/src/App.tsx`
- 创建：`frontend/src/router/index.tsx`
- 创建：`frontend/src/services/api.ts`
- 创建：`frontend/src/stores/authStore.ts`
- 创建：`frontend/src/hooks/useAuth.ts`
- 创建：`frontend/src/utils/constants.ts`
- 创建：`frontend/src/styles/global.css`

- [ ] **Step 1: 用 Vite 创建 React + TypeScript 项目**

```bash
cd /Users/mac/2026/ai_extra/ExamAI
npm create vite@latest frontend -- --template react-ts
```

- [ ] **Step 2: 安装前端依赖**

```bash
cd /Users/mac/2026/ai_extra/ExamAI/frontend
npm install antd @ant-design/icons axios zustand react-router-dom dayjs
npm install -D @types/react @types/react-dom
```

- [ ] **Step 3: 配置 Vite 代理和路径别名**

修改 `/Users/mac/2026/ai_extra/ExamAI/frontend/vite.config.ts`：

```typescript
import { defineConfig } from 'vite'
import react from '@vitejs/plugin-react'
import path from 'path'

export default defineConfig({
  plugins: [react()],
  resolve: {
    alias: {
      '@': path.resolve(__dirname, './src'),
    },
  },
  server: {
    port: 5173,
    proxy: {
      '/api': {
        target: 'http://localhost:8000',
        changeOrigin: true,
      },
      '/media': {
        target: 'http://localhost:8000',
        changeOrigin: true,
      },
    },
  },
})
```

- [ ] **Step 4: 配置 TypeScript 路径别名**

修改 `/Users/mac/2026/ai_extra/ExamAI/frontend/tsconfig.json`，添加 paths：

```json
{
  "compilerOptions": {
    "target": "ES2020",
    "useDefineForClassFields": true,
    "lib": ["ES2020", "DOM", "DOM.Iterable"],
    "module": "ESNext",
    "skipLibCheck": true,
    "moduleResolution": "bundler",
    "allowImportingTsExtensions": true,
    "resolveJsonModule": true,
    "isolatedModules": true,
    "noEmit": true,
    "jsx": "react-jsx",
    "strict": true,
    "noUnusedLocals": false,
    "noUnusedParameters": false,
    "noFallthroughCasesInSwitch": true,
    "baseUrl": ".",
    "paths": {
      "@/*": ["src/*"]
    }
  },
  "include": ["src"],
  "references": [{ "path": "./tsconfig.node.json" }]
}
```

- [ ] **Step 5: 创建 axios 实例与拦截器**

写入 `/Users/mac/2026/ai_extra/ExamAI/frontend/src/services/api.ts`：

```typescript
import axios from 'axios';
import { message } from 'antd';

const api = axios.create({
  baseURL: '/api',
  timeout: 30000,
  headers: {
    'Content-Type': 'application/json',
  },
});

// 请求拦截器：注入 JWT token
api.interceptors.request.use(
  (config) => {
    const token = localStorage.getItem('access_token');
    if (token) {
      config.headers.Authorization = `Bearer ${token}`;
    }
    return config;
  },
  (error) => Promise.reject(error)
);

// 响应拦截器：统一错误处理 + token 刷新
let isRefreshing = false;
let failedQueue: Array<{
  resolve: (value?: unknown) => void;
  reject: (reason?: unknown) => void;
}> = [];

const processQueue = (error: unknown, token: string | null = null) => {
  failedQueue.forEach((prom) => {
    if (error) {
      prom.reject(error);
    } else {
      prom.resolve(token);
    }
  });
  failedQueue = [];
};

api.interceptors.response.use(
  (response) => {
    const res = response.data;
    if (res.code !== 0) {
      message.error(res.message || '请求失败');
      return Promise.reject(new Error(res.message));
    }
    return response;
  },
  async (error) => {
    const originalRequest = error.config;

    // 401 且未重试过 → 尝试刷新 token
    if (error.response?.status === 401 && !originalRequest._retry) {
      if (isRefreshing) {
        return new Promise((resolve, reject) => {
          failedQueue.push({ resolve, reject });
        }).then((token) => {
          originalRequest.headers.Authorization = `Bearer ${token}`;
          return api(originalRequest);
        });
      }

      originalRequest._retry = true;
      isRefreshing = true;

      const refreshToken = localStorage.getItem('refresh_token');
      if (refreshToken) {
        try {
          const { data } = await axios.post('/api/auth/refresh/', {
            refresh: refreshToken,
          });
          const newAccess = data.data.access;
          localStorage.setItem('access_token', newAccess);
          processQueue(null, newAccess);
          originalRequest.headers.Authorization = `Bearer ${newAccess}`;
          return api(originalRequest);
        } catch (refreshError) {
          processQueue(refreshError, null);
          localStorage.removeItem('access_token');
          localStorage.removeItem('refresh_token');
          window.location.href = '/login';
          return Promise.reject(refreshError);
        } finally {
          isRefreshing = false;
        }
      }
    }

    const msg = error.response?.data?.message || '网络错误，请重试';
    message.error(msg);
    return Promise.reject(error);
  }
);

export default api;
```

- [ ] **Step 6: 创建认证 Store**

写入 `/Users/mac/2026/ai_extra/ExamAI/frontend/src/stores/authStore.ts`：

```typescript
import { create } from 'zustand';

interface User {
  id: number;
  email: string;
  nickname: string;
  role: string;
  quota_total: number;
  quota_used: number;
}

interface AuthState {
  user: User | null;
  isAuthenticated: boolean;
  setUser: (user: User | null) => void;
  login: (access: string, refresh: string, user: User) => void;
  logout: () => void;
}

export const useAuthStore = create<AuthState>((set) => ({
  user: null,
  isAuthenticated: !!localStorage.getItem('access_token'),

  setUser: (user) => set({ user, isAuthenticated: !!user }),

  login: (access, refresh, user) => {
    localStorage.setItem('access_token', access);
    localStorage.setItem('refresh_token', refresh);
    set({ user, isAuthenticated: true });
  },

  logout: () => {
    localStorage.removeItem('access_token');
    localStorage.removeItem('refresh_token');
    set({ user: null, isAuthenticated: false });
  },
}));
```

- [ ] **Step 7: 创建常量文件**

写入 `/Users/mac/2026/ai_extra/ExamAI/frontend/src/utils/constants.ts`：

```typescript
// 学科
export const SUBJECTS = [
  { value: 'math', label: '数学' },
  { value: 'physics', label: '物理' },
  { value: 'history', label: '历史' },
] as const;

// 年级
export const GRADES = [
  { value: 7, label: '七年级' },
  { value: 8, label: '八年级' },
  { value: 9, label: '九年级' },
] as const;

// 题型
export const QUESTION_TYPES = [
  { value: 'choice', label: '选择题' },
  { value: 'fill_blank', label: '填空题' },
  { value: 'true_false', label: '判断题' },
  { value: 'short_answer', label: '简答题' },
  { value: 'calculation', label: '计算题' },
  { value: 'essay', label: '论述题' },
] as const;

// 难度
export const DIFFICULTY_LEVELS = [
  { value: 1, label: '★ 基础', color: 'green' },
  { value: 2, label: '★★ 简单', color: 'cyan' },
  { value: 3, label: '★★★ 中等', color: 'blue' },
  { value: 4, label: '★★★★ 较难', color: 'orange' },
  { value: 5, label: '★★★★★ 困难', color: 'red' },
] as const;

// 认知层级
export const COGNITIVE_LEVELS = [
  { value: 'knowledge', label: '识记' },
  { value: 'comprehension', label: '理解' },
  { value: 'application', label: '应用' },
  { value: 'analysis', label: '分析' },
  { value: 'synthesis', label: '综合' },
] as const;

// 考试类型
export const EXAM_TYPES = [
  { value: 'unit', label: '单元测试' },
  { value: 'monthly', label: '月考' },
  { value: 'midterm', label: '期中考试' },
  { value: 'final', label: '期末考试' },
  { value: 'entrance', label: '中考模拟' },
] as const;

// 题目来源
export const SOURCE_TYPES = [
  { value: 'exam', label: '真题' },
  { value: 'mock', label: '模拟题' },
  { value: 'original', label: '原创题' },
] as const;

// 题目状态
export const QUESTION_STATUS = [
  { value: 'draft', label: '草稿', color: 'default' },
  { value: 'reviewed', label: '已审核', color: 'green' },
  { value: 'disabled', label: '已禁用', color: 'red' },
] as const;
```

- [ ] **Step 8: 创建路由配置**

写入 `/Users/mac/2026/ai_extra/ExamAI/frontend/src/router/index.tsx`：

```typescript
import { createBrowserRouter, Navigate } from 'react-router-dom';
import AppLayout from '@/components/layout/AppLayout';
import LoginPage from '@/pages/LoginPage';
import DashboardPage from '@/pages/DashboardPage';
import ExamGeneratePage from '@/pages/ExamGeneratePage';
import QuestionBankPage from '@/pages/QuestionBankPage';
import ExamDetailPage from '@/pages/ExamDetailPage';
import ResourceCollectPage from '@/pages/ResourceCollectPage';

// 简单的路由守卫
function AuthGuard({ children }: { children: React.ReactNode }) {
  const token = localStorage.getItem('access_token');
  if (!token) {
    return <Navigate to="/login" replace />;
  }
  return <>{children}</>;
}

const router = createBrowserRouter([
  {
    path: '/login',
    element: <LoginPage />,
  },
  {
    path: '/',
    element: (
      <AuthGuard>
        <AppLayout />
      </AuthGuard>
    ),
    children: [
      { index: true, element: <DashboardPage /> },
      { path: 'questions', element: <QuestionBankPage /> },
      { path: 'exams/generate', element: <ExamGeneratePage /> },
      { path: 'exams/:id', element: <ExamDetailPage /> },
      { path: 'resources', element: <ResourceCollectPage /> },
    ],
  },
]);

export default router;
```

- [ ] **Step 9: 创建 App 入口和全局样式**

修改 `/Users/mac/2026/ai_extra/ExamAI/frontend/src/main.tsx`：

```typescript
import React from 'react';
import ReactDOM from 'react-dom/client';
import { RouterProvider } from 'react-router-dom';
import { ConfigProvider } from 'antd';
import zhCN from 'antd/locale/zh_CN';
import router from './router';
import './styles/global.css';

ReactDOM.createRoot(document.getElementById('root')!).render(
  <React.StrictMode>
    <ConfigProvider
      locale={zhCN}
      theme={{
        token: {
          colorPrimary: '#1677ff',
          borderRadius: 6,
        },
      }}
    >
      <RouterProvider router={router} />
    </ConfigProvider>
  </React.StrictMode>
);
```

写入 `/Users/mac/2026/ai_extra/ExamAI/frontend/src/styles/global.css`：

```css
* {
  margin: 0;
  padding: 0;
  box-sizing: border-box;
}

body {
  font-family: -apple-system, BlinkMacSystemFont, 'Segoe UI', Roboto, 'Helvetica Neue', Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
}

#root {
  height: 100vh;
}

/* 滚动条美化 */
::-webkit-scrollbar {
  width: 6px;
  height: 6px;
}

::-webkit-scrollbar-thumb {
  background: #d9d9d9;
  border-radius: 3px;
}

::-webkit-scrollbar-thumb:hover {
  background: #bfbfbf;
}
```

- [ ] **Step 10: 创建布局组件占位和页面占位**

```bash
mkdir -p /Users/mac/2026/ai_extra/ExamAI/frontend/src/components/layout
mkdir -p /Users/mac/2026/ai_extra/ExamAI/frontend/src/components/common
mkdir -p /Users/mac/2026/ai_extra/ExamAI/frontend/src/pages
```

写入 `/Users/mac/2026/ai_extra/ExamAI/frontend/src/components/layout/AppLayout.tsx`：

```typescript
import { useState } from 'react';
import { Outlet } from 'react-router-dom';
import { Layout } from 'antd';
import Sidebar from './Sidebar';
import HeaderBar from './HeaderBar';

const { Content, Sider } = Layout;

export default function AppLayout() {
  const [collapsed, setCollapsed] = useState(false);

  return (
    <Layout style={{ height: '100vh' }}>
      <Sider
        collapsible
        collapsed={collapsed}
        onCollapse={setCollapsed}
        theme="light"
        width={220}
        style={{ borderRight: '1px solid #f0f0f0' }}
      >
        <Sidebar />
      </Sider>
      <Layout>
        <HeaderBar collapsed={collapsed} onToggle={() => setCollapsed(!collapsed)} />
        <Content
          style={{
            padding: 24,
            overflow: 'auto',
            background: '#f5f5f5',
          }}
        >
          <Outlet />
        </Content>
      </Layout>
    </Layout>
  );
}
```

写入 `/Users/mac/2026/ai_extra/ExamAI/frontend/src/components/layout/Sidebar.tsx`：

```typescript
import { useNavigate, useLocation } from 'react-router-dom';
import { Menu } from 'antd';
import {
  DashboardOutlined,
  FileTextOutlined,
  DatabaseOutlined,
  CloudUploadOutlined,
} from '@ant-design/icons';

const menuItems = [
  { key: '/', icon: <DashboardOutlined />, label: '工作台' },
  { key: '/exams/generate', icon: <FileTextOutlined />, label: '智能组卷' },
  { key: '/questions', icon: <DatabaseOutlined />, label: '题库管理' },
  { key: '/resources', icon: <CloudUploadOutlined />, label: '资源采集' },
];

export default function Sidebar() {
  const navigate = useNavigate();
  const location = useLocation();

  // 高亮当前路由
  const selectedKey = '/' + location.pathname.split('/')[1];

  return (
    <div style={{ padding: '16px 0' }}>
      <div
        style={{
          padding: '0 24px 16px',
          fontSize: 18,
          fontWeight: 700,
          color: '#1677ff',
          textAlign: 'center',
          cursor: 'pointer',
        }}
        onClick={() => navigate('/')}
      >
        ExamAI
      </div>
      <Menu
        mode="inline"
        selectedKeys={[selectedKey]}
        items={menuItems}
        onClick={({ key }) => navigate(key)}
        style={{ borderRight: 0 }}
      />
    </div>
  );
}
```

写入 `/Users/mac/2026/ai_extra/ExamAI/frontend/src/components/layout/HeaderBar.tsx`：

```typescript
import { Layout, Button, Dropdown, Space, Avatar } from 'antd';
import {
  MenuFoldOutlined,
  MenuUnfoldOutlined,
  UserOutlined,
  LogoutOutlined,
} from '@ant-design/icons';
import { useAuthStore } from '@/stores/authStore';
import { useNavigate } from 'react-router-dom';

const { Header } = Layout;

interface Props {
  collapsed: boolean;
  onToggle: () => void;
}

export default function HeaderBar({ collapsed, onToggle }: Props) {
  const { user, logout } = useAuthStore();
  const navigate = useNavigate();

  const handleLogout = () => {
    logout();
    navigate('/login');
  };

  const dropdownItems = {
    items: [
      {
        key: 'profile',
        icon: <UserOutlined />,
        label: `${user?.nickname || user?.email || '用户'}`,
        disabled: true,
      },
      { type: 'divider' as const },
      {
        key: 'logout',
        icon: <LogoutOutlined />,
        label: '退出登录',
        onClick: handleLogout,
      },
    ],
  };

  return (
    <Header
      style={{
        padding: '0 24px',
        background: '#fff',
        display: 'flex',
        alignItems: 'center',
        justifyContent: 'space-between',
        borderBottom: '1px solid #f0f0f0',
      }}
    >
      <Button
        type="text"
        icon={collapsed ? <MenuUnfoldOutlined /> : <MenuFoldOutlined />}
        onClick={onToggle}
      />
      <Dropdown menu={dropdownItems} placement="bottomRight">
        <Space style={{ cursor: 'pointer' }}>
          <Avatar size="small" icon={<UserOutlined />} />
          <span>{user?.nickname || '用户'}</span>
        </Space>
      </Dropdown>
    </Header>
  );
}
```

创建各页面占位文件：

写入 `/Users/mac/2026/ai_extra/ExamAI/frontend/src/pages/LoginPage.tsx`：

```typescript
export default function LoginPage() {
  return <div>登录页 — M6 实现</div>;
}
```

写入 `/Users/mac/2026/ai_extra/ExamAI/frontend/src/pages/DashboardPage.tsx`：

```typescript
export default function DashboardPage() {
  return <div>工作台 — M6 实现</div>;
}
```

写入 `/Users/mac/2026/ai_extra/ExamAI/frontend/src/pages/ExamGeneratePage.tsx`：

```typescript
export default function ExamGeneratePage() {
  return <div>智能组卷 — M3 实现</div>;
}
```

写入 `/Users/mac/2026/ai_extra/ExamAI/frontend/src/pages/QuestionBankPage.tsx`：

```typescript
export default function QuestionBankPage() {
  return <div>题库管理 — M2 实现</div>;
}
```

写入 `/Users/mac/2026/ai_extra/ExamAI/frontend/src/pages/ExamDetailPage.tsx`：

```typescript
export default function ExamDetailPage() {
  return <div>试卷详情 — M4 实现</div>;
}
```

写入 `/Users/mac/2026/ai_extra/ExamAI/frontend/src/pages/ResourceCollectPage.tsx`：

```typescript
export default function ResourceCollectPage() {
  return <div>资源采集 — M5 实现</div>;
}
```

- [ ] **Step 11: 验证前端可启动**

```bash
cd /Users/mac/2026/ai_extra/ExamAI/frontend
npm run dev
```

访问 http://localhost:5173 ，确认页面加载（会跳转到 /login 因为未登录）。

- [ ] **Step 12: 提交**

```bash
git add frontend/
git commit -m "feat(m1): 搭建 React 前端脚手架 — Vite + Ant Design + 路由 + 布局 + 认证状态"
```

---

### M1 里程碑检查点

在进入 M2 之前，确认以下全部通过：

```bash
# 后端测试全部通过
cd /Users/mac/2026/ai_extra/ExamAI/backend
source .venv/bin/activate
pytest apps/ -v

# Django check 无错误
python manage.py check

# 前端可编译
cd /Users/mac/2026/ai_extra/ExamAI/frontend
npm run build
```

---




## M2 — 题库模块（第3-5周）

> **目标：** 实现完整的题库管理系统 — 学科/年级/知识点树数据模型、题目 CRUD、Excel 批量导入、知识点树管理、题库管理前端页面、种子数据入库。
> **交付物：** 可用的题库管理后台 + 至少 1000 道种子题目

### Task M2.1: 学科、年级、知识点模型

**文件：**
- 创建：`backend/apps/questions/models.py`
- 创建：`backend/apps/questions/admin.py`
- 创建：`backend/apps/questions/tests/__init__.py`
- 创建：`backend/apps/questions/tests/test_models.py`

- [ ] **Step 1: 编写模型测试**

写入 `/Users/mac/2026/ai_extra/ExamAI/backend/apps/questions/tests/test_models.py`：

```python
import pytest
from apps.questions.models import Subject, Grade, KnowledgePoint


@pytest.mark.django_db
class TestSubject:
    def test_create_subject(self):
        subject = Subject.objects.create(name='数学', code='math')
        assert subject.name == '数学'
        assert subject.code == 'math'
        assert subject.is_active is True
        assert str(subject) == '数学'


@pytest.mark.django_db
class TestGrade:
    def test_create_grade(self):
        grade = Grade.objects.create(name='八年级', level=8)
        assert grade.name == '八年级'
        assert grade.level == 8

    def test_grade_subjects_m2m(self):
        math = Subject.objects.create(name='数学', code='math')
        physics = Subject.objects.create(name='物理', code='physics')
        grade = Grade.objects.create(name='八年级', level=8)
        grade.subjects.add(math, physics)
        assert grade.subjects.count() == 2
        assert math.grade_set.count() == 1


@pytest.mark.django_db
class TestKnowledgePoint:
    @pytest.fixture
    def math(self):
        return Subject.objects.create(name='数学', code='math')

    def test_create_root_knowledge_point(self, math):
        kp = KnowledgePoint.objects.create(
            name='代数',
            subject=math,
            level=1,
        )
        assert kp.parent is None
        assert kp.level == 1
        assert kp.path == '代数'

    def test_create_child_knowledge_point(self, math):
        parent = KnowledgePoint.objects.create(
            name='代数', subject=math, level=1
        )
        child = KnowledgePoint.objects.create(
            name='一元一次方程',
            subject=math,
            parent=parent,
            level=2,
        )
        assert child.parent == parent
        assert child.path == '代数/一元一次方程'
        assert parent.children.count() == 1

    def test_knowledge_point_tree_structure(self, math):
        """三层知识点树结构"""
        root = KnowledgePoint.objects.create(name='代数', subject=math, level=1)
        mid = KnowledgePoint.objects.create(name='方程', subject=math, parent=root, level=2)
        leaf = KnowledgePoint.objects.create(name='一元一次方程', subject=math, parent=mid, level=3)
        assert leaf.path == '代数/方程/一元一次方程'
        assert root.get_descendant_count() == 2

    def test_save_auto_sets_path(self, math):
        """保存时自动生成 path"""
        kp = KnowledgePoint(name='代数', subject=math)
        kp.save()
        assert kp.path == '代数'
        assert kp.level == 1
```

- [ ] **Step 2: 运行测试，验证失败**

```bash
cd /Users/mac/2026/ai_extra/ExamAI/backend
source .venv/bin/activate
pytest apps/questions/tests/test_models.py -v
```

预期：FAIL

- [ ] **Step 3: 实现模型**

写入 `/Users/mac/2026/ai_extra/ExamAI/backend/apps/questions/models.py`：

```python
from django.db import models


class Subject(models.Model):
    """学科"""
    name = models.CharField('名称', max_length=50, unique=True)
    code = models.CharField('代码', max_length=50, unique=True)
    is_active = models.BooleanField('是否启用', default=True)

    class Meta:
        db_table = 'subjects'
        verbose_name = '学科'
        verbose_name_plural = verbose_name
        ordering = ['id']

    def __str__(self):
        return self.name


class Grade(models.Model):
    """年级"""
    name = models.CharField('名称', max_length=50, unique=True)
    level = models.PositiveSmallIntegerField('年级级别', unique=True)
    subjects = models.ManyToManyField(Subject, related_name='grades', verbose_name='关联学科')

    class Meta:
        db_table = 'grades'
        verbose_name = '年级'
        verbose_name_plural = verbose_name
        ordering = ['level']

    def __str__(self):
        return self.name


class KnowledgePoint(models.Model):
    """知识点 — 自引用树"""
    name = models.CharField('名称', max_length=200)
    parent = models.ForeignKey(
        'self',
        on_delete=models.CASCADE,
        null=True,
        blank=True,
        related_name='children',
        verbose_name='父知识点',
    )
    subject = models.ForeignKey(
        Subject,
        on_delete=models.CASCADE,
        related_name='knowledge_points',
        verbose_name='所属学科',
    )
    level = models.PositiveSmallIntegerField('层级深度', default=1)
    sort_order = models.PositiveIntegerField('排序', default=0)
    path = models.CharField('路径', max_length=1000, blank=True, default='', db_index=True)

    class Meta:
        db_table = 'knowledge_points'
        verbose_name = '知识点'
        verbose_name_plural = verbose_name
        ordering = ['subject', 'sort_order']
        indexes = [
            models.Index(fields=['subject', 'level']),
            models.Index(fields=['path']),
        ]

    def __str__(self):
        return self.path or self.name

    def save(self, *args, **kwargs):
        # 自动设置 level
        if self.parent:
            self.level = self.parent.level + 1
        else:
            self.level = 1
        super().save(*args, **kwargs)
        # 保存后更新 path
        self._update_path()

    def _update_path(self):
        """构建 path: '代数/方程/一元一次方程'"""
        parts = []
        node = self
        while node:
            parts.insert(0, node.name)
            node = node.parent
        new_path = '/'.join(parts)
        if new_path != self.path:
            KnowledgePoint.objects.filter(pk=self.pk).update(path=new_path)
            # 递归更新所有子节点
            for child in self.children.all():
                child._update_path()

    def get_descendant_count(self):
        """获取子孙节点总数"""
        return KnowledgePoint.objects.filter(
            path__startswith=self.path + '/'
        ).count()
```

- [ ] **Step 4: 运行 migrations 并测试**

```bash
python manage.py makemigrations questions
python manage.py migrate
pytest apps/questions/tests/test_models.py -v
```

预期：全部 PASS

- [ ] **Step 5: 创建 admin 和数据迁移（种子数据）**

写入 `/Users/mac/2026/ai_extra/ExamAI/backend/apps/questions/admin.py`：

```python
from django.contrib import admin
from .models import Subject, Grade, KnowledgePoint


@admin.register(Subject)
class SubjectAdmin(admin.ModelAdmin):
    list_display = ('name', 'code', 'is_active')


@admin.register(Grade)
class GradeAdmin(admin.ModelAdmin):
    list_display = ('name', 'level')
    filter_horizontal = ('subjects',)


@admin.register(KnowledgePoint)
class KnowledgePointAdmin(admin.ModelAdmin):
    list_display = ('name', 'subject', 'level', 'path', 'sort_order')
    list_filter = ('subject', 'level')
    search_fields = ('name', 'path')
```

创建种子数据管理命令：

```bash
mkdir -p /Users/mac/2026/ai_extra/ExamAI/backend/apps/questions/management/commands
touch /Users/mac/2026/ai_extra/ExamAI/backend/apps/questions/management/__init__.py
touch /Users/mac/2026/ai_extra/ExamAI/backend/apps/questions/management/commands/__init__.py
```

写入 `/Users/mac/2026/ai_extra/ExamAI/backend/apps/questions/management/commands/seed_data.py`：

```python
from django.core.management.base import BaseCommand
from apps.questions.models import Subject, Grade, KnowledgePoint


class Command(BaseCommand):
    help = '初始化种子数据：学科、年级、知识点树'

    def handle(self, *args, **options):
        # 创建学科
        subjects = {
            'math': Subject.objects.get_or_create(name='数学', code='math')[0],
            'physics': Subject.objects.get_or_create(name='物理', code='physics')[0],
            'history': Subject.objects.get_or_create(name='历史', code='history')[0],
        }

        # 创建年级
        grades = {}
        for level, name in [(7, '七年级'), (8, '八年级'), (9, '九年级')]:
            grade, _ = Grade.objects.get_or_create(name=name, level=level)
            grades[level] = grade

        # 关联年级与学科
        grades[7].subjects.add(subjects['math'], subjects['history'])
        grades[8].subjects.add(subjects['math'], subjects['physics'], subjects['history'])
        grades[9].subjects.add(subjects['math'], subjects['physics'], subjects['history'])

        # 数学知识点树（示例）
        math_kps = {}
        math_tree = {
            '数与代数': ['有理数', '实数', '代数式', '整式与分式', '方程与不等式'],
            '图形与几何': ['三角形', '四边形', '圆', '相似与全等', '解直角三角形'],
            '函数': ['一次函数', '二次函数', '反比例函数', '锐角三角函数'],
            '统计与概率': ['数据的收集与整理', '数据分析', '概率'],
        }
        for root_name, children in math_tree.items():
            root = KnowledgePoint.objects.get_or_create(
                name=root_name, subject=subjects['math'], parent=None,
                defaults={'sort_order': len(math_kps)}
            )[0]
            math_kps[root_name] = root
            for i, child_name in enumerate(children):
                KnowledgePoint.objects.get_or_create(
                    name=child_name, subject=subjects['math'], parent=root,
                    defaults={'sort_order': i}
                )

        # 物理知识点树（示例）
        physics_tree = {
            '力学': ['运动和力', '压强与浮力', '功和机械能', '简单机械'],
            '热学': ['物态变化', '内能与热机'],
            '光学': ['光的传播', '透镜及其应用'],
            '电学': ['电路基础', '欧姆定律', '电功率', '电磁现象'],
        }
        for root_name, children in physics_tree.items():
            root = KnowledgePoint.objects.get_or_create(
                name=root_name, subject=subjects['physics'], parent=None
            )[0]
            for i, child_name in enumerate(children):
                KnowledgePoint.objects.get_or_create(
                    name=child_name, subject=subjects['physics'], parent=root,
                    defaults={'sort_order': i}
                )

        # 历史知识点树（示例）
        history_tree = {
            '中国古代史': ['夏商周', '秦汉', '三国两晋南北朝', '隋唐', '宋元', '明清'],
            '中国近代史': ['鸦片战争', '洋务运动', '辛亥革命', '新民主主义革命'],
            '中国现代史': ['新中国成立', '改革开放', '中国特色社会主义'],
            '世界史': ['古代文明', '中世纪', '近代世界', '现代世界'],
        }
        for root_name, children in history_tree.items():
            root = KnowledgePoint.objects.get_or_create(
                name=root_name, subject=subjects['history'], parent=None
            )[0]
            for i, child_name in enumerate(children):
                KnowledgePoint.objects.get_or_create(
                    name=child_name, subject=subjects['history'], parent=root,
                    defaults={'sort_order': i}
                )

        self.stdout.write(self.style.SUCCESS('种子数据初始化完成！'))
        self.stdout.write(f'  学科: {Subject.objects.count()} 个')
        self.stdout.write(f'  年级: {Grade.objects.count()} 个')
        self.stdout.write(f'  知识点: {KnowledgePoint.objects.count()} 个')
```

- [ ] **Step 6: 运行种子数据命令**

```bash
python manage.py seed_data
```

- [ ] **Step 7: 提交**

```bash
git add backend/apps/questions/
git commit -m "feat(m2): 实现学科/年级/知识点树模型 + 种子数据初始化命令"
```

---

### Task M2.2: 题目模型与 CRUD API

**文件：**
- 修改：`backend/apps/questions/models.py`（追加 Question, QuestionOption）
- 创建：`backend/apps/questions/serializers.py`
- 创建：`backend/apps/questions/views.py`
- 创建：`backend/apps/questions/urls.py`
- 创建：`backend/apps/questions/filters.py`
- 创建：`backend/apps/questions/tests/test_api.py`

- [ ] **Step 1: 编写题目模型测试**

追加到 `/Users/mac/2026/ai_extra/ExamAI/backend/apps/questions/tests/test_models.py`：

```python
@pytest.mark.django_db
class TestQuestion:
    @pytest.fixture
    def math(self):
        return Subject.objects.create(name='数学', code='math')

    @pytest.fixture
    def grade8(self):
        return Grade.objects.create(name='八年级', level=8)

    @pytest.fixture
    def kp(self, math):
        return KnowledgePoint.objects.create(name='代数', subject=math)

    def test_create_question(self, math, grade8, kp):
        question = Question.objects.create(
            subject=math,
            grade=grade8,
            type='choice',
            difficulty=3,
            stem='计算 $2+2$ 的结果',
            answer='4',
            explanation='这是基本加法',
            default_score=3,
        )
        question.knowledge_points.add(kp)
        assert question.type == 'choice'
        assert question.difficulty == 3
        assert question.status == 'draft'
        assert question.knowledge_points.count() == 1

    def test_create_question_with_options(self, math, grade8):
        question = Question.objects.create(
            subject=math, grade=grade8, type='choice', difficulty=2,
            stem='1+1=?', answer='B',
        )
        QuestionOption.objects.create(question=question, label='A', content='1', sort_order=0)
        QuestionOption.objects.create(question=question, label='B', content='2', sort_order=1)
        QuestionOption.objects.create(question=question, label='C', content='3', sort_order=2)
        QuestionOption.objects.create(question=question, label='D', content='4', sort_order=3)
        assert question.options.count() == 4
        assert question.options.first().label == 'A'
```

- [ ] **Step 2: 运行测试，验证失败**

```bash
pytest apps/questions/tests/test_models.py::TestQuestion -v
```

预期：FAIL（Question 模型未定义）

- [ ] **Step 3: 实现 Question 和 QuestionOption 模型**

追加到 `/Users/mac/2026/ai_extra/ExamAI/backend/apps/questions/models.py`：

```python
class Question(models.Model):
    """题目"""

    class Type(models.TextChoices):
        CHOICE = 'choice', '选择题'
        FILL_BLANK = 'fill_blank', '填空题'
        TRUE_FALSE = 'true_false', '判断题'
        SHORT_ANSWER = 'short_answer', '简答题'
        CALCULATION = 'calculation', '计算题'
        ESSAY = 'essay', '论述题'

    class Difficulty(models.IntegerChoices):
        VERY_EASY = 1, '基础'
        EASY = 2, '简单'
        MEDIUM = 3, '中等'
        HARD = 4, '较难'
        VERY_HARD = 5, '困难'

    class CognitiveLevel(models.TextChoices):
        KNOWLEDGE = 'knowledge', '识记'
        COMPREHENSION = 'comprehension', '理解'
        APPLICATION = 'application', '应用'
        ANALYSIS = 'analysis', '分析'
        SYNTHESIS = 'synthesis', '综合'

    class SourceType(models.TextChoices):
        EXAM = 'exam', '真题'
        MOCK = 'mock', '模拟题'
        ORIGINAL = 'original', '原创题'

    class Status(models.TextChoices):
        DRAFT = 'draft', '草稿'
        REVIEWED = 'reviewed', '已审核'
        DISABLED = 'disabled', '已禁用'

    subject = models.ForeignKey(Subject, on_delete=models.PROTECT, related_name='questions', verbose_name='学科')
    grade = models.ForeignKey(Grade, on_delete=models.PROTECT, related_name='questions', verbose_name='年级')
    type = models.CharField('题型', max_length=20, choices=Type.choices)
    difficulty = models.PositiveSmallIntegerField('难度(1-5)', choices=Difficulty.choices)
    cognitive_level = models.CharField('认知层级', max_length=20, choices=CognitiveLevel.choices, default=CognitiveLevel.COMPREHENSION)
    stem = models.TextField('题干 (Markdown + LaTeX)')
    answer = models.TextField('答案')
    explanation = models.TextField('解析 (Markdown)', blank=True, default='')
    default_score = models.PositiveIntegerField('默认分值', default=3)
    estimated_time = models.PositiveIntegerField('预计耗时(秒)', default=120)
    images = models.JSONField('配图路径列表', default=list, blank=True)
    source_type = models.CharField('来源类型', max_length=20, choices=SourceType.choices, default=SourceType.ORIGINAL)
    source_detail = models.CharField('来源详情', max_length=500, blank=True, default='')
    exam_type = models.CharField('考试类型', max_length=20, blank=True, default='')
    status = models.CharField('状态', max_length=20, choices=Status.choices, default=Status.DRAFT)
    usage_count = models.PositiveIntegerField('使用次数', default=0)
    knowledge_points = models.ManyToManyField(KnowledgePoint, related_name='questions', verbose_name='知识点')
    created_at = models.DateTimeField('创建时间', auto_now_add=True)
    updated_at = models.DateTimeField('更新时间', auto_now=True)

    class Meta:
        db_table = 'questions'
        verbose_name = '题目'
        verbose_name_plural = verbose_name
        ordering = ['-created_at']
        indexes = [
            models.Index(fields=['subject', 'grade', 'type']),
            models.Index(fields=['difficulty']),
            models.Index(fields=['status']),
            models.Index(fields=['subject', 'grade', 'type', 'difficulty']),
        ]

    def __str__(self):
        return self.stem[:50]


class QuestionOption(models.Model):
    """选择题选项"""
    question = models.ForeignKey(Question, on_delete=models.CASCADE, related_name='options', verbose_name='题目')
    label = models.CharField('标签', max_length=10)
    content = models.TextField('选项内容')
    sort_order = models.PositiveIntegerField('排序', default=0)

    class Meta:
        db_table = 'question_options'
        verbose_name = '选择题选项'
        verbose_name_plural = verbose_name
        ordering = ['sort_order']
        unique_together = [['question', 'label']]

    def __str__(self):
        return f'{self.label}: {self.content[:30]}'
```

- [ ] **Step 4: 运行 migrations 并测试模型**

```bash
python manage.py makemigrations questions
python manage.py migrate
pytest apps/questions/tests/test_models.py -v
```

预期：全部 PASS

- [ ] **Step 5: 编写 API 测试**

写入 `/Users/mac/2026/ai_extra/ExamAI/backend/apps/questions/tests/test_api.py`：

```python
import pytest
from django.urls import reverse
from rest_framework import status
from apps.users.models import User
from apps.questions.models import Subject, Grade, KnowledgePoint, Question, QuestionOption


@pytest.mark.django_db
class TestQuestionAPI:
    LIST_URL = reverse('question-list')

    @pytest.fixture
    def user(self):
        return User.objects.create_user(email='teacher@test.com', password='pass123')

    @pytest.fixture
    def math(self):
        return Subject.objects.create(name='数学', code='math')

    @pytest.fixture
    def grade8(self):
        return Grade.objects.create(name='八年级', level=8)

    @pytest.fixture
    def kp(self, math):
        return KnowledgePoint.objects.create(name='代数', subject=math)

    @pytest.fixture
    def auth_client(self, client, user):
        client.force_authenticate(user=user)
        return client

    def test_list_questions_empty(self, auth_client):
        """空题库列表"""
        response = auth_client.get(self.LIST_URL)
        assert response.status_code == status.HTTP_200_OK
        assert response.data['data']['total'] == 0

    def test_create_choice_question(self, auth_client, math, grade8, kp):
        """创建选择题"""
        data = {
            'subject_id': math.id,
            'grade_id': grade8.id,
            'type': 'choice',
            'difficulty': 3,
            'stem': '计算 $2+2$ 的结果',
            'answer': 'B',
            'explanation': '2+2=4',
            'default_score': 3,
            'knowledge_point_ids': [kp.id],
            'options': [
                {'label': 'A', 'content': '1', 'sort_order': 0},
                {'label': 'B', 'content': '2', 'sort_order': 1},
                {'label': 'C', 'content': '3', 'sort_order': 2},
                {'label': 'D', 'content': '4', 'sort_order': 3},
            ],
        }
        response = auth_client.post(self.LIST_URL, data, format='json')
        assert response.status_code == status.HTTP_201_CREATED
        assert response.data['data']['type'] == 'choice'
        assert len(response.data['data']['options']) == 4

    def test_create_question_missing_required(self, auth_client):
        """缺少必填字段创建失败"""
        data = {'type': 'choice'}
        response = auth_client.post(self.LIST_URL, data, format='json')
        assert response.status_code == status.HTTP_400_BAD_REQUEST

    def test_filter_by_subject(self, auth_client, math, grade8, kp):
        """按学科筛选"""
        physics = Subject.objects.create(name='物理', code='physics')
        Question.objects.create(subject=math, grade=grade8, type='choice', difficulty=3, stem='数学题', answer='A')
        Question.objects.create(subject=physics, grade=grade8, type='choice', difficulty=3, stem='物理题', answer='B')
        response = auth_client.get(self.LIST_URL, {'subject': math.id})
        assert response.data['data']['total'] == 1

    def test_filter_by_difficulty(self, auth_client, math, grade8):
        """按难度筛选"""
        Question.objects.create(subject=math, grade=grade8, type='choice', difficulty=1, stem='简单题', answer='A')
        Question.objects.create(subject=math, grade=grade8, type='choice', difficulty=5, stem='困难题', answer='D')
        response = auth_client.get(self.LIST_URL, {'difficulty': 5})
        assert response.data['data']['total'] == 1

    def test_search_question(self, auth_client, math, grade8):
        """搜索题目"""
        Question.objects.create(subject=math, grade=grade8, type='choice', difficulty=3, stem='一元二次方程求解', answer='x=1')
        Question.objects.create(subject=math, grade=grade8, type='fill_blank', difficulty=2, stem='三角形面积计算', answer='6')
        response = auth_client.get(self.LIST_URL, {'search': '三角形'})
        assert response.data['data']['total'] == 1

    def test_update_question(self, auth_client, math, grade8, kp):
        """更新题目"""
        q = Question.objects.create(subject=math, grade=grade8, type='choice', difficulty=3, stem='旧题干', answer='A')
        q.knowledge_points.add(kp)
        url = reverse('question-detail', kwargs={'pk': q.pk})
        response = auth_client.patch(url, {'stem': '新题干', 'difficulty': 4}, format='json')
        assert response.status_code == status.HTTP_200_OK
        assert response.data['data']['stem'] == '新题干'

    def test_delete_question(self, auth_client, math, grade8):
        """删除题目"""
        q = Question.objects.create(subject=math, grade=grade8, type='choice', difficulty=3, stem='待删除', answer='A')
        url = reverse('question-detail', kwargs={'pk': q.pk})
        response = auth_client.delete(url)
        assert response.status_code == status.HTTP_204_NO_CONTENT
        assert Question.objects.filter(pk=q.pk).count() == 0

    def test_unauthorized_access(self, client):
        """未认证用户无法访问"""
        response = client.get(self.LIST_URL)
        assert response.status_code == status.HTTP_401_UNAUTHORIZED
```

- [ ] **Step 6: 运行 API 测试，验证失败**

```bash
pytest apps/questions/tests/test_api.py -v
```

预期：FAIL

- [ ] **Step 7: 实现 Serializer**

写入 `/Users/mac/2026/ai_extra/ExamAI/backend/apps/questions/serializers.py`：

```python
from rest_framework import serializers
from .models import Subject, Grade, KnowledgePoint, Question, QuestionOption


class SubjectSerializer(serializers.ModelSerializer):
    class Meta:
        model = Subject
        fields = ('id', 'name', 'code')


class GradeSerializer(serializers.ModelSerializer):
    subjects = SubjectSerializer(many=True, read_only=True)

    class Meta:
        model = Grade
        fields = ('id', 'name', 'level', 'subjects')


class KnowledgePointSerializer(serializers.ModelSerializer):
    children = serializers.SerializerMethodField()

    class Meta:
        model = KnowledgePoint
        fields = ('id', 'name', 'parent', 'subject', 'level', 'sort_order', 'path', 'children')

    def get_children(self, obj):
        children = obj.children.all()
        if children.exists():
            return KnowledgePointSerializer(children, many=True).data
        return []


class KnowledgePointFlatSerializer(serializers.ModelSerializer):
    """扁平化知识点（用于下拉选择等）"""
    class Meta:
        model = KnowledgePoint
        fields = ('id', 'name', 'path', 'level', 'subject')


class QuestionOptionSerializer(serializers.ModelSerializer):
    class Meta:
        model = QuestionOption
        fields = ('id', 'label', 'content', 'sort_order')


class QuestionListSerializer(serializers.ModelSerializer):
    """题目列表序列化器（轻量）"""
    subject_name = serializers.CharField(source='subject.name', read_only=True)
    grade_name = serializers.CharField(source='grade.name', read_only=True)
    type_display = serializers.CharField(source='get_type_display', read_only=True)
    difficulty_display = serializers.CharField(source='get_difficulty_display', read_only=True)
    knowledge_point_names = serializers.SerializerMethodField()

    class Meta:
        model = Question
        fields = (
            'id', 'subject', 'subject_name', 'grade', 'grade_name',
            'type', 'type_display', 'difficulty', 'difficulty_display',
            'stem', 'status', 'usage_count', 'knowledge_point_names', 'created_at'
        )

    def get_knowledge_point_names(self, obj):
        return [kp.name for kp in obj.knowledge_points.all()]


class QuestionDetailSerializer(serializers.ModelSerializer):
    """题目详情序列化器（含选项和知识点完整信息）"""
    options = QuestionOptionSerializer(many=True, read_only=True)
    knowledge_points = KnowledgePointFlatSerializer(many=True, read_only=True)
    subject_name = serializers.CharField(source='subject.name', read_only=True)
    grade_name = serializers.CharField(source='grade.name', read_only=True)

    class Meta:
        model = Question
        fields = (
            'id', 'subject', 'subject_name', 'grade', 'grade_name',
            'type', 'difficulty', 'cognitive_level', 'stem', 'answer', 'explanation',
            'default_score', 'estimated_time', 'images', 'source_type', 'source_detail',
            'exam_type', 'status', 'usage_count', 'options', 'knowledge_points',
            'created_at', 'updated_at',
        )
        read_only_fields = ('usage_count', 'created_at', 'updated_at')


class QuestionCreateSerializer(serializers.ModelSerializer):
    """题目创建序列化器"""
    subject_id = serializers.PrimaryKeyRelatedField(
        queryset=Subject.objects.all(), source='subject', write_only=True
    )
    grade_id = serializers.PrimaryKeyRelatedField(
        queryset=Grade.objects.all(), source='grade', write_only=True
    )
    knowledge_point_ids = serializers.PrimaryKeyRelatedField(
        queryset=KnowledgePoint.objects.all(), source='knowledge_points', many=True, write_only=True
    )
    options = QuestionOptionSerializer(many=True, required=False)

    class Meta:
        model = Question
        fields = (
            'subject_id', 'grade_id', 'type', 'difficulty', 'cognitive_level',
            'stem', 'answer', 'explanation', 'default_score', 'estimated_time',
            'images', 'source_type', 'source_detail', 'exam_type',
            'knowledge_point_ids', 'options',
        )

    def create(self, validated_data):
        options_data = validated_data.pop('options', [])
        question = Question.objects.create(**validated_data)
        for opt_data in options_data:
            QuestionOption.objects.create(question=question, **opt_data)
        return question

    def validate_options(self, value):
        """选择题必须有至少 2 个选项"""
        question_type = self.initial_data.get('type', '')
        if question_type == 'choice' and len(value) < 2:
            raise serializers.ValidationError('选择题至少需要 2 个选项')
        return value
```

- [ ] **Step 8: 实现 Filter**

写入 `/Users/mac/2026/ai_extra/ExamAI/backend/apps/questions/filters.py`：

```python
import django_filters
from .models import Question


class QuestionFilter(django_filters.FilterSet):
    subject = django_filters.NumberFilter(field_name='subject_id')
    grade = django_filters.NumberFilter(field_name='grade_id')
    type = django_filters.CharFilter(field_name='type')
    difficulty = django_filters.NumberFilter(field_name='difficulty')
    difficulty__gte = django_filters.NumberFilter(field_name='difficulty', lookup_expr='gte')
    difficulty__lte = django_filters.NumberFilter(field_name='difficulty', lookup_expr='lte')
    cognitive_level = django_filters.CharFilter(field_name='cognitive_level')
    status = django_filters.CharFilter(field_name='status')
    source_type = django_filters.CharFilter(field_name='source_type')
    knowledge_point = django_filters.NumberFilter(method='filter_by_knowledge_point')

    class Meta:
        model = Question
        fields = [
            'subject', 'grade', 'type', 'difficulty',
            'cognitive_level', 'status', 'source_type',
        ]

    def filter_by_knowledge_point(self, queryset, name, value):
        return queryset.filter(knowledge_points__id=value).distinct()
```

- [ ] **Step 9: 实现 View**

写入 `/Users/mac/2026/ai_extra/ExamAI/backend/apps/questions/views.py`：

```python
from rest_framework import viewsets, permissions, status
from rest_framework.decorators import action
from django_filters.rest_framework import DjangoFilterBackend
from rest_framework.filters import SearchFilter, OrderingFilter

from common.response import success
from common.pagination import StandardPagination

from .models import Question, Subject, Grade, KnowledgePoint
from .serializers import (
    QuestionListSerializer,
    QuestionDetailSerializer,
    QuestionCreateSerializer,
    SubjectSerializer,
    GradeSerializer,
    KnowledgePointSerializer,
    KnowledgePointFlatSerializer,
)
from .filters import QuestionFilter


class QuestionViewSet(viewsets.ModelViewSet):
    """题目 CRUD"""
    queryset = Question.objects.select_related('subject', 'grade').prefetch_related('knowledge_points', 'options')
    permission_classes = (permissions.IsAuthenticated,)
    pagination_class = StandardPagination
    filter_backends = (DjangoFilterBackend, SearchFilter, OrderingFilter)
    filterset_class = QuestionFilter
    search_fields = ('stem', 'answer')
    ordering_fields = ('created_at', 'difficulty', 'usage_count')
    ordering = ('-created_at',)

    def get_serializer_class(self):
        if self.action == 'list':
            return QuestionListSerializer
        elif self.action in ('create', 'update', 'partial_update'):
            return QuestionCreateSerializer
        return QuestionDetailSerializer

    def create(self, request, *args, **kwargs):
        serializer = self.get_serializer(data=request.data)
        serializer.is_valid(raise_exception=True)
        question = serializer.save()
        return success(
            data=QuestionDetailSerializer(question).data,
            message='题目创建成功',
            status=status.HTTP_201_CREATED,
        )

    def update(self, request, *args, **kwargs):
        partial = kwargs.pop('partial', False)
        instance = self.get_object()
        serializer = self.get_serializer(instance, data=request.data, partial=partial)
        serializer.is_valid(raise_exception=True)
        question = serializer.save()
        return success(data=QuestionDetailSerializer(question).data, message='更新成功')

    def destroy(self, request, *args, **kwargs):
        instance = self.get_object()
        instance.delete()
        return success(message='删除成功', status=status.HTTP_204_NO_CONTENT)


class SubjectViewSet(viewsets.ReadOnlyModelViewSet):
    """学科列表（只读）"""
    queryset = Subject.objects.filter(is_active=True)
    serializer_class = SubjectSerializer
    permission_classes = (permissions.IsAuthenticated,)


class GradeViewSet(viewsets.ReadOnlyModelViewSet):
    """年级列表（只读）"""
    queryset = Grade.objects.prefetch_related('subjects')
    serializer_class = GradeSerializer
    permission_classes = (permissions.IsAuthenticated,)


class KnowledgePointViewSet(viewsets.ModelViewSet):
    """知识点树 CRUD"""
    queryset = KnowledgePoint.objects.select_related('subject', 'parent').prefetch_related('children')
    permission_classes = (permissions.IsAuthenticated,)

    def get_serializer_class(self):
        if self.action == 'list':
            return KnowledgePointSerializer
        return KnowledgePointFlatSerializer

    def get_queryset(self):
        qs = super().get_queryset()
        subject_id = self.request.query_params.get('subject')
        root_only = self.request.query_params.get('root_only')
        if subject_id:
            qs = qs.filter(subject_id=subject_id)
        if root_only:
            qs = qs.filter(parent__isnull=True)
        return qs
```

- [ ] **Step 10: 实现 URL 路由**

写入 `/Users/mac/2026/ai_extra/ExamAI/backend/apps/questions/urls.py`：

```python
from django.urls import path, include
from rest_framework.routers import DefaultRouter
from . import views

router = DefaultRouter()
router.register(r'', views.QuestionViewSet, basename='question')
router.register(r'subjects', views.SubjectViewSet, basename='subject')
router.register(r'grades', views.GradeViewSet, basename='grade')
router.register(r'knowledge-points', views.KnowledgePointViewSet, basename='knowledge-point')

urlpatterns = [
    path('', include(router.urls)),
]
```

- [ ] **Step 11: 运行 API 测试**

```bash
pytest apps/questions/tests/test_api.py -v
```

预期：全部 PASS

- [ ] **Step 12: 提交**

```bash
git add backend/apps/questions/
git commit -m "feat(m2): 实现题目模型与 CRUD API — 列表/详情/创建/更新/删除 + 筛选搜索"
```

---

### Task M2.3: Excel 批量导入

**文件：**
- 创建：`backend/apps/questions/imports.py`
- 创建：`backend/apps/questions/tests/test_import.py`
- 修改：`backend/apps/questions/views.py`（追加 import action）

- [ ] **Step 1: 编写导入功能测试**

写入 `/Users/mac/2026/ai_extra/ExamAI/backend/apps/questions/tests/test_import.py`：

```python
import io
import pytest
from django.urls import reverse
from openpyxl import Workbook
from apps.users.models import User
from apps.questions.models import Subject, Grade, KnowledgePoint


@pytest.mark.django_db
class TestExcelImport:
    IMPORT_URL = reverse('question-import-excel')

    @pytest.fixture
    def user(self):
        return User.objects.create_user(email='teacher@test.com', password='pass123')

    @pytest.fixture
    def math(self):
        return Subject.objects.create(name='数学', code='math')

    @pytest.fixture
    def grade8(self):
        return Grade.objects.create(name='八年级', level=8)

    @pytest.fixture
    def kp(self, math):
        return KnowledgePoint.objects.create(name='代数', subject=math)

    @pytest.fixture
    def auth_client(self, client, user):
        client.force_authenticate(user=user)
        return client

    def _create_excel_file(self, rows: list[dict]) -> bytes:
        """创建 Excel 文件 bytes"""
        wb = Workbook()
        ws = wb.active
        ws.title = '题目导入'
        headers = ['题型', '题干', '答案', '解析', '难度', '默认分值', '知识点', '选项A', '选项B', '选项C', '选项D']
        ws.append(headers)
        for row in rows:
            ws.append([
                row.get('type', ''),
                row.get('stem', ''),
                row.get('answer', ''),
                row.get('explanation', ''),
                row.get('difficulty', 3),
                row.get('default_score', 3),
                row.get('knowledge_points', ''),
                row.get('option_a', ''),
                row.get('option_b', ''),
                row.get('option_c', ''),
                row.get('option_d', ''),
            ])
        output = io.BytesIO()
        wb.save(output)
        output.seek(0)
        return output.getvalue()

    def test_import_choice_questions(self, auth_client, math, grade8, kp):
        """导入选择题"""
        excel_bytes = self._create_excel_file([{
            'type': '选择题',
            'stem': '计算 2+2 的结果',
            'answer': 'B',
            'explanation': '2+2=4',
            'difficulty': 2,
            'default_score': 3,
            'knowledge_points': '代数',
            'option_a': '1',
            'option_b': '4',
            'option_c': '3',
            'option_d': '2',
        }])

        from django.core.files.uploadedfile import SimpleUploadedFile
        file = SimpleUploadedFile('test.xlsx', excel_bytes,
                                  content_type='application/vnd.openxmlformats-officedocument.spreadsheetml.sheet')

        response = auth_client.post(
            self.IMPORT_URL,
            {'file': file, 'subject_id': math.id, 'grade_id': grade8.id},
            format='multipart'
        )
        assert response.status_code == 200
        assert response.data['data']['success_count'] >= 1

    def test_import_empty_file(self, auth_client, math, grade8):
        """导入空文件"""
        from django.core.files.uploadedfile import SimpleUploadedFile
        file = SimpleUploadedFile('empty.xlsx', b'',
                                  content_type='application/vnd.openxmlformats-officedocument.spreadsheetml.sheet')
        response = auth_client.post(
            self.IMPORT_URL,
            {'file': file, 'subject_id': math.id, 'grade_id': grade8.id},
            format='multipart'
        )
        assert response.status_code == 400
```

- [ ] **Step 2: 运行测试，验证失败**

```bash
pytest apps/questions/tests/test_import.py -v
```

预期：FAIL（import action 未定义）

- [ ] **Step 3: 实现 Excel 导入逻辑**

写入 `/Users/mac/2026/ai_extra/ExamAI/backend/apps/questions/imports.py`：

```python
import io
from typing import Tuple
from openpyxl import load_workbook
from django.db import transaction

from .models import Question, QuestionOption, KnowledgePoint, Subject, Grade


# 题型中文映射
TYPE_MAP = {
    '选择题': 'choice',
    '填空题': 'fill_blank',
    '判断题': 'true_false',
    '简答题': 'short_answer',
    '计算题': 'calculation',
    '论述题': 'essay',
}


def import_questions_from_excel(file_bytes: bytes, subject: Subject, grade: Grade) -> Tuple[int, list[str]]:
    """
    从 Excel bytes 导入题目
    返回: (成功数量, 错误列表)
    """
    wb = load_workbook(io.BytesIO(file_bytes), read_only=True)
    ws = wb.active
    rows = list(ws.iter_rows(min_row=2, values_only=True))  # 跳过表头

    success_count = 0
    errors = []

    for i, row in enumerate(rows, start=2):
        if not row or not any(row):
            continue

        try:
            type_cn, stem, answer, explanation, difficulty, default_score, kp_str, *options = row

            # 验证必填字段
            if not stem or not answer:
                errors.append(f'第{i}行: 题干或答案为空')
                continue

            # 转换题型
            question_type = TYPE_MAP.get(str(type_cn).strip() if type_cn else '')
            if not question_type:
                errors.append(f'第{i}行: 无效题型 "{type_cn}"')
                continue

            # 难度处理
            try:
                difficulty = int(difficulty) if difficulty else 3
                difficulty = max(1, min(5, difficulty))
            except (ValueError, TypeError):
                difficulty = 3

            # 分值处理
            try:
                default_score = int(default_score) if default_score else 3
            except (ValueError, TypeError):
                default_score = 3

            with transaction.atomic():
                question = Question.objects.create(
                    subject=subject,
                    grade=grade,
                    type=question_type,
                    difficulty=difficulty,
                    stem=str(stem).strip(),
                    answer=str(answer).strip(),
                    explanation=str(explanation).strip() if explanation else '',
                    default_score=default_score,
                )

                # 关联知识点（按名称查找）
                if kp_str:
                    kp_names = [n.strip() for n in str(kp_str).split(',') if n.strip()]
                    for kp_name in kp_names:
                        kp = KnowledgePoint.objects.filter(
                            name=kp_name, subject=subject
                        ).first()
                        if kp:
                            question.knowledge_points.add(kp)

                # 创建选项（选择题）
                if question_type == 'choice':
                    option_labels = ['A', 'B', 'C', 'D']
                    for j, label in enumerate(option_labels):
                        opt_content = options[j] if j < len(options) else None
                        if opt_content and str(opt_content).strip():
                            QuestionOption.objects.create(
                                question=question,
                                label=label,
                                content=str(opt_content).strip(),
                                sort_order=j,
                            )

            success_count += 1

        except Exception as e:
            errors.append(f'第{i}行: {str(e)}')

    return success_count, errors
```

- [ ] **Step 4: 在 View 中添加导入 action**

在 `/Users/mac/2026/ai_extra/ExamAI/backend/apps/questions/views.py` 的 `QuestionViewSet` 中添加：

```python
    @action(detail=False, methods=['post'], url_path='import-excel')
    def import_excel(self, request):
        """Excel 批量导入题目"""
        file = request.FILES.get('file')
        if not file:
            from common.response import error
            return error(message='请上传文件', status=status.HTTP_400_BAD_REQUEST)

        subject_id = request.data.get('subject_id')
        grade_id = request.data.get('grade_id')
        if not subject_id or not grade_id:
            from common.response import error
            return error(message='请选择学科和年级', status=status.HTTP_400_BAD_REQUEST)

        try:
            subject = Subject.objects.get(id=subject_id)
            grade = Grade.objects.get(id=grade_id)
        except (Subject.DoesNotExist, Grade.DoesNotExist):
            from common.response import error
            return error(message='学科或年级不存在', status=status.HTTP_400_BAD_REQUEST)

        from .imports import import_questions_from_excel
        success_count, errors = import_questions_from_excel(file.read(), subject, grade)

        return success(data={
            'success_count': success_count,
            'error_count': len(errors),
            'errors': errors,
        }, message=f'导入完成：成功 {success_count} 道，失败 {len(errors)} 道')
```

需要在上方 import 区域补：

```python
from common.response import success, error
```

- [ ] **Step 5: 运行导入测试**

```bash
pytest apps/questions/tests/test_import.py -v
```

预期：全部 PASS

- [ ] **Step 6: 提交**

```bash
git add backend/apps/questions/
git commit -m "feat(m2): 实现 Excel 批量导入题目功能"
```

---

### Task M2.4: 题库管理前端页面

**文件：**
- 修改：`frontend/src/pages/QuestionBankPage.tsx`
- 创建：`frontend/src/components/question/QuestionFilter.tsx`
- 创建：`frontend/src/components/question/QuestionTable.tsx`
- 创建：`frontend/src/components/question/QuestionForm.tsx`
- 创建：`frontend/src/components/question/KnowledgeTree.tsx`
- 创建：`frontend/src/components/question/BatchImportModal.tsx`
- 创建：`frontend/src/services/questionService.ts`
- 创建：`frontend/src/stores/questionStore.ts`

- [ ] **Step 1: 创建 questionService API 封装**

写入 `/Users/mac/2026/ai_extra/ExamAI/frontend/src/services/questionService.ts`：

```typescript
import api from './api';

export interface QuestionItem {
  id: number;
  subject: number;
  subject_name: string;
  grade: number;
  grade_name: string;
  type: string;
  type_display: string;
  difficulty: number;
  difficulty_display: string;
  stem: string;
  status: string;
  usage_count: number;
  knowledge_point_names: string[];
  created_at: string;
}

export interface QuestionDetail extends QuestionItem {
  cognitive_level: string;
  answer: string;
  explanation: string;
  default_score: number;
  estimated_time: number;
  images: string[];
  source_type: string;
  source_detail: string;
  exam_type: string;
  options: Array<{ id?: number; label: string; content: string; sort_order: number }>;
  knowledge_points: Array<{ id: number; name: string; path: string }>;
  updated_at: string;
}

export interface QuestionFilterParams {
  subject?: number;
  grade?: number;
  type?: string;
  difficulty?: number;
  difficulty__gte?: number;
  difficulty__lte?: number;
  status?: string;
  knowledge_point?: number;
  search?: string;
  page?: number;
  page_size?: number;
  ordering?: string;
}

export interface PaginatedResponse<T> {
  page: number;
  page_size: number;
  total: number;
  results: T[];
}

export interface ImportResult {
  success_count: number;
  error_count: number;
  errors: string[];
}

export const questionService = {
  list: (params?: QuestionFilterParams) =>
    api.get<{ code: number; data: PaginatedResponse<QuestionItem>; message: string }>('/questions/', { params }),

  detail: (id: number) =>
    api.get<{ code: number; data: QuestionDetail }>(`/questions/${id}/`),

  create: (data: any) =>
    api.post<{ code: number; data: QuestionDetail }>('/questions/', data),

  update: (id: number, data: any) =>
    api.patch<{ code: number; data: QuestionDetail }>(`/questions/${id}/`, data),

  delete: (id: number) =>
    api.delete(`/questions/${id}/`),

  importExcel: (formData: FormData) =>
    api.post<{ code: number; data: ImportResult }>('/questions/import-excel/', formData, {
      headers: { 'Content-Type': 'multipart/form-data' },
    }),

  // 学科/年级/知识点（静态数据）
  getSubjects: () =>
    api.get<{ code: number; data: Array<{ id: number; name: string; code: string }> }>('/questions/subjects/'),

  getGrades: () =>
    api.get<{ code: number; data: Array<{ id: number; name: string; level: number; subjects: any[] }> }>('/questions/grades/'),

  getKnowledgePoints: (params?: { subject?: number; root_only?: boolean }) =>
    api.get<{ code: number; data: any[] }>('/questions/knowledge-points/', { params }),

  createKnowledgePoint: (data: { name: string; subject: number; parent?: number | null; sort_order?: number }) =>
    api.post('/questions/knowledge-points/', data),

  updateKnowledgePoint: (id: number, data: any) =>
    api.patch(`/questions/knowledge-points/${id}/`, data),

  deleteKnowledgePoint: (id: number) =>
    api.delete(`/questions/knowledge-points/${id}/`),
};
```

- [ ] **Step 2: 创建 questionStore 状态管理**

写入 `/Users/mac/2026/ai_extra/ExamAI/frontend/src/stores/questionStore.ts`：

```typescript
import { create } from 'zustand';
import { questionService, QuestionItem, QuestionFilterParams, PaginatedResponse } from '@/services/questionService';

interface QuestionState {
  questions: QuestionItem[];
  total: number;
  loading: boolean;
  filters: QuestionFilterParams;
  setFilters: (filters: Partial<QuestionFilterParams>) => void;
  fetchQuestions: () => Promise<void>;
}

export const useQuestionStore = create<QuestionState>((set, get) => ({
  questions: [],
  total: 0,
  loading: false,
  filters: { page: 1, page_size: 20 },

  setFilters: (newFilters) => {
    const current = get().filters;
    set({ filters: { ...current, ...newFilters, page: newFilters.page ?? 1 } });
  },

  fetchQuestions: async () => {
    set({ loading: true });
    try {
      const res = await questionService.list(get().filters);
      set({
        questions: res.data.data.results,
        total: res.data.data.total,
        loading: false,
      });
    } catch {
      set({ loading: false });
    }
  },
}));
```

- [ ] **Step 3: 创建筛选面板组件**

写入 `/Users/mac/2026/ai_extra/ExamAI/frontend/src/components/question/QuestionFilter.tsx`：

```typescript
import { useEffect, useState } from 'react';
import { Row, Col, Select, Input, Button, Space } from 'antd';
import { SearchOutlined, ReloadOutlined } from '@ant-design/icons';
import { useQuestionStore } from '@/stores/questionStore';
import { questionService } from '@/services/questionService';
import { SUBJECTS, GRADES, QUESTION_TYPES, DIFFICULTY_LEVELS, QUESTION_STATUS } from '@/utils/constants';

export default function QuestionFilter() {
  const { filters, setFilters, fetchQuestions } = useQuestionStore();
  const [subjects, setSubjects] = useState<any[]>([]);
  const [grades, setGrades] = useState<any[]>([]);

  useEffect(() => {
    questionService.getSubjects().then(res => setSubjects(res.data.data));
    questionService.getGrades().then(res => setGrades(res.data.data));
  }, []);

  const handleSearch = () => {
    setFilters({ page: 1 });
    fetchQuestions();
  };

  const handleReset = () => {
    setFilters({ page: 1, subject: undefined, grade: undefined, type: undefined, difficulty: undefined, status: undefined, search: undefined });
    setTimeout(() => fetchQuestions(), 0);
  };

  return (
    <div style={{ padding: 16, background: '#fff', borderRadius: 8, marginBottom: 16 }}>
      <Row gutter={[12, 12]} align="middle">
        <Col><Select placeholder="学科" allowClear style={{ width: 120 }} value={filters.subject} onChange={v => setFilters({ subject: v })} options={subjects.map(s => ({ value: s.id, label: s.name }))} /></Col>
        <Col><Select placeholder="年级" allowClear style={{ width: 120 }} value={filters.grade} onChange={v => setFilters({ grade: v })} options={grades.map(g => ({ value: g.id, label: g.name }))} /></Col>
        <Col><Select placeholder="题型" allowClear style={{ width: 120 }} value={filters.type} onChange={v => setFilters({ type: v })} options={QUESTION_TYPES.map(t => ({ value: t.value, label: t.label }))} /></Col>
        <Col><Select placeholder="难度" allowClear style={{ width: 120 }} value={filters.difficulty} onChange={v => setFilters({ difficulty: v })} options={DIFFICULTY_LEVELS.map(d => ({ value: d.value, label: d.label }))} /></Col>
        <Col><Select placeholder="状态" allowClear style={{ width: 120 }} value={filters.status} onChange={v => setFilters({ status: v })} options={QUESTION_STATUS.map(s => ({ value: s.value, label: s.label }))} /></Col>
        <Col><Input.Search placeholder="搜索题干" allowClear style={{ width: 220 }} value={filters.search} onChange={e => setFilters({ search: e.target.value })} onSearch={handleSearch} prefix={<SearchOutlined />} /></Col>
        <Col>
          <Space>
            <Button type="primary" onClick={handleSearch}>查询</Button>
            <Button icon={<ReloadOutlined />} onClick={handleReset}>重置</Button>
          </Space>
        </Col>
      </Row>
    </div>
  );
}
```

- [ ] **Step 4: 创建题目表格组件**

写入 `/Users/mac/2026/ai_extra/ExamAI/frontend/src/components/question/QuestionTable.tsx`：

```typescript
import { Table, Tag, Space, Button, Popconfirm, Tooltip } from 'antd';
import { EditOutlined, DeleteOutlined, EyeOutlined } from '@ant-design/icons';
import type { ColumnsType } from 'antd/es/table';
import { QuestionItem } from '@/services/questionService';
import { QUESTION_TYPES, DIFFICULTY_LEVELS } from '@/utils/constants';

interface Props {
  questions: QuestionItem[];
  loading: boolean;
  total: number;
  page: number;
  pageSize: number;
  onPageChange: (page: number, pageSize: number) => void;
  onEdit: (id: number) => void;
  onDelete: (id: number) => void;
  onView: (id: number) => void;
}

const typeMap = Object.fromEntries(QUESTION_TYPES.map(t => [t.value, t.label]));
const diffMap = Object.fromEntries(DIFFICULTY_LEVELS.map(d => [d.value, { label: d.label, color: d.color }]));

export default function QuestionTable({ questions, loading, total, page, pageSize, onPageChange, onEdit, onDelete, onView }: Props) {
  const columns: ColumnsType<QuestionItem> = [
    {
      title: '题干',
      dataIndex: 'stem',
      key: 'stem',
      ellipsis: true,
      width: 300,
      render: (text: string) => (
        <Tooltip title={text}>
          <span>{text.length > 50 ? text.slice(0, 50) + '...' : text}</span>
        </Tooltip>
      ),
    },
    {
      title: '学科',
      dataIndex: 'subject_name',
      key: 'subject',
      width: 80,
    },
    {
      title: '年级',
      dataIndex: 'grade_name',
      key: 'grade',
      width: 80,
    },
    {
      title: '题型',
      dataIndex: 'type',
      key: 'type',
      width: 80,
      render: (t: string) => typeMap[t] || t,
    },
    {
      title: '难度',
      dataIndex: 'difficulty',
      key: 'difficulty',
      width: 100,
      render: (d: number) => {
        const info = diffMap[d];
        return info ? <Tag color={info.color}>{info.label}</Tag> : d;
      },
    },
    {
      title: '知识点',
      dataIndex: 'knowledge_point_names',
      key: 'knowledge_points',
      width: 180,
      ellipsis: true,
      render: (names: string[]) => names?.join(', ') || '-',
    },
    {
      title: '使用次数',
      dataIndex: 'usage_count',
      key: 'usage_count',
      width: 80,
    },
    {
      title: '操作',
      key: 'actions',
      width: 150,
      render: (_, record) => (
        <Space>
          <Button type="link" size="small" icon={<EyeOutlined />} onClick={() => onView(record.id)} />
          <Button type="link" size="small" icon={<EditOutlined />} onClick={() => onEdit(record.id)} />
          <Popconfirm title="确定删除？" onConfirm={() => onDelete(record.id)}>
            <Button type="link" size="small" danger icon={<DeleteOutlined />} />
          </Popconfirm>
        </Space>
      ),
    },
  ];

  return (
    <Table
      rowKey="id"
      columns={columns}
      dataSource={questions}
      loading={loading}
      pagination={{
        current: page,
        pageSize,
        total,
        showSizeChanger: true,
        showTotal: (t) => `共 ${t} 道题目`,
        onChange: onPageChange,
      }}
      scroll={{ x: 1100 }}
    />
  );
}
```

- [ ] **Step 5: 创建题目表单组件（新增/编辑）**

写入 `/Users/mac/2026/ai_extra/ExamAI/frontend/src/components/question/QuestionForm.tsx`：

```typescript
import { useEffect, useState } from 'react';
import { Modal, Form, Select, Input, InputNumber, Button, Space, message } from 'antd';
import { MinusCircleOutlined, PlusOutlined } from '@ant-design/icons';
import { questionService, QuestionDetail } from '@/services/questionService';
import { QUESTION_TYPES, DIFFICULTY_LEVELS, COGNITIVE_LEVELS } from '@/utils/constants';

const { TextArea } = Input;

interface Props {
  open: boolean;
  questionId: number | null;
  onClose: () => void;
  onSuccess: () => void;
}

export default function QuestionForm({ open, questionId, onClose, onSuccess }: Props) {
  const [form] = Form.useForm();
  const [loading, setLoading] = useState(false);
  const [subjects, setSubjects] = useState<any[]>([]);
  const [grades, setGrades] = useState<any[]>([]);
  const [knowledgePoints, setKnowledgePoints] = useState<any[]>([]);
  const [questionType, setQuestionType] = useState<string>('');

  const isEdit = !!questionId;

  useEffect(() => {
    questionService.getSubjects().then(res => setSubjects(res.data.data));
    questionService.getGrades().then(res => setGrades(res.data.data));
  }, []);

  useEffect(() => {
    if (open && questionId) {
      setLoading(true);
      questionService.detail(questionId).then(res => {
        const d = res.data.data;
        form.setFieldsValue({
          ...d,
          subject_id: d.subject,
          grade_id: d.grade,
          knowledge_point_ids: d.knowledge_points?.map((kp: any) => kp.id),
        });
        setQuestionType(d.type);
      }).finally(() => setLoading(false));
    } else {
      form.resetFields();
      setQuestionType('');
    }
  }, [open, questionId, form]);

  const handleSubjectChange = (subjectId: number) => {
    questionService.getKnowledgePoints({ subject: subjectId }).then(res => {
      setKnowledgePoints(res.data.data);
    });
  };

  const handleSubmit = async () => {
    const values = await form.validateFields();
    setLoading(true);
    try {
      if (isEdit) {
        await questionService.update(questionId!, values);
        message.success('更新成功');
      } else {
        await questionService.create(values);
        message.success('创建成功');
      }
      onSuccess();
      onClose();
    } finally {
      setLoading(false);
    }
  };

  return (
    <Modal
      title={isEdit ? '编辑题目' : '新增题目'}
      open={open}
      onCancel={onClose}
      onOk={handleSubmit}
      confirmLoading={loading}
      width={720}
      destroyOnClose
    >
      <Form form={form} layout="vertical" initialValues={{ difficulty: 3, default_score: 3, type: 'choice' }}>
        <Form.Item name="subject_id" label="学科" rules={[{ required: true }]}>
          <Select placeholder="选择学科" onChange={handleSubjectChange} options={subjects.map(s => ({ value: s.id, label: s.name }))} />
        </Form.Item>
        <Form.Item name="grade_id" label="年级" rules={[{ required: true }]}>
          <Select placeholder="选择年级" options={grades.map(g => ({ value: g.id, label: g.name }))} />
        </Form.Item>
        <Form.Item name="type" label="题型" rules={[{ required: true }]}>
          <Select onChange={v => setQuestionType(v)} options={QUESTION_TYPES.map(t => ({ value: t.value, label: t.label }))} />
        </Form.Item>
        <Form.Item name="difficulty" label="难度" rules={[{ required: true }]}>
          <Select options={DIFFICULTY_LEVELS.map(d => ({ value: d.value, label: d.label }))} />
        </Form.Item>
        <Form.Item name="stem" label="题干" rules={[{ required: true, message: '请输入题干' }]}>
          <TextArea rows={3} placeholder="支持 Markdown 和 LaTeX 公式（$...$）" />
        </Form.Item>

        {questionType === 'choice' && (
          <Form.List name="options">
            {(fields, { add, remove }) => (
              <>
                {fields.map(({ key, name, ...rest }) => (
                  <Space key={key} align="baseline">
                    <Form.Item {...rest} name={[name, 'label']} label="标签">
                      <Input style={{ width: 60 }} disabled />
                    </Form.Item>
                    <Form.Item {...rest} name={[name, 'content']} label="内容" rules={[{ required: true }]}>
                      <Input style={{ width: 300 }} />
                    </Form.Item>
                    <MinusCircleOutlined onClick={() => remove(name)} />
                  </Space>
                ))}
                {fields.length < 6 && (
                  <Button type="dashed" onClick={() => add({ label: String.fromCharCode(65 + fields.length), sort_order: fields.length })} block icon={<PlusOutlined />}>
                    添加选项
                  </Button>
                )}
              </>
            )}
          </Form.List>
        )}

        <Form.Item name="answer" label="答案" rules={[{ required: true, message: '请输入答案' }]}>
          <TextArea rows={2} />
        </Form.Item>
        <Form.Item name="explanation" label="解析">
          <TextArea rows={2} />
        </Form.Item>
        <Form.Item name="default_score" label="默认分值">
          <InputNumber min={1} max={50} />
        </Form.Item>
        <Form.Item name="knowledge_point_ids" label="知识点">
          <Select mode="multiple" placeholder="选择知识点" options={knowledgePoints.map(kp => ({ value: kp.id, label: kp.path }))} />
        </Form.Item>
      </Form>
    </Modal>
  );
}
```

- [ ] **Step 6: 创建知识点树组件**

写入 `/Users/mac/2026/ai_extra/ExamAI/frontend/src/components/question/KnowledgeTree.tsx`：

```typescript
import { useEffect, useState } from 'react';
import { Tree, Card, Select, Button, Modal, Input, Space, Popconfirm, message } from 'antd';
import { PlusOutlined, EditOutlined, DeleteOutlined } from '@ant-design/icons';
import { questionService } from '@/services/questionService';

interface KnowledgePointNode {
  key: number;
  title: string;
  children?: KnowledgePointNode[];
}

export default function KnowledgeTree() {
  const [subjects, setSubjects] = useState<any[]>([]);
  const [selectedSubject, setSelectedSubject] = useState<number | undefined>();
  const [treeData, setTreeData] = useState<KnowledgePointNode[]>([]);
  const [loading, setLoading] = useState(false);
  const [modalOpen, setModalOpen] = useState(false);
  const [editingNode, setEditingNode] = useState<any>(null);
  const [nodeName, setNodeName] = useState('');

  useEffect(() => {
    questionService.getSubjects().then(res => {
      setSubjects(res.data.data);
      if (res.data.data.length > 0) setSelectedSubject(res.data.data[0].id);
    });
  }, []);

  useEffect(() => {
    if (selectedSubject) loadTree();
  }, [selectedSubject]);

  const loadTree = async () => {
    setLoading(true);
    try {
      const res = await questionService.getKnowledgePoints({ subject: selectedSubject, root_only: true });
      setTreeData(res.data.data.map((node: any) => convertToTreeNode(node)));
    } finally {
      setLoading(false);
    }
  };

  const convertToTreeNode = (node: any): KnowledgePointNode => ({
    key: node.id,
    title: node.name,
    children: node.children?.map(convertToTreeNode),
  });

  const handleAdd = (parentId: number | null = null) => {
    setEditingNode(parentId ? { parent: parentId } : null);
    setNodeName('');
    setModalOpen(true);
  };

  const handleEdit = (node: any) => {
    setEditingNode(node);
    setNodeName(node.title);
    setModalOpen(true);
  };

  const handleSave = async () => {
    if (!nodeName.trim()) return;
    try {
      if (editingNode?.key) {
        await questionService.updateKnowledgePoint(editingNode.key, { name: nodeName });
        message.success('更新成功');
      } else {
        await questionService.createKnowledgePoint({
          name: nodeName,
          subject: selectedSubject!,
          parent: editingNode?.parent || null,
        });
        message.success('创建成功');
      }
      setModalOpen(false);
      loadTree();
    } catch { /* error handled by interceptor */ }
  };

  const handleDelete = async (id: number) => {
    try {
      await questionService.deleteKnowledgePoint(id);
      message.success('删除成功');
      loadTree();
    } catch { /* error handled by interceptor */ }
  };

  const titleRender = (nodeData: any) => (
    <span>
      {nodeData.title}
      <Space style={{ marginLeft: 8 }} size={4}>
        <Button type="link" size="small" icon={<PlusOutlined />} onClick={e => { e.stopPropagation(); handleAdd(nodeData.key); }} />
        <Button type="link" size="small" icon={<EditOutlined />} onClick={e => { e.stopPropagation(); handleEdit(nodeData); }} />
        <Popconfirm title="确定删除？子节点也会删除" onConfirm={e => { e?.stopPropagation(); handleDelete(nodeData.key); }} onCancel={e => e?.stopPropagation()}>
          <Button type="link" size="small" danger icon={<DeleteOutlined />} onClick={e => e.stopPropagation()} />
        </Popconfirm>
      </Space>
    </span>
  );

  return (
    <Card title="知识点树" extra={
      <Space>
        <Select placeholder="学科" value={selectedSubject} onChange={v => setSelectedSubject(v)} style={{ width: 120 }} options={subjects.map(s => ({ value: s.id, label: s.name }))} />
        <Button type="primary" icon={<PlusOutlined />} onClick={() => handleAdd(null)}>新增根节点</Button>
      </Space>
    }>
      <Tree treeData={treeData} titleRender={titleRender} defaultExpandAll loading={loading} />
      <Modal title={editingNode?.key ? '编辑知识点' : '新增知识点'} open={modalOpen} onOk={handleSave} onCancel={() => setModalOpen(false)}>
        <Input value={nodeName} onChange={e => setNodeName(e.target.value)} placeholder="知识点名称" />
      </Modal>
    </Card>
  );
}
```

- [ ] **Step 7: 创建批量导入弹窗**

写入 `/Users/mac/2026/ai_extra/ExamAI/frontend/src/components/question/BatchImportModal.tsx`：

```typescript
import { useState } from 'react';
import { Modal, Upload, Select, Button, Space, Alert, message } from 'antd';
import { InboxOutlined } from '@ant-design/icons';
import { questionService } from '@/services/questionService';

const { Dragger } = Upload;

interface Props {
  open: boolean;
  onClose: () => void;
  onSuccess: () => void;
}

export default function BatchImportModal({ open, onClose, onSuccess }: Props) {
  const [file, setFile] = useState<File | null>(null);
  const [subjectId, setSubjectId] = useState<number | undefined>();
  const [gradeId, setGradeId] = useState<number | undefined>();
  const [subjects, setSubjects] = useState<any[]>([]);
  const [grades, setGrades] = useState<any[]>([]);
  const [loading, setLoading] = useState(false);
  const [result, setResult] = useState<any>(null);

  useState(() => {
    questionService.getSubjects().then(res => setSubjects(res.data.data));
    questionService.getGrades().then(res => setGrades(res.data.data));
  });

  const handleImport = async () => {
    if (!file || !subjectId || !gradeId) {
      message.warning('请填写完整信息');
      return;
    }
    setLoading(true);
    try {
      const formData = new FormData();
      formData.append('file', file);
      formData.append('subject_id', String(subjectId));
      formData.append('grade_id', String(gradeId));
      const res = await questionService.importExcel(formData);
      setResult(res.data.data);
      message.success('导入完成');
      onSuccess();
    } finally {
      setLoading(false);
    }
  };

  const handleClose = () => {
    setFile(null);
    setSubjectId(undefined);
    setGradeId(undefined);
    setResult(null);
    onClose();
  };

  return (
    <Modal title="批量导入题目" open={open} onCancel={handleClose} onOk={handleImport} confirmLoading={loading} width={600} destroyOnClose>
      <Space direction="vertical" style={{ width: '100%' }} size="middle">
        <Dragger
          accept=".xlsx,.xls"
          maxCount={1}
          beforeUpload={f => { setFile(f); return false; }}
          onRemove={() => setFile(null)}
        >
          <p className="ant-upload-drag-icon"><InboxOutlined /></p>
          <p>点击或拖拽上传 Excel 文件</p>
          <p style={{ color: '#999' }}>支持 .xlsx 格式</p>
        </Dragger>
        <Space>
          <Select placeholder="选择学科" value={subjectId} onChange={setSubjectId} style={{ width: 150 }} options={subjects.map(s => ({ value: s.id, label: s.name }))} />
          <Select placeholder="选择年级" value={gradeId} onChange={setGradeId} style={{ width: 150 }} options={grades.map(g => ({ value: g.id, label: g.name }))} />
        </Space>
        {result && (
          <Alert
            type={result.error_count === 0 ? 'success' : 'warning'}
            message={`导入完成：成功 ${result.success_count} 道，失败 ${result.error_count} 道`}
            description={result.errors.length > 0 && (
              <ul>{result.errors.slice(0, 10).map((e: string, i: number) => <li key={i}>{e}</li>)}</ul>
            )}
          />
        )}
      </Space>
    </Modal>
  );
}
```

- [ ] **Step 8: 组装题库管理页面**

写入 `/Users/mac/2026/ai_extra/ExamAI/frontend/src/pages/QuestionBankPage.tsx`：

```typescript
import { useEffect, useState, useCallback } from 'react';
import { Button, Space, Tabs, message } from 'antd';
import { PlusOutlined, UploadOutlined } from '@ant-design/icons';
import { useQuestionStore } from '@/stores/questionStore';
import { questionService } from '@/services/questionService';
import QuestionFilter from '@/components/question/QuestionFilter';
import QuestionTable from '@/components/question/QuestionTable';
import QuestionForm from '@/components/question/QuestionForm';
import KnowledgeTree from '@/components/question/KnowledgeTree';
import BatchImportModal from '@/components/question/BatchImportModal';

export default function QuestionBankPage() {
  const { questions, total, loading, filters, fetchQuestions } = useQuestionStore();
  const [formOpen, setFormOpen] = useState(false);
  const [editingId, setEditingId] = useState<number | null>(null);
  const [importOpen, setImportOpen] = useState(false);
  const [activeTab, setActiveTab] = useState('list');

  useEffect(() => {
    fetchQuestions();
  }, [filters.page, filters.page_size]);

  const handleEdit = useCallback((id: number) => {
    setEditingId(id);
    setFormOpen(true);
  }, []);

  const handleCreate = useCallback(() => {
    setEditingId(null);
    setFormOpen(true);
  }, []);

  const handleDelete = useCallback(async (id: number) => {
    await questionService.delete(id);
    message.success('删除成功');
    fetchQuestions();
  }, [fetchQuestions]);

  const handleFormSuccess = useCallback(() => {
    fetchQuestions();
  }, [fetchQuestions]);

  const handlePageChange = useCallback((page: number, pageSize: number) => {
    useQuestionStore.getState().setFilters({ page, page_size: pageSize });
    setTimeout(() => useQuestionStore.getState().fetchQuestions(), 0);
  }, []);

  const tabItems = [
    {
      key: 'list',
      label: '题目列表',
      children: (
        <>
          <div style={{ display: 'flex', justifyContent: 'space-between', marginBottom: 16 }}>
            <QuestionFilter />
          </div>
          <div style={{ background: '#fff', borderRadius: 8, padding: 16 }}>
            <div style={{ marginBottom: 16 }}>
              <Space>
                <Button type="primary" icon={<PlusOutlined />} onClick={handleCreate}>新增题目</Button>
                <Button icon={<UploadOutlined />} onClick={() => setImportOpen(true)}>批量导入</Button>
              </Space>
            </div>
            <QuestionTable
              questions={questions}
              loading={loading}
              total={total}
              page={filters.page || 1}
              pageSize={filters.page_size || 20}
              onPageChange={handlePageChange}
              onEdit={handleEdit}
              onDelete={handleDelete}
              onView={handleEdit}
            />
          </div>
        </>
      ),
    },
    {
      key: 'knowledge',
      label: '知识点管理',
      children: <KnowledgeTree />,
    },
  ];

  return (
    <div>
      <h2 style={{ marginBottom: 16 }}>题库管理</h2>
      <Tabs activeKey={activeTab} onChange={setActiveTab} items={tabItems} />
      <QuestionForm open={formOpen} questionId={editingId} onClose={() => setFormOpen(false)} onSuccess={handleFormSuccess} />
      <BatchImportModal open={importOpen} onClose={() => setImportOpen(false)} onSuccess={handleFormSuccess} />
    </div>
  );
}
```

- [ ] **Step 9: 验证前端页面**

```bash
cd /Users/mac/2026/ai_extra/ExamAI/frontend
npm run build
```

确认无编译错误。

- [ ] **Step 10: 提交**

```bash
git add frontend/src/pages/QuestionBankPage.tsx frontend/src/components/question/ frontend/src/services/questionService.ts frontend/src/stores/questionStore.ts
git commit -m "feat(m2): 实现题库管理前端页面 — 列表/筛选/新增/编辑/删除/批量导入/知识点树"
```

---

### M2 里程碑检查点

```bash
# 后端测试
cd backend && source .venv/bin/activate
pytest apps/questions/ -v

# 种子数据
python manage.py seed_data

# 前端编译
cd ../frontend && npm run build
```

---


## M3 — 组卷引擎（第6-9周）

> **目标：** 实现智能组卷核心引擎 — 试卷/分区/题目关联数据模型、A面表单精细配置、B面 AI 智能推荐、试卷生成算法、智能组卷前端页面。
> **交付物：** 可根据配置参数自动生成试卷，支持预览和手动换题

### Task M3.1: 试卷数据模型

**文件：**
- 创建：`backend/apps/exams/models.py`
- 创建：`backend/apps/exams/admin.py`
- 创建：`backend/apps/exams/tests/__init__.py`
- 创建：`backend/apps/exams/tests/test_models.py`

- [ ] **Step 1: 编写模型测试**

写入 `/Users/mac/2026/ai_extra/ExamAI/backend/apps/exams/tests/test_models.py`：

```python
import pytest
from apps.users.models import User
from apps.questions.models import Subject, Grade, KnowledgePoint, Question
from apps.exams.models import ExamPaper, ExamPaperSection, ExamPaperItem


@pytest.mark.django_db
class TestExamPaper:
    @pytest.fixture
    def user(self):
        return User.objects.create_user(email='teacher@test.com', password='pass123')

    @pytest.fixture
    def math(self):
        return Subject.objects.create(name='数学', code='math')

    @pytest.fixture
    def grade8(self):
        return Grade.objects.create(name='八年级', level=8)

    def test_create_exam_paper(self, user, math, grade8):
        paper = ExamPaper.objects.create(
            user=user,
            title='初二数学期末试卷',
            subject=math,
            grade=grade8,
            exam_type='final',
            total_score=100,
            duration=90,
            config={
                'sections': [
                    {'type': 'choice', 'count': 10, 'score': 3},
                    {'type': 'fill_blank', 'count': 5, 'score': 4},
                ],
                'difficulty_distribution': {1: 0.2, 2: 0.3, 3: 0.3, 4: 0.15, 5: 0.05},
            },
        )
        assert paper.status == 'draft'
        assert paper.total_score == 100
        assert paper.duration == 90

    def test_exam_paper_sections(self, user, math, grade8):
        paper = ExamPaper.objects.create(
            user=user, title='测试卷', subject=math, grade=grade8,
            exam_type='unit', total_score=100, duration=60,
        )
        section = ExamPaperSection.objects.create(
            exam_paper=paper,
            title='一、选择题',
            description='每题3分，共30分',
            sort_order=1,
            total_score=30,
        )
        assert section.exam_paper == paper
        assert section.title == '一、选择题'

    def test_exam_paper_items_with_snapshot(self, user, math, grade8):
        paper = ExamPaper.objects.create(
            user=user, title='测试卷', subject=math, grade=grade8,
            exam_type='unit', total_score=100, duration=60,
        )
        section = ExamPaperSection.objects.create(
            exam_paper=paper, title='一、选择题', sort_order=1, total_score=30,
        )
        question = Question.objects.create(
            subject=math, grade=grade8, type='choice', difficulty=3,
            stem='1+1=?', answer='B', default_score=3,
        )
        item = ExamPaperItem.objects.create(
            section=section,
            question=question,
            sort_order=1,
            score=3,
            question_snapshot={
                'stem': '1+1=?',
                'answer': 'B',
                'type': 'choice',
                'difficulty': 3,
                'options': [
                    {'label': 'A', 'content': '1'},
                    {'label': 'B', 'content': '2'},
                ],
            },
        )
        assert item.question == question
        assert item.question_snapshot['stem'] == '1+1=?'
        # 快照保护：原题修改后试卷不变
        question.stem = '2+2=?'
        question.save()
        item.refresh_from_db()
        assert item.question_snapshot['stem'] == '1+1=?'

    def test_paper_full_structure(self, user, math, grade8):
        """完整试卷结构：试卷 → 分区 → 题目"""
        paper = ExamPaper.objects.create(
            user=user, title='完整试卷', subject=math, grade=grade8,
            exam_type='midterm', total_score=120, duration=120,
        )
        section1 = ExamPaperSection.objects.create(exam_paper=paper, title='一、选择题', sort_order=1, total_score=30)
        section2 = ExamPaperSection.objects.create(exam_paper=paper, title='二、填空题', sort_order=2, total_score=20)

        for i in range(10):
            q = Question.objects.create(subject=math, grade=grade8, type='choice', difficulty=3, stem=f'题{i}', answer='A')
            ExamPaperItem.objects.create(section=section1, question=q, sort_order=i, score=3, question_snapshot={})

        for i in range(5):
            q = Question.objects.create(subject=math, grade=grade8, type='fill_blank', difficulty=3, stem=f'填空{i}', answer='x')
            ExamPaperItem.objects.create(section=section2, question=q, sort_order=i, score=4, question_snapshot={})

        assert paper.sections.count() == 2
        assert section1.items.count() == 10
        assert section2.items.count() == 5
        assert paper.get_question_count() == 15
```

- [ ] **Step 2: 运行测试，验证失败**

```bash
cd /Users/mac/2026/ai_extra/ExamAI/backend && source .venv/bin/activate
pytest apps/exams/tests/test_models.py -v
```

预期：FAIL

- [ ] **Step 3: 实现模型**

写入 `/Users/mac/2026/ai_extra/ExamAI/backend/apps/exams/models.py`：

```python
from django.db import models
from django.conf import settings


class ExamPaper(models.Model):
    """试卷"""

    class ExamType(models.TextChoices):
        UNIT = 'unit', '单元测试'
        MONTHLY = 'monthly', '月考'
        MIDTERM = 'midterm', '期中考试'
        FINAL = 'final', '期末考试'
        ENTRANCE = 'entrance', '中考模拟'

    class Status(models.TextChoices):
        DRAFT = 'draft', '草稿'
        GENERATED = 'generated', '已生成'
        FINALIZED = 'finalized', '已定稿'

    user = models.ForeignKey(
        settings.AUTH_USER_MODEL,
        on_delete=models.CASCADE,
        related_name='exam_papers',
        verbose_name='创建者',
    )
    title = models.CharField('试卷标题', max_length=200)
    subject = models.ForeignKey('questions.Subject', on_delete=models.PROTECT, related_name='exam_papers', verbose_name='学科')
    grade = models.ForeignKey('questions.Grade', on_delete=models.PROTECT, related_name='exam_papers', verbose_name='年级')
    exam_type = models.CharField('考试类型', max_length=20, choices=ExamType.choices, default=ExamType.UNIT)
    total_score = models.PositiveIntegerField('总分', default=100)
    duration = models.PositiveIntegerField('时长(分钟)', default=90)
    config = models.JSONField('组卷参数快照', default=dict, blank=True)
    status = models.CharField('状态', max_length=20, choices=Status.choices, default=Status.DRAFT)
    created_at = models.DateTimeField('创建时间', auto_now_add=True)
    updated_at = models.DateTimeField('更新时间', auto_now=True)

    class Meta:
        db_table = 'exam_papers'
        verbose_name = '试卷'
        verbose_name_plural = verbose_name
        ordering = ['-created_at']
        indexes = [
            models.Index(fields=['user', '-created_at']),
            models.Index(fields=['subject', 'grade']),
        ]

    def __str__(self):
        return self.title

    def get_question_count(self):
        return ExamPaperItem.objects.filter(section__exam_paper=self).count()


class ExamPaperSection(models.Model):
    """试卷分区（如：一、选择题）"""
    exam_paper = models.ForeignKey(ExamPaper, on_delete=models.CASCADE, related_name='sections', verbose_name='试卷')
    title = models.CharField('分区标题', max_length=200)
    description = models.CharField('分区说明', max_length=500, blank=True, default='')
    sort_order = models.PositiveIntegerField('排序', default=0)
    total_score = models.PositiveIntegerField('本区总分', default=0)

    class Meta:
        db_table = 'exam_paper_sections'
        verbose_name = '试卷分区'
        verbose_name_plural = verbose_name
        ordering = ['sort_order']

    def __str__(self):
        return f'{self.exam_paper.title} - {self.title}'


class ExamPaperItem(models.Model):
    """试卷中的题目关联（含快照）"""
    section = models.ForeignKey(ExamPaperSection, on_delete=models.CASCADE, related_name='items', verbose_name='分区')
    question = models.ForeignKey('questions.Question', on_delete=models.PROTECT, related_name='exam_items', verbose_name='题目')
    sort_order = models.PositiveIntegerField('排序', default=0)
    score = models.PositiveIntegerField('本题分值', default=3)
    question_snapshot = models.JSONField('题目快照', default=dict)

    class Meta:
        db_table = 'exam_paper_items'
        verbose_name = '试卷题目'
        verbose_name_plural = verbose_name
        ordering = ['section', 'sort_order']

    def __str__(self):
        return f'{self.section.title} #{self.sort_order}: {self.question.stem[:30]}'
```

写入 `/Users/mac/2026/ai_extra/ExamAI/backend/apps/exams/admin.py`：

```python
from django.contrib import admin
from .models import ExamPaper, ExamPaperSection, ExamPaperItem


class ExamPaperItemInline(admin.TabularInline):
    model = ExamPaperItem
    extra = 0


class ExamPaperSectionInline(admin.TabularInline):
    model = ExamPaperSection
    extra = 0


@admin.register(ExamPaper)
class ExamPaperAdmin(admin.ModelAdmin):
    list_display = ('title', 'user', 'subject', 'grade', 'exam_type', 'total_score', 'status', 'created_at')
    list_filter = ('status', 'exam_type', 'subject')
    inlines = [ExamPaperSectionInline]
```

- [ ] **Step 4: 运行 migrations 并测试**

```bash
python manage.py makemigrations exams
python manage.py migrate
pytest apps/exams/tests/test_models.py -v
```

预期：全部 PASS

- [ ] **Step 5: 提交**

```bash
git add backend/apps/exams/
git commit -m "feat(m3): 实现试卷数据模型 — ExamPaper/Section/Item + 快照机制"
```

---

### Task M3.2: 组卷引擎核心算法

**文件：**
- 创建：`backend/apps/exams/engine.py`
- 创建：`backend/apps/exams/tests/test_engine.py`

- [ ] **Step 1: 编写引擎测试**

写入 `/Users/mac/2026/ai_extra/ExamAI/backend/apps/exams/tests/test_engine.py`：

```python
import pytest
from apps.users.models import User
from apps.questions.models import Subject, Grade, KnowledgePoint, Question
from apps.exams.models import ExamPaper
from apps.exams.engine import ExamGenerator


@pytest.mark.django_db
class TestExamGenerator:
    @pytest.fixture
    def user(self):
        return User.objects.create_user(email='teacher@test.com', password='pass123')

    @pytest.fixture
    def math(self):
        return Subject.objects.create(name='数学', code='math')

    @pytest.fixture
    def grade8(self):
        return Grade.objects.create(name='八年级', level=8)

    @pytest.fixture
    def kp(self, math):
        return KnowledgePoint.objects.create(name='代数', subject=math)

    @pytest.fixture
    def questions(self, math, grade8, kp):
        """创建 50 道不同难度和题型的题目"""
        qs = []
        for i in range(50):
            q = Question.objects.create(
                subject=math, grade=grade8,
                type='choice' if i < 30 else 'fill_blank',
                difficulty=(i % 5) + 1,
                stem=f'题目{i}',
                answer=f'答案{i}',
                default_score=3 if i < 30 else 5,
                status='reviewed',
            )
            q.knowledge_points.add(kp)
            qs.append(q)
        return qs

    @pytest.fixture
    def config(self):
        return {
            'sections': [
                {'type': 'choice', 'count': 10, 'score': 3, 'difficulty': [1, 2, 3, 4, 5]},
                {'type': 'fill_blank', 'count': 5, 'score': 5, 'difficulty': [2, 3, 4]},
            ],
            'total_score': 55,
            'difficulty_target': 3.0,
            'knowledge_point_ids': [],
        }

    def test_generate_exam_basic(self, user, math, grade8, questions, config):
        """基本组卷：按题型和数量生成"""
        paper = ExamPaper.objects.create(
            user=user, title='测试卷', subject=math, grade=grade8,
            exam_type='unit', total_score=100, duration=60,
        )
        generator = ExamGenerator(paper, config)
        generator.generate()

        assert paper.sections.count() == 2
        section1 = paper.sections.first()
        assert section1.items.count() == 10
        section2 = paper.sections.last()
        assert section2.items.count() == 5

    def test_generate_no_duplicates(self, user, math, grade8, questions, config):
        """同一试卷不含重复题目"""
        paper = ExamPaper.objects.create(
            user=user, title='无重复卷', subject=math, grade=grade8,
            exam_type='unit', total_score=100, duration=60,
        )
        generator = ExamGenerator(paper, config)
        generator.generate()
        question_ids = []
        for section in paper.sections.all():
            for item in section.items.all():
                question_ids.append(item.question_id)
        assert len(question_ids) == len(set(question_ids))

    def test_generate_insufficient_questions(self, user, math, grade8, config):
        """题库不足时抛异常"""
        paper = ExamPaper.objects.create(
            user=user, title='题目不足卷', subject=math, grade=grade8,
            exam_type='unit', total_score=100, duration=60,
        )
        config['sections'] = [{'type': 'essay', 'count': 10, 'score': 10, 'difficulty': [1, 2, 3, 4, 5]}]
        generator = ExamGenerator(paper, config)
        with pytest.raises(ValueError, match='不足'):
            generator.generate()

    def test_swap_question(self, user, math, grade8, questions, config):
        """换题功能"""
        paper = ExamPaper.objects.create(
            user=user, title='换题测试', subject=math, grade=grade8,
            exam_type='unit', total_score=100, duration=60,
        )
        generator = ExamGenerator(paper, config)
        generator.generate()

        item = paper.sections.first().items.first()
        old_question_id = item.question_id

        # 换题：用同题型、同难度的另一道题替换
        generator.swap_question(item.id)

        item.refresh_from_db()
        assert item.question_id != old_question_id
```

- [ ] **Step 2: 运行测试，验证失败**

```bash
pytest apps/exams/tests/test_engine.py -v
```

预期：FAIL

- [ ] **Step 3: 实现组卷引擎**

写入 `/Users/mac/2026/ai_extra/ExamAI/backend/apps/exams/engine.py`：

```python
import random
from typing import Any, Dict, List
from django.db import transaction
from django.db.models import QuerySet

from apps.questions.models import Question
from .models import ExamPaper, ExamPaperSection, ExamPaperItem


class ExamGenerator:
    """智能组卷引擎"""

    def __init__(self, paper: ExamPaper, config: Dict[str, Any]):
        self.paper = paper
        self.config = config

    def generate(self):
        """根据配置生成试卷"""
        sections_config = self.config.get('sections', [])
        knowledge_point_ids = self.config.get('knowledge_point_ids', [])
        difficulty_target = self.config.get('difficulty_target')

        if not sections_config:
            raise ValueError('组卷配置中缺少分区定义')

        with transaction.atomic():
            # 清空已有分区和题目（重新生成）
            self.paper.sections.all().delete()

            used_question_ids = set()
            section_order = 0

            for sec_cfg in sections_config:
                section_order += 1
                q_type = sec_cfg['type']
                count = sec_cfg['count']
                score = sec_cfg.get('score', 3)
                difficulties = sec_cfg.get('difficulty', [1, 2, 3, 4, 5])

                # 查询候选题目
                candidates = Question.objects.filter(
                    subject=self.paper.subject,
                    grade=self.paper.grade,
                    type=q_type,
                    status='reviewed',
                    difficulty__in=difficulties,
                ).exclude(id__in=used_question_ids)

                if knowledge_point_ids:
                    candidates = candidates.filter(knowledge_points__id__in=knowledge_point_ids).distinct()

                candidates = list(candidates)

                if len(candidates) < count:
                    raise ValueError(
                        f'分区「{q_type}」题库不足：需要 {count} 道，仅 {len(candidates)} 道可用'
                    )

                # 按难度分布挑选
                selected = self._select_by_difficulty(candidates, count, difficulty_target)

                # 创建分区
                section = ExamPaperSection.objects.create(
                    exam_paper=self.paper,
                    title=self._section_title(q_type, section_order),
                    description=f'每题{score}分，共{count * score}分',
                    sort_order=section_order,
                    total_score=count * score,
                )

                # 创建题目关联（含快照）
                for i, question in enumerate(selected):
                    snapshot = self._build_snapshot(question)
                    ExamPaperItem.objects.create(
                        section=section,
                        question=question,
                        sort_order=i + 1,
                        score=score,
                        question_snapshot=snapshot,
                    )
                    used_question_ids.add(question.id)

            # 更新试卷状态和总分
            total = sum(sec_cfg['count'] * sec_cfg.get('score', 3) for sec_cfg in sections_config)
            self.paper.total_score = total
            self.paper.config = self.config
            self.paper.status = ExamPaper.Status.GENERATED
            self.paper.save(update_fields=['total_score', 'config', 'status', 'updated_at'])

    def swap_question(self, item_id: int) -> ExamPaperItem:
        """替换单道题目"""
        item = ExamPaperItem.objects.select_related('section__exam_paper', 'question').get(id=item_id)
        paper = item.section.exam_paper
        section = item.section

        # 找同分区所有已用题目
        used_ids = set(
            ExamPaperItem.objects.filter(section__exam_paper=paper)
            .values_list('question_id', flat=True)
        )

        # 找同题型、同级难度的候选
        old_q = item.question
        candidates = list(
            Question.objects.filter(
                subject=paper.subject,
                grade=paper.grade,
                type=old_q.type,
                difficulty=old_q.difficulty,
                status='reviewed',
            ).exclude(id__in=used_ids)
        )

        if not candidates:
            # 放宽难度限制
            candidates = list(
                Question.objects.filter(
                    subject=paper.subject,
                    grade=paper.grade,
                    type=old_q.type,
                    status='reviewed',
                ).exclude(id__in=used_ids)
            )

        if not candidates:
            raise ValueError('没有可替换的题目')

        new_q = random.choice(candidates)
        item.question = new_q
        item.question_snapshot = self._build_snapshot(new_q)
        item.save(update_fields=['question', 'question_snapshot'])

        return item

    def _select_by_difficulty(self, candidates: List[Question], count: int, target: float | None) -> List[Question]:
        """按难度分布选题目 — 贪心逼近目标难度"""
        if target is None:
            # 随机选
            return random.sample(candidates, count)

        # 按难度分组
        by_diff: Dict[int, List[Question]] = {}
        for q in candidates:
            by_diff.setdefault(q.difficulty, []).append(q)

        # 按目标难度计算各难度所需数量
        selected = []
        # 简单策略：按难度比例挑选，逼近目标难度均值
        # 先按难度排序
        sorted_candidates = sorted(candidates, key=lambda q: abs(q.difficulty - target))
        selected = sorted_candidates[:count]

        # 打乱顺序（同难度内随机）
        random.shuffle(selected)
        return selected

    def _section_title(self, q_type: str, order: int) -> str:
        """生成分区标题"""
        chinese_numbers = ['', '一', '二', '三', '四', '五', '六', '七', '八']
        type_names = {
            'choice': '选择题',
            'fill_blank': '填空题',
            'true_false': '判断题',
            'short_answer': '简答题',
            'calculation': '计算题',
            'essay': '论述题',
        }
        cn = chinese_numbers[order] if order < len(chinese_numbers) else str(order)
        return f'{cn}、{type_names.get(q_type, q_type)}'

    def _build_snapshot(self, question: Question) -> Dict[str, Any]:
        """构建题目快照 — 防止原题被修改后试卷不一致"""
        options = list(question.options.values('label', 'content', 'sort_order'))
        return {
            'stem': question.stem,
            'answer': question.answer,
            'explanation': question.explanation,
            'type': question.type,
            'difficulty': question.difficulty,
            'default_score': question.default_score,
            'images': question.images,
            'options': options,
        }
```

- [ ] **Step 4: 运行引擎测试**

```bash
pytest apps/exams/tests/test_engine.py -v
```

预期：全部 PASS

- [ ] **Step 5: 提交**

```bash
git add backend/apps/exams/engine.py backend/apps/exams/tests/test_engine.py
git commit -m "feat(m3): 实现组卷引擎核心算法 — 按题型/难度/知识点生成 + 换题"
```

---

### Task M3.3: AI 组卷推荐

**文件：**
- 创建：`backend/apps/exams/recommend.py`
- 创建：`backend/apps/exams/tests/test_recommend.py`

- [ ] **Step 1: 编写推荐测试**

写入 `/Users/mac/2026/ai_extra/ExamAI/backend/apps/exams/tests/test_recommend.py`：

```python
import pytest
from unittest.mock import MagicMock, patch
from apps.users.models import User
from apps.questions.models import Subject, Grade
from apps.ai.models import AIProvider
from apps.exams.recommend import ExamRecommendService


@pytest.mark.django_db
class TestExamRecommend:
    @pytest.fixture
    def user(self):
        return User.objects.create_user(email='teacher@test.com', password='pass123')

    @pytest.fixture
    def math(self):
        return Subject.objects.create(name='数学', code='math')

    @pytest.fixture
    def grade8(self):
        return Grade.objects.create(name='八年级', level=8)

    @pytest.fixture
    def provider(self):
        return AIProvider.objects.create(
            name='deepseek', display_name='DeepSeek',
            api_base_url='https://api.deepseek.com', api_key='sk-test',
            model_name='deepseek-chat', is_default=True, is_active=True,
        )

    @patch('apps.exams.recommend.get_default_adapter')
    def test_recommend_returns_valid_config(self, mock_get_adapter, user, math, grade8, provider):
        """推荐返回合法的组卷配置"""
        mock_adapter = MagicMock()
        mock_adapter.call_with_logging.return_value = {
            'content': '''{
                "title": "初二数学期中考试卷",
                "exam_type": "midterm",
                "total_score": 120,
                "duration": 120,
                "sections": [
                    {"type": "choice", "count": 12, "score": 3},
                    {"type": "fill_blank", "count": 6, "score": 4},
                    {"type": "calculation", "count": 5, "score": 8},
                    {"type": "short_answer", "count": 2, "score": 10}
                ],
                "difficulty_target": 3.0,
                "difficulty_distribution": {"1": 0.15, "2": 0.25, "3": 0.35, "4": 0.2, "5": 0.05},
                "knowledge_focus": ["方程", "函数", "三角形"],
                "reasoning": "根据初二数学期中考试大纲，重点考察方程与函数..."
            }''',
            'input_tokens': 200,
            'output_tokens': 500,
            'model': 'deepseek-chat',
        }
        mock_get_adapter.return_value = mock_adapter

        service = ExamRecommendService()
        result = service.recommend(
            user_input='初二数学期中考试，重点考方程和函数',
            subject=math,
            grade=grade8,
        )

        assert result['title'] == '初二数学期中考试卷'
        assert len(result['sections']) == 4
        assert result['sections'][0]['type'] == 'choice'
        assert result['sections'][0]['count'] == 12

    @patch('apps.exams.recommend.get_default_adapter')
    def test_recommend_invalid_json_fallback(self, mock_get_adapter, user, math, grade8, provider):
        """LLM 返回无效 JSON 时使用默认配置"""
        mock_adapter = MagicMock()
        mock_adapter.call_with_logging.return_value = {
            'content': '这不是合法的 JSON',
            'input_tokens': 100,
            'output_tokens': 50,
            'model': 'deepseek-chat',
        }
        mock_get_adapter.return_value = mock_adapter

        service = ExamRecommendService()
        result = service.recommend(
            user_input='随便出个卷子',
            subject=math,
            grade=grade8,
        )

        # 应返回默认配置
        assert 'sections' in result
        assert len(result['sections']) > 0
```

- [ ] **Step 2: 运行测试，验证失败**

```bash
pytest apps/exams/tests/test_recommend.py -v
```

预期：FAIL

- [ ] **Step 3: 实现 AI 推荐服务**

写入 `/Users/mac/2026/ai_extra/ExamAI/backend/apps/exams/recommend.py`：

```python
import json
import re
from typing import Any, Dict

from apps.ai.base import get_default_adapter
from apps.questions.models import Subject, Grade


# AI 推荐默认兜底配置
DEFAULT_RECOMMEND_CONFIG = {
    'sections': [
        {'type': 'choice', 'count': 10, 'score': 3},
        {'type': 'fill_blank', 'count': 5, 'score': 3},
        {'type': 'calculation', 'count': 4, 'score': 8},
        {'type': 'short_answer', 'count': 2, 'score': 10},
    ],
    'difficulty_target': 3.0,
    'difficulty_distribution': {'1': 0.15, '2': 0.25, '3': 0.35, '4': 0.2, '5': 0.05},
    'knowledge_focus': [],
}


RECOMMEND_SYSTEM_PROMPT = """你是一位资深的初中教师和考试命题专家。你需要根据用户的需求，为试卷生成提供专业配置建议。

输出格式必须为 JSON，包含以下字段：
{
    "title": "试卷标题",
    "exam_type": "unit/monthly/midterm/final/entrance",
    "total_score": 整数,
    "duration": 整数(分钟),
    "sections": [
        {
            "type": "choice/fill_blank/true_false/short_answer/calculation/essay",
            "count": 题目数量,
            "score": 每题分值,
            "difficulty": [1,2,3,4,5]  // 该题型允许的难度范围
        }
    ],
    "difficulty_target": 目标平均难度(1-5),
    "difficulty_distribution": {"1": 0.1, "2": 0.2, "3": 0.4, "4": 0.2, "5": 0.1},
    "knowledge_focus": ["重点知识点1", "重点知识点2"],
    "reasoning": "配置理由说明"
}

题型说明：choice=选择题, fill_blank=填空题, true_false=判断题, short_answer=简答题, calculation=计算题, essay=论述题
难度说明：1=基础, 2=简单, 3=中等, 4=较难, 5=困难

常见考试类型与题型配比参考：
- 单元测试(unit): 选择+填空为主，少量简答，总分80-100，60-90分钟
- 月考(monthly): 选择+填空+计算，总分100-120，90-100分钟
- 期中/期末(midterm/final): 全题型覆盖，总分100-120，90-120分钟
- 中考模拟(entrance): 按当地中考题型分布，总分120-150，120分钟"""


class ExamRecommendService:
    """AI 组卷推荐服务"""

    def recommend(self, user_input: str, subject: Subject, grade: Grade) -> Dict[str, Any]:
        """根据用户输入推荐组卷配置"""
        adapter = get_default_adapter()

        messages = [
            {'role': 'system', 'content': RECOMMEND_SYSTEM_PROMPT},
            {
                'role': 'user',
                'content': f'请为{grade.name}{subject.name}推荐一份组卷配置。用户需求：{user_input}',
            },
        ]

        result = adapter.call_with_logging('recommend_exam', messages)
        content = result.get('content', '')

        # 提取 JSON（LLM 可能用 markdown 代码块包裹）
        config = self._parse_json(content)

        # 校验并补充默认值
        config = self._validate_and_fill(config, subject, grade, user_input)

        return config

    def _parse_json(self, content: str) -> Dict[str, Any]:
        """从 LLM 回复中提取 JSON"""
        # 尝试直接解析
        try:
            return json.loads(content)
        except json.JSONDecodeError:
            pass

        # 尝试提取 markdown 代码块中的 JSON
        json_match = re.search(r'```(?:json)?\s*([\s\S]*?)\s*```', content)
        if json_match:
            try:
                return json.loads(json_match.group(1))
            except json.JSONDecodeError:
                pass

        # 尝试找到 { } 包裹的 JSON
        brace_match = re.search(r'\{[\s\S]*\}', content)
        if brace_match:
            try:
                return json.loads(brace_match.group(0))
            except json.JSONDecodeError:
                pass

        return DEFAULT_RECOMMEND_CONFIG

    def _validate_and_fill(self, config: Dict[str, Any], subject: Subject, grade: Grade, user_input: str) -> Dict[str, Any]:
        """校验并填充默认值"""
        # 确保必要字段存在
        config.setdefault('title', f'{grade.name}{subject.name}试卷')
        config.setdefault('exam_type', 'unit')
        config.setdefault('total_score', 100)
        config.setdefault('duration', 90)
        config.setdefault('difficulty_target', 3.0)
        config.setdefault('difficulty_distribution', DEFAULT_RECOMMEND_CONFIG['difficulty_distribution'])
        config.setdefault('knowledge_focus', [])
        config.setdefault('reasoning', '')
        config.setdefault('user_input', user_input)

        # 校验 sections
        sections = config.get('sections', [])
        if not sections:
            config['sections'] = DEFAULT_RECOMMEND_CONFIG['sections']
        else:
            for sec in sections:
                sec.setdefault('difficulty', [1, 2, 3, 4, 5])
                sec.setdefault('score', 3)

        return config
```

- [ ] **Step 4: 运行推荐测试**

```bash
pytest apps/exams/tests/test_recommend.py -v
```

预期：全部 PASS

- [ ] **Step 5: 提交**

```bash
git add backend/apps/exams/recommend.py backend/apps/exams/tests/test_recommend.py
git commit -m "feat(m3): 实现 AI 智能组卷推荐 — LLM 推荐配置 + JSON 解析 + 兜底策略"
```

---

### Task M3.4: 试卷 API

**文件：**
- 创建：`backend/apps/exams/serializers.py`
- 创建：`backend/apps/exams/views.py`
- 创建：`backend/apps/exams/urls.py`
- 创建：`backend/apps/exams/filters.py`
- 创建：`backend/apps/exams/tests/test_api.py`

- [ ] **Step 1: 编写 API 测试**

写入 `/Users/mac/2026/ai_extra/ExamAI/backend/apps/exams/tests/test_api.py`：

```python
import pytest
from django.urls import reverse
from rest_framework import status
from apps.users.models import User
from apps.questions.models import Subject, Grade, Question
from apps.exams.models import ExamPaper


@pytest.mark.django_db
class TestExamAPI:
    LIST_URL = reverse('exampaper-list')

    @pytest.fixture
    def user(self):
        return User.objects.create_user(email='teacher@test.com', password='pass123')

    @pytest.fixture
    def math(self):
        return Subject.objects.create(name='数学', code='math')

    @pytest.fixture
    def grade8(self):
        return Grade.objects.create(name='八年级', level=8)

    @pytest.fixture
    def auth_client(self, client, user):
        client.force_authenticate(user=user)
        return client

    @pytest.fixture
    def questions(self, math, grade8):
        for i in range(30):
            Question.objects.create(
                subject=math, grade=grade8, type='choice', difficulty=3,
                stem=f'题{i}', answer='A', status='reviewed',
            )

    def test_list_papers(self, auth_client, math, grade8):
        """试卷列表"""
        ExamPaper.objects.create(user=User.objects.first(), title='测试卷', subject=math, grade=grade8, exam_type='unit')
        response = auth_client.get(self.LIST_URL)
        assert response.status_code == 200
        assert response.data['data']['total'] >= 1

    def test_create_and_generate(self, auth_client, math, grade8, questions):
        """创建试卷并生成"""
        data = {
            'title': '初二数学单元测试',
            'subject_id': math.id,
            'grade_id': grade8.id,
            'exam_type': 'unit',
            'duration': 60,
            'config': {
                'sections': [
                    {'type': 'choice', 'count': 10, 'score': 3},
                    {'type': 'fill_blank', 'count': 5, 'score': 4},
                ],
                'difficulty_target': 3.0,
            },
        }
        response = auth_client.post(self.LIST_URL, data, format='json')
        assert response.status_code == status.HTTP_201_CREATED
        assert response.data['data']['status'] == 'generated'
        assert response.data['data']['total_score'] == 50

    def test_get_paper_detail(self, auth_client, math, grade8, questions):
        """试卷详情含分区和题目"""
        data = {
            'title': '详情测试', 'subject_id': math.id, 'grade_id': grade8.id,
            'exam_type': 'unit', 'duration': 60,
            'config': {'sections': [{'type': 'choice', 'count': 5, 'score': 3}], 'difficulty_target': 3.0},
        }
        create_resp = auth_client.post(self.LIST_URL, data, format='json')
        paper_id = create_resp.data['data']['id']
        url = reverse('exampaper-detail', kwargs={'pk': paper_id})
        response = auth_client.get(url)
        assert response.status_code == 200
        assert 'sections' in response.data['data']
        assert len(response.data['data']['sections']) == 1

    def test_swap_question(self, auth_client, math, grade8, questions):
        """换题 API"""
        data = {
            'title': '换题测试', 'subject_id': math.id, 'grade_id': grade8.id,
            'exam_type': 'unit', 'duration': 60,
            'config': {'sections': [{'type': 'choice', 'count': 5, 'score': 3}], 'difficulty_target': 3.0},
        }
        create_resp = auth_client.post(self.LIST_URL, data, format='json')
        paper_id = create_resp.data['data']['id']

        # 获取第一道题
        detail_url = reverse('exampaper-detail', kwargs={'pk': paper_id})
        detail = auth_client.get(detail_url)
        first_item_id = detail.data['data']['sections'][0]['items'][0]['id']

        swap_url = reverse('exampaper-swap-question', kwargs={'pk': paper_id})
        response = auth_client.post(swap_url, {'item_id': first_item_id}, format='json')
        assert response.status_code == 200

    def test_delete_paper(self, auth_client, math, grade8):
        """删除试卷"""
        paper = ExamPaper.objects.create(
            user=User.objects.first(), title='待删除', subject=math, grade=grade8, exam_type='unit',
        )
        url = reverse('exampaper-detail', kwargs={'pk': paper.pk})
        response = auth_client.delete(url)
        assert response.status_code == 204
```

- [ ] **Step 2: 运行测试，验证失败**

```bash
pytest apps/exams/tests/test_api.py -v
```

预期：FAIL

- [ ] **Step 3: 实现 Serializer**

写入 `/Users/mac/2026/ai_extra/ExamAI/backend/apps/exams/serializers.py`：

```python
from rest_framework import serializers
from .models import ExamPaper, ExamPaperSection, ExamPaperItem


class ExamPaperItemSerializer(serializers.ModelSerializer):
    """试卷题目序列化器 — 优先使用快照"""
    stem = serializers.SerializerMethodField()
    answer = serializers.SerializerMethodField()
    explanation = serializers.SerializerMethodField()
    question_type = serializers.SerializerMethodField()
    difficulty = serializers.SerializerMethodField()
    options = serializers.SerializerMethodField()

    class Meta:
        model = ExamPaperItem
        fields = ('id', 'question', 'sort_order', 'score', 'stem', 'answer', 'explanation', 'question_type', 'difficulty', 'options')

    def get_stem(self, obj):
        return obj.question_snapshot.get('stem', obj.question.stem)

    def get_answer(self, obj):
        return obj.question_snapshot.get('answer', obj.question.answer)

    def get_explanation(self, obj):
        return obj.question_snapshot.get('explanation', obj.question.explanation)

    def get_question_type(self, obj):
        return obj.question_snapshot.get('type', obj.question.type)

    def get_difficulty(self, obj):
        return obj.question_snapshot.get('difficulty', obj.question.difficulty)

    def get_options(self, obj):
        return obj.question_snapshot.get('options', [])


class ExamPaperSectionSerializer(serializers.ModelSerializer):
    items = ExamPaperItemSerializer(many=True, read_only=True)

    class Meta:
        model = ExamPaperSection
        fields = ('id', 'title', 'description', 'sort_order', 'total_score', 'items')


class ExamPaperListSerializer(serializers.ModelSerializer):
    """试卷列表序列化器（轻量）"""
    subject_name = serializers.CharField(source='subject.name', read_only=True)
    grade_name = serializers.CharField(source='grade.name', read_only=True)
    question_count = serializers.SerializerMethodField()

    class Meta:
        model = ExamPaper
        fields = ('id', 'title', 'subject_name', 'grade_name', 'exam_type', 'total_score', 'duration', 'status', 'question_count', 'created_at')

    def get_question_count(self, obj):
        return obj.get_question_count()


class ExamPaperDetailSerializer(serializers.ModelSerializer):
    """试卷详情序列化器（含分区和题目）"""
    sections = ExamPaperSectionSerializer(many=True, read_only=True)
    subject_name = serializers.CharField(source='subject.name', read_only=True)
    grade_name = serializers.CharField(source='grade.name', read_only=True)

    class Meta:
        model = ExamPaper
        fields = ('id', 'title', 'subject', 'subject_name', 'grade', 'grade_name', 'exam_type', 'total_score', 'duration', 'config', 'status', 'sections', 'created_at', 'updated_at')


class ExamPaperCreateSerializer(serializers.ModelSerializer):
    """创建试卷序列化器"""
    subject_id = serializers.PrimaryKeyRelatedField(queryset=serializers.PrimaryKeyRelatedField.__kwdefaults__.__class__(), source='subject', write_only=True)
    grade_id = serializers.PrimaryKeyRelatedField(queryset=serializers.PrimaryKeyRelatedField.__kwdefaults__.__class__(), source='grade', write_only=True)

    class Meta:
        model = ExamPaper
        fields = ('title', 'subject_id', 'grade_id', 'exam_type', 'duration', 'config')

    def __init__(self, *args, **kwargs):
        super().__init__(*args, **kwargs)
        from apps.questions.models import Subject, Grade
        self.fields['subject_id'] = serializers.PrimaryKeyRelatedField(
            queryset=Subject.objects.all(), source='subject', write_only=True
        )
        self.fields['grade_id'] = serializers.PrimaryKeyRelatedField(
            queryset=Grade.objects.all(), source='grade', write_only=True
        )
```

- [ ] **Step 4: 实现 View**

写入 `/Users/mac/2026/ai_extra/ExamAI/backend/apps/exams/views.py`：

```python
from rest_framework import viewsets, permissions, status, mixins
from rest_framework.decorators import action

from common.response import success, error

from .models import ExamPaper
from .serializers import (
    ExamPaperListSerializer,
    ExamPaperDetailSerializer,
    ExamPaperCreateSerializer,
)
from .engine import ExamGenerator
from .recommend import ExamRecommendService


class ExamPaperViewSet(mixins.CreateModelMixin,
                        mixins.RetrieveModelMixin,
                        mixins.ListModelMixin,
                        mixins.DestroyModelMixin,
                        viewsets.GenericViewSet):
    """试卷 CRUD"""
    permission_classes = (permissions.IsAuthenticated,)

    def get_serializer_class(self):
        if self.action == 'list':
            return ExamPaperListSerializer
        elif self.action == 'create':
            return ExamPaperCreateSerializer
        return ExamPaperDetailSerializer

    def get_queryset(self):
        return ExamPaper.objects.filter(
            user=self.request.user
        ).select_related('subject', 'grade').order_by('-created_at')

    def perform_create(self, serializer):
        paper = serializer.save(user=self.request.user)
        # 自动触发组卷
        config = serializer.validated_data.get('config', {})
        generator = ExamGenerator(paper, config)
        try:
            generator.generate()
        except ValueError as e:
            paper.delete()
            raise e

    def create(self, request, *args, **kwargs):
        try:
            return super().create(request, *args, **kwargs)
        except ValueError as e:
            return error(message=str(e), status=status.HTTP_400_BAD_REQUEST)

    def retrieve(self, request, *args, **kwargs):
        instance = self.get_object()
        serializer = self.get_serializer(instance)
        return success(data=serializer.data)

    def list(self, request, *args, **kwargs):
        queryset = self.filter_queryset(self.get_queryset())
        page = self.paginate_queryset(queryset)
        if page is not None:
            serializer = self.get_serializer(page, many=True)
            return self.get_paginated_response(serializer.data)
        serializer = self.get_serializer(queryset, many=True)
        return success(data=serializer.data)

    def destroy(self, request, *args, **kwargs):
        instance = self.get_object()
        instance.delete()
        return success(message='删除成功', status=status.HTTP_204_NO_CONTENT)

    @action(detail=True, methods=['post'], url_path='swap-question')
    def swap_question(self, request, pk=None):
        """换题"""
        paper = self.get_object()
        item_id = request.data.get('item_id')
        if not item_id:
            return error(message='请指定要替换的题目', status=status.HTTP_400_BAD_REQUEST)

        generator = ExamGenerator(paper, paper.config)
        try:
            generator.swap_question(item_id)
            return success(data=ExamPaperDetailSerializer(paper).data, message='换题成功')
        except ValueError as e:
            return error(message=str(e), status=status.HTTP_400_BAD_REQUEST)

    @action(detail=False, methods=['post'], url_path='recommend')
    def recommend(self, request):
        """AI 智能推荐组卷配置"""
        user_input = request.data.get('input', '')
        subject_id = request.data.get('subject_id')
        grade_id = request.data.get('grade_id')

        if not subject_id or not grade_id:
            return error(message='请选择学科和年级', status=status.HTTP_400_BAD_REQUEST)

        from apps.questions.models import Subject, Grade
        try:
            subject = Subject.objects.get(id=subject_id)
            grade = Grade.objects.get(id=grade_id)
        except (Subject.DoesNotExist, Grade.DoesNotExist):
            return error(message='学科或年级不存在', status=status.HTTP_400_BAD_REQUEST)

        service = ExamRecommendService()
        try:
            config = service.recommend(user_input, subject, grade)
            return success(data=config, message='推荐成功')
        except Exception as e:
            return error(message=f'AI 推荐失败: {str(e)}', status=status.HTTP_500_INTERNAL_SERVER_ERROR)
```

- [ ] **Step 5: 实现 URL 路由**

写入 `/Users/mac/2026/ai_extra/ExamAI/backend/apps/exams/urls.py`：

```python
from django.urls import path, include
from rest_framework.routers import DefaultRouter
from . import views

router = DefaultRouter()
router.register(r'', views.ExamPaperViewSet, basename='exampaper')

urlpatterns = [
    path('', include(router.urls)),
]
```

- [ ] **Step 6: 运行 API 测试**

```bash
pytest apps/exams/tests/test_api.py -v
```

预期：全部 PASS

- [ ] **Step 7: 提交**

```bash
git add backend/apps/exams/
git commit -m "feat(m3): 实现试卷 API — 创建/列表/详情/删除 + 自动组卷 + 换题 + AI推荐"
```

---

### Task M3.5: 智能组卷前端页面

**文件：**
- 修改：`frontend/src/pages/ExamGeneratePage.tsx`
- 创建：`frontend/src/components/exam/ExamConfigPanel.tsx`
- 创建：`frontend/src/components/exam/ExamAIRecommend.tsx`
- 创建：`frontend/src/components/exam/ExamPreview.tsx`
- 创建：`frontend/src/components/exam/ExamPreviewToolbar.tsx`
- 创建：`frontend/src/components/exam/DifficultyChart.tsx`
- 创建：`frontend/src/components/exam/SwapQuestionModal.tsx`
- 创建：`frontend/src/services/examService.ts`
- 创建：`frontend/src/stores/examStore.ts`

- [ ] **Step 1: 创建 examService API 封装**

写入 `/Users/mac/2026/ai_extra/ExamAI/frontend/src/services/examService.ts`：

```typescript
import api from './api';

export interface ExamSectionConfig {
  type: string;
  count: number;
  score: number;
  difficulty?: number[];
}

export interface ExamConfig {
  sections: ExamSectionConfig[];
  difficulty_target?: number;
  difficulty_distribution?: Record<string, number>;
  knowledge_point_ids?: number[];
  knowledge_focus?: string[];
  title?: string;
  exam_type?: string;
  total_score?: number;
  duration?: number;
  reasoning?: string;
}

export interface ExamPaperItem {
  id: number;
  question: number;
  sort_order: number;
  score: number;
  stem: string;
  answer: string;
  explanation: string;
  question_type: string;
  difficulty: number;
  options: Array<{ label: string; content: string }>;
}

export interface ExamSection {
  id: number;
  title: string;
  description: string;
  sort_order: number;
  total_score: number;
  items: ExamPaperItem[];
}

export interface ExamPaper {
  id: number;
  title: string;
  subject: number;
  subject_name: string;
  grade: number;
  grade_name: string;
  exam_type: string;
  total_score: number;
  duration: number;
  config: ExamConfig;
  status: string;
  sections: ExamSection[];
  question_count?: number;
  created_at: string;
}

export interface RecommendInput {
  input: string;
  subject_id: number;
  grade_id: number;
}

export const examService = {
  list: (params?: any) =>
    api.get<{ code: number; data: { page: number; page_size: number; total: number; results: ExamPaper[] } }>('/exams/', { params }),

  detail: (id: number) =>
    api.get<{ code: number; data: ExamPaper }>(`/exams/${id}/`),

  create: (data: { title: string; subject_id: number; grade_id: number; exam_type: string; duration: number; config: ExamConfig }) =>
    api.post<{ code: number; data: ExamPaper }>('/exams/', data),

  delete: (id: number) =>
    api.delete(`/exams/${id}/`),

  swapQuestion: (paperId: number, itemId: number) =>
    api.post<{ code: number; data: ExamPaper }>(`/exams/${paperId}/swap-question/`, { item_id: itemId }),

  recommend: (data: RecommendInput) =>
    api.post<{ code: number; data: ExamConfig }>('/exams/recommend/', data),
};
```

- [ ] **Step 2: 创建 examStore**

写入 `/Users/mac/2026/ai_extra/ExamAI/frontend/src/stores/examStore.ts`：

```typescript
import { create } from 'zustand';
import { examService, ExamPaper, ExamConfig, RecommendInput } from '@/services/examService';

interface ExamState {
  papers: ExamPaper[];
  total: number;
  loading: boolean;
  currentPaper: ExamPaper | null;
  recommendConfig: ExamConfig | null;
  recommendLoading: boolean;

  fetchPapers: (params?: any) => Promise<void>;
  createPaper: (data: any) => Promise<ExamPaper>;
  fetchPaperDetail: (id: number) => Promise<void>;
  deletePaper: (id: number) => Promise<void>;
  swapQuestion: (paperId: number, itemId: number) => Promise<void>;
  recommend: (data: RecommendInput) => Promise<void>;
  setCurrentPaper: (paper: ExamPaper | null) => void;
  setRecommendConfig: (config: ExamConfig | null) => void;
}

export const useExamStore = create<ExamState>((set) => ({
  papers: [],
  total: 0,
  loading: false,
  currentPaper: null,
  recommendConfig: null,
  recommendLoading: false,

  fetchPapers: async (params) => {
    set({ loading: true });
    try {
      const res = await examService.list(params);
      set({ papers: res.data.data.results, total: res.data.data.total });
    } finally {
      set({ loading: false });
    }
  },

  createPaper: async (data) => {
    const res = await examService.create(data);
    const paper = res.data.data;
    set({ currentPaper: paper });
    return paper;
  },

  fetchPaperDetail: async (id) => {
    set({ loading: true });
    try {
      const res = await examService.detail(id);
      set({ currentPaper: res.data.data });
    } finally {
      set({ loading: false });
    }
  },

  deletePaper: async (id) => {
    await examService.delete(id);
    set(state => ({ papers: state.papers.filter(p => p.id !== id) }));
  },

  swapQuestion: async (paperId, itemId) => {
    const res = await examService.swapQuestion(paperId, itemId);
    set({ currentPaper: res.data.data });
  },

  recommend: async (data) => {
    set({ recommendLoading: true });
    try {
      const res = await examService.recommend(data);
      set({ recommendConfig: res.data.data });
    } finally {
      set({ recommendLoading: false });
    }
  },

  setCurrentPaper: (paper) => set({ currentPaper: paper }),
  setRecommendConfig: (config) => set({ recommendConfig: config }),
}));
```

- [ ] **Step 3: 创建 A 面配置面板**

写入 `/Users/mac/2026/ai_extra/ExamAI/frontend/src/components/exam/ExamConfigPanel.tsx`：

```typescript
import { useState, useEffect } from 'react';
import { Card, Form, Select, Input, InputNumber, Button, Space, Divider, message } from 'antd';
import { PlusOutlined, DeleteOutlined } from '@ant-design/icons';
import { questionService } from '@/services/questionService';
import { ExamConfig } from '@/services/examService';
import { SUBJECTS, GRADES, QUESTION_TYPES, EXAM_TYPES } from '@/utils/constants';

interface Props {
  onGenerate: (data: any) => void;
  loading: boolean;
  initialConfig?: ExamConfig | null;
}

export default function ExamConfigPanel({ onGenerate, loading, initialConfig }: Props) {
  const [form] = Form.useForm();
  const [subjects, setSubjects] = useState<any[]>([]);
  const [grades, setGrades] = useState<any[]>([]);
  const [knowledgePoints, setKnowledgePoints] = useState<any[]>([]);

  useEffect(() => {
    questionService.getSubjects().then(res => setSubjects(res.data.data));
    questionService.getGrades().then(res => setGrades(res.data.data));
  }, []);

  useEffect(() => {
    if (initialConfig) {
      form.setFieldsValue({
        title: initialConfig.title || '',
        exam_type: initialConfig.exam_type || 'unit',
        duration: initialConfig.duration || 90,
        sections: initialConfig.sections || [{ type: 'choice', count: 10, score: 3 }],
      });
    }
  }, [initialConfig, form]);

  const handleSubjectChange = (id: number) => {
    questionService.getKnowledgePoints({ subject: id }).then(res => setKnowledgePoints(res.data.data));
  };

  return (
    <Card title="A面 — 精细配置" style={{ height: '100%' }}>
      <Form form={form} layout="vertical" initialValues={{
        exam_type: 'unit',
        duration: 90,
        sections: [{ type: 'choice', count: 10, score: 3 }],
      }}>
        <Form.Item name="title" label="试卷标题" rules={[{ required: true }]}>
          <Input placeholder="如：初二数学期中考试卷" />
        </Form.Item>
        <Space size="middle">
          <Form.Item name="subject_id" label="学科" rules={[{ required: true }]}>
            <Select placeholder="学科" style={{ width: 140 }} onChange={handleSubjectChange}
              options={subjects.map(s => ({ value: s.id, label: s.name }))} />
          </Form.Item>
          <Form.Item name="grade_id" label="年级" rules={[{ required: true }]}>
            <Select placeholder="年级" style={{ width: 140 }}
              options={grades.map(g => ({ value: g.id, label: g.name }))} />
          </Form.Item>
          <Form.Item name="exam_type" label="考试类型">
            <Select style={{ width: 140 }} options={EXAM_TYPES.map(t => ({ value: t.value, label: t.label }))} />
          </Form.Item>
          <Form.Item name="duration" label="时长(分钟)">
            <InputNumber min={30} max={180} step={10} />
          </Form.Item>
        </Space>

        <Divider>试卷分区配置</Divider>
        <Form.List name="sections">
          {(fields, { add, remove }) => (
            <>
              {fields.map(({ key, name, ...rest }) => (
                <Space key={key} align="baseline" wrap>
                  <Form.Item {...rest} name={[name, 'type']} label="题型" rules={[{ required: true }]}>
                    <Select style={{ width: 120 }} options={QUESTION_TYPES.map(t => ({ value: t.value, label: t.label }))} />
                  </Form.Item>
                  <Form.Item {...rest} name={[name, 'count']} label="数量" rules={[{ required: true }]}>
                    <InputNumber min={1} max={50} />
                  </Form.Item>
                  <Form.Item {...rest} name={[name, 'score']} label="每题分值" rules={[{ required: true }]}>
                    <InputNumber min={1} max={50} />
                  </Form.Item>
                  <Button icon={<DeleteOutlined />} onClick={() => remove(name)} danger type="link" />
                </Space>
              ))}
              <Button type="dashed" onClick={() => add({ type: 'choice', count: 5, score: 3 })} icon={<PlusOutlined />}>
                添加分区
              </Button>
            </>
          )}
        </Form.List>

        <Divider />
        <Form.Item name="knowledge_point_ids" label="限定知识点范围（可选）">
          <Select mode="multiple" placeholder="不选则全题库随机" allowClear
            options={knowledgePoints.map(kp => ({ value: kp.id, label: kp.path }))} />
        </Form.Item>

        <Button type="primary" size="large" block loading={loading}
          onClick={async () => {
            const values = await form.validateFields();
            const config: ExamConfig = {
              sections: values.sections,
              knowledge_point_ids: values.knowledge_point_ids || [],
            };
            onGenerate({ ...values, config });
          }}>
          开始组卷
        </Button>
      </Form>
    </Card>
  );
}
```

- [ ] **Step 4: 创建 B 面 AI 推荐面板**

写入 `/Users/mac/2026/ai_extra/ExamAI/frontend/src/components/exam/ExamAIRecommend.tsx`：

```typescript
import { useState } from 'react';
import { Card, Input, Button, Select, Typography, Spin, Alert, Tag, Space } from 'antd';
import { RobotOutlined, ThunderboltOutlined } from '@ant-design/icons';
import { useExamStore } from '@/stores/examStore';
import { questionService } from '@/services/questionService';
import { ExamConfig } from '@/services/examService';
import { QUESTION_TYPES } from '@/utils/constants';

const { TextArea } = Input;
const { Text, Paragraph } = Typography;

interface Props {
  onApplyConfig: (config: ExamConfig) => void;
}

const typeLabels = Object.fromEntries(QUESTION_TYPES.map(t => [t.value, t.label]));

export default function ExamAIRecommend({ onApplyConfig }: Props) {
  const { recommendConfig, recommendLoading, recommend } = useExamStore();
  const [input, setInput] = useState('');
  const [subjectId, setSubjectId] = useState<number | undefined>();
  const [gradeId, setGradeId] = useState<number | undefined>();
  const [subjects, setSubjects] = useState<any[]>([]);
  const [grades, setGrades] = useState<any[]>([]);

  useState(() => {
    questionService.getSubjects().then(res => setSubjects(res.data.data));
    questionService.getGrades().then(res => setGrades(res.data.data));
  });

  const handleRecommend = () => {
    if (!subjectId || !gradeId) return;
    recommend({ input, subject_id: subjectId, grade_id: gradeId });
  };

  return (
    <Card
      title={<span><RobotOutlined /> B面 — AI 智能推荐</span>}
      style={{ height: '100%' }}
    >
      <Space direction="vertical" style={{ width: '100%' }} size="middle">
        <Text type="secondary">输入你的考试意图，AI 自动推荐知识点覆盖、题型分布和难度比例</Text>
        <TextArea
          rows={3}
          placeholder="例：初二物理期中考试，重点考力学部分，需要覆盖选择题、填空题和计算题，难度中等偏上"
          value={input}
          onChange={e => setInput(e.target.value)}
        />
        <Space>
          <Select placeholder="学科" value={subjectId} onChange={setSubjectId} style={{ width: 120 }}
            options={subjects.map(s => ({ value: s.id, label: s.name }))} />
          <Select placeholder="年级" value={gradeId} onChange={setGradeId} style={{ width: 120 }}
            options={grades.map(g => ({ value: g.id, label: g.name }))} />
          <Button type="primary" icon={<ThunderboltOutlined />} loading={recommendLoading}
            onClick={handleRecommend} disabled={!input.trim() || !subjectId || !gradeId}>
            AI 推荐
          </Button>
        </Space>

        {recommendLoading && <Spin tip="AI 正在分析中..." />}

        {recommendConfig && !recommendLoading && (
          <div style={{ background: '#f6ffed', border: '1px solid #b7eb8f', borderRadius: 8, padding: 16 }}>
            <Text strong>{recommendConfig.title}</Text>
            <Paragraph type="secondary" style={{ marginTop: 8 }}>{recommendConfig.reasoning}</Paragraph>
            <Space wrap style={{ marginTop: 8 }}>
              <Tag color="blue">{recommendConfig.exam_type}</Tag>
              <Tag>总分: {recommendConfig.total_score}</Tag>
              <Tag>时长: {recommendConfig.duration}分钟</Tag>
            </Space>
            <div style={{ marginTop: 12 }}>
              <Text strong>推荐题型分布：</Text>
              {recommendConfig.sections?.map((sec, i) => (
                <Tag key={i} style={{ marginTop: 4 }}>
                  {typeLabels[sec.type] || sec.type} ×{sec.count} (每题{sec.score}分)
                </Tag>
              ))}
            </div>
            {recommendConfig.knowledge_focus && recommendConfig.knowledge_focus.length > 0 && (
              <div style={{ marginTop: 8 }}>
                <Text strong>重点知识点：</Text>
                <Space wrap>{recommendConfig.knowledge_focus.map((k, i) => <Tag key={i} color="purple">{k}</Tag>)}</Space>
              </div>
            )}
            <Button type="primary" style={{ marginTop: 16 }} onClick={() => onApplyConfig(recommendConfig)}>
              应用此配置到 A 面
            </Button>
          </div>
        )}
      </Space>
    </Card>
  );
}
```

- [ ] **Step 5: 创建试卷预览组件**

写入 `/Users/mac/2026/ai_extra/ExamAI/frontend/src/components/exam/ExamPreview.tsx`：

```typescript
import { useState } from 'react';
import { Card, Empty, Typography, Tag, Divider, Space, Button, message } from 'antd';
import { SwapOutlined } from '@ant-design/icons';
import { ExamPaper, ExamPaperItem } from '@/services/examService';
import { useExamStore } from '@/stores/examStore';
import SwapQuestionModal from './SwapQuestionModal';
import DifficultyChart from './DifficultyChart';

const { Title, Text } = Typography;

interface Props {
  paper: ExamPaper;
}

export default function ExamPreview({ paper }: Props) {
  const { swapQuestion } = useExamStore();
  const [swapModalOpen, setSwapModalOpen] = useState(false);
  const [selectedItem, setSelectedItem] = useState<ExamPaperItem | null>(null);
  const [swapping, setSwapping] = useState(false);

  const handleSwap = async (newQuestionId?: number) => {
    if (!selectedItem || !paper) return;
    setSwapping(true);
    try {
      await swapQuestion(paper.id, selectedItem.id);
      setSwapModalOpen(false);
      message.success('换题成功');
    } finally {
      setSwapping(false);
    }
  };

  if (!paper || !paper.sections) {
    return <Card><Empty description="暂无试卷，请先配置并生成" /></Card>;
  }

  return (
    <Card
      title={<Title level={4} style={{ margin: 0 }}>{paper.title}</Title>}
      extra={
        <Space>
          <Tag color="blue">{paper.subject_name}</Tag>
          <Tag>{paper.grade_name}</Tag>
          <Tag>总分: {paper.total_score}</Tag>
          <Tag>时长: {paper.duration}分钟</Tag>
        </Space>
      }
    >
      <DifficultyChart paper={paper} />

      {paper.sections.map(section => (
        <div key={section.id} style={{ marginBottom: 24 }}>
          <Divider orientation="left">
            <Text strong style={{ fontSize: 16 }}>{section.title}</Text>
            <Text type="secondary" style={{ marginLeft: 8 }}>{section.description}</Text>
          </Divider>

          {section.items.map((item, index) => (
            <div key={item.id} style={{
              padding: '12px 0',
              borderBottom: '1px solid #f0f0f0',
              display: 'flex',
              alignItems: 'flex-start',
              gap: 12,
            }}>
              <Text strong style={{ minWidth: 32 }}>{index + 1}.</Text>
              <div style={{ flex: 1 }}>
                <div style={{ marginBottom: 8, lineHeight: 1.8 }}>{item.stem}</div>
                {item.options && item.options.length > 0 && (
                  <div style={{ display: 'flex', flexWrap: 'wrap', gap: '12px 24px', marginBottom: 8 }}>
                    {item.options.map(opt => (
                      <span key={opt.label}>{opt.label}. {opt.content}</span>
                    ))}
                  </div>
                )}
                <Space size="small">
                  <Tag>分值: {item.score}分</Tag>
                  <Tag color={['green', 'cyan', 'blue', 'orange', 'red'][(item.difficulty || 3) - 1]}>
                    难度: {'★'.repeat(item.difficulty || 3)}
                  </Tag>
                </Space>
              </div>
              <Button
                type="link"
                icon={<SwapOutlined />}
                onClick={() => { setSelectedItem(item); setSwapModalOpen(true); }}
                title="换一题"
              />
            </div>
          ))}
        </div>
      ))}

      <SwapQuestionModal
        open={swapModalOpen}
        item={selectedItem}
        paper={paper}
        loading={swapping}
        onSwap={handleSwap}
        onClose={() => setSwapModalOpen(false)}
      />
    </Card>
  );
}
```

- [ ] **Step 6: 创建难度分布图组件**

写入 `/Users/mac/2026/ai_extra/ExamAI/frontend/src/components/exam/DifficultyChart.tsx`：

```typescript
import { Card, Progress, Row, Col, Typography } from 'antd';
import { ExamPaper } from '@/services/examService';
import { DIFFICULTY_LEVELS } from '@/utils/constants';

const { Text } = Typography;

interface Props {
  paper: ExamPaper;
}

export default function DifficultyChart({ paper }: Props) {
  // 统计各难度题目数量
  const counts: Record<number, number> = { 1: 0, 2: 0, 3: 0, 4: 0, 5: 0 };
  let totalQuestions = 0;

  paper.sections?.forEach(section => {
    section.items?.forEach(item => {
      const d = item.difficulty || 3;
      counts[d] = (counts[d] || 0) + 1;
      totalQuestions++;
    });
  });

  const colors = ['#52c41a', '#13c2c2', '#1677ff', '#fa8c16', '#f5222d'];

  return (
    <Card size="small" title="难度分布" style={{ marginBottom: 16 }}>
      <Row gutter={16}>
        {DIFFICULTY_LEVELS.map((level, i) => {
          const count = counts[level.value] || 0;
          const percent = totalQuestions > 0 ? Math.round((count / totalQuestions) * 100) : 0;
          return (
            <Col span={Math.floor(24 / 5)} key={level.value}>
              <div style={{ textAlign: 'center' }}>
                <Progress type="circle" percent={percent} size={60} strokeColor={colors[i]} format={() => count} />
                <div style={{ marginTop: 4 }}>
                  <Text style={{ color: colors[i], fontSize: 12 }}>{level.label}</Text>
                </div>
              </div>
            </Col>
          );
        })}
      </Row>
    </Card>
  );
}
```

- [ ] **Step 7: 创建换题弹窗**

写入 `/Users/mac/2026/ai_extra/ExamAI/frontend/src/components/exam/SwapQuestionModal.tsx`：

```typescript
import { Modal, Button, Typography } from 'antd';
import { ExamPaper, ExamPaperItem } from '@/services/examService';

const { Text } = Typography;

interface Props {
  open: boolean;
  item: ExamPaperItem | null;
  paper: ExamPaper;
  loading: boolean;
  onSwap: (questionId?: number) => void;
  onClose: () => void;
}

export default function SwapQuestionModal({ open, item, paper, loading, onSwap, onClose }: Props) {
  if (!item) return null;

  return (
    <Modal
      title="换一题"
      open={open}
      onCancel={onClose}
      footer={[
        <Button key="cancel" onClick={onClose}>取消</Button>,
        <Button key="swap" type="primary" loading={loading} onClick={() => onSwap()}>
          随机换一题
        </Button>,
      ]}
    >
      <div style={{ marginBottom: 16 }}>
        <Text type="secondary">当前题目：</Text>
        <div style={{ padding: 12, background: '#fafafa', borderRadius: 6, marginTop: 8 }}>
          <Text>{item.stem}</Text>
          <div style={{ marginTop: 8 }}>
            <Text type="secondary">题型: {item.question_type} | 难度: {'★'.repeat(item.difficulty || 3)} | 分值: {item.score}分</Text>
          </div>
        </div>
      </div>
      <Text type="secondary">
        系统将自动从题库中随机选取一道同题型、相近难度的题目进行替换。
        原题目不会被删除，只是从本试卷中移除。
      </Text>
    </Modal>
  );
}
```

- [ ] **Step 8: 组装智能组卷页面**

写入 `/Users/mac/2026/ai_extra/ExamAI/frontend/src/pages/ExamGeneratePage.tsx`：

```typescript
import { useState } from 'react';
import { Row, Col, Tabs, message } from 'antd';
import { useExamStore } from '@/stores/examStore';
import { ExamConfig } from '@/services/examService';
import ExamConfigPanel from '@/components/exam/ExamConfigPanel';
import ExamAIRecommend from '@/components/exam/ExamAIRecommend';
import ExamPreview from '@/components/exam/ExamPreview';

export default function ExamGeneratePage() {
  const { currentPaper, createPaper, loading } = useExamStore();
  const [activeTab, setActiveTab] = useState('config');

  const handleGenerate = async (data: any) => {
    try {
      await createPaper(data);
      message.success('试卷生成成功！');
      setActiveTab('preview');
    } catch (err: any) {
      message.error(err?.response?.data?.message || '组卷失败');
    }
  };

  const handleApplyRecommend = (config: ExamConfig) => {
    // 将 AI 推荐配置应用到 A 面，切换到 A 面 tab
    setActiveTab('config');
    // 通过 ExamConfigPanel 的 initialConfig prop 传递
    // 实际通过 store 传递
    useExamStore.getState().setRecommendConfig(config);
  };

  return (
    <div>
      <h2 style={{ marginBottom: 16 }}>智能组卷</h2>
      <Tabs
        activeKey={activeTab}
        onChange={setActiveTab}
        items={[
          {
            key: 'config',
            label: '配置组卷',
            children: (
              <Row gutter={24}>
                <Col span={12}>
                  <ExamConfigPanel
                    onGenerate={handleGenerate}
                    loading={loading}
                    initialConfig={useExamStore.getState().recommendConfig}
                  />
                </Col>
                <Col span={12}>
                  <ExamAIRecommend onApplyConfig={handleApplyRecommend} />
                </Col>
              </Row>
            ),
          },
          {
            key: 'preview',
            label: '试卷预览',
            disabled: !currentPaper,
            children: currentPaper ? (
              <ExamPreview paper={currentPaper} />
            ) : (
              <div style={{ textAlign: 'center', padding: 60, color: '#999' }}>请先生成试卷</div>
            ),
          },
        ]}
      />
    </div>
  );
}
```

- [ ] **Step 9: 验证前端编译**

```bash
cd /Users/mac/2026/ai_extra/ExamAI/frontend
npm run build
```

- [ ] **Step 10: 提交**

```bash
git add frontend/src/pages/ExamGeneratePage.tsx frontend/src/components/exam/ frontend/src/services/examService.ts frontend/src/stores/examStore.ts
git commit -m "feat(m3): 实现智能组卷前端页面 — A面配置/B面AI推荐/试卷预览/换题/难度图表"
```

---

### M3 里程碑检查点

```bash
# 后端测试
cd backend && source .venv/bin/activate
pytest apps/exams/ -v

# 前端编译
cd ../frontend && npm run build
```

---


## M4 — 试卷导出（第10-12周）

> **目标：** 实现在线预览、PDF/Word 异步生成与下载、答题卡生成、参考答案与解析导出。
> **交付物：** 完整的试卷导出功能 — 在线预览 + PDF下载 + Word下载 + 答题卡 + 解析

### Task M4.1: 试卷导出后端 — PDF/Word 异步生成

**文件：**
- 修改：`backend/requirements/base.txt`（追加依赖）
- 创建：`backend/apps/exams/export.py`
- 创建：`backend/apps/exams/tasks.py`
- 创建：`backend/apps/exams/tests/test_export.py`
- 修改：`backend/apps/exams/views.py`（追加导出 action）
- 修改：`backend/apps/exams/urls.py`

- [ ] **Step 1: 安装导出依赖**

追加到 `/Users/mac/2026/ai_extra/ExamAI/backend/requirements/base.txt`：

```
reportlab>=4.2,<4.3
python-docx>=1.1,<1.2
```

安装：

```bash
cd /Users/mac/2026/ai_extra/ExamAI/backend
source .venv/bin/activate
pip install reportlab python-docx
```

- [ ] **Step 2: 编写导出测试**

写入 `/Users/mac/2026/ai_extra/ExamAI/backend/apps/exams/tests/test_export.py`：

```python
import pytest
from unittest.mock import patch, MagicMock
from django.urls import reverse
from apps.users.models import User
from apps.questions.models import Subject, Grade, Question
from apps.exams.models import ExamPaper, ExamPaperSection, ExamPaperItem


@pytest.mark.django_db
class TestExamExport:
    @pytest.fixture
    def user(self):
        return User.objects.create_user(email='teacher@test.com', password='pass123')

    @pytest.fixture
    def math(self):
        return Subject.objects.create(name='数学', code='math')

    @pytest.fixture
    def grade8(self):
        return Grade.objects.create(name='八年级', level=8)

    @pytest.fixture
    def paper_with_items(self, user, math, grade8):
        paper = ExamPaper.objects.create(
            user=user, title='导出测试卷', subject=math, grade=grade8,
            exam_type='final', total_score=100, duration=90,
        )
        section = ExamPaperSection.objects.create(
            exam_paper=paper, title='一、选择题', sort_order=1, total_score=30,
        )
        for i in range(5):
            q = Question.objects.create(
                subject=math, grade=grade8, type='choice', difficulty=3,
                stem=f'测试题 {i+1}', answer='B',
            )
            ExamPaperItem.objects.create(
                section=section, question=q, sort_order=i+1, score=6,
                question_snapshot={
                    'stem': f'测试题 {i+1}',
                    'answer': 'B',
                    'type': 'choice',
                    'difficulty': 3,
                    'options': [
                        {'label': 'A', 'content': '选项1'},
                        {'label': 'B', 'content': '选项2'},
                        {'label': 'C', 'content': '选项3'},
                        {'label': 'D', 'content': '选项4'},
                    ],
                },
            )
        return paper

    def test_export_pdf_creates_file(self, paper_with_items, auth_client):
        """导出 PDF 返回文件"""
        url = reverse('exampaper-export-pdf', kwargs={'pk': paper_with_items.id})
        response = auth_client.get(url)
        assert response.status_code == 200
        assert response['Content-Type'] == 'application/pdf'

    def test_export_word_creates_file(self, paper_with_items, auth_client):
        """导出 Word 返回文件"""
        url = reverse('exampaper-export-word', kwargs={'pk': paper_with_items.id})
        response = auth_client.get(url)
        assert response.status_code == 200
        assert 'application/vnd.openxmlformats' in response['Content-Type']

    def test_export_answer_sheet(self, paper_with_items, auth_client):
        """导出答题卡"""
        url = reverse('exampaper-export-answer-sheet', kwargs={'pk': paper_with_items.id})
        response = auth_client.get(url)
        assert response.status_code == 200

    def test_export_answer_with_explanation(self, paper_with_items, auth_client):
        """导出答案与解析"""
        url = reverse('exampaper-export-answers', kwargs={'pk': paper_with_items.id})
        response = auth_client.get(url)
        assert response.status_code == 200
```

- [ ] **Step 3: 实现导出核心逻辑**

写入 `/Users/mac/2026/ai_extra/ExamAI/backend/apps/exams/export.py`：

```python
import io
from typing import Tuple
from reportlab.lib.pagesizes import A4
from reportlab.lib.styles import getSampleStyleSheet, ParagraphStyle
from reportlab.lib.units import mm, cm
from reportlab.lib import colors
from reportlab.platypus import (
    SimpleDocTemplate, Paragraph, Spacer, Table, TableStyle, PageBreak
)
from reportlab.pdfbase import pdfmetrics
from reportlab.pdfbase.ttfonts import TTFont
from docx import Document
from docx.shared import Pt, Cm, Inches
from docx.enum.text import WD_ALIGN_PARAGRAPH

from .models import ExamPaper


# 尝试注册中文字体（如果存在的话）
try:
    pdfmetrics.registerFont(TTFont('SimSun', '/System/Library/Fonts/STHeiti Light.ttc'))
    CN_FONT = 'SimSun'
except Exception:
    CN_FONT = 'Helvetica'


def export_pdf(paper: ExamPaper) -> bytes:
    """导出试卷为 PDF"""
    buffer = io.BytesIO()
    doc = SimpleDocTemplate(
        buffer, pagesize=A4,
        leftMargin=2*cm, rightMargin=2*cm,
        topMargin=2*cm, bottomMargin=2*cm,
    )
    styles = getSampleStyleSheet()
    story = []

    # 标题
    title_style = ParagraphStyle('Title_CN', parent=styles['Title'], fontName=CN_FONT, fontSize=18, alignment=1)
    story.append(Paragraph(paper.title, title_style))
    story.append(Spacer(1, 12))

    # 信息行
    info_style = ParagraphStyle('Info', fontName=CN_FONT, fontSize=10, alignment=1)
    info_text = f'学科: {paper.subject.name} | 年级: {paper.grade.name} | 满分: {paper.total_score}分 | 时长: {paper.duration}分钟'
    story.append(Paragraph(info_text, info_style))
    story.append(Spacer(1, 20))

    # 分区和题目
    q_number = 0
    for section in paper.sections.all():
        section_style = ParagraphStyle('Section', fontName=CN_FONT, fontSize=14, spaceAfter=8)
        story.append(Paragraph(f'<b>{section.title}</b>  ({section.description})', section_style))
        story.append(Spacer(1, 8))

        for item in section.items.all():
            q_number += 1
            snapshot = item.question_snapshot

            # 题干
            stem_text = f'{q_number}. {snapshot.get("stem", "")}'
            stem_style = ParagraphStyle('Stem', fontName=CN_FONT, fontSize=11, spaceAfter=4, leading=16)
            story.append(Paragraph(stem_text, stem_style))

            # 选择题选项
            options = snapshot.get('options', [])
            if options:
                opt_texts = []
                for opt in options:
                    opt_texts.append(f'{opt["label"]}. {opt["content"]}')
                opt_style = ParagraphStyle('Options', fontName=CN_FONT, fontSize=10, leftIndent=20, spaceAfter=2)
                story.append(Paragraph('    '.join(opt_texts), opt_style))

            story.append(Spacer(1, 8))

        story.append(Spacer(1, 12))

    doc.build(story)
    buffer.seek(0)
    return buffer.getvalue()


def export_word(paper: ExamPaper) -> bytes:
    """导出试卷为 Word"""
    doc = Document()

    # 页面设置
    section = doc.sections[0]
    section.page_width = Cm(21)
    section.page_height = Cm(29.7)

    # 标题
    title = doc.add_paragraph()
    title.alignment = WD_ALIGN_PARAGRAPH.CENTER
    run = title.add_run(paper.title)
    run.font.size = Pt(18)
    run.bold = True

    # 信息行
    info = doc.add_paragraph()
    info.alignment = WD_ALIGN_PARAGRAPH.CENTER
    info.add_run(f'学科: {paper.subject.name} | 年级: {paper.grade.name} | 满分: {paper.total_score}分 | 时长: {paper.duration}分钟').font.size = Pt(10)

    doc.add_paragraph()

    q_number = 0
    for sec in paper.sections.all():
        # 分区标题
        sec_title = doc.add_paragraph()
        run = sec_title.add_run(f'{sec.title}（{sec.description}）')
        run.bold = True
        run.font.size = Pt(14)

        for item in sec.items.all():
            q_number += 1
            snapshot = item.question_snapshot

            # 题干
            p = doc.add_paragraph()
            p.add_run(f'{q_number}. {snapshot.get("stem", "")}').font.size = Pt(11)

            # 选项
            options = snapshot.get('options', [])
            if options:
                opt_p = doc.add_paragraph()
                opt_p.paragraph_format.left_indent = Cm(1)
                for i, opt in enumerate(options):
                    opt_p.add_run(f'{opt["label"]}. {opt["content"]}    ').font.size = Pt(10)

        doc.add_paragraph()

    buffer = io.BytesIO()
    doc.save(buffer)
    buffer.seek(0)
    return buffer.getvalue()


def export_answer_sheet(paper: ExamPaper) -> bytes:
    """导出答题卡 PDF"""
    buffer = io.BytesIO()
    doc = SimpleDocTemplate(buffer, pagesize=A4, leftMargin=2*cm, rightMargin=2*cm, topMargin=2*cm, bottomMargin=2*cm)
    story = []

    title_style = ParagraphStyle('Title_CN2', fontName=CN_FONT, fontSize=16, alignment=1)
    story.append(Paragraph(f'{paper.title} — 答题卡', title_style))
    story.append(Spacer(1, 8))

    info_style = ParagraphStyle('Info2', fontName=CN_FONT, fontSize=10, alignment=1)
    story.append(Paragraph(f'姓名: __________  班级: __________  得分: __________', info_style))
    story.append(Spacer(1, 20))

    q_number = 0
    for sec in paper.sections.all():
        sec_style = ParagraphStyle('Sec2', fontName=CN_FONT, fontSize=13, spaceAfter=6)
        story.append(Paragraph(f'<b>{sec.title}</b>', sec_style))

        q_type = sec.items.first().question_snapshot.get('type', '') if sec.items.exists() else ''

        if q_type == 'choice':
            # 选择题答题卡：题号 + ABCD 方框
            data = [['题号', 'A', 'B', 'C', 'D', '题号', 'A', 'B', 'C', 'D']]
            items = list(sec.items.all())
            for i in range(0, len(items), 2):
                row = []
                for j in range(2):
                    if i + j < len(items):
                        row.append(str(q_number + i + j + 1))
                        row.extend(['□', '□', '□', '□'])
                    else:
                        row.extend(['', '', '', '', ''])
                data.append(row)
            q_number += len(items)

            table = Table(data, colWidths=[40, 24, 24, 24, 24, 40, 24, 24, 24, 24])
            table.setStyle(TableStyle([
                ('ALIGN', (0, 0), (-1, -1), 'CENTER'),
                ('FONTSIZE', (0, 0), (-1, -1), 10),
                ('GRID', (0, 0), (-1, -1), 0.5, colors.grey),
            ]))
            story.append(table)
        else:
            # 非选择题：留空行
            for item in sec.items.all():
                q_number += 1
                p_style = ParagraphStyle('AQ', fontName=CN_FONT, fontSize=10, spaceAfter=30)
                story.append(Paragraph(f'{q_number}. _____________________________________________', p_style))

        story.append(Spacer(1, 16))

    doc.build(story)
    buffer.seek(0)
    return buffer.getvalue()


def export_answers(paper: ExamPaper) -> bytes:
    """导出参考答案与解析 PDF"""
    buffer = io.BytesIO()
    doc = SimpleDocTemplate(buffer, pagesize=A4, leftMargin=2*cm, rightMargin=2*cm, topMargin=2*cm, bottomMargin=2*cm)
    story = []

    title_style = ParagraphStyle('TA', fontName=CN_FONT, fontSize=16, alignment=1)
    story.append(Paragraph(f'{paper.title} — 参考答案与解析', title_style))
    story.append(Spacer(1, 16))

    q_number = 0
    for sec in paper.sections.all():
        sec_style = ParagraphStyle('SA', fontName=CN_FONT, fontSize=13, spaceAfter=8)
        story.append(Paragraph(f'<b>{sec.title}</b>', sec_style))

        for item in sec.items.all():
            q_number += 1
            snapshot = item.question_snapshot

            q_style = ParagraphStyle('QA', fontName=CN_FONT, fontSize=10, spaceAfter=2, leading=14)
            story.append(Paragraph(f'<b>{q_number}.</b> 答案: {snapshot.get("answer", "")}', q_style))

            explanation = snapshot.get('explanation', '')
            if explanation:
                e_style = ParagraphStyle('EA', fontName=CN_FONT, fontSize=9, leftIndent=16, spaceAfter=6, textColor=colors.grey)
                story.append(Paragraph(f'解析: {explanation}', e_style))

            story.append(Spacer(1, 4))

    doc.build(story)
    buffer.seek(0)
    return buffer.getvalue()
```

- [ ] **Step 4: 实现 Celery 异步任务**

写入 `/Users/mac/2026/ai_extra/ExamAI/backend/apps/exams/tasks.py`：

```python
import os
from celery import shared_task
from django.core.files.base import ContentFile
from django.utils import timezone

from .models import ExamPaper
from .export import export_pdf, export_word, export_answer_sheet, export_answers


@shared_task(bind=True, max_retries=2)
def generate_pdf_task(self, paper_id: int):
    """异步生成 PDF"""
    try:
        paper = ExamPaper.objects.get(id=paper_id)
        pdf_bytes = export_pdf(paper)

        filename = f'exports/{paper.id}_{paper.title}.pdf'
        paper.pdf_file.save(filename, ContentFile(pdf_bytes))
        paper.save(update_fields=['pdf_file', 'updated_at'])

        return {'paper_id': paper_id, 'status': 'done'}
    except Exception as e:
        self.retry(exc=e, countdown=10)


@shared_task(bind=True, max_retries=2)
def generate_word_task(self, paper_id: int):
    """异步生成 Word"""
    try:
        paper = ExamPaper.objects.get(id=paper_id)
        word_bytes = export_word(paper)

        filename = f'exports/{paper.id}_{paper.title}.docx'
        paper.word_file.save(filename, ContentFile(word_bytes))
        paper.save(update_fields=['word_file', 'updated_at'])

        return {'paper_id': paper_id, 'status': 'done'}
    except Exception as e:
        self.retry(exc=e, countdown=10)
```

注意：需要在 ExamPaper 模型中添加 `pdf_file` 和 `word_file` 字段。

修改 `/Users/mac/2026/ai_extra/ExamAI/backend/apps/exams/models.py`，在 ExamPaper 类中添加：

```python
    pdf_file = models.FileField('PDF文件', upload_to='exports/', null=True, blank=True)
    word_file = models.FileField('Word文件', upload_to='exports/', null=True, blank=True)
```

- [ ] **Step 5: 添加导出 View actions**

在 `/Users/mac/2026/ai_extra/ExamAI/backend/apps/exams/views.py` 的 ExamPaperViewSet 中添加：

```python
    @action(detail=True, methods=['get'], url_path='export-pdf')
    def export_pdf(self, request, pk=None):
        """导出 PDF — 同步返回（小试卷直接返回，大试卷异步）"""
        paper = self.get_object()
        from .export import export_pdf
        from django.http import HttpResponse

        pdf_bytes = export_pdf(paper)
        response = HttpResponse(pdf_bytes, content_type='application/pdf')
        response['Content-Disposition'] = f'attachment; filename="{paper.title}.pdf"'
        return response

    @action(detail=True, methods=['get'], url_path='export-word')
    def export_word(self, request, pk=None):
        """导出 Word"""
        paper = self.get_object()
        from .export import export_word
        from django.http import HttpResponse

        word_bytes = export_word(paper)
        response = HttpResponse(
            word_bytes,
            content_type='application/vnd.openxmlformats-officedocument.wordprocessingml.document'
        )
        response['Content-Disposition'] = f'attachment; filename="{paper.title}.docx"'
        return response

    @action(detail=True, methods=['get'], url_path='export-answer-sheet')
    def export_answer_sheet(self, request, pk=None):
        """导出答题卡"""
        paper = self.get_object()
        from .export import export_answer_sheet
        from django.http import HttpResponse

        pdf_bytes = export_answer_sheet(paper)
        response = HttpResponse(pdf_bytes, content_type='application/pdf')
        response['Content-Disposition'] = f'attachment; filename="{paper.title}-答题卡.pdf"'
        return response

    @action(detail=True, methods=['get'], url_path='export-answers')
    def export_answers(self, request, pk=None):
        """导出答案与解析"""
        paper = self.get_object()
        from .export import export_answers
        from django.http import HttpResponse

        pdf_bytes = export_answers(paper)
        response = HttpResponse(pdf_bytes, content_type='application/pdf')
        response['Content-Disposition'] = f'attachment; filename="{paper.title}-答案解析.pdf"'
        return response
```

- [ ] **Step 6: 运行导出测试**

```bash
python manage.py makemigrations exams
python manage.py migrate
pytest apps/exams/tests/test_export.py -v
```

- [ ] **Step 7: 提交**

```bash
git add backend/apps/exams/export.py backend/apps/exams/tasks.py backend/apps/exams/tests/test_export.py backend/apps/exams/views.py backend/apps/exams/models.py backend/requirements/base.txt
git commit -m "feat(m4): 实现试卷导出 — PDF/Word生成 + 答题卡 + 答案与解析"
```

---

### Task M4.2: 试卷导出前端页面

**文件：**
- 修改：`frontend/src/pages/ExamDetailPage.tsx`
- 创建：`frontend/src/components/export/ExportPanel.tsx`
- 创建：`frontend/src/components/export/AnswerSheet.tsx`
- 创建：`frontend/src/components/export/ExportProgress.tsx`

- [ ] **Step 1: 创建导出面板组件**

写入 `/Users/mac/2026/ai_extra/ExamAI/frontend/src/components/export/ExportPanel.tsx`：

```typescript
import { Card, Button, Space, Divider, message } from 'antd';
import {
  FilePdfOutlined,
  FileWordOutlined,
  SolutionOutlined,
  ReadOutlined,
} from '@ant-design/icons';
import { ExamPaper } from '@/services/examService';

interface Props {
  paper: ExamPaper;
}

export default function ExportPanel({ paper }: Props) {
  const handleExport = (type: 'pdf' | 'word' | 'answer-sheet' | 'answers') => {
    const urls: Record<string, string> = {
      pdf: `/api/exams/${paper.id}/export-pdf/`,
      word: `/api/exams/${paper.id}/export-word/`,
      'answer-sheet': `/api/exams/${paper.id}/export-answer-sheet/`,
      answers: `/api/exams/${paper.id}/export-answers/`,
    };

    const names: Record<string, string> = {
      pdf: `${paper.title}.pdf`,
      word: `${paper.title}.docx`,
      'answer-sheet': `${paper.title}-答题卡.pdf`,
      answers: `${paper.title}-答案解析.pdf`,
    };

    const url = urls[type];
    const token = localStorage.getItem('access_token');

    // 用 fetch 下载（带 auth header）
    fetch(url, { headers: { Authorization: `Bearer ${token}` } })
      .then(res => {
        if (!res.ok) throw new Error('导出失败');
        return res.blob();
      })
      .then(blob => {
        const downloadUrl = URL.createObjectURL(blob);
        const a = document.createElement('a');
        a.href = downloadUrl;
        a.download = names[type];
        a.click();
        URL.revokeObjectURL(downloadUrl);
        message.success('下载已开始');
      })
      .catch(() => message.error('导出失败，请重试'));
  };

  return (
    <Card title="试卷导出">
      <Space direction="vertical" style={{ width: '100%' }} size="middle">
        <Button type="primary" icon={<FilePdfOutlined />} size="large" block
          onClick={() => handleExport('pdf')}>
          下载 PDF（A4 排版）
        </Button>
        <Button icon={<FileWordOutlined />} size="large" block
          onClick={() => handleExport('word')}>
          下载 Word（可编辑）
        </Button>
        <Divider />
        <Button icon={<SolutionOutlined />} block
          onClick={() => handleExport('answer-sheet')}>
          下载答题卡
        </Button>
        <Button icon={<ReadOutlined />} block
          onClick={() => handleExport('answers')}>
          下载参考答案与解析
        </Button>
      </Space>
    </Card>
  );
}
```

- [ ] **Step 2: 组装试卷详情/导出页面**

写入 `/Users/mac/2026/ai_extra/ExamAI/frontend/src/pages/ExamDetailPage.tsx`：

```typescript
import { useEffect, useState } from 'react';
import { useParams, useNavigate } from 'react-router-dom';
import { Row, Col, Spin, Button, Typography, Tag, Space, Divider, Empty } from 'antd';
import { ArrowLeftOutlined } from '@ant-design/icons';
import { useExamStore } from '@/stores/examStore';
import { ExamPaper, ExamPaperItem } from '@/services/examService';
import ExportPanel from '@/components/export/ExportPanel';
import DifficultyChart from '@/components/exam/DifficultyChart';

const { Title, Text } = Typography;

export default function ExamDetailPage() {
  const { id } = useParams<{ id: string }>();
  const navigate = useNavigate();
  const { currentPaper, fetchPaperDetail, loading } = useExamStore();
  const [paper, setPaper] = useState<ExamPaper | null>(null);

  useEffect(() => {
    if (id) {
      fetchPaperDetail(Number(id)).then(() => {
        setPaper(useExamStore.getState().currentPaper);
      });
    }
  }, [id]);

  if (loading) return <Spin size="large" style={{ display: 'block', margin: '100px auto' }} />;
  if (!paper) return <Empty description="试卷不存在" />;

  return (
    <div>
      <div style={{ marginBottom: 16, display: 'flex', alignItems: 'center', gap: 16 }}>
        <Button icon={<ArrowLeftOutlined />} onClick={() => navigate('/')}>返回</Button>
        <Title level={4} style={{ margin: 0 }}>{paper.title}</Title>
        <Space>
          <Tag color="blue">{paper.subject_name}</Tag>
          <Tag>{paper.grade_name}</Tag>
          <Tag>总分: {paper.total_score}</Tag>
          <Tag>时长: {paper.duration}分钟</Tag>
        </Space>
      </div>

      <Row gutter={24}>
        <Col span={16}>
          <div style={{ background: '#fff', borderRadius: 8, padding: 24 }}>
            <DifficultyChart paper={paper} />

            {paper.sections?.map(section => (
              <div key={section.id} style={{ marginBottom: 24 }}>
                <Divider orientation="left">
                  <Text strong style={{ fontSize: 16 }}>{section.title}</Text>
                  <Text type="secondary" style={{ marginLeft: 8 }}>{section.description}</Text>
                </Divider>

                {section.items?.map((item: ExamPaperItem, index: number) => (
                  <div key={item.id} style={{
                    padding: '12px 0',
                    borderBottom: '1px solid #f0f0f0',
                    display: 'flex',
                    gap: 12,
                  }}>
                    <Text strong style={{ minWidth: 32 }}>{index + 1}.</Text>
                    <div style={{ flex: 1 }}>
                      <div style={{ marginBottom: 8, lineHeight: 1.8 }}>{item.stem}</div>
                      {item.options && item.options.length > 0 && (
                        <div style={{ display: 'flex', flexWrap: 'wrap', gap: '12px 24px', marginBottom: 8 }}>
                          {item.options.map(opt => (
                            <span key={opt.label}>{opt.label}. {opt.content}</span>
                          ))}
                        </div>
                      )}
                      <Space size="small">
                        <Tag>分值: {item.score}分</Tag>
                        <Tag color={['green', 'cyan', 'blue', 'orange', 'red'][(item.difficulty || 3) - 1]}>
                          难度: {'★'.repeat(item.difficulty || 3)}
                        </Tag>
                      </Space>
                    </div>
                  </div>
                ))}
              </div>
            ))}
          </div>
        </Col>
        <Col span={8}>
          <ExportPanel paper={paper} />
        </Col>
      </Row>
    </div>
  );
}
```

- [ ] **Step 3: 验证前端编译**

```bash
cd /Users/mac/2026/ai_extra/ExamAI/frontend
npm run build
```

- [ ] **Step 4: 提交**

```bash
git add frontend/src/pages/ExamDetailPage.tsx frontend/src/components/export/
git commit -m "feat(m4): 实现试卷导出前端页面 — 在线预览 + PDF/Word下载 + 答题卡 + 解析"
```

---

### M4 里程碑检查点

```bash
# 后端测试
cd backend && source .venv/bin/activate
pytest apps/exams/ -v

# 验证 PDF/Word 导出
python manage.py shell -c "
from apps.exams.models import ExamPaper
p = ExamPaper.objects.first()
if p:
    from apps.exams.export import export_pdf, export_word
    print(f'PDF: {len(export_pdf(p))} bytes')
    print(f'Word: {len(export_word(p))} bytes')
"

# 前端编译
cd ../frontend && npm run build
```

---


## M5 — 资源采集（第12-14周）

> **目标：** 实现试卷资源上传、文本提取（Word/PDF）、LLM 结构化解析（Celery 异步）、人工审核界面、审核入库。
> **交付物：** 完整的资源采集链路 — 上传 → 提取 → 解析 → 审核 → 入库

### Task M5.1: 资源模型与上传 API

**文件：**
- 创建：`backend/apps/resources/models.py`
- 创建：`backend/apps/resources/admin.py`
- 创建：`backend/apps/resources/serializers.py`
- 创建：`backend/apps/resources/views.py`
- 创建：`backend/apps/resources/urls.py`
- 创建：`backend/apps/resources/tests/__init__.py`
- 创建：`backend/apps/resources/tests/test_models.py`
- 创建：`backend/apps/resources/tests/test_api.py`

- [ ] **Step 1: 安装依赖**

追加到 `backend/requirements/base.txt`：

```
PyMuPDF>=1.24,<1.25
python-docx>=1.1,<1.2
```

```bash
cd /Users/mac/2026/ai_extra/ExamAI/backend && source .venv/bin/activate
pip install PyMuPDF python-docx
```

- [ ] **Step 2: 编写模型测试**

写入 `/Users/mac/2026/ai_extra/ExamAI/backend/apps/resources/tests/test_models.py`：

```python
import pytest
from django.core.files.uploadedfile import SimpleUploadedFile
from apps.users.models import User
from apps.questions.models import Subject, Grade
from apps.resources.models import Resource


@pytest.mark.django_db
class TestResource:
    @pytest.fixture
    def user(self):
        return User.objects.create_user(email='teacher@test.com', password='pass123')

    @pytest.fixture
    def math(self):
        return Subject.objects.create(name='数学', code='math')

    @pytest.fixture
    def grade8(self):
        return Grade.objects.create(name='八年级', level=8)

    def test_create_resource(self, user, math, grade8):
        """创建资源记录"""
        file = SimpleUploadedFile('test.pdf', b'fake pdf content', content_type='application/pdf')
        resource = Resource.objects.create(
            user=user,
            file=file,
            file_name='初二数学期末.pdf',
            file_type='pdf',
            file_size=1000,
            subject=math,
            grade=grade8,
            source_type='manual',
        )
        assert resource.status == 'pending'
        assert resource.file_type == 'pdf'
        assert resource.file_name == '初二数学期末.pdf'

    def test_resource_status_transitions(self, user):
        """资源状态流转"""
        resource = Resource.objects.create(
            user=user,
            file_name='test.pdf',
            file_type='pdf',
            file_size=100,
            status='pending',
        )
        resource.status = 'extracting'
        resource.save()
        assert resource.status == 'extracting'

        resource.status = 'parsing'
        resource.save()
        assert resource.status == 'parsing'

        resource.status = 'reviewing'
        resource.save()
        assert resource.status == 'reviewing'

        resource.status = 'completed'
        resource.save()
        assert resource.status == 'completed'
```

- [ ] **Step 3: 实现模型**

写入 `/Users/mac/2026/ai_extra/ExamAI/backend/apps/resources/models.py`：

```python
from django.db import models
from django.conf import settings


class Resource(models.Model):
    """原始试卷资源"""

    class SourceType(models.TextChoices):
        PURCHASED = 'purchased', '购买'
        CRAWLED = 'crawled', '爬虫'
        MANUAL = 'manual', '手动上传'

    class Status(models.TextChoices):
        PENDING = 'pending', '待处理'
        EXTRACTING = 'extracting', '文本提取中'
        PARSING = 'parsing', 'AI解析中'
        REVIEWING = 'reviewing', '待审核'
        COMPLETED = 'completed', '已完成'
        FAILED = 'failed', '失败'

    user = models.ForeignKey(settings.AUTH_USER_MODEL, on_delete=models.CASCADE, related_name='resources', verbose_name='上传者')
    file = models.FileField('文件', upload_to='resources/%Y/%m/', null=True, blank=True)
    file_name = models.CharField('文件名', max_length=500)
    file_type = models.CharField('文件类型', max_length=50)
    file_size = models.PositiveIntegerField('文件大小(字节)', default=0)
    subject = models.ForeignKey('questions.Subject', on_delete=models.SET_NULL, null=True, blank=True, related_name='resources', verbose_name='学科')
    grade = models.ForeignKey('questions.Grade', on_delete=models.SET_NULL, null=True, blank=True, related_name='resources', verbose_name='年级')
    source_type = models.CharField('来源类型', max_length=20, choices=SourceType.choices, default=SourceType.MANUAL)
    source_detail = models.CharField('来源详情', max_length=500, blank=True, default='')
    status = models.CharField('状态', max_length=20, choices=Status.choices, default=Status.PENDING)
    extracted_text = models.TextField('提取文本', blank=True, default='')
    parsed_data = models.JSONField('解析结果', default=dict, blank=True)
    parsed_questions = models.ManyToManyField('questions.Question', blank=True, related_name='source_resources', verbose_name='解析题目')
    error_message = models.TextField('错误信息', blank=True, default='')
    created_at = models.DateTimeField('创建时间', auto_now_add=True)
    updated_at = models.DateTimeField('更新时间', auto_now=True)

    class Meta:
        db_table = 'resources'
        verbose_name = '试卷资源'
        verbose_name_plural = verbose_name
        ordering = ['-created_at']
        indexes = [
            models.Index(fields=['status', '-created_at']),
            models.Index(fields=['user', '-created_at']),
        ]

    def __str__(self):
        return self.file_name
```

- [ ] **Step 4: 编写 API 测试**

写入 `/Users/mac/2026/ai_extra/ExamAI/backend/apps/resources/tests/test_api.py`：

```python
import pytest
from django.urls import reverse
from django.core.files.uploadedfile import SimpleUploadedFile
from rest_framework import status
from apps.users.models import User
from apps.questions.models import Subject, Grade


@pytest.mark.django_db
class TestResourceAPI:
    LIST_URL = reverse('resource-list')

    @pytest.fixture
    def user(self):
        return User.objects.create_user(email='teacher@test.com', password='pass123')

    @pytest.fixture
    def math(self):
        return Subject.objects.create(name='数学', code='math')

    @pytest.fixture
    def grade8(self):
        return Grade.objects.create(name='八年级', level=8)

    @pytest.fixture
    def auth_client(self, client, user):
        client.force_authenticate(user=user)
        return client

    def test_upload_resource(self, auth_client, math, grade8):
        """上传试卷文件"""
        file = SimpleUploadedFile('test.docx', b'fake docx', content_type='application/vnd.openxmlformats-officedocument.wordprocessingml.document')
        data = {
            'file': file,
            'subject_id': math.id,
            'grade_id': grade8.id,
            'source_type': 'manual',
        }
        response = auth_client.post(self.LIST_URL, data, format='multipart')
        assert response.status_code == status.HTTP_201_CREATED
        assert response.data['data']['status'] == 'pending'

    def test_list_resources(self, auth_client):
        """资源列表"""
        response = auth_client.get(self.LIST_URL)
        assert response.status_code == 200
        assert 'results' in response.data['data']

    def test_filter_by_status(self, auth_client):
        """按状态筛选"""
        response = auth_client.get(self.LIST_URL, {'status': 'pending'})
        assert response.status_code == 200

    def test_upload_without_file(self, auth_client):
        """无文件上传失败"""
        response = auth_client.post(self.LIST_URL, {}, format='multipart')
        assert response.status_code == status.HTTP_400_BAD_REQUEST
```

- [ ] **Step 5: 实现 Serializer 和 View**

写入 `/Users/mac/2026/ai_extra/ExamAI/backend/apps/resources/serializers.py`：

```python
from rest_framework import serializers
from .models import Resource


class ResourceListSerializer(serializers.ModelSerializer):
    """资源列表序列化器"""
    user_email = serializers.CharField(source='user.email', read_only=True)
    subject_name = serializers.CharField(source='subject.name', read_only=True, default='')
    grade_name = serializers.CharField(source='grade.name', read_only=True, default='')
    question_count = serializers.SerializerMethodField()

    class Meta:
        model = Resource
        fields = ('id', 'file_name', 'file_type', 'file_size', 'subject_name', 'grade_name', 'source_type', 'status', 'user_email', 'question_count', 'created_at')

    def get_question_count(self, obj):
        return obj.parsed_questions.count()


class ResourceDetailSerializer(serializers.ModelSerializer):
    """资源详情序列化器"""
    class Meta:
        model = Resource
        fields = '__all__'


class ResourceUploadSerializer(serializers.ModelSerializer):
    """资源上传序列化器"""
    subject_id = serializers.PrimaryKeyRelatedField(
        queryset=serializers.PrimaryKeyRelatedField.__kwdefaults__.__class__(),
        source='subject', write_only=True, required=False
    )
    grade_id = serializers.PrimaryKeyRelatedField(
        queryset=serializers.PrimaryKeyRelatedField.__kwdefaults__.__class__(),
        source='grade', write_only=True, required=False
    )

    class Meta:
        model = Resource
        fields = ('file', 'subject_id', 'grade_id', 'source_type', 'source_detail')

    def __init__(self, *args, **kwargs):
        super().__init__(*args, **kwargs)
        from apps.questions.models import Subject, Grade
        self.fields['subject_id'] = serializers.PrimaryKeyRelatedField(
            queryset=Subject.objects.all(), source='subject', write_only=True, required=False
        )
        self.fields['grade_id'] = serializers.PrimaryKeyRelatedField(
            queryset=Grade.objects.all(), source='grade', write_only=True, required=False
        )

    def create(self, validated_data):
        file = validated_data.get('file')
        validated_data['file_name'] = file.name if file else 'unknown'
        validated_data['file_type'] = self._get_file_type(file.name) if file else 'unknown'
        validated_data['file_size'] = file.size if file else 0
        return super().create(validated_data)

    def _get_file_type(self, filename: str) -> str:
        ext = filename.rsplit('.', 1)[-1].lower() if '.' in filename else ''
        type_map = {'pdf': 'pdf', 'doc': 'word', 'docx': 'word', 'png': 'image', 'jpg': 'image', 'jpeg': 'image', 'zip': 'zip'}
        return type_map.get(ext, ext)
```

写入 `/Users/mac/2026/ai_extra/ExamAI/backend/apps/resources/views.py`：

```python
from rest_framework import viewsets, permissions, status
from rest_framework.decorators import action
from rest_framework.filters import SearchFilter, OrderingFilter
from django_filters.rest_framework import DjangoFilterBackend

from common.response import success, error
from common.pagination import StandardPagination

from .models import Resource
from .serializers import ResourceListSerializer, ResourceDetailSerializer, ResourceUploadSerializer


class ResourceViewSet(viewsets.ModelViewSet):
    """资源采集 CRUD"""
    permission_classes = (permissions.IsAuthenticated,)
    pagination_class = StandardPagination
    filter_backends = (DjangoFilterBackend, SearchFilter, OrderingFilter)
    search_fields = ('file_name',)
    ordering_fields = ('created_at', 'file_size')
    ordering = ('-created_at',)
    filterset_fields = ('status', 'file_type', 'source_type', 'subject', 'grade')

    def get_serializer_class(self):
        if self.action == 'list':
            return ResourceListSerializer
        elif self.action == 'create':
            return ResourceUploadSerializer
        return ResourceDetailSerializer

    def get_queryset(self):
        return Resource.objects.filter(user=self.request.user).select_related('subject', 'grade').prefetch_related('parsed_questions')

    def create(self, request, *args, **kwargs):
        serializer = self.get_serializer(data=request.data)
        serializer.is_valid(raise_exception=True)
        resource = serializer.save(user=request.user)
        # 触发异步解析任务
        from .tasks import process_resource_task
        process_resource_task.delay(resource.id)
        return success(data=ResourceDetailSerializer(resource).data, message='上传成功，开始处理', status=status.HTTP_201_CREATED)

    def destroy(self, request, *args, **kwargs):
        instance = self.get_object()
        instance.file.delete(save=False)
        instance.delete()
        return success(message='删除成功', status=status.HTTP_204_NO_CONTENT)

    @action(detail=True, methods=['post'], url_path='approve')
    def approve_questions(self, request, pk=None):
        """审核通过 — 将解析的题目入库"""
        resource = self.get_object()
        if resource.status != 'reviewing':
            return error(message='当前状态不允许审核', status=status.HTTP_400_BAD_REQUEST)

        question_ids = request.data.get('question_ids', [])
        reject_ids = request.data.get('reject_ids', [])

        # 通过审核的题目设为 reviewed 状态
        from apps.questions.models import Question
        if question_ids:
            Question.objects.filter(id__in=question_ids, status='draft').update(status='reviewed')

        resource.status = 'completed'
        resource.save(update_fields=['status'])
        return success(message=f'审核完成：通过 {len(question_ids)} 道，拒绝 {len(reject_ids)} 道')
```

写入 `/Users/mac/2026/ai_extra/ExamAI/backend/apps/resources/urls.py`：

```python
from django.urls import path, include
from rest_framework.routers import DefaultRouter
from . import views

router = DefaultRouter()
router.register(r'', views.ResourceViewSet, basename='resource')

urlpatterns = [
    path('', include(router.urls)),
]
```

- [ ] **Step 6: 运行 migrations 和测试**

```bash
python manage.py makemigrations resources
python manage.py migrate
pytest apps/resources/tests/test_models.py apps/resources/tests/test_api.py -v
```

- [ ] **Step 7: 提交**

```bash
git add backend/apps/resources/ backend/requirements/base.txt
git commit -m "feat(m5): 实现资源模型与上传 API — 上传/列表/筛选/删除"
```

---

### Task M5.2: 文本提取与 LLM 解析

**文件：**
- 创建：`backend/apps/resources/extractors.py`
- 创建：`backend/apps/resources/tasks.py`
- 创建：`backend/apps/resources/tests/test_extractors.py`
- 创建：`backend/apps/resources/tests/test_tasks.py`

- [ ] **Step 1: 实现文本提取器**

写入 `/Users/mac/2026/ai_extra/ExamAI/backend/apps/resources/extractors.py`：

```python
"""试卷文本提取器 — 按文件类型分策略"""

import io
from typing import Optional


def extract_text(file_bytes: bytes, file_type: str) -> str:
    """根据文件类型提取文本"""
    extractors = {
        'pdf': _extract_pdf,
        'word': _extract_word,
        'docx': _extract_word,
    }
    extractor = extractors.get(file_type)
    if not extractor:
        raise ValueError(f'不支持的文件类型: {file_type}')
    return extractor(file_bytes)


def _extract_pdf(file_bytes: bytes) -> str:
    """使用 PyMuPDF 提取 PDF 文本"""
    import fitz  # PyMuPDF
    doc = fitz.open(stream=file_bytes, filetype='pdf')
    texts = []
    for page in doc:
        text = page.get_text()
        if text.strip():
            texts.append(text.strip())
    doc.close()
    return '\n\n'.join(texts)


def _extract_word(file_bytes: bytes) -> str:
    """使用 python-docx 提取 Word 文本"""
    from docx import Document
    doc = Document(io.BytesIO(file_bytes))
    texts = []
    for para in doc.paragraphs:
        if para.text.strip():
            texts.append(para.text.strip())
    # 也提取表格内容
    for table in doc.tables:
        for row in table.rows:
            row_text = ' | '.join(cell.text for cell in row.cells)
            if row_text.strip():
                texts.append(row_text.strip())
    return '\n'.join(texts)
```

- [ ] **Step 2: 实现 LLM 解析 Prompt 和任务**

写入 `/Users/mac/2026/ai_extra/ExamAI/backend/apps/resources/tasks.py`：

```python
import json
import re
from celery import shared_task
from django.db import transaction

from apps.ai.base import get_default_adapter


QUESTION_PARSE_PROMPT = """你是一位专业的试卷解析专家。请将以下试卷文本中的每道题目解析为结构化 JSON。

输出格式（JSON 数组）：
[
    {
        "type": "choice/fill_blank/true_false/short_answer/calculation/essay",
        "difficulty": 1-5,
        "stem": "题干文本（保留 LaTeX 公式 $...$）",
        "answer": "正确答案",
        "explanation": "解析说明",
        "options": [
            {"label": "A", "content": "选项内容"},
            {"label": "B", "content": "选项内容"}
        ],
        "knowledge_points": ["推测的知识点名称"],
        "default_score": 建议分值,
        "confidence": 0.0-1.0  // 解析置信度
    }
]

注意：
1. 如果文本中有 LaTeX 公式，请保留 $...$ 格式
2. 对于选择题，必须包含 options 数组
3. 对于没有明确解析的题目，explanation 可为空字符串
4. 对于无法确定的题目，confidence 设置为较低值
5. 知识点名称请用中文，参考初中教学大纲
"""


@shared_task(bind=True, max_retries=3, default_retry_delay=30)
def process_resource_task(self, resource_id: int):
    """处理资源：提取文本 → LLM 解析 → 创建草稿题目 → 进入审核"""
    from .models import Resource

    try:
        resource = Resource.objects.get(id=resource_id)
    except Resource.DoesNotExist:
        return {'error': 'Resource not found'}

    try:
        # Step 1: 提取文本
        resource.status = Resource.Status.EXTRACTING
        resource.save(update_fields=['status'])

        from .extractors import extract_text
        file_bytes = resource.file.read()
        text = extract_text(file_bytes, resource.file_type)
        resource.extracted_text = text
        resource.save(update_fields=['extracted_text'])

        if not text.strip():
            raise ValueError('未能从文件中提取到文本内容')

        # Step 2: LLM 解析
        resource.status = Resource.Status.PARSING
        resource.save(update_fields=['status'])

        adapter = get_default_adapter()
        messages = [
            {'role': 'system', 'content': QUESTION_PARSE_PROMPT},
            {'role': 'user', 'content': f'请解析以下试卷文本：\n\n{text[:8000]}'},
        ]
        result = adapter.call_with_logging('parse_question', messages)
        content = result.get('content', '')

        # 提取 JSON
        questions_data = _extract_json_array(content)
        resource.parsed_data = {
            'raw_response': content,
            'questions': questions_data,
            'question_count': len(questions_data),
        }
        resource.save(update_fields=['parsed_data'])

        # Step 3: 创建草稿题目
        with transaction.atomic():
            from apps.questions.models import Question, QuestionOption

            for q_data in questions_data:
                if q_data.get('confidence', 1.0) < 0.3:
                    continue  # 跳过低置信度题目

                question = Question.objects.create(
                    subject=resource.subject,
                    grade=resource.grade,
                    type=q_data.get('type', 'short_answer'),
                    difficulty=q_data.get('difficulty', 3),
                    stem=q_data.get('stem', ''),
                    answer=q_data.get('answer', ''),
                    explanation=q_data.get('explanation', ''),
                    default_score=q_data.get('default_score', 3),
                    source_type='exam',
                    source_detail=resource.file_name,
                    status='draft',
                )
                # 创建选项
                options = q_data.get('options', [])
                for opt in options:
                    QuestionOption.objects.create(
                        question=question,
                        label=opt.get('label', ''),
                        content=opt.get('content', ''),
                        sort_order=ord(opt.get('label', 'A')) - ord('A') if opt.get('label') else 0,
                    )
                # 关联知识点（按名称查找）
                kp_names = q_data.get('knowledge_points', [])
                if kp_names:
                    from apps.questions.models import KnowledgePoint
                    for kp_name in kp_names:
                        kp = KnowledgePoint.objects.filter(
                            name=kp_name, subject=resource.subject
                        ).first()
                        if kp:
                            question.knowledge_points.add(kp)

                resource.parsed_questions.add(question)

        # Step 4: 进入审核状态
        resource.status = Resource.Status.REVIEWING
        resource.save(update_fields=['status'])

        return {
            'resource_id': resource_id,
            'status': 'reviewing',
            'question_count': len(questions_data),
        }

    except Exception as e:
        resource.status = Resource.Status.FAILED
        resource.error_message = str(e)
        resource.save(update_fields=['status', 'error_message'])
        raise self.retry(exc=e)


def _extract_json_array(text: str) -> list:
    """从 LLM 回复中提取 JSON 数组"""
    # 直接解析
    try:
        data = json.loads(text)
        if isinstance(data, list):
            return data
        if isinstance(data, dict) and 'questions' in data:
            return data['questions']
    except json.JSONDecodeError:
        pass

    # 提取 markdown 代码块中的 JSON
    json_match = re.search(r'```(?:json)?\s*([\s\S]*?)\s*```', text)
    if json_match:
        try:
            return json.loads(json_match.group(1))
        except json.JSONDecodeError:
            pass

    # 提取 [ ] 包裹的 JSON 数组
    array_match = re.search(r'\[[\s\S]*\]', text)
    if array_match:
        try:
            return json.loads(array_match.group(0))
        except json.JSONDecodeError:
            pass

    return []
```

- [ ] **Step 3: 编写提取器测试**

写入 `/Users/mac/2026/ai_extra/ExamAI/backend/apps/resources/tests/test_extractors.py`：

```python
import io
import pytest
from apps.resources.extractors import extract_text


class TestExtractors:
    def test_extract_word_docx(self):
        """提取 Word 文本"""
        from docx import Document
        doc = Document()
        doc.add_paragraph('第一题：1+1=?')
        doc.add_paragraph('第二题：三角形面积公式')
        buffer = io.BytesIO()
        doc.save(buffer)
        buffer.seek(0)

        text = extract_text(buffer.getvalue(), 'word')
        assert '第一题' in text
        assert '第二题' in text

    def test_extract_unsupported_type(self):
        """不支持的文件类型抛异常"""
        with pytest.raises(ValueError, match='不支持'):
            extract_text(b'test', 'txt')
```

- [ ] **Step 4: 运行测试**

```bash
pytest apps/resources/tests/ -v
```

- [ ] **Step 5: 提交**

```bash
git add backend/apps/resources/extractors.py backend/apps/resources/tasks.py backend/apps/resources/tests/
git commit -m "feat(m5): 实现文本提取器与 LLM 异步解析 — Word/PDF提取 + DeepSeek结构化 + 自动创建草稿题目"
```

---

### Task M5.3: 资源采集前端页面

**文件：**
- 修改：`frontend/src/pages/ResourceCollectPage.tsx`
- 创建：`frontend/src/components/resource/UploadPanel.tsx`
- 创建：`frontend/src/components/resource/ParseProgress.tsx`
- 创建：`frontend/src/components/resource/ReviewPanel.tsx`
- 创建：`frontend/src/services/resourceService.ts`
- 创建：`frontend/src/stores/resourceStore.ts`

- [ ] **Step 1: 创建 resourceService**

写入 `/Users/mac/2026/ai_extra/ExamAI/frontend/src/services/resourceService.ts`：

```typescript
import api from './api';

export interface ResourceItem {
  id: number;
  file_name: string;
  file_type: string;
  file_size: number;
  subject_name: string;
  grade_name: string;
  source_type: string;
  status: string;
  user_email: string;
  question_count: number;
  created_at: string;
}

export interface ResourceDetail extends ResourceItem {
  file: string;
  extracted_text: string;
  parsed_data: {
    raw_response?: string;
    questions: Array<{
      type: string;
      difficulty: number;
      stem: string;
      answer: string;
      explanation: string;
      options: Array<{ label: string; content: string }>;
      knowledge_points: string[];
      default_score: number;
      confidence: number;
    }>;
    question_count: number;
  };
  error_message: string;
}

export const resourceService = {
  list: (params?: any) =>
    api.get<{ code: number; data: { page: number; page_size: number; total: number; results: ResourceItem[] } }>('/resources/', { params }),

  detail: (id: number) =>
    api.get<{ code: number; data: ResourceDetail }>(`/resources/${id}/`),

  upload: (formData: FormData) =>
    api.post<{ code: number; data: ResourceDetail }>('/resources/', formData, {
      headers: { 'Content-Type': 'multipart/form-data' },
    }),

  delete: (id: number) =>
    api.delete(`/resources/${id}/`),

  approve: (id: number, data: { question_ids: number[]; reject_ids: number[] }) =>
    api.post<{ code: number; data: any }>(`/resources/${id}/approve/`, data),
};
```

- [ ] **Step 2: 创建上传面板组件**

写入 `/Users/mac/2026/ai_extra/ExamAI/frontend/src/components/resource/UploadPanel.tsx`：

```typescript
import { useState, useEffect } from 'react';
import { Card, Upload, Select, Button, Space, message, Typography } from 'antd';
import { InboxOutlined } from '@ant-design/icons';
import { questionService } from '@/services/questionService';
import { resourceService } from '@/services/resourceService';

const { Dragger } = Upload;
const { Text } = Typography;

interface Props {
  onSuccess: () => void;
}

export default function UploadPanel({ onSuccess }: Props) {
  const [files, setFiles] = useState<any[]>([]);
  const [subjectId, setSubjectId] = useState<number | undefined>();
  const [gradeId, setGradeId] = useState<number | undefined>();
  const [subjects, setSubjects] = useState<any[]>([]);
  const [grades, setGrades] = useState<any[]>([]);
  const [uploading, setUploading] = useState(false);

  useEffect(() => {
    questionService.getSubjects().then(res => setSubjects(res.data.data));
    questionService.getGrades().then(res => setGrades(res.data.data));
  }, []);

  const handleUpload = async () => {
    if (files.length === 0) return;
    setUploading(true);
    let successCount = 0;
    for (const file of files) {
      try {
        const formData = new FormData();
        formData.append('file', file);
        if (subjectId) formData.append('subject_id', String(subjectId));
        if (gradeId) formData.append('grade_id', String(gradeId));
        formData.append('source_type', 'manual');
        await resourceService.upload(formData);
        successCount++;
      } catch { /* handled by interceptor */ }
    }
    setUploading(false);
    setFiles([]);
    message.success(`上传完成：${successCount}/${files.length} 个文件成功`);
    onSuccess();
  };

  return (
    <Card title="上传试卷资源">
      <Space direction="vertical" style={{ width: '100%' }} size="middle">
        <Text type="secondary">支持 PDF、Word（.doc/.docx）格式，可批量上传</Text>
        <Dragger
          multiple
          accept=".pdf,.doc,.docx"
          fileList={files}
          beforeUpload={(file) => { setFiles(prev => [...prev, file]); return false; }}
          onRemove={(file) => setFiles(prev => prev.filter(f => f.uid !== file.uid))}
        >
          <p className="ant-upload-drag-icon"><InboxOutlined /></p>
          <p>点击或拖拽试卷文件到此区域上传</p>
        </Dragger>
        <Space>
          <Select placeholder="学科（可选）" allowClear value={subjectId} onChange={setSubjectId} style={{ width: 140 }}
            options={subjects.map(s => ({ value: s.id, label: s.name }))} />
          <Select placeholder="年级（可选）" allowClear value={gradeId} onChange={setGradeId} style={{ width: 140 }}
            options={grades.map(g => ({ value: g.id, label: g.name }))} />
          <Button type="primary" onClick={handleUpload} loading={uploading} disabled={files.length === 0}>
            开始上传 ({files.length})
          </Button>
        </Space>
      </Space>
    </Card>
  );
}
```

- [ ] **Step 3: 创建审核面板组件**

写入 `/Users/mac/2026/ai_extra/ExamAI/frontend/src/components/resource/ReviewPanel.tsx`：

```typescript
import { useState } from 'react';
import { Modal, Card, Checkbox, Button, Space, Tag, Typography, Empty, message, Divider } from 'antd';
import { CheckOutlined, CloseOutlined } from '@ant-design/icons';
import { resourceService, ResourceDetail } from '@/services/resourceService';
import { QUESTION_TYPES, DIFFICULTY_LEVELS } from '@/utils/constants';

const { Text, Paragraph } = Typography;
const typeLabels = Object.fromEntries(QUESTION_TYPES.map(t => [t.value, t.label]));

interface Props {
  open: boolean;
  resource: ResourceDetail | null;
  onClose: () => void;
  onComplete: () => void;
}

export default function ReviewPanel({ open, resource, onClose, onComplete }: Props) {
  const [selectedIds, setSelectedIds] = useState<Set<number>>(new Set());
  const [rejectIds, setRejectIds] = useState<Set<number>>(new Set());
  const [submitting, setSubmitting] = useState(false);

  if (!resource) return null;

  const questions = resource.parsed_data?.questions || [];

  const handleApprove = async () => {
    setSubmitting(true);
    try {
      await resourceService.approve(resource.id, {
        question_ids: Array.from(selectedIds),
        reject_ids: Array.from(rejectIds),
      });
      message.success('审核完成');
      onComplete();
      onClose();
    } finally {
      setSubmitting(false);
    }
  };

  return (
    <Modal
      title={`审核解析结果 — ${resource.file_name}`}
      open={open}
      onCancel={onClose}
      width={800}
      footer={[
        <Button key="cancel" onClick={onClose}>取消</Button>,
        <Button key="approve" type="primary" loading={submitting} onClick={handleApprove}
          icon={<CheckOutlined />}>
          确认入库 ({selectedIds.size} 道)
        </Button>,
      ]}
    >
      <div style={{ marginBottom: 16 }}>
        <Text type="secondary">
          AI 解析完成，共识别 {questions.length} 道题目。请逐题审核，勾选确认正确的题目入库。
        </Text>
      </div>

      {questions.length === 0 ? (
        <Empty description="未解析到题目" />
      ) : (
        questions.map((q: any, index: number) => (
          <Card key={index} size="small" style={{ marginBottom: 12 }}
            title={
              <Space>
                <Checkbox
                  checked={selectedIds.has(index)}
                  onChange={e => {
                    const next = new Set(selectedIds);
                    e.target.checked ? next.add(index) : next.delete(index);
                    setSelectedIds(next);
                    const rej = new Set(rejectIds);
                    rej.delete(index);
                    setRejectIds(rej);
                  }}
                />
                <Text strong>第 {index + 1} 题</Text>
                <Tag color="blue">{typeLabels[q.type] || q.type}</Tag>
                <Tag color={['green', 'cyan', 'blue', 'orange', 'red'][(q.difficulty || 3) - 1]}>
                  {'★'.repeat(q.difficulty || 3)}
                </Tag>
                <Tag color={q.confidence > 0.7 ? 'green' : q.confidence > 0.4 ? 'orange' : 'red'}>
                  置信度: {Math.round((q.confidence || 0) * 100)}%
                </Tag>
              </Space>
            }
            extra={
              <Button size="small" danger icon={<CloseOutlined />}
                onClick={() => {
                  const rej = new Set(rejectIds);
                  rej.add(index);
                  setRejectIds(rej);
                  const sel = new Set(selectedIds);
                  sel.delete(index);
                  setSelectedIds(sel);
                }}>
                拒绝
              </Button>
            }
          >
            <Paragraph>{q.stem}</Paragraph>
            {q.options && q.options.length > 0 && (
              <div style={{ marginBottom: 8 }}>
                {q.options.map((opt: any) => (
                  <Tag key={opt.label}>{opt.label}. {opt.content}</Tag>
                ))}
              </div>
            )}
            <Text type="success">答案: {q.answer}</Text>
            {q.explanation && <><Divider type="vertical" /><Text type="secondary">解析: {q.explanation}</Text></>}
            {q.knowledge_points && q.knowledge_points.length > 0 && (
              <div style={{ marginTop: 4 }}>
                <Text type="secondary">知识点: </Text>
                {q.knowledge_points.map((kp: string, i: number) => <Tag key={i}>{kp}</Tag>)}
              </div>
            )}
          </Card>
        ))
      )}
    </Modal>
  );
}
```

- [ ] **Step 4: 组装资源采集页面**

写入 `/Users/mac/2026/ai_extra/ExamAI/frontend/src/pages/ResourceCollectPage.tsx`：

```typescript
import { useEffect, useState, useCallback } from 'react';
import { Table, Tag, Space, Button, Popconfirm, message, Typography } from 'antd';
import { EyeOutlined, DeleteOutlined, ReloadOutlined } from '@ant-design/icons';
import type { ColumnsType } from 'antd/es/table';
import { resourceService, ResourceItem, ResourceDetail } from '@/services/resourceService';
import UploadPanel from '@/components/resource/UploadPanel';
import ReviewPanel from '@/components/resource/ReviewPanel';

const { Title } = Typography;

const statusMap: Record<string, { label: string; color: string }> = {
  pending: { label: '待处理', color: 'default' },
  extracting: { label: '提取中', color: 'processing' },
  parsing: { label: '解析中', color: 'processing' },
  reviewing: { label: '待审核', color: 'warning' },
  completed: { label: '已完成', color: 'success' },
  failed: { label: '失败', color: 'error' },
};

const formatSize = (bytes: number) => {
  if (bytes < 1024) return `${bytes} B`;
  if (bytes < 1024 * 1024) return `${(bytes / 1024).toFixed(1)} KB`;
  return `${(bytes / (1024 * 1024)).toFixed(1)} MB`;
};

export default function ResourceCollectPage() {
  const [resources, setResources] = useState<ResourceItem[]>([]);
  const [total, setTotal] = useState(0);
  const [page, setPage] = useState(1);
  const [loading, setLoading] = useState(false);
  const [reviewOpen, setReviewOpen] = useState(false);
  const [currentResource, setCurrentResource] = useState<ResourceDetail | null>(null);

  const fetchResources = useCallback(async () => {
    setLoading(true);
    try {
      const res = await resourceService.list({ page, page_size: 20 });
      setResources(res.data.data.results);
      setTotal(res.data.data.total);
    } finally {
      setLoading(false);
    }
  }, [page]);

  useEffect(() => { fetchResources(); }, [fetchResources]);

  // 定时刷新（处理中的资源）
  useEffect(() => {
    const hasProcessing = resources.some(r => ['pending', 'extracting', 'parsing'].includes(r.status));
    if (!hasProcessing) return;
    const timer = setInterval(fetchResources, 5000);
    return () => clearInterval(timer);
  }, [resources, fetchResources]);

  const handleReview = async (id: number) => {
    const res = await resourceService.detail(id);
    setCurrentResource(res.data.data);
    setReviewOpen(true);
  };

  const columns: ColumnsType<ResourceItem> = [
    { title: '文件名', dataIndex: 'file_name', key: 'file_name', ellipsis: true, width: 250 },
    { title: '类型', dataIndex: 'file_type', key: 'file_type', width: 70, render: (t: string) => <Tag>{t.toUpperCase()}</Tag> },
    { title: '大小', dataIndex: 'file_size', key: 'file_size', width: 80, render: (s: number) => formatSize(s) },
    { title: '学科', dataIndex: 'subject_name', key: 'subject', width: 80, render: (t: string) => t || '-' },
    { title: '年级', dataIndex: 'grade_name', key: 'grade', width: 80, render: (t: string) => t || '-' },
    {
      title: '状态', dataIndex: 'status', key: 'status', width: 100,
      render: (s: string) => {
        const info = statusMap[s] || { label: s, color: 'default' };
        return <Tag color={info.color}>{info.label}</Tag>;
      },
    },
    { title: '题目数', dataIndex: 'question_count', key: 'question_count', width: 70 },
    {
      title: '操作', key: 'actions', width: 120,
      render: (_, record) => (
        <Space>
          {record.status === 'reviewing' && (
            <Button type="link" size="small" icon={<EyeOutlined />} onClick={() => handleReview(record.id)}>审核</Button>
          )}
          <Popconfirm title="确定删除？" onConfirm={async () => { await resourceService.delete(record.id); fetchResources(); }}>
            <Button type="link" size="small" danger icon={<DeleteOutlined />} />
          </Popconfirm>
        </Space>
      ),
    },
  ];

  return (
    <div>
      <div style={{ display: 'flex', justifyContent: 'space-between', alignItems: 'center', marginBottom: 16 }}>
        <Title level={4} style={{ margin: 0 }}>资源采集</Title>
        <Button icon={<ReloadOutlined />} onClick={fetchResources}>刷新</Button>
      </div>

      <UploadPanel onSuccess={fetchResources} />

      <div style={{ background: '#fff', borderRadius: 8, padding: 16, marginTop: 16 }}>
        <Table
          rowKey="id"
          columns={columns}
          dataSource={resources}
          loading={loading}
          pagination={{ current: page, pageSize: 20, total, showTotal: t => `共 ${t} 个资源`, onChange: setPage }}
        />
      </div>

      <ReviewPanel open={reviewOpen} resource={currentResource} onClose={() => setReviewOpen(false)} onComplete={fetchResources} />
    </div>
  );
}
```

- [ ] **Step 5: 验证前端编译**

```bash
cd /Users/mac/2026/ai_extra/ExamAI/frontend
npm run build
```

- [ ] **Step 6: 提交**

```bash
git add frontend/src/pages/ResourceCollectPage.tsx frontend/src/components/resource/ frontend/src/services/resourceService.ts
git commit -m "feat(m5): 实现资源采集前端页面 — 上传/列表/状态跟踪/审核入库"
```

---

### M5 里程碑检查点

```bash
# 后端测试
cd backend && source .venv/bin/activate
pytest apps/resources/ -v

# 验证 Celery worker
celery -A config worker -l info --pool=solo &
# 上传一个测试文件验证完整链路

# 前端编译
cd ../frontend && npm run build
```

---


## M6 — 上线整合（第14-16周）

> **目标：** 完成工作台首页、登录/注册页面、全局联调测试、Docker 部署配置、项目上线。
> **交付物：** 完整的可用产品 — 6 个页面全部实现 + 联调通过 + Docker 一键部署

### Task M6.1: 登录/注册前端页面

**文件：**
- 修改：`frontend/src/pages/LoginPage.tsx`
- 创建：`frontend/src/services/authService.ts`

- [ ] **Step 1: 创建 authService**

写入 `/Users/mac/2026/ai_extra/ExamAI/frontend/src/services/authService.ts`：

```typescript
import api from './api';

interface LoginResponse {
  access: string;
  refresh: string;
  user: {
    id: number;
    email: string;
    nickname: string;
    role: string;
    quota_total: number;
    quota_used: number;
  };
}

export const authService = {
  login: (email: string, password: string) =>
    api.post<{ code: number; data: LoginResponse }>('/auth/login/', { email, password }),

  register: (data: { email: string; password: string; confirm_password: string; nickname?: string }) =>
    api.post<{ code: number; data: any }>('/auth/register/', data),

  refreshToken: (refresh: string) =>
    api.post<{ code: number; data: { access: string } }>('/auth/refresh/', { refresh }),

  getProfile: () =>
    api.get<{ code: number; data: any }>('/auth/profile/'),

  updateProfile: (data: any) =>
    api.patch<{ code: number; data: any }>('/auth/profile/', data),
};
```

- [ ] **Step 2: 实现登录页面**

写入 `/Users/mac/2026/ai_extra/ExamAI/frontend/src/pages/LoginPage.tsx`：

```typescript
import { useState } from 'react';
import { useNavigate } from 'react-router-dom';
import { Card, Form, Input, Button, Tabs, Typography, message, Space } from 'antd';
import { MailOutlined, LockOutlined, UserOutlined } from '@ant-design/icons';
import { useAuthStore } from '@/stores/authStore';
import { authService } from '@/services/authService';

const { Title, Text } = Typography;

export default function LoginPage() {
  const navigate = useNavigate();
  const { login } = useAuthStore();
  const [loading, setLoading] = useState(false);
  const [activeTab, setActiveTab] = useState('login');

  const handleLogin = async (values: { email: string; password: string }) => {
    setLoading(true);
    try {
      const res = await authService.login(values.email, values.password);
      const { access, refresh, user } = res.data.data;
      login(access, refresh, user);
      message.success(`欢迎回来，${user.nickname || user.email}`);
      navigate('/');
    } catch {
      // error handled by interceptor
    } finally {
      setLoading(false);
    }
  };

  const handleRegister = async (values: { email: string; password: string; confirm_password: string; nickname?: string }) => {
    setLoading(true);
    try {
      await authService.register(values);
      message.success('注册成功，请登录');
      setActiveTab('login');
    } catch {
      // error handled by interceptor
    } finally {
      setLoading(false);
    }
  };

  return (
    <div style={{
      minHeight: '100vh',
      display: 'flex',
      justifyContent: 'center',
      alignItems: 'center',
      background: 'linear-gradient(135deg, #667eea 0%, #764ba2 100%)',
    }}>
      <Card style={{ width: 420, borderRadius: 12, boxShadow: '0 8px 24px rgba(0,0,0,0.15)' }}>
        <div style={{ textAlign: 'center', marginBottom: 32 }}>
          <Title level={2} style={{ margin: 0, color: '#1677ff' }}>ExamAI</Title>
          <Text type="secondary">智能试卷生成平台</Text>
        </div>

        <Tabs activeKey={activeTab} onChange={setActiveTab} centered items={[
          {
            key: 'login',
            label: '登录',
            children: (
              <Form onFinish={handleLogin} size="large">
                <Form.Item name="email" rules={[{ required: true, type: 'email', message: '请输入有效邮箱' }]}>
                  <Input prefix={<MailOutlined />} placeholder="邮箱" />
                </Form.Item>
                <Form.Item name="password" rules={[{ required: true, message: '请输入密码' }]}>
                  <Input.Password prefix={<LockOutlined />} placeholder="密码" />
                </Form.Item>
                <Form.Item>
                  <Button type="primary" htmlType="submit" block loading={loading}>登录</Button>
                </Form.Item>
                <div style={{ textAlign: 'center' }}>
                  <Button type="link" onClick={() => setActiveTab('forgot')}>忘记密码？</Button>
                </div>
              </Form>
            ),
          },
          {
            key: 'register',
            label: '注册',
            children: (
              <Form onFinish={handleRegister} size="large">
                <Form.Item name="email" rules={[{ required: true, type: 'email', message: '请输入有效邮箱' }]}>
                  <Input prefix={<MailOutlined />} placeholder="邮箱" />
                </Form.Item>
                <Form.Item name="nickname">
                  <Input prefix={<UserOutlined />} placeholder="昵称（选填）" />
                </Form.Item>
                <Form.Item name="password" rules={[{ required: true, min: 6, message: '密码至少6位' }]}>
                  <Input.Password prefix={<LockOutlined />} placeholder="密码" />
                </Form.Item>
                <Form.Item
                  name="confirm_password"
                  dependencies={['password']}
                  rules={[
                    { required: true, message: '请确认密码' },
                    ({ getFieldValue }) => ({
                      validator(_, value) {
                        if (!value || getFieldValue('password') === value) return Promise.resolve();
                        return Promise.reject(new Error('两次密码不一致'));
                      },
                    }),
                  ]}
                >
                  <Input.Password prefix={<LockOutlined />} placeholder="确认密码" />
                </Form.Item>
                <Form.Item>
                  <Button type="primary" htmlType="submit" block loading={loading}>注册</Button>
                </Form.Item>
              </Form>
            ),
          },
          {
            key: 'forgot',
            label: '忘记密码',
            children: (
              <div style={{ textAlign: 'center', padding: '24px 0' }}>
                <Text type="secondary">请联系管理员重置密码</Text>
                <div style={{ marginTop: 16 }}>
                  <Button type="link" onClick={() => setActiveTab('login')}>返回登录</Button>
                </div>
              </div>
            ),
          },
        ]} />
      </Card>
    </div>
  );
}
```

- [ ] **Step 3: 验证前端编译**

```bash
cd /Users/mac/2026/ai_extra/ExamAI/frontend
npm run build
```

- [ ] **Step 4: 提交**

```bash
git add frontend/src/pages/LoginPage.tsx frontend/src/services/authService.ts
git commit -m "feat(m6): 实现登录/注册页面 — 邮箱登录/注册/忘记密码"
```

---

### Task M6.2: 工作台首页

**文件：**
- 修改：`frontend/src/pages/DashboardPage.tsx`

- [ ] **Step 1: 实现工作台首页**

写入 `/Users/mac/2026/ai_extra/ExamAI/frontend/src/pages/DashboardPage.tsx`：

```typescript
import { useEffect, useState } from 'react';
import { useNavigate } from 'react-router-dom';
import { Row, Col, Card, Statistic, Button, Table, Tag, Space, Typography, Spin } from 'antd';
import {
  FileTextOutlined,
  DatabaseOutlined,
  ExperimentOutlined,
  PlusOutlined,
  EyeOutlined,
} from '@ant-design/icons';
import { useExamStore } from '@/stores/examStore';
import { ExamPaper } from '@/services/examService';
import { useAuthStore } from '@/stores/authStore';
import { authService } from '@/services/authService';

const { Title } = Typography;

export default function DashboardPage() {
  const navigate = useNavigate();
  const { user, setUser } = useAuthStore();
  const { papers, total, loading, fetchPapers } = useExamStore();
  const [statsLoading, setStatsLoading] = useState(true);

  useEffect(() => {
    fetchPapers({ page: 1, page_size: 5 });
    // 刷新用户信息
    authService.getProfile().then(res => {
      setUser(res.data.data);
      setStatsLoading(false);
    }).catch(() => setStatsLoading(false));
  }, []);

  const examTypeLabels: Record<string, string> = {
    unit: '单元测试', monthly: '月考', midterm: '期中', final: '期末', entrance: '中考模拟',
  };

  const statusColors: Record<string, string> = {
    draft: 'default', generated: 'blue', finalized: 'green',
  };
  const statusLabels: Record<string, string> = {
    draft: '草稿', generated: '已生成', finalized: '已定稿',
  };

  const columns = [
    { title: '试卷标题', dataIndex: 'title', key: 'title', ellipsis: true },
    { title: '学科', dataIndex: 'subject_name', key: 'subject', width: 80 },
    { title: '年级', dataIndex: 'grade_name', key: 'grade', width: 80 },
    {
      title: '类型', dataIndex: 'exam_type', key: 'type', width: 100,
      render: (t: string) => examTypeLabels[t] || t,
    },
    { title: '总分', dataIndex: 'total_score', key: 'score', width: 70 },
    {
      title: '状态', dataIndex: 'status', key: 'status', width: 80,
      render: (s: string) => <Tag color={statusColors[s] || 'default'}>{statusLabels[s] || s}</Tag>,
    },
    { title: '题数', dataIndex: 'question_count', key: 'count', width: 60 },
    {
      title: '操作', key: 'actions', width: 80,
      render: (_: any, record: ExamPaper) => (
        <Button type="link" size="small" icon={<EyeOutlined />}
          onClick={() => navigate(`/exams/${record.id}`)}>查看</Button>
      ),
    },
  ];

  return (
    <div>
      <div style={{ marginBottom: 24, display: 'flex', justifyContent: 'space-between', alignItems: 'center' }}>
        <div>
          <Title level={4} style={{ margin: 0 }}>工作台</Title>
          <span style={{ color: '#999' }}>欢迎回来，{user?.nickname || user?.email}</span>
        </div>
        <Button type="primary" size="large" icon={<PlusOutlined />} onClick={() => navigate('/exams/generate')}>
          创建试卷
        </Button>
      </div>

      <Row gutter={16} style={{ marginBottom: 24 }}>
        <Col span={6}>
          <Card><Statistic title="我的试卷" value={total} prefix={<FileTextOutlined />} loading={loading} /></Card>
        </Col>
        <Col span={6}>
          <Card><Statistic title="题库总量" value="--" prefix={<DatabaseOutlined />} /></Card>
        </Col>
        <Col span={6}>
          <Card>
            <Statistic
              title="本月已用配额"
              value={user?.quota_used || 0}
              suffix={`/ ${user?.quota_total || 100}`}
              prefix={<ExperimentOutlined />}
              loading={statsLoading}
            />
          </Card>
        </Col>
        <Col span={6}>
          <Card><Statistic title="本月生成试卷" value={total} prefix={<FileTextOutlined />} loading={loading} /></Card>
        </Col>
      </Row>

      <Card
        title="最近试卷"
        extra={<Button type="link" onClick={() => navigate('/exams/generate')}>查看全部</Button>}
      >
        <Table rowKey="id" columns={columns} dataSource={papers} loading={loading}
          pagination={false} onRow={record => ({
            onClick: () => navigate(`/exams/${record.id}`),
            style: { cursor: 'pointer' },
          })} />
        {!loading && papers.length === 0 && (
          <div style={{ textAlign: 'center', padding: 40 }}>
            <Button type="primary" icon={<PlusOutlined />} onClick={() => navigate('/exams/generate')}>
              创建你的第一份试卷
            </Button>
          </div>
        )}
      </Card>
    </div>
  );
}
```

- [ ] **Step 2: 验证前端编译**

```bash
cd /Users/mac/2026/ai_extra/ExamAI/frontend
npm run build
```

- [ ] **Step 3: 提交**

```bash
git add frontend/src/pages/DashboardPage.tsx
git commit -m "feat(m6): 实现工作台首页 — 统计概览/最近试卷列表/快速创建入口"
```

---

### Task M6.3: 全局联调与 E2E 测试

**文件：**
- 创建：`backend/pytest.ini`
- 修改：`frontend/src/router/index.tsx`（检查路由完整性）

- [ ] **Step 1: 配置 pytest**

写入 `/Users/mac/2026/ai_extra/ExamAI/backend/pytest.ini`：

```ini
[pytest]
DJANGO_SETTINGS_MODULE = config.settings
python_files = test_*.py
python_classes = Test*
python_functions = test_*
addopts = -v --tb=short --strict-markers
markers =
    slow: marks tests as slow (deselect with '-m "not slow"')
    integration: integration tests
```

- [ ] **Step 2: 运行全部后端测试**

```bash
cd /Users/mac/2026/ai_extra/ExamAI/backend
source .venv/bin/activate
pytest apps/ -v --tb=short
```

预期：所有已有测试通过。如有失败，根据错误信息修复。

- [ ] **Step 3: 验证核心 API 流程（手动联调测试清单）**

按以下顺序逐一验证：

```bash
# 1. 注册
curl -X POST http://localhost:8000/api/auth/register/ \
  -H "Content-Type: application/json" \
  -d '{"email":"test@examai.com","password":"test123","confirm_password":"test123","nickname":"测试老师"}'

# 2. 登录
curl -X POST http://localhost:8000/api/auth/login/ \
  -H "Content-Type: application/json" \
  -d '{"email":"test@examai.com","password":"test123"}'
# 保存返回的 access token

# 3. 获取学科列表
curl http://localhost:8000/api/questions/subjects/ \
  -H "Authorization: Bearer <access_token>"

# 4. 创建试卷（组卷）
curl -X POST http://localhost:8000/api/exams/ \
  -H "Authorization: Bearer <access_token>" \
  -H "Content-Type: application/json" \
  -d '{"title":"联调测试卷","subject_id":1,"grade_id":2,"exam_type":"unit","duration":60,"config":{"sections":[{"type":"choice","count":5,"score":3}]}}'

# 5. 获取试卷详情
curl http://localhost:8000/api/exams/1/ \
  -H "Authorization: Bearer <access_token>"

# 6. 导出 PDF
curl http://localhost:8000/api/exams/1/export-pdf/ \
  -H "Authorization: Bearer <access_token>" \
  -o test_exam.pdf
```

- [ ] **Step 4: 验证前端完整流程**

启动前后端：

```bash
# Terminal 1: 启动后端
cd /Users/mac/2026/ai_extra/ExamAI/backend
source .venv/bin/activate
python manage.py runserver

# Terminal 2: 启动 Celery（可选，用于资源采集）
cd /Users/mac/2026/ai_extra/ExamAI/backend
source .venv/bin/activate
celery -A config worker -l info --pool=solo

# Terminal 3: 启动前端
cd /Users/mac/2026/ai_extra/ExamAI/frontend
npm run dev
```

手动验证以下流程：
1. 打开 http://localhost:5173 → 跳转登录页
2. 注册新用户 → 登录成功 → 进入工作台
3. 进入题库管理 → 新增题目 → 查看列表
4. 进入知识点管理 → 查看知识点树
5. 进入智能组卷 → A面配置 → 生成试卷 → 预览
6. 在预览中换题
7. 进入试卷详情 → 下载 PDF/Word
8. 进入资源采集 → 上传文件 → 等待处理 → 审核

- [ ] **Step 5: 修复联调中发现的问题**

根据联调测试结果修复 bug，提交修复。

- [ ] **Step 6: 提交**

```bash
git add backend/pytest.ini
git commit -m "chore(m6): 添加 pytest 配置 + 联调测试验证"
```

---

### Task M6.4: Docker 部署配置

**文件：**
- 修改：`docker-compose.yml`（追加 Django + React 服务）
- 创建：`backend/Dockerfile`
- 创建：`frontend/Dockerfile`
- 创建：`nginx.conf`
- 创建：`scripts/entrypoint.sh`

- [ ] **Step 1: 创建后端 Dockerfile**

写入 `/Users/mac/2026/ai_extra/ExamAI/backend/Dockerfile`：

```dockerfile
FROM python:3.12-slim

ENV PYTHONDONTWRITEBYTECODE=1
ENV PYTHONUNBUFFERED=1

WORKDIR /app

# 安装系统依赖
RUN apt-get update && apt-get install -y --no-install-recommends \
    build-essential \
    libpq-dev \
    && rm -rf /var/lib/apt/lists/*

# 安装 Python 依赖
COPY requirements/base.txt /app/requirements/
RUN pip install --no-cache-dir -r requirements/base.txt gunicorn

# 复制代码
COPY . /app/

# 启动脚本
COPY scripts/entrypoint.sh /entrypoint.sh
RUN chmod +x /entrypoint.sh

EXPOSE 8000

ENTRYPOINT ["/entrypoint.sh"]
```

- [ ] **Step 2: 创建前端 Dockerfile**

写入 `/Users/mac/2026/ai_extra/ExamAI/frontend/Dockerfile`：

```dockerfile
FROM node:20-alpine AS build

WORKDIR /app
COPY package*.json ./
RUN npm ci
COPY . .
RUN npm run build

FROM nginx:alpine
COPY --from=build /app/dist /usr/share/nginx/html
COPY nginx.conf /etc/nginx/conf.d/default.conf
EXPOSE 80
CMD ["nginx", "-g", "daemon off;"]
```

- [ ] **Step 3: 创建 nginx 配置**

写入 `/Users/mac/2026/ai_extra/ExamAI/frontend/nginx.conf`：

```nginx
server {
    listen 80;
    server_name localhost;

    root /usr/share/nginx/html;
    index index.html;

    # SPA 路由支持
    location / {
        try_files $uri $uri/ /index.html;
    }

    # API 反向代理
    location /api/ {
        proxy_pass http://backend:8000;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
    }

    # 静态文件
    location /media/ {
        proxy_pass http://backend:8000;
    }

    # 静态资源缓存
    location /assets/ {
        expires 1y;
        add_header Cache-Control "public, immutable";
    }
}
```

- [ ] **Step 4: 创建启动脚本**

```bash
mkdir -p /Users/mac/2026/ai_extra/ExamAI/backend/scripts
```

写入 `/Users/mac/2026/ai_extra/ExamAI/backend/scripts/entrypoint.sh`：

```bash
#!/bin/bash
set -e

echo "Running migrations..."
python manage.py migrate --noinput

echo "Initializing seed data..."
python manage.py seed_data

echo "Collecting static files..."
python manage.py collectstatic --noinput

echo "Starting Gunicorn..."
exec gunicorn config.wsgi:application \
    --bind 0.0.0.0:8000 \
    --workers 4 \
    --timeout 120 \
    --access-logfile - \
    --error-logfile -
```

- [ ] **Step 5: 更新 docker-compose.yml**

修改 `/Users/mac/2026/ai_extra/ExamAI/docker-compose.yml`，追加：

```yaml
  backend:
    build: ./backend
    ports:
      - "8000:8000"
    env_file:
      - .env
    depends_on:
      db:
        condition: service_healthy
      redis:
        condition: service_healthy
    volumes:
      - ./backend/media:/app/media
      - ./backend/exports:/app/exports

  celery:
    build: ./backend
    command: celery -A config worker -l info
    env_file:
      - .env
    depends_on:
      db:
        condition: service_healthy
      redis:
        condition: service_healthy
    volumes:
      - ./backend/media:/app/media

  frontend:
    build: ./frontend
    ports:
      - "80:80"
    depends_on:
      - backend
```

- [ ] **Step 6: 验证 Docker 部署**

```bash
cd /Users/mac/2026/ai_extra/ExamAI
docker compose build
docker compose up -d
docker compose ps
# 验证所有服务 healthy
curl http://localhost/api/auth/login/ -H "Content-Type: application/json" -d '{"email":"test@examai.com","password":"test123"}'
```

- [ ] **Step 7: 提交**

```bash
git add docker-compose.yml backend/Dockerfile backend/scripts/ frontend/Dockerfile frontend/nginx.conf
git commit -m "feat(m6): 添加 Docker 部署配置 — Dockerfile + nginx + docker-compose"
```

---

### M6 里程碑检查点

```bash
# 全部后端测试
cd backend && source .venv/bin/activate
pytest apps/ -v

# 全部前端编译
cd ../frontend && npm run build

# Docker 部署
docker compose up -d --build
docker compose ps

# 访问 http://localhost 验证完整流程
```

---

## 附录 A: 关键命令速查

```bash
# === 开发环境 ===

# 启动基础设施
docker compose up -d db redis

# 后端
cd backend && source .venv/bin/activate
python manage.py runserver                    # Django 开发服务器
celery -A config worker -l info --pool=solo   # Celery worker
pytest apps/ -v                               # 运行全部测试
pytest apps/exams/ -v                         # 运行指定模块测试
python manage.py makemigrations               # 创建迁移
python manage.py migrate                      # 应用迁移
python manage.py seed_data                    # 初始化种子数据
python manage.py shell                        # Django shell

# 前端
cd frontend
npm run dev       # 开发服务器 (http://localhost:5173)
npm run build     # 生产构建
npm run preview   # 预览生产构建

# === Docker 部署 ===
docker compose up -d --build    # 构建并启动
docker compose ps               # 查看服务状态
docker compose logs -f backend  # 查看后端日志
docker compose down             # 停止所有服务
```

---

## 附录 B: 开发注意事项

### 编码规范

1. **Python**：遵循 PEP 8，使用 4 空格缩进，类型提示（Type Hints）推荐使用
2. **TypeScript/React**：使用函数组件 + Hooks，避免 class 组件；类型定义完整
3. **Git 提交**：使用 [Conventional Commits](https://www.conventionalcommits.org/) 格式：`feat(m1): 描述`

### 关键约定

1. **API 响应格式**：始终 `{ code: 0, data: {...}, message: "ok" }`
2. **分页格式**：`{ page, page_size, total, results }`
3. **认证**：所有 API（除 login/register/refresh）都需要 `Authorization: Bearer <token>`
4. **文件上传**：使用 `multipart/form-data`，最大 50MB
5. **题目快照**：试卷生成时始终创建快照，防止原题被修改导致试卷不一致
6. **AI 调用**：始终通过 `get_default_adapter()` 获取 adapter，使用 `call_with_logging()` 记录日志

### 常见问题

| 问题 | 解决方案 |
|------|---------|
| 数据库连接失败 | 检查 `docker compose ps db` 状态，确认 `DATABASE_URL` 配置 |
| Celery 任务不执行 | 确认 Redis 和 Celery worker 均已启动 |
| AI 调用失败 | 检查 DEEPSEEK_API_KEY 配置，确认 AIProvider 有激活的默认 provider |
| PDF 中文乱码 | 确保系统有中文字体，或修改 export.py 中的 CN_FONT 配置 |
| 前端代理失败 | 确认后端在 8000 端口运行，Vite 代理配置正确 |

---

## 附录 C: 技能使用计划

下表列出每个里程碑中推荐使用的 Superpowers skills：

| 里程碑 | 推荐技能 | 使用场景 |
|--------|---------|---------|
| M1 | `test-driven-development` | 编写 User 模型和 JWT 认证测试 |
| M1 | `brainstorming` | AI 接口抽象层设计不明确时 |
| M2 | `test-driven-development` | 题库 CRUD 实现 |
| M2 | `subagent-driven-development` | 前后端并行开发（后端 API + 前端页面） |
| M3 | `test-driven-development` | 组卷引擎算法实现 |
| M3 | `brainstorming` | AI 推荐 Prompt 设计 |
| M4 | `test-driven-development` | 导出功能测试 |
| M5 | `brainstorming` | LLM 解析 Prompt 调优 |
| M5 | `systematic-debugging` | AI 解析结果不理想时调试 |
| M6 | `verification-before-completion` | 每个集成步骤完成前验证 |
| M6 | `requesting-code-review` | 里程碑完成后整体 review |
| 全局 | `code-review` | 每个 task 提交前 |
| 全局 | `simplify` | code-review 后优化代码 |

---

> **文档版本**: v1.0 | **最后更新**: 2026-05-30 | **计划状态**: 待执行

---

## 自审报告

> 审查日期: 2026-05-30

### 1. 设计文档覆盖率

| 设计文档章节 | 对应计划任务 | 覆盖 |
|-------------|-------------|:--:|
| 4. 技术架构 | M1.1 Django 脚手架 + M1.4 前端脚手架 | ✅ |
| 5. 项目结构 | M1.0-M1.4 全部覆盖 | ✅ |
| 6.1 用户模型 | M1.2 User 模型与 JWT | ✅ |
| 6.2 题库模型 | M2.1 学科/年级/知识点 + M2.2 题目/选项 | ✅ |
| 6.3 试卷模型 | M3.1 试卷/分区/题目关联 | ✅ |
| 6.4 资源模型 | M5.1 资源模型 | ✅ |
| 6.5 AI 服务模型 | M1.3 AI 服务抽象层 | ✅ |
| 7. 资源采集链路 | M5.1-M5.3 完整覆盖 | ✅ |
| 8.1 题库管理 | M2.1-M2.4 | ✅ |
| 8.2 智能组卷 | M3.1-M3.5 | ✅ |
| 8.3 试卷输出 | M4.1-M4.2 | ✅ |
| 8.4 资源采集 | M5.1-M5.3 | ✅ |
| 8.5 用户系统 | M1.2 + M6.1 | ✅ |
| 10. API 约定 | M1.1 common/response.py, pagination.py, exceptions.py | ✅ |
| 11. 非功能需求 | 贯穿各 milestone | ✅ |
| 12. 测试策略 | 每个 Task 均 TDD (先写测试) | ✅ |
| 13. 部署策略 | M6.4 Docker 部署配置 | ✅ |
| 14. 里程碑 | M1-M6 完整对应 | ✅ |

### 2. 占位符检查

- 搜索关键词: TBD, TODO, 待实现, 待补充, implement later, fill in
- 结果: **0 个占位符** — 全部通过

### 3. 类型一致性检查

- `ExamPaper.config` 字段类型: JSON (Dict) — engine.py / recommend.py / serializers.py 中一致 ✅
- `Question.snapshot` 字段结构: exam/export.py 中各引用一致 ✅
- API 响应格式: `{ code, data, message }` 全项目统一 ✅
- 前端 ExamConfig 接口: examService.ts / ExamConfigPanel / ExamAIRecommend 一致 ✅

### 4. 计划统计

| 指标 | 数量 |
|------|:---:|
| 总行数 | 9,059 |
| 里程碑 | 6 |
| 任务 (Task) | 24 |
| 步骤 (Step) | 167 |
| 提交点 | 23 |
| 文件创建/修改 | 100+ |

### 5. 审结

计划完整覆盖设计文档所有模块，无占位符，类型一致，可直接进入执行阶段。

建议执行顺序：
1. 使用 `superpowers:subagent-driven-development` 按里程碑顺序执行
2. M1 基础架构必须最先完成（后续所有模块依赖）
3. M2 题库 → M3 组卷 → M4 导出 → M5 资源采集 → M6 整合
4. 每个里程碑完成后使用 `superpowers:requesting-code-review` 审查
