# 节点管理 API

> 对应页面：`MVP_frontend/light/coordinator/node_management.html`（节点管理）
> 接口前缀：`/coordinator/node`
> 权限：仅协调员

---

## 1. GET /coordinator/node/list

**用途**：获取全部节点列表与健康概览，渲染节点拓扑图、健康评分统计卡、节点列表。

**查询参数**

| 参数 | 类型 | 必填 | 说明 |
| --- | --- | --- | --- |
| status | str | 否 | 节点状态过滤：`all` / `online`（在线）/ `warning`（需关注）/ `offline`（离线）/ `unknown`（未知），缺省 `all` |

**响应字段**

| 字段 | 类型 | 说明 |
| --- | --- | --- |
| time | str | 服务端时间戳 |
| health_summary | object | 全联盟健康概览，用于顶部统计卡，结构见下 |
| nodes | array | 节点数组，元素结构见下 |

**health_summary 字段**

| 字段 | 类型 | 说明 |
| --- | --- | --- |
| avg | int | 全部节点健康评分均值（0–100） |
| online_ratio | float | 在线节点占比（0–1），如 `0.83` 表示 83% |
| bucket | object | 各健康档位节点数：`excellent`（≥85）/ `good`（72–84）/ `warning`（60–71）/ `critical`（<60）/ `offline`（离线） |

**nodes[] 元素字段**

| 字段 | 类型 | 说明 |
| --- | --- | --- |
| id | str | 节点标识，如 `n01` |
| name | str | 节点显示名称，如"节点 A01" |
| code | str | 节点编码，如 `NODE-A01`（通知/过滤使用） |
| status | str | 节点状态：`online` / `warning` / `offline` / `unknown`，决定节点配色 |
| health | int | 健康评分（0–100），离线节点为低分 |
| last | str | 最近心跳时间，`HH:mm`；离线为"未激活" |
| scene | str | 所属场景描述，如"垂直联盟 / 主场景" |
| task | str | 当前任务描述，如"第 38 轮同步完成" |
| event | str | 当前事件摘要，如"运行正常，延迟稳定" |
| telemetry | object | 实时遥测数据，结构见下 |

**nodes[].telemetry 字段**

| 字段 | 类型 | 说明 |
| --- | --- | --- |
| cpu | int | CPU 使用率（%），离线节点为 0 |
| gpu | int | GPU 使用率（%），离线节点为 0 |
| memory | int | 内存使用率（%） |
| latency | int | 通信延迟（ms） |
| packet_loss | float | 丢包率（%），离线节点为 100 |
| round | str | 训练轮次进度，如 `38/40`；未参与为 `--` |
| contribution | float | Fisher 模型贡献度（0–1），贡献为 0 表示搭便车/异常 |
| accuracy | str | 本地检测率（%），如 `94.2%`；无数据为 `--` |
| comm | str | 通信质量评级：`Excellent` / `Good` / `Degraded` / `Lost` / `Pending` / `Core Link` |

**示例**

```json
{
    "time": "2026-08-03 15:30:00",
    "health_summary": {
        "avg": 82,
        "online_ratio": 0.8,
        "bucket": {
            "excellent": 3,
            "good": 4,
            "warning": 1,
            "critical": 1,
            "offline": 1
        }
    },
    "nodes": [
        {
            "id": "n01",
            "name": "节点 A01",
            "code": "NODE-A01",
            "status": "online",
            "health": 94,
            "last": "14:32",
            "scene": "垂直联盟 / 主场景",
            "task": "第 38 轮同步完成",
            "event": "运行正常，延迟稳定",
            "telemetry": {
                "cpu": 42,
                "gpu": 68,
                "memory": 61,
                "latency": 42,
                "packet_loss": 0.2,
                "round": "38/40",
                "contribution": 0.31,
                "accuracy": "94.2%",
                "comm": "Excellent"
            }
        }
    ]
}
```

---

## 2. GET /coordinator/node/{id}

**用途**：获取单个节点完整详情，渲染节点详情面板（含通知默认文案）。

**路径参数**

| 参数 | 类型 | 说明 |
| --- | --- | --- |
| id | str | 节点标识（`n01` 等） |

**响应字段**

| 字段 | 类型 | 说明 |
| --- | --- | --- |
| time | str | 服务端时间戳 |
| node | object | 节点对象，字段与 `list` 接口的 nodes[] 元素一致，另含 `payload` |

**node 附加字段**

| 字段 | 类型 | 说明 |
| --- | --- | --- |
| payload | str | 发送通知时的默认文案（可编辑） |

**示例**

```json
{
    "time": "2026-08-03 15:30:00",
    "node": {
        "id": "n01",
        "name": "节点 A01",
        "code": "NODE-A01",
        "status": "online",
        "health": 94,
        "last": "14:32",
        "scene": "垂直联盟 / 主场景",
        "task": "第 38 轮同步完成",
        "event": "运行正常，延迟稳定",
        "payload": "请确认节点 A01 在下一轮同步前保持在线。",
        "telemetry": {
            "cpu": 42,
            "gpu": 68,
            "memory": 61,
            "latency": 42,
            "packet_loss": 0.2,
            "round": "38/40",
            "contribution": 0.31,
            "accuracy": "94.2%",
            "comm": "Excellent"
        }
    }
}
```

---

## 3. POST /coordinator/node/{id}/notify

**用途**：向指定节点发送通知，对应节点详情面板"发送通知"操作。

**路径参数**

| 参数 | 类型 | 说明 |
| --- | --- | --- |
| id | str | 节点标识（`n01` 等） |

**请求体字段**

| 字段 | 类型 | 必填 | 说明 |
| --- | --- | --- | --- |
| type | str | 是 | 通知类型：`状态检查` / `同步提醒` / `异常处理` / `接入确认` |
| priority | str | 是 | 优先级：`普通` / `较高` / `紧急` |
| note | str | 否 | 补充说明，可选 |
| content | str | 是 | 通知正文（默认取节点 `payload`，可修改） |

**响应字段**

| 字段 | 类型 | 说明 |
| --- | --- | --- |
| time | str | 服务端时间戳 |
| notify_id | str | 通知唯一标识 |
| status | str | 发送状态，固定为 `sent` |

**示例**

```json
{
    "time": "2026-08-03 15:30:00",
    "notify_id": "ntf-20260803-042",
    "status": "sent"
}
```
