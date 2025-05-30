# 内部网关：RIP与OSPF协议简介

## 路由信息协议（RIP）

### 1. 基本概念

-   **协议类型**：分布式、基于距离向量的内部网关协议（IGP）
-   **适用场景**：小型同类网络，自治系统（AS）内部路由信息传递
-   **度量标准**：以**跳数**衡量到达目标地址的路由距离

### 2. 工作原理

#### 路由更新机制

-   **报文格式**：由若干 `(V, D)` 元组组成路由表
    -   `V`：目标网络/主机的矢量标识
    -   `D`：对应路由的跳数值
-   **更新周期**：每30秒向相邻路由器广播一次路由表
-   **路径限制**：最大跳数不超过15（16跳视为不可达）

#### 路由收敛特性

-   仅与相邻路由器交换路由信息
-   通过迭代更新逐步收敛路由表

### 3. 协议特点

| 特征           | 说明                        |
|----------------|-----------------------------|
| 算法类型       | 距离向量算法                |
| 更新方式       | 周期性广播（30秒间隔）      |
| 路径选择依据   | 最小跳数优先                |
| 网络规模适应性 | 适用于小型网络（≤15跳限制） |

------------------------------------------------------------------------

## 最短路径优先协议（OSPF）

### 1. 协议概述

-   **协议全称**：开放最短路径优先（Open Shortest Path First）
-   **协议类型**：链路状态型内部网关协议（IGP）
-   **核心算法**：基于Dijkstra的最短路径优先（SPF）算法
-   **设计目标**：克服RIP的局限性，支持大规模网络部署

### 2. 核心机制

#### 分层网络架构

-   **区域划分**：
    -   通过划分区域提高路由收敛速度
    -   每个区域拥有32位标识符（点分十进制格式）
    -   单区域路由器数量建议 ≤200
-   **拓扑可见性**：
    -   区域内路由器仅掌握本区域完整拓扑
    -   区域间路由通过骨干区域（Area 0）转发

#### 链路状态更新

-   **度量指标**：费用、距离、延时、带宽等复合参数
-   **更新策略**：
    -   触发式更新：链路状态变化时立即发送
    -   洪泛法传播：向所有路由器广播更新信息
-   **数据库结构**：
    -   存储全网链路状态拓扑图
    -   不直接保存完整路由表

### 3. 协议特点

| 特征           | 说明                         |
|----------------|------------------------------|
| 算法类型       | 链路状态算法                 |
| 更新方式       | 触发式洪泛更新               |
| 路径选择依据   | 综合链路状态度量             |
| 网络规模适应性 | 支持超大规模网络（分层设计） |

------------------------------------------------------------------------

## RIP vs OSPF 对比

| 比较维度         | RIP                  | OSPF                       |
|------------------|----------------------|----------------------------|
| **协议类型**     | 距离向量协议         | 链路状态协议               |
| **度量标准**     | 跳数（≤15）          | 复合指标（带宽/延迟等）    |
| **更新方式**     | 周期性广播（30秒）   | 触发式洪泛更新             |
| **拓扑可见性**   | 仅维护距离向量表     | 存储完整链路状态数据库     |
| **收敛速度**     | 慢（逐跳迭代）       | 快（立即计算最短路径树）   |
| **网络规模支持** | 小型网络（15跳限制） | 大型网络（分层架构设计）   |
| **资源消耗**     | 低（简单距离计算）   | 高（需维护链路状态数据库） |

> **注**：OSPF通过划分区域实现层次化路由，区域边界路由器（ABR）负责区域间路由汇总和转发。
