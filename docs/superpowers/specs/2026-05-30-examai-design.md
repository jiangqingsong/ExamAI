# ExamAI — 智能试卷生成平台 设计文档

> **版本**: v2.0 | **日期**: 2026-05-30 | **状态**: 设计确认，待实施计划

---

## 1. 产品愿景

面向初中教师的智能考试试卷生成平台。从网上搜集各学科试卷资源构建题库，基于教师个性化需求自动生成定制化考试试卷，以 Web 网站形式管理和运营。

**路线图：** 校内教师个人工具（MVP验证）→ 商业化SaaS付费平台

---

## 2. 目标用户

| 阶段 | 用户 | 核心场景 |
|------|------|---------|
| MVP | 3-5位种子教师（数学/物理/历史） | "我是初二物理老师，期末了，帮我出一份覆盖力学+电学的试卷" |
| 商业化 | 全国初中教师 | 付费订阅，团队协作，学校版 |

---

## 3. MVP 范围定义

### 3.1 MVP 学科

- 数学（七年级/八年级/九年级）
- 物理（八年级/九年级）
- 历史（七年级/八年级/九年级）

### 3.2 MVP 时间目标

**3-4个月**，交付可用的核心流程。

### 3.3 MVP 核心页面（6个）

| # | 页面 | 功能 | 优先级 |
|---|------|------|--------|
| 1 | 登录/注册页 | 邮箱注册、登录、找回密码 | P0 |
| 2 | 工作台首页 | 我的试卷列表、快速创建入口、使用统计 | P0 |
| 3 | 智能组卷页 | A面表单配置 + B面AI推荐 → 生成试卷 → 预览 → 换题 | P0 |
| 4 | 题库管理页 | 搜索/筛选、知识点树、手动录入、批量导入 | P0 |
| 5 | 试卷详情/导出页 | 在线预览、PDF下载、Word下载、答题卡+解析 | P0 |
| 6 | 资源采集页 | 手动上传试卷、AI解析进度、审核入库 | P1 |

### 3.4 核心用户流程

```
登录 → 工作台 → 创建试卷 → 配置组卷(AI辅助) → 预览微调 → 导出PDF/Word
```

---

## 4. 技术架构

### 4.1 技术选型

| 层级 | 技术 | 理由 |
|------|------|------|
| 前端 | Vite + React + Ant Design + React Router | 轻量现代，组件库丰富，适合管理后台 |
| 状态管理 | Zustand | 轻量，API 简洁，适合中型 SPA |
| 后端 | Django REST Framework | 开发效率高，Python AI生态优势 |
| 异步任务 | Celery + Redis | AI解析/PDF生成等耗时任务 |
| 数据库 | PostgreSQL | 关系型数据，JSON字段支持灵活配置 |
| 缓存/队列 | Redis | 缓存 + Celery消息代理 |
| 文件存储 | 本地存储（MVP） → MinIO/S3（后期） | MVP简化，后期迁移 |
| 认证 | JWT (simplejwt) | 无状态认证，前后端分离标准方案 |

### 4.2 架构图

```
React SPA (Vite + Ant Design)  →  Django REST API  →  Celery Worker  →  PostgreSQL
        ↕                              ↕                    ↕
   浏览器缓存                      Redis (缓存+队列)      本地文件存储
```

### 4.3 LLM 策略

- **默认模型：** DeepSeek (deepseek-chat)
- **接口抽象：** `apps/ai/providers/` 下每个模型一个 adapter，统一接口
- **配置入口：** 管理后台可动态添加/切换模型提供商
- **后续扩展：** 通义千问、智谱 GLM 等国内模型

### 4.4 AI 能力

| AI能力 | 用途 | MVP |
|--------|------|-----|
| LLM 题目结构化解析 | 非结构化试卷文本 → 结构化题目（题干/答案/解析/题型/难度/知识点） | ✅ |
| LLM 智能组卷推荐 | 输入"初二物理期中" → 自动推荐知识点覆盖+题型+难度配置 | ✅ |
| OCR 图片识别 | 识别试卷扫描件/图片中的文字和公式 | ⏳ MVP后期 |

