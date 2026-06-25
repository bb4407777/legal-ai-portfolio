# Lawlink 系统架构

> 基于 lawflow-boop/LawLink 的深度定制与重构。

---

## 技术栈

| 层 | 技术 |
|----|------|
| 前端框架 | Next.js 16 (App Router) |
| UI 组件 | shadcn/ui + Tailwind CSS |
| 后端 | Next.js Server Actions + API Routes |
| 数据库 | PostgreSQL 16 |
| ORM | Prisma 5 |
| 鉴权 | NextAuth.js |
| 部署 | Docker Compose |

## 数据模型设计

### 核心实体关系

```
Client ──→ Matter ←── OpposingParty
              │
              ├── MatterProcedure (程序节点)
              ├── Task (待办任务)
              ├── Document (文书)
              ├── Note (办案记录)
              ├── Hearing (开庭)
              ├── Preservation (保全)
              └── TimelineEvent (时间线)
```

### 案件（Matter）核心 Schema

```
Matter {
  internalCode      // 所内案号（可配置模板）
  displayCode       // 显示案号
  title             // 案件名称
  causeOfAction     // 案由
  status            // 案件状态（19 个阶段）
  caseType          // 案件类型（诉讼/非诉/仲裁）
  court             // 受理法院
  amount            // 标的额
  filingDate        // 立案日期
  closeDate         // 结案日期
  // ...（共 30+ 字段）
}
```

### 当事人模型重构

原项目使用简单的当事人字段，我们重构为：

```
Party → 三种角色
  ├── 相对方（OpposingParty）：对方当事人
  ├── 第三方（ThirdParty）：无独立请求权第三人
  └── 关联方（RelatedParty）：关联案件当事人

Client → 委托人体系
  ├── 自然人客户（基本信息 + WeChat + 证件）
  └── 企业客户（统一社会信用代码 + 法定代表人 + 委托代理人）
```

### 案件状态机

案件通过 19 个状态字段进行自动推进：

```
收案 ─→ 立案材料 ─→ 网上立案 ─→ 审核中 ─→ 已立案
                                                    │
                          ┌──────────────────────────┘
                          ▼
                    审理中 ─→ 庭后 ─→ 待判决
                          │
                          ▼
                     判决/调解 ─→ 上诉/执行 ─→ 归档
```

## 自动化工作流架构

参见 [WORKFLOWS.md](./WORKFLOWS.md)。

## OCR 管道

```
微信聊天截图（MP4）
       │
       ▼
ffmpeg 截帧
       │
       ▼
ocr-vision 文本识别
       │
       ▼
结构化提取（证据清单）
       │
       ▼
PDF / 案件关联
```

## 部署架构

```
┌──────────────┐     ┌──────────────┐     ┌──────────────┐
│  Next.js App  │────▶│  PostgreSQL   │────▶│   存储卷      │
│  (Server)     │     │  (数据)       │     │  (文件/文书)   │
└──────────────┘     └──────────────┘     └──────────────┘
       │
       ▼
┌──────────────┐
│  Docker       │
│  Compose      │
└──────────────┘
```
