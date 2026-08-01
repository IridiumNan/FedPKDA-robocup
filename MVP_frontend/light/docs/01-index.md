# FedPKDA 联邦入侵检测协作平台 — 文档索引

> 面向 RoboCup 网络安全赛道的系统设计文档中心。
> 核心算法：FedPKDA（AAAI 2026）

---

## 文档总览

| # | 文档 | 内容 | 来源 |
| --- | --- | --- | --- |
| 01 | **本文档** | 文档索引 & 快速导航 | — |
| 02 | **系统概述** | 现实痛点、目标场景、系统定位、核心创新 | `plan_A.md` + `frontend_plan.md` |
| 03 | **五大技术突破** | FedPKDA 五大核心技术突破的详细对比说明 | `plan_A.md` |
| 04 | **角色设计与页面架构** | 三个角色定义、权力边界、各页面信息架构 | `frontend_plan.md` |
| 05 | **系统架构与数据流** | 训练流程、事件触发、角色可见边界、算法-前端映射 | `frontend_plan.md` |
| 06 | **比赛交付计划** | 第一轮比赛交付物清单、时间线、技术选型 | `practical_plan.md`（更新版） |
| 07 | **前端方案对比** | Legacy 前端 vs MVP 设计的逐维对比分析 | `frontend_compare.md` |
| 08 | **附录：调研与参考** | ISAC 治理模式调研、比赛规则分析 | `local/links.md` + `local/docs/prepare.md` |

---

## 快速开始

- **想了解这个系统做什么？** → [02-系统概述](./02-system-overview.md)
- **想知道技术核心亮点？** → [03-五大技术突破](./03-five-breakthroughs.md)
- **想查看 UI 设计逻辑？** → [04-角色设计与页面架构](./04-role-design.md)
- **想看系统怎么运转？** → [05-系统架构与数据流](./05-system-architecture.md)
- **想看比赛排期？** → [06-比赛交付计划](./06-competition-plan.md)
- **想对比新旧前端？** → [07-前端方案对比](./07-frontend-comparison.md)

---

## 文档层级说明

```
docs/                         ← 对外文档中心（比赛评审可查看）
├── 01-index.md               ← 本文档
├── 02-system-overview.md     ← 系统概述
├── 03-five-breakthroughs.md  ← 五大技术突破
├── 04-role-design.md         ← 角色设计与页面架构
├── 05-system-architecture.md ← 系统架构与数据流
├── 06-competition-plan.md    ← 比赛交付计划
├── 07-frontend-comparison.md ← 前端方案对比
└── 08-appendix-research.md   ← 调研附录

MVP_frontend/                 ← 交互原型（HTML 页面）
└── light/                    ← 亮色主题（当前版本）
    ├── coordinator/          ← 协调员端
    └── operator/             ← 运营员端

legacy_frontend/              ← 原有前端（参考，不采用）
```

---

## 相关链接

- **交互原型**：`MVP_frontend/light/` — 可直接浏览器打开 HTML 查看
- **产品设计原稿**：`frontend_plan.md` — 本文档中心的核心信息来源
- **前端分析**：`frontend_analysis.md` — UI/UX 设计原则
