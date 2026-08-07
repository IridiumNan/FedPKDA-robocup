
```
                   协调服务器（Coordinator）
   ┌─────────────────────────────────────────────────────┐
   │ ① 收集各客户端的带噪原型                               │
   │ ② K-Means 聚类 → 马氏距离加权（异常自动降权）            │
   │ ③ 生成全局原型 → 广播回客户端                           │
   │ ④ Fisher 加权聚合全局模型 → 下发                       │
   └──────────────┬──────────────────────────┬───────────┘
                  ↕ 上行：带噪原型           ↕ 下行：全局原型 + 模型
   ┌─────────┐   ┌─────────┐   ┌─────────┐   ┌─────────┐
   │ 机构 A   │   │ 机构 B  │   │ 机构 C   │   │  ...    │
   │ 金融流量  │   │ 政务流量 │   │ 能源流量 │   │  其他    │
   ├─────────┤   ├─────────┤   ├─────────┤   ├─────────┤
   │ 本地训练 │   │ 本地训练  │   │ 本地训练 │   │         │
   │ 原型+噪声│   │ 原型+噪声 │   │ 原型+噪声│   │         │
   └─────────┘   └─────────┘   └─────────┘   └─────────┘
        原始数据仅存在于机构本地，协调员从技术上拿不到原型与数据
```

```mermaid
graph TB
    %% 定义样式
    classDef server fill:#1E3A8A,color:#fff,stroke:#0F172A,stroke-width:2px;
    classDef module fill:#3B82F6,color:#fff,stroke:#1D4ED8,stroke-width:2px;
    classDef client fill:#10B981,color:#fff,stroke:#047857,stroke-width:2px;
    classDef data fill:#F59E0B,color:#fff,stroke:#B45309,stroke-width:2px;
    classDef arrow fill:none,stroke:#94A3B8,stroke-width:3px;

    subgraph Coordinator [🔷 联盟协调中心 Coordinator]
        direction TB
        subgraph Proto_Engine [原型聚合引擎]
            direction LR
            P1[接收带噪原型] --> P2[K-Means 聚类] --> P3[马氏距离加权<br>异常节点自动降权] --> P4[生成全局原型]
        end
        subgraph Model_Engine [模型聚合引擎]
            direction LR
            M1[收集本地模型参数] --> M2[Fisher 信息矩阵加权] --> M3[生成全局模型]
        end
    end

    subgraph Clients [🏢 机构节点 Institutions]
        direction LR
        C1[机构 A<br>金融/银行流量]
        C2[机构 B<br>政务云流量]

        C4[机构 ...<br>更多联盟成员]
    end

    %% 数据流连接
    C1 -- "⬆ 上行：带噪类原型" --> Coordinator
    C2 -- "⬆ 上行：带噪类原型" --> Coordinator

    C4 -- "⬆ 上行：带噪类原型" --> Coordinator

    Coordinator -- "⬇ 下行：全局原型 + 全局模型" --> C1
    Coordinator -- "⬇ 下行：全局原型 + 全局模型" --> C2
    Coordinator -- "⬇ 下行：全局原型 + 全局模型" --> C4

    %% 标注节点内部细节（通过CSS或隐藏备注）
    C1 --- D1[🔒 原始流量数据]
    C2 --- D2[🔒 原始流量数据]

    C4 --- D4[🔒 原始流量数据]

    D1 -.-> L1[裁剪+Laplace噪声]
    D2 -.-> L2[裁剪+Laplace噪声]

    D4 -.-> L4[裁剪+Laplace噪声]

    %% 应用样式
    class Coordinator server;
    class Proto_Engine,Model_Engine module;
    class C1,C2,C4 client;
    class D1,D2,D4 data;
```

```mermaid
---
title: FedPKDA System
---
graph TB
    %% 浅色卡片样式（圆角 + 细边框）
    classDef module fill:#DBEAFE,stroke:#3B82F6,stroke-width:1.5px,color:#1E40AF,rx:8px,ry:8px,font-size:14px;
    classDef client fill:#ECFDF5,stroke:#10B981,stroke-width:1.5px,color:#065F46,rx:8px,ry:8px,font-size:14px;
    classDef data fill:#FFFBEB,stroke:#F59E0B,stroke-width:1.5px,color:#92400E,rx:8px,ry:8px,font-size:14px;

    style Proto_Engine fill:#F8FAFC,stroke:#E2E8F0,stroke-width:1px;
    style Client_A fill:#F8FAFC,stroke:#E2E8F0,stroke-width:1px;
    style Model_Engine fill:#F8FAFC,stroke:#E2E8F0,stroke-width:1px;

    subgraph Proto_Engine [原型聚合引擎]
        direction LR
        receive@{ shape: rect, label: "接受带噪原型"} --> culster@{ shape: rect, label: "K-Means聚类" }
        culster --> ma_distance@{ shape:rect, label: "马氏距离加权<br/>异常节点自动降权" }
        ma_distance --> global_generation@{ shape: rect, label: "生成全局原型"}
    end

    subgraph Client_A [客户端 A]
        direction LR
        local_db_A@{shape: cyl, label: "本地数据"} -->|特征提取|prototype_A@{shape: docs, label: "本地初始原型"}
        prototype_A --> |裁剪,加噪|cliped_prototype_A@{ shape: procs, label: "本地原型"}
    end

    subgraph Model_Engine [模型聚合引擎]
        direction LR
        M1[收集本地模型参数] --> M2[Fisher 信息矩阵加权] --> M3[生成全局模型]
    end

    Client_A --> |上行 本地带噪类原型| Proto_Engine
    Proto_Engine --> |全局原型|Client_A
    Client_A --> Model_Engine
    Model_Engine --> |下行 全局模型|Client_A

    %% 应用样式
    class receive,culster,ma_distance,global_generation module;
    class local_db_A data;
    class prototype_A,cliped_prototype_A client;
    class M1,M2,M3 module;
```

```mermaid
graph LR
    %% 浅色卡片样式（圆角 + 细边框）
    classDef server fill:#EFF6FF,stroke:#2563EB,stroke-width:1.5px,color:#1E3A8A,rx:8px,ry:8px,font-size:14px;
    classDef module fill:#DBEAFE,stroke:#3B82F6,stroke-width:1.5px,color:#1E40AF,rx:8px,ry:8px,font-size:14px;
    classDef client fill:#ECFDF5,stroke:#10B981,stroke-width:1.5px,color:#065F46,rx:8px,ry:8px,font-size:14px;

    style Coordinator fill:#F8FAFC,stroke:#E2E8F0,stroke-width:1px;

    subgraph Coordinator [协调服务端]
        direction TB
        Model_Engine@{label: "模型聚合引擎"} ~~~ Proto_Engine@{label: "原型聚合引擎"} ~~~ Panel@{label: "协调员面板"}
    end

    Client_A[客户端A] & Client_B[客户端B] & Client_C[客户端C] --> Coordinator
    Coordinator --> Client_A & Client_B & Client_C

    %% 应用样式
    class Coordinator server;
    class Model_Engine,Proto_Engine,Panel module;
    class Client_A,Client_B,Client_C client;
```
