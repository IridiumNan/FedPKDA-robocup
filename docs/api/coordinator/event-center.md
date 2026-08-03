# 事件中心 API

> 对应页面：`MVP_frontend/light/coordinator/event_center.html`（事件中心）
> 接口前缀：`/coordinator/event`
> 权限：仅协调员

---

## 1. GET /coordinator/event/all_event

**用途**：按条件查询事件列表，渲染右侧"实时风险事件"列表（支持按类型/处理状态过滤与分页）。

**查询参数**

| 参数 | 类型 | 必填 | 说明 |
| --- | --- | --- | --- |
| type | str | 否 | 事件类型过滤：`risk`（风险告警）/ `watch`（关注事件）/ `normal`（普通记录）/ `other`（其他），缺省返回全部 |
| status | str | 否 | 处理状态过滤：`pending`（未处理）/ `done`（已处理），缺省返回全部 |
| page | int | 否 | 页码，从 1 开始，缺省 1 |
| size | int | 否 | 每页条数，缺省 20 |

**响应字段**

| 字段 | 类型 | 说明 |
| --- | --- | --- |
| time | str | 服务端时间戳 |
| total | int | 符合条件的事件总数（用于分页） |
| events | array | 事件数组，元素结构见下 |

**events[] 元素字段**

| 字段 | 类型 | 说明 |
| --- | --- | --- |
| id | int | 事件唯一标识 |
| type | str | 事件类型：`risk` / `watch` / `normal` / `other`，决定卡片级别配色与过滤归属 |
| time | str | 事件发生时间，格式 `HH:mm` |
| duration | str | 持续时间描述，如"持续 18m"、"连续 5 轮"，用于判断异常持续时长 |
| title | str | 事件标题（卡片主标题） |
| source | str | 事件来源描述（关联机构/节点或系统），如"BANK-COM-03 · 金融节点" |
| node_code | str | 关联节点编码，非节点事件为空字符串 |
| status | str | 状态文案，如"紧急" / "关注" / "已记录" |
| metrics | array | 指标键值对数组，形如 `[["聚合权重","0.14 → 0.03"],["风险","High"]]`，每种事件展示自身最相关指标 |
| detail | str | 事件详情说明（卡片"查看详情"折叠内容） |
| primary | str | 主要操作标识：`broadcast`（广播通知）/ `mark_watch`（标记关注）/ `view_summary`（查看摘要），对应前端事件数据中的 `primary` 字段 |
| done | bool | 是否已处理，`true` 时卡片按钮置灰 |

**示例**

```json
{
    "time": "2026-08-03 15:30:00",
    "total": 5,
    "events": [
        {
            "id": 0,
            "type": "risk",
            "time": "14:32",
            "duration": "持续 18m",
            "title": "商行数据中心行为偏离 4.2σ",
            "source": "BANK-COM-03 · 金融节点",
            "node_code": "BANK-COM-03",
            "status": "紧急",
            "metrics": [
                ["聚合权重", "0.14 → 0.03"],
                ["风险", "High"]
            ],
            "detail": "本轮上传的带噪原型与全局质心距离显著异常，系统已将该节点聚合权重从 0.14 降至 0.03。",
            "primary": "broadcast",
            "done": false
        }
    ]
}
```

---

## 2. GET /coordinator/event/summary

**用途**：获取事件中心统计概览，渲染顶部统计卡（待处理 / 高风险）、事件类型环形分布图、近 24 小时事件趋势折线图。

**查询参数**：无

**响应字段**

| 字段 | 类型 | 说明 |
| --- | --- | --- |
| time | str | 服务端时间戳 |
| pending | int | 待处理事件总数（`done=false`） |
| high_risk | int | 高风险事件数（`type=risk` 且未处理） |
| categories | array | 各类型事件数量，用于环形分布图，元素见下 |
| trend | array | 事件数量随时间变化序列，用于趋势折线图，元素见下 |

**categories[] 元素字段**

| 字段 | 类型 | 说明 |
| --- | --- | --- |
| id | str | 事件类型：`risk` / `watch` / `normal` / `other` |
| count | int | 该类型事件数量 |

**trend[] 元素字段**

| 字段 | 类型 | 说明 |
| --- | --- | --- |
| round | int | 训练轮次（横轴） |
| count | int | 该轮产生的事件数（纵轴） |

**示例**

```json
{
    "time": "2026-08-03 15:30:00",
    "pending": 4,
    "high_risk": 2,
    "categories": [
        { "id": "risk", "count": 2 },
        { "id": "watch", "count": 2 },
        { "id": "normal", "count": 1 },
        { "id": "other", "count": 0 }
    ],
    "trend": [
        { "round": 30, "count": 2 },
        { "round": 31, "count": 3 }
    ]
}
```

---

## 3. POST /coordinator/event/{id}/handle

**用途**：处理事件，对应卡片"处理"按钮（标记关注 / 标记已完成）。

**路径参数**

| 参数 | 类型 | 说明 |
| --- | --- | --- |
| id | int | 事件唯一标识 |

**请求体字段**

| 字段 | 类型 | 必填 | 说明 |
| --- | --- | --- | --- |
| action | str | 是 | 处理动作：`mark_watch`（标记关注）/ `done`（标记已完成） |
| note | str | 否 | 处理备注，可选 |

**响应字段**

| 字段 | 类型 | 说明 |
| --- | --- | --- |
| time | str | 服务端时间戳 |
| id | int | 被处理的事件标识 |
| status | str | 处理后的状态：`watched` / `done` |

**示例**

```json
{
    "time": "2026-08-03 15:30:00",
    "id": 1,
    "status": "done"
}
```

---

## 4. POST /coordinator/event/broadcast

**用途**：向全部或指定节点广播通知，对应事件卡"广播通知"操作。

**请求体字段**

| 字段 | 类型 | 必填 | 说明 |
| --- | --- | --- | --- |
| title | str | 是 | 通知标题 |
| content | str | 是 | 通知正文内容 |
| level | str | 是 | 通知级别：`info`（普通）/ `warning`（警告）/ `urgent`（紧急） |
| target | str / array | 是 | 通知对象：字符串 `all` 表示全体节点；或节点编码数组，如 `["BANK-COM-03", "ENG-PWR-03"]` |

**响应字段**

| 字段 | 类型 | 说明 |
| --- | --- | --- |
| time | str | 服务端时间戳 |
| message_id | str | 通知唯一标识（用于后续追踪） |
| target_num | int | 实际送达的节点数量 |

**示例**

```json
{
    "time": "2026-08-03 15:30:00",
    "message_id": "msg-20260803-001",
    "target_num": 24
}
```
