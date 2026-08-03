# 运营员端 · 节点状态 API

> 对应页面：`MVP_frontend/light/operator/node_status.html`（节点状态）
> 接口前缀：`/operator/node`
> 权限：仅本机构运营员（只能访问本机构节点）

---

## 1. GET /operator/node/status

**用途**：获取本节点运行状态总览，渲染页面顶部运行状态摘要、训练质量、安全与隐私、最近同步记录四块内容。

**查询参数**：无

**响应字段**

| 字段 | 类型 | 说明 |
| --- | --- | --- |
| time | str | 服务端时间戳 |
| organization | str | 机构名称，如"省农商行清算中心" |
| node_code | str | 本机构节点编码，如 `BANK-AGR-01` |
| runtime | object | 运行状态摘要（顶部四个指标块），结构见下 |
| quality | object | 训练质量（本地检测率 / 损失 / 趋势图），结构见下 |
| privacy | object | 安全与隐私状态，结构见下 |
| sync_records | array | 最近同步记录时间线，元素见下 |

**runtime 字段**

| 字段 | 类型 | 说明 |
| --- | --- | --- |
| connection | str | 连接状态：`online` / `offline` |
| latency | int | 通信延迟（ms） |
| sync_version | str | 已同步的模型版本，如 `v38` |
| version_diff | int | 与全局最新版本的差，0 表示已同步 |
| perf_swing | float | 检测率较历史峰值的波动（百分点），负数表示回落 |
| rank | int | Fisher 贡献度排名 |
| total | int | 联盟节点总数（用于 "rank/total"） |

**quality 字段**

| 字段 | 类型 | 说明 |
| --- | --- | --- |
| local_acc | float | 本地检测率（%） |
| loss | float | 本地损失值 |
| health | int | 健康评分（0–100） |
| warn_line | int | 健康预警线（低于该值触发告警） |
| peak_acc | float | 历史最高检测率（%） |
| trend | array | 训练质量趋势序列，元素见下 |

**quality.trend[] 元素字段**

| 字段 | 类型 | 说明 |
| --- | --- | --- |
| round | int | 训练轮次（横轴） |
| acc | float | 该轮本地检测率（纵轴） |

**privacy 字段**

| 字段 | 类型 | 说明 |
| --- | --- | --- |
| budget | int | 差分隐私预算已使用比例（%） |
| eps_used | float | 已消耗的 ε |
| eps_total | float | 总隐私预算 ε |
| noise_active | bool | 噪声注入是否活跃 |
| encryption | str | 传输加密协议，如 `TLS 1.3` |
| prototype_dim | int | 原型向量维度 |
| data_local | bool | 原始数据是否留存在本机构（`true` = 未离开） |

**sync_records[] 元素字段**

| 字段 | 类型 | 说明 |
| --- | --- | --- |
| time | str | 记录时间，`HH:mm` |
| title | str | 记录标题 |
| detail | str | 记录详情 |
| level | str | 级别（影响时间线配色）：`success` / `warning` / `info` |

**示例**

```json
{
    "time": "2026-08-03 15:30:00",
    "organization": "省农商行清算中心",
    "node_code": "BANK-AGR-01",
    "runtime": {
        "connection": "online",
        "latency": 84,
        "sync_version": "v38",
        "version_diff": 0,
        "perf_swing": -1.6,
        "rank": 1,
        "total": 24
    },
    "quality": {
        "local_acc": 94.2,
        "loss": 0.312,
        "health": 91,
        "warn_line": 70,
        "peak_acc": 95.8,
        "trend": [
            { "round": 30, "acc": 91.0 },
            { "round": 38, "acc": 94.2 }
        ]
    },
    "privacy": {
        "budget": 42,
        "eps_used": 2.4,
        "eps_total": 5.8,
        "noise_active": true,
        "encryption": "TLS 1.3",
        "prototype_dim": 512,
        "data_local": true
    },
    "sync_records": [
        { "time": "14:30", "title": "模型版本同步完成", "detail": "已拉取全局模型 v38，版本差恢复为 0。", "level": "success" },
        { "time": "14:26", "title": "检测率轻微回落", "detail": "较历史峰值下降 1.6pp，系统建议继续观察 2 轮。", "level": "warning" }
    ]
}
```

---

## 2. GET /operator/params

**用途**：获取本节点本地参数当前值，初始化"操作控制"滑杆/下拉框（节点状态页与训练控制页共用此接口）。

**查询参数**：无

**响应字段**

| 字段 | 类型 | 说明 |
| --- | --- | --- |
| time | str | 服务端时间戳 |
| mode | str | 参与模式：`train_eval`（训练+评估）/ `train`（仅训练）/ `eval`（仅评估）/ `idle`（空闲） |
| noise | float | 噪声注入强度（0–0.20） |
| epoch | int | 本地训练轮次（1–20） |

**示例**

```json
{
    "time": "2026-08-03 15:30:00",
    "mode": "train_eval",
    "noise": 0.05,
    "epoch": 5
}
```

---

## 3. POST /operator/params

**用途**：保存本地配置（"保存配置"按钮）。调整仅影响本节点，在下一轮生效，不改变联盟全局策略。

**请求体字段**

| 字段 | 类型 | 必填 | 说明 |
| --- | --- | --- | --- |
| mode | str | 是 | 参与模式：`train_eval` / `train` / `eval` / `idle` |
| noise | float | 是 | 噪声注入强度（0–0.20），越高隐私越强但可能压低检测率 |
| epoch | int | 是 | 本地训练轮次（1–20），越高训练越充分但耗时与过拟合风险上升 |

**响应字段**

| 字段 | 类型 | 说明 |
| --- | --- | --- |
| time | str | 服务端时间戳 |
| status | str | 生效状态，固定为 `saved_next_round` |

**示例**

```json
{
    "time": "2026-08-03 15:30:00",
    "status": "saved_next_round"
}
```

---

## 4. POST /operator/node/isolate

**用途**：隔离本节点（危险操作，红色按钮）。隔离后本节点不再上传原型、不参与本轮聚合。

**请求体字段**：无

**响应字段**

| 字段 | 类型 | 说明 |
| --- | --- | --- |
| time | str | 服务端时间戳 |
| status | str | 隔离状态，固定为 `isolated` |

**示例**

```json
{
    "time": "2026-08-03 15:30:00",
    "status": "isolated"
}
```

---

## 5. POST /operator/node/recover

**用途**：恢复本节点参与（"恢复参与"按钮），重新加入训练队列。

**请求体字段**：无

**响应字段**

| 字段 | 类型 | 说明 |
| --- | --- | --- |
| time | str | 服务端时间戳 |
| status | str | 恢复状态，固定为 `recovered` |

**示例**

```json
{
    "time": "2026-08-03 15:30:00",
    "status": "recovered"
}
```
