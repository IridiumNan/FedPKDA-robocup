# 通信协议与算法映射

> 本文档说明两层内容：
> ① **训练通信协议**——联邦学习数据面（上行加噪原型 / 下行全局原型+模型）的传输链路；
> ② **接口与算法函数映射**——前端 REST 接口与 FedPKDA 算法内部函数的对应关系。
>
> 算法核心流程详见 [`../05-system-architecture.md`](../05-system-architecture.md)。

---

## 一、训练通信协议

### 1.1 控制面与数据面的关系

系统通信分为两层，职责分离：

| 层面 | 载体 | 用途 | 对应文档 |
| --- | --- | --- | --- |
| **控制面** | HTTP REST（本文档中心全部接口） | 前端展示、状态查询、参数调整、事件处理 | `00-conventions.md` 及各页面文档 |
| **数据面** | 联邦学习训练协议（基于 Flower 框架） | 每轮训练中原型上传 / 全局原型与模型下发 | 本文档 |

数据面不走 HTTP：原型向量维度高（如 512 维）、每轮高频传输，需要低延迟连接；REST 只读取数据面运行产生的结果。

### 1.2 每轮训练的数据链路（8 步循环中的通信环节）

```
① 本地训练 ──────────────── 原始数据不离开机构服务器（无上行）
② 原型提取 ──────────────── 计算每类攻击的特征中心（本地）
③ 加噪上传 ──【上行】───── 裁剪到 [-1,1] + Laplace 噪声 → 上传带噪原型
④ 服务端聚合 ───────────── K-Means 聚类 → 马氏距离计算 → 距离加权平均 → 全局原型
                              ↳ 异常节点自动降权；触发事件写入事件中心（REST 可读）
⑤ 全局原型广播 ──【下行】─ 全局原型发回各机构
⑥ 双对齐训练 ───────────── 本地对齐 L_LA + 全局对齐 L_GA，φ(t) 调节权重（本地）
⑦ Fisher 加权聚合 ──────── 各节点贡献度量化 → 加权聚合全局模型
                              ↳ 贡献度排行可通过 REST 读取
⑧ 进入下一轮
```

### 1.3 上行链路（Client → Server）：加噪原型

```
本地数据 → 特征提取器 → 特征向量
         → 裁剪到 [-1,1]（限制原型范围）
         → 添加 Laplace 噪声（尺度 ε 由本地参数控制）
         → 上传带噪原型
```

- **载荷**：带噪原型向量（类别中心），含节点标识、轮次、类别、噪声尺度等元信息
- **隐私保证**：PSNR 6.9 dB，数学上不可逆；原始数据、攻击详情、业务数据均不离开机构
- **噪声参数来源**：运营员通过 `POST /operator/params` 调整噪声强度，作用于 `get_noisy_local_prototypes()` 的 Laplace scale

### 1.4 下行链路（Server → Client）：全局原型 + 模型

```
收集所有在线节点的带噪原型
→ K-Means 聚类求中心
→ 马氏距离（考虑协方差分布）加权生成全局原型
→ 广播全局原型给客户端
→ Fisher 信息矩阵计算贡献度 → 加权聚合全局模型参数 → 下发
```

- **载荷**：全局原型 + 聚合后的全局模型参数
- **隔离机制**：被隔离的节点不再上传原型，自然退出第④步聚合

### 1.5 事件触发与算法环节对应

训练循环中产生的告警事件进入事件中心（REST 可读），触发点与算法环节严格对应：

| 事件 | 算法来源 | 触发条件 | 对应 REST 数据 |
| --- | --- | --- | --- |
| 节点行为偏离 | 马氏距离（步骤④） | 距离 > 3σ | `/coordinator/event/all_event`（type=risk） |
| 节点离线 | 心跳超时 | N 轮未响应 | `/coordinator/event/all_event`（type=risk） |
| 第 N 轮训练完成 | 每轮聚合（步骤④） | 每轮完成 | `/coordinator/event/all_event`（type=normal） |
| 检测率持续下滑 | 全局评估曲线 | 连续 N 轮下降 | `/coordinator/event/all_event`（type=risk） |
| 新攻击模式 | 本地推理（步骤②） | 未知类别置信度 > 87% | `/coordinator/event/all_event`（匿名摘要） |

### 1.6 参数生效时机

- **全局参数**（`POST /coordinator/training/params`）：不中断当前训练，下一轮开始前生效
- **本地参数**（`POST /operator/params`）：仅影响本节点，下一轮生效
- 两类调整均不触发数据面中断，由训练协议在轮次边界应用新配置

---

## 二、接口与算法函数映射

### 2.1 算法输出 → REST 接口字段

前端展示的每个核心指标都可追溯到算法内部输出：

| 算法输出 | REST 接口 | 对应字段 |
| --- | --- | --- |
| `rs_test_acc`（每轮全局测试精度） | `GET /overall/best_acc` | `acc` |
| `rs_test_acc` 轮次序列 | `GET /coordinator/training/status` | `trend[].acc` |
| vs FedAvg 基线对比（实验数据） | `GET /overall/best_acc` | `compare` |
| 马氏距离异常分数（`compute_global_prototypes()`） | `GET /coordinator/node/list` | `nodes[].health`（节点行为评分） |
| 马氏距离 > 阈值（异常节点） | `GET /coordinator/event/all_event` | type=risk 事件 |
| Fisher 信息聚合贡献权重 | `GET /coordinator/training/contribution` | `ranking[].contribution` |
| Fisher 权重（本节点） | `GET /operator/training/status` | `contribution` / `rank` |
| φ(t) 调度状态（`Func`） | `GET /coordinator/training/status` | `alpha.state` / `alpha.local_par` / `alpha.global_par` |
| 全局模型本地推理（攻击检测） | `GET /coordinator/event/all_event` | "新攻击模式"事件（匿名） |
| Laplace 噪声尺度（`get_noisy_local_prototypes()`） | `GET /operator/node/status` | `privacy.eps_used` / `noise_active` |

### 2.2 用户操作（REST 写接口）→ 算法变更

| REST 写接口 | 算法函数 | 变更内容 |
| --- | --- | --- |
| `POST /coordinator/training/params`（global_par） | `set_parameters()` | 修改全局对齐权重 |
| `POST /coordinator/training/params`（participation_ratio） | `send_selected_models()` 选择逻辑 | 每轮参与节点比例 |
| `POST /coordinator/training/params`（anomaly_threshold） | `compute_global_prototypes()` | 马氏距离隔离阈值 σ |
| `POST /operator/params`（noise） | `get_noisy_local_prototypes()` | Laplace 噪声尺度 |
| `POST /operator/params`（epoch） | 本地训练循环 | 本地训练轮次 |
| `POST /operator/params`（mode） | `send_selected_models()` | 客户端身份（训练/评估/空闲） |
| `POST /operator/node/isolate` | `compute_global_prototypes()` | 该节点不再上传原型，自动降权 |
| `POST /operator/node/recover` | `send_selected_models()` | 重新加入训练队列 |
| `POST /coordinator/event/broadcast` | — | 不改变算法，仅生成通知 |
| `POST /coordinator/event/{id}/handle` | — | 不改变算法，仅更新事件状态 |
| `POST /operator/training/watch` | — | 不改变算法，记录观察策略 |

> 权限边界保持一致：协调员只能触及全局参数与事件（2.2 表上半部），运营员只能调整本节点参数（2.2 表下半部）。
