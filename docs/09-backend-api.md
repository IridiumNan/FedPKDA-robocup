# 后端 API 设计（入口）

> 本文档为后端接口设计的**入口索引**，完整接口定义按界面拆分在 `docs/api/` 下。
> 接口从 `MVP_frontend/light/` 前端页面真实数据需求反推而来。

---

## 文档目录

| # | 界面 | 文档 | 覆盖接口 |
| --- | --- | --- | --- |
| 00 | 通用约定 | [`docs/api/00-conventions.md`](./api/00-conventions.md) | 响应结构、鉴权、错误码、刷新方式 |
| 01 | 平台概览 | [`docs/api/01-overall.md`](./api/01-overall.md) | `/overall/*`：最佳检测率、机构数、训练轮次 |
| 02 | 协调员 · 事件中心 | [`docs/api/coordinator/event-center.md`](./api/coordinator/event-center.md) | `/coordinator/event/*`：事件列表、统计、处理、广播 |
| 03 | 协调员 · 节点管理 | [`docs/api/coordinator/node-management.md`](./api/coordinator/node-management.md) | `/coordinator/node/*`：节点列表、详情、通知 |
| 04 | 协调员 · 训练控制 | [`docs/api/coordinator/training.md`](./api/coordinator/training.md) | `/coordinator/training/*`：状态、参数、贡献排行 |
| 05 | 运营员 · 节点状态 | [`docs/api/operator/node-status.md`](./api/operator/node-status.md) | `/operator/node/*`：状态、参数、隔离/恢复 |
| 06 | 运营员 · 训练控制 | [`docs/api/operator/training.md`](./api/operator/training.md) | `/operator/training/*`：状态、参数、标记观察 |
| 07 | 系统架构 | — | 静态说明页面，无动态数据接口 |

---

## 快速导航

- 想知道接口通用规则（鉴权 / 错误码 / 时间格式）？ → [`00-conventions.md`](./api/00-conventions.md)
- 想知道某个页面调哪些接口？ → 对照上表"界面"列进入对应文档
- 想知道角色权限边界？ → [`04-role-design.md`](./04-role-design.md)
- 想知道算法函数与前端展示的映射？ → [`05-system-architecture.md`](./05-system-architecture.md)

---

## 接口一览

### 平台概览（`/overall`）

| 方法 | 路径 | 用途 |
| --- | --- | --- |
| GET | `/overall/best_acc` | 最佳检测率及 vs FedAvg 提升 |
| GET | `/overall/node_num` | 参与机构数及新增 |
| GET | `/overall/round` | 当前训练轮次 |

### 协调员端（`/coordinator`）

| 方法 | 路径 | 用途 |
| --- | --- | --- |
| GET | `/coordinator/event/all_event` | 事件列表（过滤 + 分页） |
| GET | `/coordinator/event/summary` | 事件统计概览（分布 + 趋势） |
| POST | `/coordinator/event/{id}/handle` | 处理事件（关注 / 完成） |
| POST | `/coordinator/event/broadcast` | 广播通知 |
| GET | `/coordinator/node/list` | 节点列表 + 健康概览 |
| GET | `/coordinator/node/{id}` | 节点详情 |
| POST | `/coordinator/node/{id}/notify` | 向节点发送通知 |
| GET | `/coordinator/training/status` | 训练状态 + 趋势 |
| GET | `/coordinator/training/params` | 读取全局参数 |
| POST | `/coordinator/training/params` | 更新全局参数 |
| GET | `/coordinator/training/contribution` | Fisher 贡献度排行 |

### 运营员端（`/operator`）

| 方法 | 路径 | 用途 |
| --- | --- | --- |
| GET | `/operator/node/status` | 本节点状态总览 |
| GET | `/operator/params` | 读取本地参数（节点状态 / 训练控制两页共用） |
| POST | `/operator/params` | 保存本地参数 |
| POST | `/operator/node/isolate` | 隔离本节点 |
| POST | `/operator/node/recover` | 恢复参与 |
| GET | `/operator/training/status` | 本地训练状态 + 本地/全局对比 |
| POST | `/operator/training/watch` | 标记观察 |

> 完整字段说明见各界面文档。