---

## 5. 项目结构

```
ExamAI/
├── backend/                        # Django 项目根
│   ├── config/                     # settings.py, urls.py, wsgi.py, celery.py
│   ├── apps/
│   │   ├── users/                  # 用户认证 (JWT)、注册登录、配额
│   │   ├── questions/              # 题目 CRUD、知识点树、批量导入
│   │   ├── exams/                  # 试卷、分区、组卷引擎
│   │   ├── resources/              # 资源上传、AI 解析任务、审核入库
│   │   └── ai/                     # LLM 接口抽象层（多模型切换）
│   ├── common/                     # 公共工具：分页、响应格式、权限
│   └── requirements/
│       ├── base.txt                # Django + DRF + Celery + psycopg2
│       └── dev.txt                 # django-extensions 等
├── frontend/                       # Vite + React SPA
│   ├── src/
│   │   ├── pages/                  # 6 个页面组件
│   │   ├── components/             # 共享组件
│   │   ├── services/               # API 请求封装 (axios)
│   │   ├── stores/                 # 状态管理 (zustand)
│   │   └── router/                 # React Router 路由配置
│   └── vite.config.ts
├── docker-compose.yml              # 本地开发：Django + React + PG + Redis
└── docs/
    └── superpowers/
        ├── specs/                  # 设计文档
        └── plans/                  # 实施计划
```

---

## 6. 数据模型

### 6.1 用户与认证 (`apps/users`)

```
User (继承 Django AbstractUser)
├── email (唯一标识，登录用)
├── nickname
├── role: teacher / admin
├── quota_total: 每月可用配额
├── quota_used: 本月已用
└── created_at
```

### 6.2 题库模块 (`apps/questions`)

```
Subject (学科)
├── name: "数学" / "物理" / "历史"
├── code: "math" / "physics" / "history"
└── is_active

Grade (年级)
├── name: "七年级" / "八年级" / "九年级"
├── level: 7 / 8 / 9
└── subjects: M2M → Subject

KnowledgePoint (知识点自引用树)
├── name
├── parent: FK → self (null=根节点)
├── subject: FK → Subject
├── level: 层级深度
├── sort_order: 排序
└── path: 冗余路径字段，加速查询 (如 "物理/力学/牛顿定律/牛二律")

Question (题目)
├── subject, grade: FK
├── type: choice / fill_blank / true_false / short_answer / calculation / essay
├── difficulty: 1-5
├── cognitive_level: 识记/理解/应用/分析/综合
├── stem (题干，Markdown + LaTeX)
├── answer (答案)
├── explanation (解析，Markdown)
├── default_score
├── estimated_time (秒)
├── images: JSON (配图路径列表)
├── source_type: 真题/模拟/原创
├── source_detail: "2024北京中考"
├── exam_type: 单元/月考/期中/期末/中考
├── status: draft / reviewed / disabled
├── usage_count: 被使用次数
├── knowledge_points: M2M → KnowledgePoint
└── created_at, updated_at

QuestionOption (选择题选项)
├── question: FK
├── label: "A" / "B" / "C" / "D"
├── content
└── sort_order
```

### 6.3 试卷模块 (`apps/exams`)

```
ExamPaper (试卷)
├── user: FK → User
├── title: "初二物理期中考试卷"
├── subject, grade: FK
├── exam_type
├── total_score, duration (分钟)
├── config: JSON (组卷参数快照，可追溯复现)
├── status: draft / generated / finalized
└── created_at, updated_at

ExamPaperSection (试卷分区)
├── exam_paper: FK
├── title: "一、选择题"
├── description: "每题3分，共30分"
├── sort_order
└── total_score

ExamPaperItem (题目关联)
├── section: FK → ExamPaperSection
├── question: FK → Question
├── sort_order
├── score (本题分值)
└── question_snapshot: JSON (生成时的题目快照，防止原题被修改后试卷不一致)
```

### 6.4 资源模块 (`apps/resources`)

