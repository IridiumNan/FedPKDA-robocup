# 运营员端 · 训练控制 API

> 对应页面：`MVP_frontend/light/operator/training.html`（训练控制）
> 接口前缀：`/operator/training`
> 权限：仅本机构运营员（只能访问本机构节点）

---

## 1. GET /operator/training/status

**用途**：获取本机构训练状态，渲染顶部状态栏、四个统计卡、本地与全局趋势双线图、训练质量摘要、风险与建议。

**查询参数**：无

**响应字段**

| 字段 | 类型 | 说明 |
| --- | --- | --- |
| time | str | 服务端时间戳 |
| round | int | 全局当前轮次 |
| total_round | int | 训练计划总轮次 |
| local_acc | float | 本地检测率（%） |
| local_acc_compare | float | 本地检测率较近 7 轮均值的变化（百分点） |
| rank | int | 贡献度排名 |
| total | int | 联盟节点总数 |
| contribution | float | 本节点 Fisher 贡献度（0–1） |
| privacy_budget | int | 差分隐私预算已使用比例（%） |
| eps | float | 当前已消耗的 ε |
| sync_version | str | 已同步模型版本，如 `v38` |
| next_round_time | str | 下一轮预计开始时间，`HH:mm` |
| trend | array | 本地与全局检测率对比曲线，元素见下 |
| summary | object | 训练质量摘要，结构见下 |
| risk | object | 风险与建议，结构见下 |

**trend[] 元素字段**

| 字段 | 类型 | 说明 |
| --- | --- | --- |
| round | int | 训练轮次（横轴） |
| local_acc | float | 该轮本地检测率（本地实线） |
| global_acc | float | 该轮全局检测率（全局虚线） |

**summary 字段**

| 字段 | 类型 | 说明 |
| --- | --- | --- |
| loss | float | 本地损失值 |
| version_diff | int | 模型版本差，0 表示已同步 |
| peak_acc | float | 历史最高检测率（%） |
| peak_round | int | 达到历史最高的轮次 |
| contribution_note | str | 本轮贡献解释文案，如"高质量样本与稳定梯度" |

**risk 字段**

| 字段 | 类型 | 说明 |
| --- | --- | --- |
| level | str | 风险级别：`ok` / `info` / `warning` |
| message | str | 风险建议文案（"风险与建议"卡片内容） |
| suggest_watch | bool | 是否建议标记观察（为 `true` 时前端突出显示"标记观察"按钮） |

**示例**

```json
{
    "time": "2026-08-03 15:30:00",
    "round": 38,
    "total_round": 40,
    "local_acc": 94.2,
    "local_acc_compare": 0.4,
    "rank": 1,
    "total": 24,
    "contribution": 0.31,
    "privacy_budget": 42,
    "eps": 2.4,
    "sync_version": "v38",
    "next_round_time": "14:38",
    "trend": [
        { "round": 30, "local_acc": 91.0, "global_acc": 41.5 },
        { "round": 38, "local_acc": 94.2, "global_acc": 45.1 }
    ],
    "summary": {
        "loss": 0.312,
        "version_diff": 0,
        "peak_acc": 95.8,
        "peak_round": 35,
        "contribution_note": "高质量样本与稳定梯度"
    },
    "risk": {
        "level": "warning",
        "message": "本地检测率较峰值下降 1.6pp。建议维持当前噪声强度，等待下一轮全局模型回传后再扩大本地轮次。",
        "suggest_watch": true
    }
}
```

---

## 2. GET /operator/params

**用途**：读取本地参数当前值（噪声强度 / 本地轮次 / 参与模式滑杆初始值）。

> 与节点状态页共用，完整定义见 [`node-status.md`](./node-status.md) 的 `GET /operator/params`。

**查询参数**：无

---

## 3. POST /operator/params

**用途**：保存本地参数（"保存本地参数"按钮），在下一轮生效。

> 与节点状态页共用，完整定义见 [`node-status.md`](./node-status.md) 的 `POST /operator/params`。

**请求体字段**：`mode`（train_eval / train / eval / idle）、`noise`（0–0.20）、`epoch`（1–20）

---

## 4. POST /operator/training/watch

**用途**：标记观察（"标记观察"按钮），记录当前风险策略以便后续追踪。

**请求体字段**

| 字段 | 类型 | 必填 | 说明 |
| --- | --- | --- | --- |
| note | str | 否 | 观察备注，可选 |

**响应字段**

| 字段 | 类型 | 说明 |
| --- | --- | --- |
| time | str | 服务端时间戳 |
| status | str | 标记状态，固定为 `watched` |

**示例**

```json
{
    "time": "2026-08-03 15:30:00",
    "status": "watched"
}
```
