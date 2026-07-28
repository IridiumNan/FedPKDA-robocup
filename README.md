# FedPKDA 联邦入侵检测协作平台

> 基于 FedPKDA（AAAI 2026）的联邦学习入侵检测系统。
> 面向网络安全联盟场景，让多家机构在不共享原始数据的前提下联合训练检测模型，同时自动隔离恶意节点。

## 核心创新

传统 ISAC 共享机制依赖法律协议和控制制度来保护数据安全——信任基于"你是谁"。FedPKDA 通过算法本身剥夺了数据被滥用的可能性——信任基于**你技术上做不了坏事**。即使联盟协调员被攻陷，也拿不到任何可滥用的原始数据。

- **传输原型而非梯度**：PSNR 6.9 dB，数学上不可逆
- **马氏距离自动隔离**：异常节点无需人工举报，自动降权
- **Fisher 量化贡献**：搭便车在数学上无意义
- **φ(t) 自适应调度**：早期个性化，后期全局泛化，无需人工调参

## 实验验证

| 算法 | 模型 | 参数量 | 最佳精度 | vs FedAvg |
| ------ | ------ | -------- | --------- | ----------- |
| FedAvg | CNN | 5.6M | 27.8% | — |
| **FedPKDA** | **CNN** | **5.6M** | **45.1%** | **+17.3pp** |
| FedAvg | ResNet-18 | 11.2M | 24.9% | — |
| **FedPKDA** | **ResNet-18** | **11.2M** | **42.1%** | **+17.2pp** |

*Cifar100, 20 客户端, Dirichlet α=0.3, 40 轮*

## 项目结构

```
├── README.md
├── docs/                          # 文档中心（比赛评审查看入口）
│   ├── 01-index.md               # 文档索引
│   ├── 02-system-overview.md     # 系统概述
│   ├── 03-five-breakthroughs.md  # 五大技术突破
│   ├── 04-role-design.md         # 角色设计与页面架构
│   ├── 05-system-architecture.md # 系统架构与数据流
│   ├── 06-competition-plan.md    # 比赛交付计划
│   ├── 07-frontend-comparison.md # 前端方案对比
│   └── 08-appendix-research.md   # 调研附录
├── legacy_frontend/              # 原有前端（参考，不采用）
├── MVP_frontend/
│   ├── dark/                     # 暗色主题原型
│   └── light/
│       ├── coordinator/          # 联盟协调员端
│       │   ├── event_center      # 事件中心 — 自动告警 + 广播通知
│       │   ├── node_management   # 节点管理 — 24 机构管理
│       │   ├── training          # 训练控制 — 全局参数 + 贡献度
│       │   └── architecture      # 系统架构 — 工作流程 + 价值说明
│       └── operator/             # 机构运营员端
│           ├── node_status       # 节点状态 — 详情 + 本地操作
│           ├── training          # 训练控制 — 本地对比 + 参数
│           └── architecture      # 系统架构（机构视角）
└── FedPKDA →                     # 核心算法（软链接）
```

## 角色与权力边界

| | 能看到什么 | 能做什么 | 不能做什么 |
| --- | --- | --- | --- |
| **联盟协调员** | 聚合统计 + 脱敏事件 | 建议隔离、广播通知、调全局参数 | 不能看原始数据、原型、攻击事件 |
| **机构运营员** | 自己的完整信息 | 隔离/恢复节点、调本地参数 | 不能看其他机构任何信息 |

---

## 快速导航

- **架构白皮书入口** → [`docs/01-index.md`](./docs/01-index.md)
- **系统概述** → [`docs/02-system-overview.md`](./docs/02-system-overview.md)
- **五大技术突破** → [`docs/03-five-breakthroughs.md`](./docs/03-five-breakthroughs.md)
- **角色设计与页面架构** → [`docs/04-role-design.md`](./docs/04-role-design.md)
- **系统架构与数据流** → [`docs/05-system-architecture.md`](./docs/05-system-architecture.md)
- **前端方案对比** → [`docs/07-frontend-comparison.md`](./docs/07-frontend-comparison.md)
- **交互原型** → `MVP_frontend/light/`