```
Resource (原始资源文件)
├── user: FK → User (上传者)
├── file: FileField
├── file_name, file_type, file_size
├── subject, grade: FK (可空，解析后填充)
├── source_type: 购买/爬虫/手动上传
├── source_detail: 出处描述
├── status: pending / extracting / parsing / reviewing / completed / failed
├── extracted_text: TextField (文本提取结果)
├── parsed_data: JSON (LLM 解析结果，含 confidence/图片引用/源页码)
├── parsed_questions: M2M → Question (审核确认后关联)
├── error_message: TextField
└── created_at, updated_at
```

### 6.5 AI 服务 (`apps/ai`)

```
AIProvider (LLM 提供商配置)
├── name: "deepseek" / "qwen" / "glm"
├── display_name: "DeepSeek"
├── api_base_url
├── api_key (加密存储)
├── model_name: "deepseek-chat"
├── is_active, is_default
├── config: JSON (temperature, max_tokens 等)
└── created_at

AITaskLog (调用日志/成本统计)
├── provider: FK → AIProvider
├── task_type: parse_question / recommend_exam
├── input_tokens, output_tokens
├── cost
├── duration_ms
├── success: bool
├── error_message
└── created_at
```

---

## 7. 资源采集完整链路

从原始试卷文件到结构化题库的全流程：

```
原始试卷 (Word/PDF/图片)
    │
    ▼
① 上传 & 存储 → Resource 记录（单份或多份并行上传）
    │
    ▼
② 文本提取（按文件类型策略）
   ├── Word → python-docx（保留段落/表格结构）
   ├── PDF  → PyMuPDF（按页提取，保留排版）
   └── 图片 → OCR（MVP后期）
    │
    ▼
③ LLM 结构化解析（Celery 异步，DeepSeek）
   输入：原始试卷文本
   输出：结构化题目 JSON（题干/答案/解析/题型/难度/知识点/图片引用/置信度）
   优化：大题拆 chunk 并行处理
    │
    ▼
④ 人工审核校对
   审核界面：原始文件预览 + 解析结果并排对比
   支持：修正/确认/拒绝/批量操作
   重点审核：含图题目、低置信度题目
    │
    ▼
⑤ 入库 → Question 表，关联知识点树
```

### 图片处理分层策略

| 题目类型 | 占比 | MVP 自动对齐 | 处理方式 |
|----------|:----:|:----------:|------|
| 纯文字题 | ~70% | ~95% | 文本提取 → LLM解析 → 直接入库 |
| 含公式题 | ~20% | ~85% | 提取时转 LaTeX，LLM 还原，前端 KaTeX/MathJax 渲染 |
| 含图几何/函数题 | ~7% | ~60% | 提取图片文件，审核时人工确认关联 |
| 复杂图文混排 | ~3% | ~40% | 审核时标记，可手动录入或后期优化 |

---

## 8. 功能模块

### 8.1 题库管理中心

- 题目列表（分页、搜索、多条件筛选）
- 题目详情与编辑
- 知识点树管理（增删改、拖拽排序）
- 批量导入（Excel模板）
- 题目审核流程（草稿→已审核→已禁用）

### 8.2 智能组卷引擎

- **A面 - 表单精细配置**：学科/年级/考试类型/知识点范围/题型分布/难度比例/总分/时长
- **B面 - AI智能推荐**：输入考试意图，AI自动推荐配置参数
- 试卷实时预览
- 手动调换题目
- 难度分布可视化

### 8.3 试卷输出中心

- 在线预览（标准试卷排版，支持 LaTeX + 图片渲染）
- PDF 下载（A4排版，异步生成）
- Word 下载（可编辑，异步生成）
- 答题卡生成
- 参考答案 + 详细解析

### 8.4 资源采集系统

- 手动上传试卷文件（PDF/Word/图片，支持批量+ZIP）
- 单份/批量并行处理
- AI 自动解析（Celery 异步，分块并行）
- 解析结果审核（原始文件+解析结果并排对比）
- 状态机管理：待解析 → 提取中 → 解析中 → 待审核 → 已完成/失败

