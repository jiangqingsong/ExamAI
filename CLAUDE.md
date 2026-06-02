# ExamAI — 智能试卷生成平台

## 项目概述

面向初中教师的智能考试试卷生成平台。从网上搜集各学科试卷资源构建题库，基于教师个性化需求自动生成定制化考试试卷，以 Web 网站形式管理和运营。

**路线：** 校内教师个人工具（MVP验证）→ 商业化SaaS付费平台

## 技术栈

| 层级 | 技术 |
|------|------|
| 前端 | React + Ant Design (SPA) |
| 后端 | Django REST Framework |
| 异步 | Celery + Redis |
| 数据库 | PostgreSQL |
| 缓存 | Redis |
| 文件 | 本地存储 → MinIO/S3 |
| 认证 | JWT (djangorestframework-simplejwt) |

## 当前阶段

**⏸️ ExamAI 已暂停** — 优先推进 DATA for AI 职业转型计划。ExamAI 将作为方案三（AI 数据工程改造版）在项目一完成后启动。

## 关键文档

- 设计文档: `docs/superpowers/specs/2026-05-30-examai-design.md`
- **职业转型计划**: `docs/superpowers/specs/2026-06-02-career-ai-data-engineering-plan.md`
- 实施计划: `docs/superpowers/plans/` (待编写)

## MVP 范围

- **学科：** 数学（7/8/9年级）、物理（8/9年级）、历史（7/8/9年级）
- **时间：** 3-4个月
- **核心页面（6个）：**
  1. 登录/注册页 — 邮箱注册、登录、找回密码
  2. 工作台首页 — 我的试卷列表、快速创建、统计概览
  3. 智能组卷页 — A面表单配置 + B面AI推荐 → 生成 → 预览 → 换题
  4. 题库管理页 — 搜索筛选、知识点树、手动录入、批量导入
  5. 试卷导出页 — 在线预览、PDF/Word下载、答题卡+解析
  6. 资源采集页 — 上传试卷、AI解析、审核入库

## 产品关键决策

- 组卷方式：A面表单精细配置 + B面AI智能推荐，组合使用
- 题目标签：D级全量（学科/年级/题型/难度/认知层级/知识点树/来源/统计）
- 试卷输出：在线预览 + PDF下载 + Word下载
- 资源来源：购买种子数据 + 网络爬虫 + 手动上传 混合策略
- 数据模型：知识点自引用树（无限层级）、题目-知识点M2M、试卷→分区→题目三级结构
- AI能力：LLM题目结构化解析、LLM智能组卷推荐

## 待办

- [ ] 编写实施计划（writing-plans skill）
- [ ] 按里程碑M1-M7逐步实现

## Git

- 仓库: `git@github.com:jiangqingsong/ExamAI.git`
- 分支: `main`
