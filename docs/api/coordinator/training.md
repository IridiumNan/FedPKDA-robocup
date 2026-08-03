# 训练控制 API

> 对应页面：`MVP_frontend/light/coordinator/training.html`（训练控制）
> 接口前缀：`/coordinator/training`
> 权限：仅协调员

---

## 1. GET /coordinator/training/status

**用途**：获取当前训练运行状态，渲染顶部统计卡（轮次 / 检测率 / α(t) 调度状态 / 效率）与全局检测率趋势图。

**查询参数**：无

**响应字段**

| 字段 | 类型 | 说明 |
| --- | --- | --- |
| time | str | 服务端时间戳 |
| round | int | 当前已完成的训练轮次 |
| total_round | int | 训练计划总轮次 |
| acc | float | 全局检测率（%），如 `45.1` |
| acc_compare | float | 相比上一轮的检测率变化（百分点），正数提升、负数下降 |
| alpha | object | φ(t) 动态调度状态，结构见下 |
| efficiency | float | 训练效率（%），反映闭环稳定程度 |
| trend | array | 全局检测率随轮次变化序列，用于趋势图，元素见下 |

**alpha 字段**

| 字段 | 类型 | 说明 |
| --- | --- | --- |
| local_par | float | 本地对齐权重（0–1），训练早期偏大（个性化） |
| global_par | float | 全局对齐权重（0–1），训练后期偏大（泛化） |
| state | str | 当前调度阶段：`local`（偏本地）/ `balanced`（平衡）/ `global`（偏全局） |

**trend[] 元素字段**

| 字段 | 类型 | 说明 |
| --- | --- | --- |
| round | int | 训练轮次（横轴） |
| acc | float | 该轮全局检测率（纵轴） |

**示例**

```json
{
    "time": "2026-08-03 15:30:00",
    "round": 38,
    "total_round": 40,
    "acc": 45.1,
    "acc_compare": 1.2,
    "alpha": {
        "local_par": 0.2,
        "global_par": 0.8,
        "state": "global"
    },
    "efficiency": 95.0,
    "trend": [
        { "round": 30, "acc": 42.1 },
        { "round": 38, "acc": 45.1 }
    ]
}
```

---

## 2. GET /coordinator/training/params

**用途**：获取当前全局参数，用于初始化"全局参数控制"三个滑杆。

**查询参数**：无

**响应字段**

| 字段 | 类型 | 说明 |
| --- | --- | --- |
| time | str | 服务端时间戳 |
| global_par | float | 全局对齐权重（0–1），对应 Global Alignment Weight 滑杆 |
| participation_ratio | float | 每轮参与训练节点比例（0–1），对应 Participation Ratio 滑杆 |
| anomaly_threshold | float | 异常检测阈值（σ），对应 Anomaly Threshold 滑杆 |

**示例**

```json
{
    "time": "2026-08-03 15:30:00",
    "global_par": 0.5,
    "participation_ratio": 0.2,
    "anomaly_threshold": 3.0
}
```

---

## 3. POST /coordinator/training/params

**用途**：更新全局参数（"应用到下一轮"按钮）。调整不中断当前训练，在下一轮开始前生效。

**请求体字段**

| 字段 | 类型 | 必填 | 说明 |
| --- | --- | --- | --- |
| global_par | float | 是 | 全局对齐权重（0–1），调高偏全局泛化、调低偏本地个性化 |
| participation_ratio | float | 是 | 每轮参与节点比例（0–1），越高通信成本越高 |
| anomaly_threshold | float | 是 | 异常检测阈值（σ），越小越敏感（误报多）、越大越宽松（漏报多） |

**响应字段**

| 字段 | 类型 | 说明 |
| --- | --- | --- |
| time | str | 服务端时间戳 |
| status | str | 生效状态，固定为 `applied_next_round`（下一轮生效） |

**示例**

```json
{
    "time": "2026-08-03 15:30:00",
    "status": "applied_next_round"
}
```

---

## 4. GET /coordinator/training/contribution

**用途**：获取 Fisher 贡献度排行，渲染训练控制页"Fisher 贡献度排行"表格。

**查询参数**：无

**响应字段**

| 字段 | 类型 | 说明 |
| --- | --- | --- |
| time | str | 服务端时间戳 |
| ranking | array | 按贡献度降序的节点列表，元素见下 |

**ranking[] 元素字段**

| 字段 | 类型 | 说明 |
| --- | --- | --- |
| node_code | str | 节点编码 |
| node_name | str | 机构/节点显示名 |
| contribution | float | Fisher 贡献度（0–1），贡献低说明搭便车或行为异常 |
| status | str | 节点状态：`online` / `warning` / `offline` |

**示例**

```json
{
    "time": "2026-08-03 15:30:00",
    "ranking": [
        { "node_code": "BANK-COM-01", "node_name": "省农商行清算中心", "contribution": 0.31, "status": "online" },
        { "node_code": "ENG-PWR-03", "node_name": "区域能源网络中心", "contribution": 0.00, "status": "offline" }
    ]
}
```