### 8.5 用户系统

- 注册/登录/找回密码
- 个人试卷夹
- 使用配额管理

---

## 9. 试卷资源来源策略

| 来源 | 方式 | 阶段 |
|------|------|------|
| 淘宝/闲鱼购买 | 获取高质量PDF/Word试卷作为种子数据 | MVP 前期 (M1/M2) |
| 网络爬虫 | 从公开教育资源站抓取试卷 | MVP+ |
| 手动上传 | 老师自行上传试卷文件 | MVP |
| 合作/API | 对接学科网、菁优网等题库 | 商业化 |

**种子数据：** M1/M2 期间采购并录入首批 1000 道核心题。

---

## 10. API 设计约定

- RESTful 风格
- 统一响应格式：`{ code: 0, data: {...}, message: "ok" }`
- 分页格式：`{ page, page_size, total, results }`
- 认证：JWT Header `Authorization: Bearer <token>`
- 异步任务：返回 `{ task_id }`，前端轮询获取进度
- 文件上传：限制 50MB，类型白名单 PDF/Word/图片/ZIP
- 错误处理：DRF 全局异常处理，LLM 调用失败自动重试 2 次

---

## 11. 非功能需求

- **安全性**：JWT认证、API鉴权、AIProvider api_key 加密存储
- **性能**：试卷生成 3秒内完成，题库搜索支持 10万+ 题目量
- **可用性**：响应式设计，PC端为主 + 平板端兼容
- **可扩展性**：前后端分离，LLM 接口抽象支持多模型切换，存储可迁移至 OSS

---

## 12. 测试策略

MVP 阶段核心 API 测试覆盖：

| 测试对象 | 验证内容 |
|----------|------|
| 组卷生成 API | 题型分布、难度分布、知识点覆盖准确性 |
| AI 解析 API | 结构化输出格式正确性、异常输入处理 |
| 试卷导出 API | PDF/Word 生成完整性 |

前端暂不写自动化测试，后期补充。

---

## 13. 部署策略

- **MVP 阶段：** 本地开发跑通（Docker Compose 编排 Django + React + PostgreSQL + Redis）
- **上线阶段：** 部署至云服务器（阿里云 ECS 等），具体方案届时确定

---

## 14. 里程碑规划

| 里程碑 | 内容 | 前后端 | 时间 |
|--------|------|:----:|------|
| M1 — 基础架构 | Django + React 脚手架、DB模型、JWT认证、LLM接口抽象 | 基础搭建 | 第1-2周 |
| M2 — 题库模块 | 知识点树 + 题目CRUD + Excel批量导入 + 题库管理页面 + 种子数据入库 | 前后端同步 | 第3-5周 |
| M3 — 组卷引擎 | A面表单 + B面AI推荐 + 试卷生成 + 智能组卷页面 | 前后端同步 | 第6-9周 |
| M4 — 试卷导出 | 预览 + PDF/Word异步生成 + 答题卡 + 导出页面 | 前后端同步 | 第10-12周 |
| M5 — 资源采集 | 手动上传 + AI解析(异步) + 审核入库 + 采集页面 | 前后端同步 | 第12-14周 |
| M6 — 上线整合 | 工作台首页 + 登录注册页 + 联调测试 + 部署 | 整体整合 | 第14-16周 |

---

## 15. 风险与应对

| 风险 | 影响 | 应对 |
|------|------|------|
| 题库冷启动（题目太少无法组卷） | 高 | M1/M2 采购种子数据 + 手动录入首批 1000 道核心题 |
| AI 解析准确率不足 | 中 | 人工审核环节兜底，逐步优化 Prompt |
| 图片题自动处理率低 | 中 | 分层策略，MVP 先处理纯文字+公式题（占 90%），图题后期集中攻克 |
| 试卷版权问题 | 中 | 仅收录公开来源，注明出处 |
| 3-4月时间紧张 | 中 | 资源采集降为 P1，保证 P0 核心流程完整 |
